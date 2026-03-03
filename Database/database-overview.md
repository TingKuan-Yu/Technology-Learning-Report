# DB Test



你想要获取能读取 PostgreSQL 中 `wikiuser` 相关数据的命令，我会按「连接数据库」「查询用户数据」「查看权限/表结构」等场景提供实用命令，适配你之前 Docker 部署的 Wiki.js 环境。

### 前提说明
- 以下命令默认你的 PostgreSQL 是**本地安装**（非 Docker）；如果 PostgreSQL 运行在 Docker 容器中，需先进入容器（命令见下方）。
- `wikiuser` 是你给 Wiki.js 配置的数据库用户，命令中会标注需要替换的参数（如密码、数据库名）。

---

### 一、基础：连接 PostgreSQL 并切换到 Wiki.js 数据库
#### 1. 本地 PostgreSQL 连接命令
```bash
# 1. 切换到 postgres 系统用户（PostgreSQL 管理员）
sudo su - postgres

# 2. 用 psql 连接数据库（以 wikiuser 身份）
psql -U wikiuser -d wikidb -h 127.0.0.1 -p 5432
# 执行后会提示输入 wikiuser 的密码（即你配置的 DB_PASS）- The same as login password
```

#### 2. Docker 运行的 PostgreSQL 连接命令
如果 PostgreSQL 在 Docker 容器中，先进入容器再连接：
```bash
# 1. 查看 PostgreSQL 容器名/ID
docker ps | grep postgres

# 2. 进入容器（替换为你的容器名/ID，如 postgres-14）
docker exec -it 你的postgres容器名 psql -U wikiuser -d wikidb
```

#### 3. 简化连接（免手动输入密码）
如果不想每次输密码，可临时设置环境变量：
```bash
export PGPASSWORD='你的wikiuser密码'
psql -U wikiuser -d wikidb -h 127.0.0.1
```

---

### 二、核心：读取 wikiuser 相关数据的命令
连接成功后（终端出现 `wikidb=>` 提示符），执行以下 SQL 命令：

#### 1. 查看 `wikiuser` 这个数据库用户的基本信息
```sql
-- 1. 查看用户（角色）的基本属性（用户名、创建时间、是否超级用户等）
\du wikiuser;
-- 或用 SQL 语句查询（更详细）
SELECT usename, usesysid, passwd, created, superuser, createdb, login 
FROM pg_user WHERE usename = 'wikiuser';
```
**输出说明**：
- `usename`：用户名（wikiuser）
- `superuser`：是否超级用户（Wiki.js 建议设为 `f`，即否）
- `login`：是否允许登录（必须为 `t`，否则 Wiki.js 无法连接）
- `created`：用户创建时间

#### 2. 查看 `wikiuser` 对 `wikidb` 数据库的权限
```sql
-- 1. 查看 wikiuser 在 wikidb 数据库的权限
\l wikidb;
-- 2. 查看 wikiuser 对所有表的权限
SELECT table_catalog, table_name, grantee, privilege_type 
FROM information_schema.table_privileges 
WHERE grantee = 'wikiuser';
```

#### 3. 读取 Wiki.js 应用中的「用户数据」（重点）
Wiki.js 会把**应用内的用户**（如登录 Wiki 的编辑者）存在数据库表中，而非 PostgreSQL 的 `wikiuser` 数据库用户，查询命令：
```sql
-- 1. 查看 Wiki.js 应用用户表结构（确认字段）
\d users;

-- 2. 查询所有 Wiki.js 应用用户（核心）
SELECT id, email, name, username, created_at, is_admin 
FROM users;

-- 3. 筛选管理员用户
SELECT name, email FROM users WHERE is_admin = true;

-- 4. 查看用户的编辑记录（关联活动日志）
SELECT u.name, a.action, a.resource_title, a.created_at 
FROM activity_logs a
JOIN users u ON a.user_id = u.id
ORDER BY a.created_at DESC;
```

#### 4. 查看 `wikiuser` 可访问的表（Wiki.js 所有表）
```sql
-- 1. 列出 wikidb 数据库中所有表（Wiki.js 相关）
\dt;

-- 2. 查看指定表的前 10 条数据（如 pages 页面表）
SELECT id, title, slug, created_by, updated_at FROM pages LIMIT 10;

-- 3. 查看谁创建/编辑了哪个 Wiki 页面
SELECT 
  p.title AS 页面标题,
  u1.name AS 创建人,
  u2.name AS 最后编辑人,
  p.updated_at AS 最后编辑时间
FROM pages p
LEFT JOIN users u1 ON p.created_by = u1.id
LEFT JOIN users u2 ON p.updated_by = u2.id;
```

#### 5. 退出 psql 终端
```sql
\q  -- 退出 psql
exit -- 退出 postgres 系统用户（如果之前切换过）
```

---

### 三、进阶：直接在终端执行查询（无需进入 psql 交互模式）
如果不想进入 psql 交互界面，可直接在 Ubuntu 终端执行单条查询：
```bash
# 示例：查询所有 Wiki.js 应用用户
export PGPASSWORD='你的wikiuser密码'
psql -U wikiuser -d wikidb -h 127.0.0.1 -c "SELECT id, name, email FROM users;"

# 示例：导出用户数据到 CSV 文件（方便查看）
psql -U wikiuser -d wikidb -c "COPY (SELECT name, email, created_at FROM users) TO '/tmp/wiki_users.csv' WITH (FORMAT csv, HEADER);"
# 导出后可查看文件：cat /tmp/wiki_users.csv
```

---

### 四、常用辅助命令
```sql
-- 1. 查看当前连接的用户/数据库
SELECT current_user, current_database();

-- 2. 查看 Wiki.js 数据库的大小
SELECT pg_size_pretty(pg_database_size('wikidb'));

-- 3. 查看 users 表的记录数（有多少个 Wiki 应用用户）
SELECT COUNT(*) FROM users;
```

### 总结
1. 核心连接命令：本地用 `psql -U wikiuser -d wikidb -h 127.0.0.1`，Docker 需先 `docker exec` 进入容器；
2. 查看数据库用户 `wikiuser`：用 `\du wikiuser` 或 `SELECT * FROM pg_user WHERE usename='wikiuser';`；
3. 查看 Wiki.js 应用内的用户/编辑记录：重点查 `users` 表和 `activity_logs` 表，如 `SELECT name, email FROM users;`；
4. 可通过 `COPY` 命令将数据导出为 CSV，方便查看和分析。

关键点：执行命令前确保 `wikiuser` 有足够权限（至少 `SELECT` 权限），如果提示「权限不足」，可切换到 `postgres` 管理员用户执行 `GRANT SELECT ON ALL TABLES IN SCHEMA public TO wikiuser;` 赋予查询权限。