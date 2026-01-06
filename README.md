## easy-docker-compose 使用说明

本仓库是一个**方便通过 `docker-compose` 一键启动常用服务**的示例集合，目前主要包含：

- **PostgreSQL（含中文分词 zhparser）**
- **PgBouncer 连接池**

你可以根据需要，进入对应子目录，直接使用 `docker compose up -d` 启动服务。

---

**1. 已包含的服务与目录结构**

- `postgresql-zhparser/`  
  - 提供带中文分词扩展 `zhparser` 的 PostgreSQL 16 环境  
  - 支持通过初始化 SQL 自动创建 `zhparser` 扩展和中文全文检索配置  
  - 详情参见该目录下的 `README.md`

- `pgbouncer/`  
  - 提供 PgBouncer 连接池配置，用于为 PostgreSQL 增加连接池能力  
  - 可与上述 `postgresql-zhparser` 联合使用，并支持 SCRAM-SHA-256 密码  
  - 详情参见该目录下的 `README.md`

后续如有新增服务（如 Redis、MySQL 等），会以同样的目录形式添加，并在此总说明中补充。

---

**2. 一键启动示例**

启动前请确认本机已安装：
- Docker
- Docker Compose（或 Docker 自带的 `docker compose` 子命令）

常见使用方式：

- **启动 PostgreSQL（含 zhparser）**

```bash
cd postgresql-zhparser
docker compose up -d
```

- **启动 PgBouncer**

```bash
cd pgbouncer
docker compose up -d
```

根据各子目录 `README.md` 中的说明，按需修改环境变量、端口与卷挂载后再启动。

---

**3. 自定义与扩展建议**

- 可以把本仓库当作**个人或团队的 Docker Compose 模板库**，按需增减目录与配置
- 每添加一个新服务，建议：
  - 新建独立子目录（例如 `redis/`、`mysql/` 等）
  - 在子目录中维护自己的 `docker-compose.yml`、`Dockerfile` 和 `README.md`
  - 在本文件中补充该服务的简单说明和一键启动命令

这样可以逐步沉淀出一套适合自己环境的“常用服务一键启动”合集。

