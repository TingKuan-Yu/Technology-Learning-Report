[TOC]



# SELinux Kernel 核心機制與檢查介面

SELinux 在內核中並非獨立運作，而是作為 **LSM (Linux Security Modules)** 框架的一個插件實現。它透過在核心物件操作路徑上設置「鉤子（Hooks）」來執行強制訪問控制。

### 1. SELinux 在 Kernel 中的核心架構

#### A. LSM (Linux Security Modules) 框架

LSM 是 Kernel 中的一個抽象層。當用戶空間進程嘗試執行系統調用（如 `open()`, `read()`, `socket()`）時：

1. Kernel 先執行傳統的 **DAC** 檢查（UID/GID）。
2. 若 DAC 通過，核心會觸發 **LSM Hook**。
3. LSM Hook 調用 SELinux 模組進行 **MAC** 檢查。

#### B. AVC (Access Vector Cache)

為了避免每次操作都去查詢複雜的策略數據庫（Security Server），Kernel 維護了一個 **AVC** 緩存：

- **命中 (Hit)：** 直接從快取返回允許或拒絕的結果。
- **缺失 (Miss)：** 查詢後端的 Security Server，並將結果存入 AVC。
- **日誌輸出：** 當權限被拒絕時，AVC 負責產生 `avc: denied` 訊息並透過 `printk` 輸出。

### 2. 核心檢查介面 (Kernel Interfaces)

在 Linux/Android 中，有幾個關鍵的檔案系統介面可用於觀察 SELinux 的內核狀態。

#### A. selinuxfs (/sys/fs/selinux)

這是與內核 SELinux 模組交互的主要窗口。

- `/sys/fs/selinux/enforce`: 讀取可得知當前模式（0 為 Permissive，1 為 Enforcing）。寫入可切換模式。
- `/sys/fs/selinux/policy`: 導出當前核心正在使用的二進制策略檔案。
- `/sys/fs/selinux/avc/hash_stats`: 查看 AVC 緩存的統計資訊，用於評估效能。
- `/sys/fs/selinux/context`: 可將標籤傳入以驗證其合法性。

#### B. procfs (/proc)

觀察特定進程運行時的狀態。

- `/proc/<pid>/attr/current`: 查看特定 PID 的進程標籤（Process Context）。
- `/proc/<pid>/attr/prev`: 查看切換到當前標籤之前的上一個標籤（常用於追蹤 `execve` 轉換）。

#### C. Debugfs & Tracepoints

對於效能調優與深度追踪非常有用：

- **位置：** `/sys/kernel/debug/tracing/events/avc/`
- **功能：** 可以使用 `ftrace` 監控 `avc:selinux_audited` 事件，精確掌握權限檢查發生的頻率與時機，而不需要依賴寫入磁碟的 Log。

### 3. 如何利用介面檢查 V4L2/UVC 相關權限？

當您在處理 V4L2 設備（如 `/dev/video0`）時，可以透過以下方式與 Kernel 介面互動：

#### 步驟 1：確認設備節點的 Kernel 標籤

除了使用 `ls -Z`，您也可以查看內核如何記錄它：

```
# 查看設備在內核中的標籤屬性
getfattr -n security.selinux /dev/video0
```

#### 步驟 2：使用運行時檢查工具 (Checkers)

Android 提供了一些基於 Kernel 介面的工具，不需要看源碼即可檢查：

- **`sepolicy-analyze`**: 分析當前加載到內核的 `policy` 文件。

- **`sesearch`**: 直接查詢內核當前規則：

  ```
  # 檢查 hal_camera 是否有權限執行 ioctl 於 video_device
  sesearch --allow -s hal_camera_default -t video_device -c chr_file -p ioctl
  ```

#### 步驟 3：攔截核心系統調用 (Strace + SELinux)

如果您懷疑某個特定的 V4L2 `ioctl` 被攔截，可以使用 `strace` 觀察系統調用的返回：

- 若返回 `EPERM (Operation not permitted)`，通常是 SELinux 在內核 LSM Hook 階段直接攔截了該 `ioctl` code。

### 4. Kernel 配置清單 (Kconfig)

若要確保核心完整支持 SELinux，必須在內核編譯時啟用以下選項：

```
CONFIG_SECURITY_SELINUX=y
CONFIG_SECURITY_NETWORK=y
CONFIG_LSM="selinux"
CONFIG_NETLABEL=y
```

### 5. 工程建議

當同事問到如何證明「是 SELinux 擋住了驅動」時，您可以展示 `/proc/<pid>/attr/current` 與 `/sys/fs/selinux/enforce` 的對比，並搭配 `dmesg` 中的 AVC 輸出。這套從 Kernel 介面出發的證詞是最具說服力的。



