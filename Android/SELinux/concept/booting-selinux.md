[TOC]



# Android 开机时，SELinux 策略（sepolicy）的**合并、存储与加载**

Android 开机时，SELinux 策略（sepolicy）的**合并、存储与加载**是由 `init` 进程主导的关键流程。合并后的策略不会以文件形式持久存盘，而是**直接加载进内核内存**生效；原始策略源文件则分布在各分区的 `etc/selinux/` 目录下。

下面按 Android 8.0+（Treble 架构）详细拆解：

---

### 一、策略源文件：分布在哪些分区？
Android 8.0 后，SELinux 策略模块化拆分到不同分区，编译后生成 **CIL（Common Intermediate Language）** 格式文件：

| 分区           | 策略路径                   | 说明                                     |
| -------------- | -------------------------- | ---------------------------------------- |
| **system**     | `/system/etc/selinux/`     | AOSP 平台基础策略（`plat_sepolicy.cil`） |
| **system_ext** | `/system_ext/etc/selinux/` | 系统扩展策略                             |
| **product**    | `/product/etc/selinux/`    | 产品定制策略                             |
| **vendor**     | `/vendor/etc/selinux/`     | 芯片/厂商定制策略                        |
| **odm**        | `/odm/etc/selinux/`        | ODM 定制策略                             |
| **apex**       | `/apex/*/etc/selinux/`     | APEX 模块自带策略                        |

---

### 二、开机合并与加载：init 的 SetupSelinux 阶段
`init` 进程在启动的 **SetupSelinux 阶段**完成策略合并与加载，核心代码在 `system/core/init/selinux.cpp`。

#### 1. 前置：挂载必要分区
`init` 先挂载 `system`/`vendor`/`odm`/`system_ext`/`product` 等分区，确保能读取各分区的 `.cil` 文件。

#### 2. 合并策略：两种路径
##### 路径 A：使用预编译策略（优先，开机快）
`init` 按顺序查找预编译好的二进制策略文件：
1. `/odm/etc/selinux/precompiled_sepolicy`
2. `/vendor/etc/selinux/precompiled_sepolicy`
3. 找到则直接读取，跳过编译。

##### 路径 B：动态编译合并（无预编译时）
1. 收集所有分区的 `.cil` 文件。
2. 调用 `secilc` 工具，将所有 `.cil` 合并编译成**单一二进制 sepolicy**（内存中）。
3. 生成的二进制策略仅在内存中，**不写入磁盘**。

#### 3. 加载到内核
`init` 将合并后的二进制策略通过内核接口 `/sys/fs/selinux/load` 写入内核，内核解析并生效。

#### 4. 切换 enforcing 模式
加载完成后，`init` 调用 `SelinuxSetEnforcement()`，将 SELinux 从 permissive 切到 **enforcing 模式**。

#### 5. init 重执行（re-exec）
`init` 调用 `execv("/system/bin/init", ...)` 重新执行自己，让新的 SELinux 域规则对 `init` 自身生效。

---

### 三、合并后的策略：存在哪里？
- **内核内存**：合并后的二进制 sepolicy 仅存在于内核内存，**无磁盘文件**。
- **内核接口查看**：
  ```bash
  # 查看内核当前加载的策略（只读）
  cat /sys/fs/selinux/policy
  ```
- **源文件**：各分区的 `.cil` 仍在原地，供下次开机重新编译。

---

### 四、关键路径与命令（调试用）
#### 1. 查看当前加载的策略版本/状态
```bash
# 查看 SELinux 模式
getenforce

# 查看内核策略状态
cat /sys/fs/selinux/policyvers
cat /sys/fs/selinux/commit_pending_bools
```

#### 2. 查看各分区策略文件
```bash
# 查看平台基础策略
ls -l /system/etc/selinux/

# 查看厂商策略
ls -l /vendor/etc/selinux/

# 查看预编译策略（如有）
ls -l /vendor/etc/selinux/precompiled_sepolicy
```

#### 3. 手动重新编译策略（调试）
```bash
# 收集所有 .cil 并编译（需 root）
secilc -o /data/local/tmp/sepolicy /system/etc/selinux/plat_sepolicy.cil /vendor/etc/selinux/vendor_sepolicy.cil ...

# 加载到内核（临时生效，重启失效）
cat /data/local/tmp/sepolicy > /sys/fs/selinux/load
```

---

### 五、总结
1. **源文件**：各分区 `etc/selinux/` 下的 `.cil`。
2. **合并时机**：开机 `init` 的 `SetupSelinux` 阶段。
3. **合并方式**：优先用预编译 `precompiled_sepolicy`；否则动态编译所有 `.cil`。
4. **存储位置**：**仅在内核内存**，无磁盘文件；通过 `/sys/fs/selinux/policy` 访问。
5. **生效**：加载内核 → 切 enforcing → init 重执行。

需要我整理一份 **Android SELinux 策略加载失败/avc  denied 开机卡 logo 的排查步骤**吗？