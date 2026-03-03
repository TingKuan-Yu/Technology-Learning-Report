[TOC]



# How to mount NFS remote folder on Linux

你想知道如何在 Linux 系统上挂载 NFS（Network File System）远程文件夹，核心是先确认本地和远程的 NFS 服务环境，再通过 `mount` 命令或配置开机自动挂载，将远程服务器的共享目录映射到本地目录。

下面我会一步步带你完成完整的挂载流程，包括环境准备、手动挂载、开机自动挂载，以及常见问题排查，确保你能快速上手。

---

### 一、前置准备（本地+远程服务器）
NFS 挂载依赖两端的服务和权限配置，先完成以下基础步骤：

#### 1. 远程 NFS 服务器端（共享目录的机器）
##### （1）安装 NFS 服务
```bash
# CentOS/RHEL 系统
yum install -y nfs-utils rpcbind

# Ubuntu/Debian 系统
apt update && apt install -y nfs-kernel-server rpcbind
```

##### （2）启动并开机自启服务
```bash
# CentOS/RHEL 系统
systemctl start rpcbind nfs-server
systemctl enable rpcbind nfs-server

# Ubuntu/Debian 系统
systemctl start rpcbind nfs-kernel-server
systemctl enable rpcbind nfs-kernel-server
```

##### （3）配置共享目录
编辑 NFS 核心配置文件 `/etc/exports`，指定要共享的目录、允许访问的客户端，以及权限：
```bash
# 示例：共享 /data/nfs 目录，允许 192.168.1.0/24 网段访问
# 权限说明：rw(读写)、sync(同步写入)、no_root_squash(允许root权限)、no_subtree_check(关闭子目录检查)
echo "/data/nfs 192.168.1.0/24(rw,sync,no_root_squash,no_subtree_check)" >> /etc/exports
```
- 替换 `/data/nfs` 为你要共享的实际目录；
- 替换 `192.168.1.0/24` 为允许访问的客户端 IP/网段（如单个 IP：`192.168.1.100`，全部允许：`*`，但生产环境不建议）。

##### （4）生效共享配置并验证
```bash
# 重新加载 exports 配置
exportfs -rv

# 查看已配置的 NFS 共享
showmount -e localhost
# 正常输出示例：
# Export list for localhost:
# /data/nfs 192.168.1.0/24
```

##### （5）开放防火墙（可选）
如果服务器开启了防火墙，需放行 NFS 相关端口：
```bash
# CentOS/RHEL 7+（firewalld）
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload

# Ubuntu/Debian（ufw）
ufw allow nfs
ufw allow rpc-bind
ufw reload
```

#### 2. 本地客户端（挂载的机器）
##### （1）安装 NFS 客户端工具
```bash
# CentOS/RHEL 系统
yum install -y nfs-utils rpcbind

# Ubuntu/Debian 系统
apt update && apt install -y nfs-common rpcbind
```

##### （2）启动 rpcbind 服务
```bash
systemctl start rpcbind
systemctl enable rpcbind
```

##### （3）创建本地挂载点
挂载点是本地的空目录，用于映射远程 NFS 共享：
```bash
# 示例：创建 /mnt/nfs_mount 作为挂载点
mkdir -p /mnt/nfs_mount
```

---

### 二、手动挂载 NFS 远程目录
完成准备后，在本地客户端执行挂载命令：
```bash
# 核心命令格式：mount -t nfs 远程服务器IP:/共享目录 本地挂载点
mount -t nfs 192.168.1.200:/data/nfs /mnt/nfs_mount
```
- 替换 `192.168.1.200` 为远程 NFS 服务器的实际 IP；
- 替换 `/data/nfs` 为远程共享目录；
- 替换 `/mnt/nfs_mount` 为本地挂载点。

#### 验证挂载是否成功
```bash
# 查看已挂载的 NFS 目录
mount | grep nfs
# 正常输出示例：
# 192.168.1.200:/data/nfs on /mnt/nfs_mount type nfs (rw,relatime,vers=3,rsize=1048576,wsize=1048576,namlen=255,...)

# 或查看所有挂载
df -h
# 会看到 /mnt/nfs_mount 对应的远程 NFS 目录信息
```

---

### 三、配置开机自动挂载（持久化）
手动挂载重启后会失效，需配置 `/etc/fstab` 实现开机自动挂载：

#### 1. 编辑 fstab 文件
```bash
vim /etc/fstab
```
添加以下内容（按实际信息替换）：
```
# 格式：远程服务器IP:/共享目录  本地挂载点  nfs  挂载选项  0  0
192.168.1.200:/data/nfs  /mnt/nfs_mount  nfs  defaults  0  0
```
- 挂载选项说明：`defaults` 包含 rw、suid、dev、exec、auto、nouser、async，也可自定义（如 `rw,sync,_netdev`，`_netdev` 表示网络就绪后再挂载，避免开机挂载失败）。

#### 2. 验证 fstab 配置（关键！避免开机故障）
```bash
# 重新加载 fstab 并测试挂载
mount -a
```
如果无报错，说明配置正确；若报错，立即检查 `/etc/fstab` 内容，避免重启后系统无法正常启动。

---

### 四、卸载 NFS 挂载目录
如需卸载已挂载的 NFS 目录：
```bash
# 确保目录未被占用（否则用 fuser -km /mnt/nfs_mount 强制终止进程）
umount /mnt/nfs_mount
```

---

### 五、常见问题排查
1. **挂载时报错 "Connection refused"**
   - 原因：远程服务器 NFS/rpcbind 服务未启动，或防火墙未放行端口；
   - 解决：检查远程服务状态（`systemctl status nfs-server`），确认防火墙规则。