# 為什麼不是所有檔案系統都能受 SELinux 完整控制？

SELinux 要能控制一個檔案系統中的對象（Object），核心在於如何獲取該對象的安全上下文（Security Context）。並非所有檔案系統都能滿足這個要求，主要受限於以下兩個關鍵技術指標。

### 1. 核心指標一：是否支援擴展屬性 (xattr)

這是檔案系統能否支援「精確標籤管控」的最重要條件。

- **機制：** 像 ext4, f2fs 或 xfs 這種現代磁碟檔案系統，支援在磁碟節點（inode）中存儲額外的元數據，稱為 **Extended Attributes (xattr)**。SELinux 的標籤就存儲在 `security.selinux` 這個命名空間下。
- **運作：** 當你下達 `chcon` 或 `restorecon` 時，內核會將標籤字串寫入磁碟。
- **不支援的例子：**
  - **FAT32 / exFAT：** 這些格式設計簡單，磁碟結構中沒有位置存放 xattr。因此，你無法為 SD 卡中的不同檔案設定不同的 SELinux 標籤。
  - **vfat：** 這是 Android 掛載外部儲存常用的格式，它無法區分單個檔案的標籤。

### 2. 核心指標二：檔案系統的本質（實體 vs 虛擬）

檔案系統的生成方式決定了內核如何分配標籤。

#### A. 實體檔案系統 (Disk-based)

- 如 `system`, `vendor`, `userdata` (ext4/f2fs)。
- **控制方式：** 使用 **Labeling**。內核讀取磁碟上的標籤，並與政策（Policy）比對。

#### B. 虛擬/偽檔案系統 (Pseudo Filesystems)

這些檔案系統只存在於記憶體中，反映的是內核狀態，而不是磁碟上的數據。

- **例子：** `/proc`, `/sys`, `/dev/pts`, `tmpfs`。
- **問題：** 它們沒有物理節點來存儲 xattr。
- **解決方案 (genfscon)：** 既然不能存儲標籤，內核就必須在掛載時透過 `genfscon` 指令，將特定的標籤「釘」在特定的路徑或類型上。例如：
  - `genfscon proc / u:object_r:proc:s0`
  - 這導致你無法像在 `/data` 下那樣靈活地為 `/proc` 內的不同節點動態更改標籤。

### 3. SELinux 的三種處理模式

根據檔案系統的特性，SELinux 使用三種不同的方式來決定權限：

| 模式             | 說明                                                    | 代表檔案系統                 |
| ---------------- | ------------------------------------------------------- | ---------------------------- |
| **fs_use_xattr** | 完整支援。標籤存在磁碟 inode，支援個別檔案不同標籤。    | ext4, f2fs, btrfs            |
| **fs_use_task**  | 標籤繼承自建立該物件的進程標籤。常用於 Pipe 或 Socket。 | pipefs, sockfs               |
| **genfscon**     | 預設標籤。由內核根據路徑強制分配，無法動態修改。        | sysfs, proc, selinuxfs, vfat |

### 4. 為什麼這在 Android 開發中很重要？

1. **SD 卡問題：**
   - 因為 SD 卡（vfat/exFAT）不支持 xattr，Android 只能在掛載時給予整張卡一個統一的標籤（通常是 `sdcardfs` 或 `fuse` 相關標籤）。這就是為什麼你很難針對 SD 卡內的單個資料夾設定精細的 SELinux 規則。
2. **Debugfs 的限制：**
   - `debugfs` 也是一種虛擬檔案系統。在 user 版本中，為了安全，Google 通常會透過 SELinux 封死整個 `debugfs` 的存取，因為我們很難對其內部成千上萬的動態節點做精細標籤控管。
3. **V4L2 設備節點 (/dev)：**
   - `/dev` 是 `tmpfs`。雖然它是虛擬的，但 Android 透過 `ueventd` 在設備建立時，根據 `file_contexts` 強制設定標籤（如 `video_device`），這補足了 `tmpfs` 不支持 xattr 的遺憾。

### 5. 總結給同事的說法

「SELinux 能不能控制一個檔案系統，要看內核能不能幫這個檔案『貼上標籤』。

- **高級貨 (ext4/f2fs)** 可以在磁碟上記住標籤，所以管控最精細。
- **陽春貨 (FAT32)** 沒地方記標籤，只能全家共用一個標籤。
- **虛擬貨 (proc/sys)** 是內核生成的，標籤是寫死在代碼（Policy）裡的，不能隨便改。」



# genfscon

