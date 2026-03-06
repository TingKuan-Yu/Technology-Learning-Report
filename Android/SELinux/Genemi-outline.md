# Android SELinux 全方位解析

## 第一部分：SELinux 核心概念與 Android 引入背景

### 1. 什麼是 SELinux？

- **核心目的：** 強制存取控制 (MAC, Mandatory Access Control)。
- **DAC vs MAC：** * **DAC (傳統 Linux):** 基於用戶/群組 (UID/GID)，權限過於寬鬆（Root 擁有一切）。
  - **MAC (SELinux):** 基於策略規則，即使是 Root 只要沒寫規則也無法存取特定資源。
- **Android 為何需要它？**
  - **收斂權限：** 限制被攻擊後的損害範圍（例如某進程被破獲，也無法存取其他分區）。
  - **進程隔離：** 嚴格區分 App 與 系統進程。
  - **CTS 認證要求：** Google 強制要求生產設備必須處於 Enforcing 模式。

### 2. 三種運作模式 (日常調試必知)

- **Enforcing：** 強制模式。攔截未授權操作並記錄日誌。
- **Permissive：** 寬容模式。**不攔截**操作，但會在 `dmesg` 記錄 `avc: denied`。這是調試的首選。
- **Disabled：** 完全關閉（Android 通常無法輕易關閉，需修改 Kernel 啟動參數）。

## 第二部分：標籤管控與文件系統

### 1. 標籤 (Security Context) 的組成

- 格式：`u:object_r:type:s0` (使用者:角色:類型:安全級別)。
- **File Context (客體):** 文件、目錄、設備節點的標籤。
- **Process Context (主體):** 進程運行的標籤。
- **匹配規則：** 進程標籤 (Subject) 必須擁有對目標標籤 (Object) 的特定操作權限。

### 2. 文件系統支持的條件

- **xattr (擴展屬性)：** 如 ext4, f2fs。標籤直接存儲在磁碟節點上。
- **適配層：** 若文件系統不支持 xattr（如 vfat, sysfs, debugfs），需使用 `genfscon` 或 `context=` 掛載選項來分配標籤。

## 第三部分：Android Sepolicy 設定與語法

### 1. 基礎規則格式

- **語法：** `allow <主體類型> <客體類型>:<權限類別> { 權限名稱 };`
- **範例：** 允許 system_app 訪問 V4L2 設備。
  - `allow system_app video_device:chr_file { read write open ioctl };`

### 2. 常用 Macro (宏) 與特別用法

- **常用宏：**
  - `unix_socket_connect()`: 簡化 Socket 連接規則。
  - `typeattribute`: 將類型關聯到某個屬性（如 `domain`）。
  - **Neverallow：** 最強警告！定義「絕對禁止」的行為，編譯時會檢查。若衝突需重新設計權限，而非簡單添加 allow。
- **genfscon：** 用於為無法支持標籤的文件系統（如 `proc` 節點）指定標籤。

## 第四部分：HAL 層與 V4L2/UVC 實戰範例

### 1. HAL 層 Sepolicy 範例

- **場景：** HAL 進程需要存取 `/dev/videoX`。
- **文件定義：**
  - 在 `device/qcom/sepolicy/.../file_contexts` 中定義：
    - `/dev/video[0-9]+  u:object_r:video_device:s0`
  - 在 `hal_camera_default.te` 中添加：
    - `allow hal_camera_default video_device:chr_file rw_file_perms;`
    - `allowxperm hal_camera_default video_device:chr_file ioctl { VIDIOC_STREAMON VIDIOC_ENUM_FMT };`

### 2. HAL 與 Kernel 交互注意事項

- 存取 `/sys/class/video4linux` 等節點需另外授權。
- 核心原則：**最小權限原則**。HAL 不應擁有 `sysadmin` 權限。

## 第五部分：核心機制：SELinux 與 Kernel

### 1. 內核組件

- **CONFIG_SECURITY_SELINUX=y**：編譯開關。
- **LSM (Linux Security Modules)**：SELinux 在 Kernel 中的掛鉤點。
- **selinuxfs (/sys/fs/selinux)**：Kernel 與 User 空間交互的門戶。

### 2. UVC Video 驅動相關

- 核心在執行 `v4l2_open` 時，會觸發 LSM 檢查該進程是否有對 `video_device` 標籤的 `open` 權限。

## 第六部分：配置與合併 (System vs Vendor)

### 1. 分區隔離 (Android 10+)

- **System Policy：** `/system/etc/selinux`，定義核心組件規則。
- **Vendor Policy：** `/vendor/etc/selinux`，定義 SoC 特有驅動規則。
- **合併機制：** 編譯時會將兩者合併成最終的 `precompiled_sepolicy`。

### 2. 合併避坑點

- **Avoid Global Modification：** 盡量不要在 Vendor 側修改 System 標籤的屬性。
- **Attribute 衝突：** 若 System 定義了 `neverallow`，Vendor 則無法違反，否則編譯失敗。
- **集中與分散：** 建議功能相關的規則放在對應的 `.te` 文件，不要全部塞進 `device.te`。

## 第七部分：調試與實戰方法 (Debug Guide)

### 1. 核心調試指令

- `getenforce` / `setenforce 0` (臨時切換模式)。
- `ls -Z`：查看文件標籤。
- `ps -AZ`：查看進程標籤。

### 2. 如何分析 AVC Denied？

- **日誌捕獲：** `dmesg | grep avc` 或 `logcat | grep avc`。
- **分析範例：** * `avc: denied { ioctl } for comm="camerahalserver" path="/dev/video0" scontext=u:r:hal_camera_default:s0 tcontext=u:object_r:video_device:s0`
- **工具：**
  - **audit2allow：** 將 Log 直接轉換為規則（`dmesg | audit2allow -p out/target/.../sepolicy`）。**注意：** 不要盲目複製，需評估安全性。

### 3. V4L2 專屬案例

- 若 `VIDIOC_STREAMON` 失敗但 errno 為 `EACCES` 或 `EPERM`：
  - 先 `setenforce 0` 測試。
  - 若成功，則是 SELinux 攔截了特定的 `ioctl` 控制字。
  - 需使用 `allowxperm` 精確授權特定 ioctl code。

## 結語：SELinux 不是障礙，而是保護傘

- 開發初期使用 Permissive 模式收集完整 Log。
- 上線前務必確保所有規則已進入 Enforcing 模式並通過 CTS。