# Check PostgreSQL installation status

你想确认自己 Ubuntu 系统中的 PostgreSQL 是安装在 Docker 容器里，还是直接安装在本地系统中，我会教你几个简单又准确的检查方法，新手也能轻松操作。

### 方法 1：检查 Docker 容器（最直接）
首先查看当前运行/已创建的 Docker 容器，确认是否有 PostgreSQL 相关容器：
```bash
# 查看正在运行的容器
docker ps

# 查看所有容器（包括停止的）
docker ps -a
```
- 如果输出结果中能看到 `postgres`、`postgresql` 等关键词的容器名/镜像名，说明 PostgreSQL 运行在 Docker 中。
- 如果没有任何 PostgreSQL 相关容器，再继续检查本地安装。

### 方法 2：检查本地系统服务
PostgreSQL 若安装在本地（非 Docker），会以系统服务形式运行，执行以下命令检查：
```bash
# 检查 PostgreSQL 服务状态（Ubuntu 16.04+ 用 systemctl）
sudo systemctl status postgresql

# 若系统是老版本（如 Ubuntu 14.04），用 service 命令
sudo service postgresql status
```
- 如果输出显示 `active (running)`（正在运行），说明是**本地安装**的 PostgreSQL。
- 如果显示 `inactive (dead)` 或 `Unit postgresql.service could not be found`，说明本地未安装。

### 方法 3：检查进程和端口（辅助验证）
通过进程和端口进一步确认 PostgreSQL 的运行方式：
```bash
# 1. 查找 PostgreSQL 进程
ps aux | grep postgres

# 2. 查看 5432 端口（PostgreSQL 默认端口）被哪个进程占用
sudo lsof -i :5432
# 或用 netstat（需先安装：sudo apt install net-tools）
sudo netstat -tulpn | grep 5432
```
- 如果进程路径包含 `docker`（如 `/var/lib/docker/...`），或端口占用进程的 `PID/PPID` 指向 Docker 相关进程，说明是 Docker 运行的。
- 如果进程路径是 `/usr/lib/postgresql/...`（系统默认路径），说明是本地安装的。

### 方法 4：检查安装包（最终确认）
直接查看系统是否通过 apt 安装了 PostgreSQL：
```bash
# 检查本地是否安装了 PostgreSQL 包
dpkg -l | grep postgresql
```
- 如果输出有 `ii` 开头的记录（如 `ii  postgresql-14  14.10-0ubuntu0.22.04.1  amd64`），说明是**本地安装**。
- 如果无输出，说明本地未安装，PostgreSQL 大概率在 Docker 中。

### 实操示例（快速判断）
举个例子，你可以按以下顺序快速验证：
1. 先执行 `docker ps | grep postgres` → 有结果 = Docker 运行；
2. 若无结果，执行 `sudo systemctl status postgresql` → 显示 running = 本地安装；
3. 若都无结果，说明 PostgreSQL 既没装在本地也没在 Docker 运行（可能未安装）。

### 总结
1. 优先用 `docker ps -a` 检查是否有 PostgreSQL 容器，有则是 Docker 部署；
2. 无容器时，用 `sudo systemctl status postgresql` 检查本地服务，显示运行则是本地安装；
3. 辅助用 `ps aux | grep postgres` 或 `dpkg -l | grep postgresql` 验证，可100%确认安装方式。

关键点：Docker 运行的 PostgreSQL 进程/端口会关联到 Docker 容器，而本地安装的会有系统服务和 apt 安装记录，两者不会同时存在（除非你刻意部署了两套，这种情况极少）。