`genfscon` 是 **SELinux 策略配置语句**，全称 *generic file system context*，核心作用是为**不支持扩展属性（xattr）的文件系统/内核虚拟文件系统**配置**默认 SELinux 安全上下文（标签）**。

因为这类文件系统无法像 `ext4`/`F2FS` 那样在文件上直接存储 `security.selinux` 扩展属性，所以需要通过 `genfscon` 统一指定其标签规则。

### 一、 `genfscon` 的核心适用场景
主要针对两类无法存储自定义 SELinux 标签的文件系统：
1.  **内核虚拟文件系统**：`procfs`、`sysfs`、`devtmpfs`、`devpts`
2.  **不支持 xattr 的传统文件系统**：`FAT32`、`NTFS`、`ISO 9660`

### 二、 `genfscon` 的语法架构
#### 1.  基本语法格式
```
genfscon  <fs_type>  <path_prefix>  <context>
```
| 字段            | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| `<fs_type>`     | 文件系统类型（如 `proc`、`sysfs`、`vfat`、`devpts`）         |
| `<path_prefix>` | 该文件系统下的路径前缀（`*` 表示整个文件系统；具体路径如 `/net`） |
| `<context>`     | 要分配的 SELinux 安全上下文（格式：`u:r:domain:s0`）         |

#### 2.  优先级规则
- 路径前缀**越具体，优先级越高**；
- 若多个 `genfscon` 匹配同一路径，优先应用最精确的配置。

#### 3.  配置文件位置
- Android 系统：配置写在 sepolicy 策略文件中（如 `system/sepolicy/genfs_contexts`）；
- 通用 Linux 系统：通常写在 `/etc/selinux/targeted/contexts/files/genfs_contexts`。

### 三、 典型示例（Android & Linux）
#### 1.  为 `procfs` 配置默认标签
`procfs` 是内核虚拟文件系统，无法存储 xattr，需用 `genfscon` 指定全局和局部标签：
```
# 为整个 proc 目录分配默认标签 u:r:proc:s0
genfscon proc * u:r:proc:s0

# 为 proc/net 子目录分配更具体的标签（优先级更高）
genfscon proc /net u:r:proc_net:s0
```

#### 2.  为 `sysfs` 配置硬件节点标签
Android 中 `sysfs` 用于暴露硬件信息，需为特定路径配置 HAL 可访问的标签：
```
# 为 sys/class/video4linux 分配视频设备标签（适配 V4L2/摄像头）
genfscon sysfs /class/video4linux u:r:video_device:s0
```

#### 3.  为 FAT32 外置 SD 卡配置标签
SD 卡通常为 FAT32 格式，无法自定义文件标签，需统一分配 `sdcardfs` 标签：
```
# 为 vfat 类型的外置存储分配默认标签
genfscon vfat /mnt/media_rw/sdcard u:r:sdcardfs:s0
```

### 四、 `genfscon` 的工作架构（执行流程）
1.  **策略加载阶段**：SELinux 内核模块读取 `genfs_contexts` 配置文件，将 `genfscon` 规则加载到内核哈希表中。
2.  **文件系统挂载阶段**：当系统挂载 `procfs`/`vfat` 等文件系统时，内核检查该文件系统是否支持 xattr。
3.  **标签匹配阶段**：当进程访问该文件系统的文件/目录时，内核根据路径前缀匹配 `genfscon` 规则，分配对应的安全上下文。
4.  **权限检查阶段**：SELinux 用分配的上下文，结合 `allow` 规则判断进程是否有访问权限。

### 五、 与 `file_contexts` 的区别
很多人会混淆 `genfscon` 和 `file_contexts`，两者核心差异如下：
| 特性             | `genfscon`                              | `file_contexts`                       |
| ---------------- | --------------------------------------- | ------------------------------------- |
| **适用对象**     | 不支持 xattr 的文件系统                 | 支持 xattr 的文件系统（ext4/F2FS）    |
| **标签存储方式** | 内核内存中动态匹配，不写入文件          | 写入文件的 xattr 扩展属性，持久化存储 |
| **配置方式**     | 按文件系统+路径前缀配置                 | 按绝对路径配置                        |
| **优先级**       | 仅作用于无 xattr 的文件系统，优先级最低 | 直接写入文件，优先级最高              |

### 总结
1.  `genfscon` 是 SELinux 为**无 xattr 能力的文件系统**设计的默认标签配置语句；
2.  语法架构简单清晰：`文件系统类型 + 路径前缀 + 安全上下文`，支持路径优先级匹配；
3.  核心应用场景是内核虚拟文件系统（procfs/sysfs）和传统文件系统（FAT32/NTFS）；
4.  与 `file_contexts` 互补，共同完成 SELinux 对所有文件系统的标签管控。

