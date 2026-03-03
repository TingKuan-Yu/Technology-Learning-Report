

[TOC]



# Android ufs-utils

你想了解的是 Android 平台下用于管理和调试 UFS 设备的 `ufs-utils` 工具集，它是一套专门针对 UFS（Universal Flash Storage）协议的命令行工具，核心作用是让开发者/工程师能直接与 UFS 设备的固件、LUN 配置、协议参数等底层模块交互，是 Android 系统中调试 UFS LUN 分割、性能调优、故障排查的核心工具。

下面我会从工具定位、核心功能、常用命令、使用场景和注意事项几个维度，帮你全面理解 `ufs-utils`：

---

### 一、工具核心定位
`ufs-utils` 是 **UFS 主机侧的管理工具集**，适配 Android 内核的 UFS 驱动框架（基于 SCSI 子系统），主要解决以下问题：
- 替代 Android 系统中无法直接通过常规文件系统命令（如 `fdisk`）操作 UFS LUN 的痛点；
- 直接读写 UFS 设备的固件配置、查询 LUN 信息、调试 UFS 协议交互（如 UPIU 命令）；
- 适配 UFS 特有的硬件特性（如 RPMB、BOOT LUN、安全特性），这是普通 Linux 存储工具（如 `lsblk`）无法覆盖的。

> 注意：`ufs-utils` 通常集成在 Android 源码的 `external/ufs-utils` 目录下，需编译后在 Android 设备（一般是工程机/root 设备）上运行，普通消费级 Android 设备默认不预装。-> **在mdep上沒找到**

### 二、核心功能与常用命令
`ufs-utils` 包含多个子命令，核心围绕 UFS 设备的 **信息查询、LUN 管理、协议调试、安全配置** 四大类，以下是最常用的命令及作用：

| 命令          | 核心功能                                                   | 典型用法示例                                                 |
| ------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| `ufs-info`    | 查询 UFS 设备基本信息（厂商、版本、LUN 列表、协议版本）    | `ufs-info /dev/sda`（UFS 设备节点，Android 中也可能是 `/dev/block/sda`） |
| `ufs-lun`     | 管理/查询 UFS LUN 配置（LUN 数量、类型、容量、属性）       | `ufs-lun -l /dev/sda`（列出所有 LUN 及类型：普通 LUN/BOOT LUN/RPMB） |
| `ufs-rpmb`    | 操作 RPMB（安全存储区），如写入/读取密钥、验证数据         | `ufs-rpmb -w key.bin -d data.bin /dev/sda`（向 RPMB 写入加密数据） |
| `ufs-feature` | 查询/配置 UFS 设备特性（如功耗模式、读写超时、队列数）     | `ufs-feature -q /dev/sda`（查询所有支持的 UFS 特性）         |
| `ufs-trace`   | 抓取 UFS 协议交互日志（UPIU 命令、响应），用于调试通信故障 | `ufs-trace -o ufs_log.txt /dev/sda`（将协议日志输出到文件）  |
| `ufs-format`  | 格式化指定 UFS LUN（仅普通 LUN 可用，BOOT/RPMB 禁止）      | `ufs-format -l 2 /dev/sda`（格式化 LUN2 为 Android 兼容的 GPT 格式） |

#### 关键命令输出示例（以 `ufs-lun` 为例）
```bash
# 执行 ufs-lun -l /dev/block/sda
UFS LUN Information:
Total LUNs: 5
LUN 0: Type=BOOT, Capacity=512MB, State=Read-Only (Firmware Protected)
LUN 1: Type=BOOT (Backup), Capacity=512MB, State=Read-Only
LUN 2: Type=GENERAL, Capacity=128GB, State=Read-Write (Main Data)
LUN 3: Type=RPMB, Capacity=16MB, State=Secure (HMAC Protected)
LUN 4: Type=DEVICE, Capacity=8MB, State=Firmware Only
```
从输出能清晰看到 Android UFS LUN 的核心特点：**BOOT/RPMB LUN 受固件保护，仅普通 GENERAL LUN 可读写**，这也是和 Linux NVMe 命名空间最大的区别。

### 三、典型使用场景
1. **调试 UFS LUN 分割问题**
   若 Android 设备启动失败、数据分区挂载异常，可通过 `ufs-lun` 检查 LUN 状态，确认是否存在 LUN 识别失败、容量异常等问题；通过 `ufs-info` 验证 UFS 设备是否正常枚举。

2. **配置 UFS 性能参数**
   针对 Android 设备的功耗/性能需求，用 `ufs-feature` 调整 UFS 的队列深度、读写超时时间，优化 LUN 级并行 I/O 性能（UFS 每个 LUN 有独立队列）。

3. **RPMB 安全调试**
   Android 中的 DRM 密钥、指纹数据等敏感信息存储在 RPMB LUN，可通过 `ufs-rpmb` 验证 RPMB 的读写权限、密钥有效性，排查安全相关故障。

4. **固件版本验证**
   通过 `ufs-info` 查询 UFS 设备的固件版本、协议版本（如 UFS 3.1/4.0），确认是否适配 Android 系统的驱动要求。

### 四、使用注意事项
1. **权限要求**：必须以 `root` 权限运行（`su` 后执行），普通用户无访问 UFS 底层的权限。
2. **风险提示**：操作 BOOT/RPMB LUN 可能导致设备变砖（如误格式化 BOOT LUN），仅建议在工程机/测试机上使用。
3. **兼容性**：不同芯片厂商（高通、联发科）的 `ufs-utils` 可能有定制化修改，需匹配设备的内核版本和 UFS 驱动。
4. **与 Linux 工具的区别**：`ufs-utils` 聚焦 UFS 协议/固件层，而 `lsblk`/`mount` 仅能看到 UFS 映射后的块设备，无法操作 LUN 底层配置。

### 总结
1. `ufs-utils` 是 Android 专属的 UFS 底层管理工具，核心用于查询/调试 UFS LUN 配置、RPMB 安全区、协议交互等，是普通文件系统工具的补充。
2. 其核心价值在于适配 Android UFS LUN “固件主导、硬件隔离”的特点，能直接操作 BOOT/RPMB 等特殊 LUN（仅查询/调试，禁止修改）。
3. 使用时需注意 root 权限和设备兼容性，且仅建议在测试/工程场景下操作，避免损坏系统分区。