2. **挂载时报错 "No such file or directory"**
   - 原因：远程共享目录路径错误，或客户端 IP 不在服务器 exports 允许列表；
   - 解决：用 `showmount -e 远程IP` 查看服务器实际共享目录，检查 exports 配置。

3. **开机挂载失败**
   - 原因：网络未就绪就执行挂载；
   - 解决：在 fstab 挂载选项中添加 `_netdev`（如 `defaults,_netdev`）。

4. **挂载后读写权限不足**
   - 原因：远程共享目录的本地权限（如属主/属组），或 exports 配置中未加 `no_root_squash`；
   - 解决：修改远程目录权限（`chmod 777 /data/nfs`，测试用），或调整 exports 权限配置。

---

### 总结
1. 挂载 NFS 核心步骤：服务器端配置共享目录 → 客户端安装工具并创建挂载点 → 手动挂载/配置开机自动挂载。
2. 关键验证点：服务器端用 `showmount -e` 确认共享，客户端用 `mount -a` 验证 fstab 配置。
3. 注意事项：生产环境避免用 `*` 开放所有客户端访问，fstab 配置后务必用 `mount -a` 测试，防止开机故障。



# 掛載nfs後, 會知遠端的filesystem是那種type嗎?

挂载NFS远程目录后，**本地Linux系统不会直接识别到远程服务器的底层文件系统类型**（比如远程是ext4、xfs、btrfs、ZFS等），本地仅能看到挂载的文件系统类型为`nfs`/`nfs4`（依使用的NFS协议版本而定）；而远程的底层文件系统，是NFS服务器端的本地存储格式，对NFS客户端完全**透明**。

### 核心原因：NFS的协议本质
NFS是**网络文件系统协议**，作用是通过网络将服务器的**文件目录抽象为远程存储资源**提供给客户端，而非直接暴露服务器的底层存储介质和文件系统。
客户端与服务器之间仅通过NFS协议交互**文件读写、目录遍历、权限查询**等操作指令，由服务器端自行将这些指令映射到底层文件系统执行，客户端无需（也无法）感知服务器的本地文件系统类型。

### 一、本地查看：仅能看到NFS协议类型
在客户端执行挂载查询命令，只能看到文件系统类型为`nfs`/`nfs4`，无法获取远程底层类型：
```bash
# 客户端查看挂载信息
mount | grep nfs
# 示例输出（NFSv3）：
192.168.1.200:/data/nfs on /mnt/nfs_mount type nfs (rw,relatime,vers=3,...)
# 示例输出（NFSv4）：
192.168.1.200:/data/nfs on /mnt/nfs_mount type nfs4 (rw,relatime,vers=4.2,...)

# 或用df命令
df -T /mnt/nfs_mount
# 输出示例：
Filesystem                Type  Size  Used Avail Use% Mounted on
192.168.1.200:/data/nfs   nfs4  100G   20G   80G  20% /mnt/nfs_mount
```

### 二、如何获取远程的底层文件系统类型？
必须**在NFS服务器端**执行查询命令，才能看到共享目录所在的本地文件系统类型，常用两种方式：

#### 方式1：通过共享目录路径查询（最直接）
```bash
# 在NFS服务器端执行，替换为实际共享目录
df -T /data/nfs
# 输出示例（远程底层为xfs）：
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sda3      xfs   100G   20G   80G  20% /data/nfs
```

#### 方式2：查看服务器端所有挂载的文件系统
```bash
# 在NFS服务器端执行
lsblk -f  # 或 df -T
# 输出会列出服务器本地所有磁盘/分区的文件系统类型，匹配共享目录的挂载点即可
```

### 三、补充：NFS协议版本与本地显示类型的对应
客户端看到的`type`字段由**使用的NFS协议版本**决定，与远程底层无关，对应关系：
- NFSv2/NFSv3：本地显示为 `nfs`
- NFSv4（v4.0/v4.1/v4.2）：本地显示为 `nfs4`

可通过挂载参数指定协议版本，比如强制用NFSv4：
```bash
mount -t nfs4 192.168.1.200:/data/nfs /mnt/nfs_mount
# 或fstab中指定：192.168.1.200:/data/nfs /mnt/nfs_mount nfs4 defaults,_netdev 0 0
```

### 四、关键注意点
1. 远程底层文件系统的特性**不会直接传递**给NFS客户端：比如服务器端是ZFS的快照、配额特性，客户端无法直接感知，需通过NFS服务器端配置间接生效；
2. NFS客户端的**文件系统操作限制**由NFS协议决定，而非远程底层：比如NFS不支持部分本地文件系统的高级特性（如ext4的稀疏文件优化、xfs的实时子卷）；
3. 远程底层文件系统的**故障/性能**会间接影响客户端：比如服务器端底层磁盘IO高、文件系统损坏，会导致客户端NFS挂载目录读写卡顿/失败，但客户端仍仅能检测到NFS协议层的异常。

### 总结
✅ 客户端视角：仅能看到挂载类型为`nfs`/`nfs4`，无远程底层文件系统信息；
✅ 服务器视角：可通过`df -T`/`lsblk -f`查询共享目录的本地文件系统类型；
✅ 核心逻辑：NFS协议做了**底层存储与远程访问的解耦**，对客户端屏蔽了服务器的本地存储细节。

需要我教你**如何在NFS客户端通过脚本快速获取远程服务器的底层文件系统类型**（免手动登录服务器）吗？