

# Ubuntu 系统上如何备份并导出 PostgreSQL 数据库

你想要了解在 Ubuntu 系统上如何备份并导出 PostgreSQL 数据库，我会为你详细讲解最常用、最实用的备份方法。

PostgreSQL 官方推荐且最常用的备份工具是 `pg_dump`，它可以将数据库导出为 SQL 脚本文件，非常适合新手使用，操作简单且恢复方便。

### 一、基础备份操作（pg_dump）
#### 1. 基本备份命令
首先登录到你的 Ubuntu 服务器，打开终端，执行以下核心命令：
```bash
# 基本格式：pg_dump -U 用户名 -d 数据库名 -f 备份文件路径/文件名.sql
pg_dump -U postgres -d mydatabase -f /home/ubuntu/backup/mydatabase_backup.sql
```

**参数解释**：
- `-U postgres`：指定连接数据库的用户（PostgreSQL 默认管理员用户是 postgres）
- `-d mydatabase`：指定要备份的数据库名称（替换成你实际的数据库名）
- `-f`：指定备份文件的保存路径和文件名（建议存放在非系统盘，且确保路径有写入权限）

#### 2. 带密码免交互备份（适合脚本/定时任务）
如果执行上述命令时需要手动输入密码，可通过环境变量指定密码，避免交互：
```bash
# 临时设置密码环境变量（替换为你的postgres密码）
export PGPASSWORD='你的数据库密码'
# 执行备份
pg_dump -U postgres -d mydatabase -h localhost -p 5432 -f /home/ubuntu/backup/mydatabase_backup.sql
# 清空密码环境变量（可选，增强安全性）
unset PGPASSWORD
```

#### 3. 压缩备份（节省磁盘空间）
如果数据库较大，建议直接导出为压缩文件：
```bash
pg_dump -U postgres -d mydatabase | gzip > /home/ubuntu/backup/mydatabase_backup.sql.gz
```

### 二、备份整个PostgreSQL实例（pg_dumpall）
如果你需要备份服务器上所有数据库（包括用户、权限等全局信息），使用 `pg_dumpall`：
```bash
pg_dumpall -U postgres -f /home/ubuntu/backup/all_databases_backup.sql
```

### 三、备份注意事项
1. **权限问题**：
   - 如果当前用户不是 `postgres`，可以先切换到 postgres 用户：`sudo su - postgres`，再执行备份命令。
   - 确保备份目录存在且有写入权限：`mkdir -p /home/ubuntu/backup && chmod 755 /home/ubuntu/backup`。
2. **远程备份（可选）**：
   如果需要从本地备份远程 Ubuntu 服务器的 PostgreSQL，添加 `-h`（主机）和 `-p`（端口）参数：
   ```bash
   pg_dump -U postgres -h 192.168.1.100 -p 5432 -d mydatabase -f ./mydatabase_backup.sql
   ```
3. **验证备份文件**：
   备份完成后，可通过查看文件大小确认是否备份成功：
   ```bash
   ls -lh /home/ubuntu/backup/mydatabase_backup.sql
   ```
   正常情况下文件大小应大于 0。

### 四、恢复备份（可选，便于你验证备份有效性）
如果需要恢复备份的数据库，执行以下命令：
```bash
# 先创建空数据库（如果目标数据库不存在）
createdb -U postgres -O postgres mydatabase_restore
# 恢复备份
psql -U postgres -d mydatabase_restore -f /home/ubuntu/backup/mydatabase_backup.sql
```

### 总结
1. Ubuntu 下备份 PostgreSQL 最核心的工具是 `pg_dump`，基本命令为 `pg_dump -U 用户名 -d 数据库名 -f 备份文件路径`。
2. 可通过压缩（`gzip`）减少备份文件体积，通过环境变量 `PGPASSWORD` 实现免交互备份。
3. `pg_dumpall` 适用于备份所有数据库及全局配置，恢复时用 `psql` 执行备份的 SQL 文件即可。

关键点：备份前确保 `postgres` 用户有数据库访问权限，备份目录有写入权限；重要备份建议定期执行，并将备份文件拷贝到外部存储（如U盘、云存储）。