需要我帮你整理一份**Android 常用 genfscon 配置清单**吗？里面会包含 procfs/sysfs/vfat 等场景的实战配置。



# SELinux 內核機制：genfscon 的存儲與查詢

當 `sepolicy` 被載入內核時，所有的 `genfscon` 語句會被解析並儲存在內核安全伺服器（Security Server）的資料結構中。

### 1. 內核中的資料結構 (Data Structures)

在 Linux Kernel 源碼中（主要位於 `security/selinux/ss/`），`genfscon` 規則由 `struct ocontext` 和 `struct genfs` 共同組成。

- **`struct genfs`**: 代表一個檔案系統類型（如 `proc`, `sysfs`）。
- **`struct ocontext`**: 代表該檔案系統下的具體路徑與標籤（Context）。

#### 數據結構示意（簡化版）：

```
struct genfs {
    char *fstype;               // 檔案系統名稱，如 "proc"
    struct ocontext *head;      // 指向該 FS 下的所有路徑規則鏈表
    struct genfs *next;         // 指向雜湊桶中的下一個 FS
};
```

### 2. genfscon 雜湊表的運作

為了快速找到特定檔案系統（fstype）的規則，核心並不使用單一的大列表，而是使用一個**雜湊表（Hash Table）**來組織這些 `genfs` 結構。

#### A. 雜湊函數 (Hashing)

當內核需要查詢規則時，它會對檔案系統名稱字串（例如 `"sysfs"`）進行雜湊運算。

- **Key:** 檔案系統名稱字串。
- **Bucket:** 雜湊表的大小通常是固定的（在現行內核中通常為一個簡單的陣列）。

#### B. 雜湊桶與衝突處理

- 如果兩個不同的 fstype 算出相同的雜湊值，它們會透過 `struct genfs` 中的 `next` 指標連成一個單向鏈表（Chaining）。

#### C. 查詢流程

1. **第一步 (雜湊定位)：** 計算目標 `fstype` 的雜湊值，找到對應的雜湊桶。
2. **第二步 (名稱匹配)：** 在該桶的鏈表中進行字串比對，找到正確的 `struct genfs`。
3. **第三步 (路徑掃描)：** 進入該 `genfs` 下的 `ocontext` 鏈表。**注意：** 這裡通常是線性鏈表，並依據路徑長度排序。

### 3. 路徑匹配邏輯：為什麼需要排序？

雖然檔案系統層級是用雜湊表找，但「路徑」匹配卻是另一回事。

- **最長前綴匹配 (Longest Prefix Match)：** 假設你有兩條規則：
  1. `genfscon proc / u:object_r:proc:s0`
  2. `genfscon proc /net u:object_r:proc_net:s0`
- 內核在載入 Policy 時，會將 `ocontext` 鏈表依照**路徑長度降冪排序**。
- **查詢時：** 從鏈表頭開始掃描，第一個匹配到的就是最長、最精確的規則。這保證了 `/proc/net/tcp` 會匹配到 `proc_net` 而不是 `proc`。

### 4. 效能與記憶體開銷 (Memory Overhead)

- **空間利用：** 使用雜湊表避免了為每個檔案系統掃描整個 Policy 的開銷。
- **開銷來源：** 每個 `genfscon` 規則都需要分配內核記憶體來儲存路徑字串和 SID (Security Identifier)。
- **不可變性：** 這些結構在 Policy 載入後是唯讀的（Read-only），並受到內核讀寫鎖（rwlock）保護，這確保了多核心併發存取時的安全性。

### 5. 如何在內核層面「看見」這些雜湊結果？

如果你有內核調試權限（例如使用 `kgdb` 或撰寫內核模組），你可以觀察 `policydb` 結構體：

- **`policydb->genfs`**: 這是指向 `genfs` 雜湊表頭的指標。

在 Android 日誌中，如果你看到： `SELinux:  2048 avtab hash slots, 12345 rules` 這通常是指 `avtab`（訪問向量表）的雜湊狀態，而 `genfscon` 則是儲存在另一個較小、針對檔案系統類型的雜湊結構中。

### 6. 總結給同事的技術點

1. **二級尋址：** 內核先用「雜湊」找到檔案系統（例如 `sysfs`），再用「鏈表遍歷」找到路徑。
2. **效能穩定：** 檔案系統種類不多（通常幾十個），所以雜湊衝突極低，搜尋速度極快。
3. **排序精髓：** `genfscon` 的鏈表必須排序，否則路徑匹配會出錯（例如短路徑會蓋掉長路徑）。