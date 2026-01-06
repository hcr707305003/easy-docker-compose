## PgBouncer 使用说明

**1. 在 PostgreSQL 中查询用户的 `username` 和 `passwd`**

在目标 PostgreSQL 数据库中执行下面的 SQL，获取需要通过 PgBouncer 连接的用户账号和加密密码（例如 `postgres` 用户）。  
**推荐使用 SCRAM-SHA-256，而不是 MD5**，以避免兼容性和安全性问题：

```sql
SELECT usename, passwd
FROM pg_shadow
WHERE usename = 'postgres';
```

> 注意：`pg_shadow` 视图只能由超级用户查询，请确保使用具有足够权限的账号连接。

---

**2. 将查询到的账号信息填入 `userlist.txt`**

`userlist.txt` 的每一行格式为（PgBouncer 会直接使用 PostgreSQL 中存储的加密密码，这里同样支持 SCRAM-SHA-256）：

```txt
"usename" "passwd"
```

示例（请替换为你实际查询到的用户名和密码，以下示例使用 SCRAM-SHA-256，占位符仅为示意）：

```txt
"postgres" "SCRAM-SHA-256$xxxxxx"
```

- **第一个字段**：PostgreSQL 用户名，对应查询结果中的 `usename`
- **第二个字段**：加密后的密码，对应查询结果中的 `passwd`（如果已启用 SCRAM-SHA-256，则这里会是以 `SCRAM-SHA-256$` 开头的字符串）

> 提示：
> - 确保 PostgreSQL 侧已配置使用 SCRAM-SHA-256（例如 `password_encryption = 'scram-sha-256'`，并确保相关用户密码是按 SCRAM 方式重新设置过的）。
> - MD5 密码在部分环境和新版本中存在兼容性问题，不再建议使用。

如果有多个需要通过 PgBouncer 访问的用户，可以写多行。

---

**3. 调整 `pgbouncer.ini` 中的 `databases` 配置**

打开 `pgbouncer.ini`，找到 `[databases]` 段落，根据你的实际 PostgreSQL 实例信息进行修改，例如：

```ini
[databases]
mydb = host=postgres port=5432 dbname=mydb user=postgres
```

- **mydb**：通过 PgBouncer 访问时使用的逻辑数据库名
- **host**：实际 PostgreSQL 服务的主机名或容器名
- **port**：PostgreSQL 的端口
- **dbname**：后端真实数据库名
- **user**：连接后端 PostgreSQL 的用户名（需要有相应权限）

根据你的部署情况，调整为对应的主机、端口、数据库名和用户。

---

**4. 启动容器（示例）**

在当前目录下执行：

```bash
docker compose up -d
```

确保：
- `userlist.txt` 已正确填写；
- `pgbouncer.ini` 的 `databases` 段配置指向正确的 PostgreSQL。

