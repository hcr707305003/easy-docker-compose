## PostgreSQL（含 zhparser）使用说明

本目录提供一个带中文分词扩展 `zhparser` 的 PostgreSQL 16 容器环境，通过 `docker-compose` 一键启动。

---

**1. 目录结构说明**

- `docker-compose.yml`：编排 PostgreSQL 服务的配置文件
- `postgres/Dockerfile`：自定义 PostgreSQL 镜像，内置 `scws` 和 `zhparser`
- `postgres/init-db.sql`：数据库初始化脚本，自动创建 `zhparser` 扩展和文本搜索配置
- `data/`：PostgreSQL 数据持久化目录（由 `docker-compose.yml` 挂载）

> 提示：`docker-compose.yml` 中的卷映射可以根据你的本机目录结构自行调整。

---

**2. `docker-compose.yml` 关键配置说明**

```yaml
services:
  postgres:
    build:
      context: ./postgres
      dockerfile: Dockerfile
    container_name: postgres_with_zhparser
    environment:
      POSTGRES_USER: postgresql
      POSTGRES_PASSWORD: hcr707305003
      POSTGRES_DB: ai-moments
    ports:
      - "5432:5432"
    volumes:
      - ./data:/var/lib/postgresql
      - ./postgres/init-db.sql:/docker-entrypoint-initdb.d/init-db.sql
    restart: unless-stopped
```

- **POSTGRES_USER / POSTGRES_PASSWORD / POSTGRES_DB**：初始化超级用户、密码和默认数据库名，可按需修改
- **ports**：将容器的 5432 端口映射到宿主机 5432 端口
- **volumes**：
  - `./data:/var/lib/postgresql`：数据持久化目录，**如有需要可修改为你自己的路径**
  - `./postgres/init-db.sql:/docker-entrypoint-initdb.d/init-db.sql`：挂载初始化 SQL 脚本

如果你需要调整数据存储位置，只需修改 `./data` 前面的主机路径部分即可。

---

**3. zhparser 初始化脚本说明（`postgres/init-db.sql`）**

初始化脚本主要做了两件事：

1. 创建 `zhparser` 扩展（如果不存在）：

```sql
CREATE EXTENSION IF NOT EXISTS zhparser;
```

2. 创建中文全文检索配置 `zhparsercfg`（如果不存在），并绑定常见词性到 `simple` 字典：

```sql
DO $$
BEGIN
    IF NOT EXISTS (
        SELECT 1 FROM pg_ts_config WHERE cfgname = 'zhparsercfg'
    ) THEN
        CREATE TEXT SEARCH CONFIGURATION zhparsercfg (PARSER = zhparser);
        ALTER TEXT SEARCH CONFIGURATION zhparsercfg
            ADD MAPPING FOR n,v,a,i,e,l WITH simple;
    END IF;
END
$$;
```

之后可以在应用中直接使用 `zhparsercfg` 做中文全文搜索。

---

**4. 构建与启动**

在 `postgresql` 目录下执行：

```bash
docker compose up -d
```

Docker 会：
- 使用 `postgres/Dockerfile` 构建带 `zhparser` 的 PostgreSQL 镜像；
- 启动容器并执行 `init-db.sql` 完成扩展和配置初始化；
- 将数据写入你在 `docker-compose.yml` 中挂载的 `./data` 目录。

---

**5. 与 PgBouncer 协同使用的建议**

如果你同时使用本仓库中的 `pgbouncer` 目录：
- PostgreSQL 中可以启用 **SCRAM-SHA-256** 作为密码加密方式（例如在 `postgresql.conf` 中设置 `password_encryption = 'scram-sha-256'`，并为相关用户重置密码）
- 然后在 PgBouncer 容器内按 `pgbouncer/README.md` 的说明，从本 PostgreSQL 中查询 `pg_shadow`，将 `usename` / `passwd` 同步到 PgBouncer 的 `userlist.txt` 中。

