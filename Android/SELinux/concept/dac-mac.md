[TOC]



# **DAC（自主访问控制）** 和 **MAC（强制访问控制

在 Linux 权限控制体系中，**DAC（自主访问控制）** 和 **MAC（强制访问控制）** 是两种核心的权限模型，二者的核心差异在于 **权限的决策主体、控制粒度和强制程度**，MAC 是对 DAC 的补充和强化。

### 一、核心定义与核心差异
| 对比维度     | **DAC (Discretionary Access Control)**                       | **MAC (Mandatory Access Control)**                           |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **决策主体** | **资源所有者** 决定谁能访问资源，权限可自主分配/修改         | **系统内核/安全策略** 强制决定访问权限，用户（包括所有者）无权干预 |
| **控制依据** | 基于 **用户 (UID)、用户组 (GID)、文件权限位 (rwx)**（如 `chmod 755`） | 基于 **安全标签 (Security Label)**，为主体（进程）和客体（文件/端口）分配标签，按预设策略匹配 |
| **权限粒度** | 粗粒度：仅区分所有者、同组用户、其他用户                     | 细粒度：可针对单个进程、文件、端口设置差异化权限             |
| **灵活性**   | 高：所有者可自由修改权限（如 `chown` `chmod`）               | 低：策略由管理员配置，普通用户无法修改，严格遵循“最小权限”   |
| **典型实现** | Linux 传统权限（rwx）、ACL（访问控制列表）                   | SELinux（Android/Linux 主流）、AppArmor                      |
| **适用场景** | 普通桌面/服务器环境，满足日常权限管理需求                    | 高安全需求场景（如服务器、Android 系统、军工/金融系统）      |

### 二、具体工作原理拆解
#### 1. DAC（自主访问控制）
这是 Linux 最基础的权限模型，我们日常使用的 `ls -l` 看到的权限就是 DAC：
- **主体**：进程对应的用户/用户组。
- **客体**：文件、目录、设备等资源。
- **规则**：资源所有者通过 `chmod` 设定 `r（读）w（写）x（执行）` 权限，系统仅检查主体的 UID/GID 是否匹配客体的权限位。
- **局限性**：**权限可被所有者随意转让**，且无法限制“越权访问”——比如 root 用户默认拥有所有资源的访问权限，一旦 root 进程被劫持，系统将完全失控。

  **示例**：
  ```bash
  # 所有者给文件设置 rwxr-xr-- 权限
  chmod 754 test.txt
  # 此时：所有者可读可写可执行，同组用户可读可执行，其他用户仅可读
  ```

#### 2. MAC（强制访问控制）
MAC 是在 DAC 之上增加的一层强制安全策略，**即使 DAC 权限满足，MAC 策略不通过也无法访问**：
- **核心机制**：为所有主体（进程）和客体（文件）分配 **安全标签**（如 SELinux 的 `user:role:type:level`）。
- **决策逻辑**：系统内核根据预设的安全策略（如 SELinux 的 `allow` 规则），检查主体标签是否允许访问客体标签，**这个决策不受用户/所有者干预**。
- **典型优势**：**限制 root 权限**——即使是 root 进程，若 SELinux 策略不允许其访问某文件，访问也会被拒绝；同时可实现“进程沙箱化”，比如 Android 中每个 App 进程都有独立标签，无法访问其他 App 的数据。

  **示例（SELinux 场景）**：
  ```bash
  # 查看文件的 SELinux 标签
  ls -Z test.txt
  # 输出：unconfined_u:object_r:user_home_t:s0 test.txt
  # 标签含义：用户:角色:类型:级别
  # 只有进程标签匹配 user_home_t 类型，且策略允许，才能访问该文件
  ```

### 三、关键总结：DAC 与 MAC 的关系
1. **DAC 是基础，MAC 是补充**：Linux 权限检查的顺序是 **先 DAC，后 MAC**——DAC 权限不通过，直接拒绝；DAC 通过后，再检查 MAC 策略。
2. **权限控制的“自由度”相反**：DAC 是“所有者说了算”，灵活性高但安全性低；MAC 是“系统策略说了算”，安全性高但灵活性低。
3. **应用场景互补**：普通桌面环境用 DAC 即可满足需求；服务器、Android、高安全系统必须启用 MAC（如 SELinux）来防范恶意攻击。

### 四、技术延伸（贴合你的工作场景）
你日常接触的 **Android 系统** 和 **Linux 服务器** 中，SELinux 就是 MAC 的典型实现：
- Android 中，SELinux 强制限制 App 进程只能访问自身沙箱内的文件，防止恶意 App 窃取数据；
- 服务器中，SELinux 可限制 `nginx` `mysql` 等进程的权限范围，即使进程被入侵，攻击者也无法跨权限访问系统核心文件。

我可以帮你整理一份 **SELinux 常用命令速查表**，包含标签查看、策略切换、权限排查的指令，方便你在 Android/服务器运维中使用。需要吗？



# Discretionary

### 单词：**Discretionary**
1.  **音标**
    - 英式发音：/dɪˈskreʃənəri/
    - 美式发音：/dɪˈskreʃəneri/
2.  **词性**
    形容词（adj.）
3.  **核心释义**
    | 释义类型              | 英文解释                                                     | 中文翻译               |
    | --------------------- | ------------------------------------------------------------ | ---------------------- |
    | 常用义（权限/决策类） | based on or decided by individual judgment or choice, rather than fixed rules | 自主决定的；自由裁量的 |
    | 金融/商务义           | available for use as needed or desired                       | 可自由支配的           |
4.  **搭配与例句**
    - **高频搭配（技术/权限场景）**
      1. discretionary access control (DAC) 自主访问控制
      2. discretionary permission 自主权限
      3. discretionary decision 自主决策
    - **中英双语例句**
      1.  **Linux 权限核心场景**
          > In **Discretionary Access Control**, the file owner has discretionary rights to grant or revoke access permissions to other users.
          > （在自主访问控制模型中，文件所有者拥有自主决定是否向其他用户授予或撤销访问权限的权利。）
      2.  **技术会议场景**
          > The discretionary permission settings in DAC are flexible but less secure than MAC for high-risk systems.
          > （自主访问控制中的自主权限设置很灵活，但对于高风险系统来说，安全性不如强制访问控制。）
      3.  **金融拓展例句**
          > The company has a discretionary budget for emergency technical upgrades.
          > （该公司有一笔可自由支配的预算，用于紧急技术升级。）
5.  **词根词缀拆解**
    - 词根：`discretion`（n. 自由裁量权；谨慎）
    - 后缀：`-ary`（形容词后缀，表“与……相关的”）
    - 衍生逻辑：`discretion`（自由裁量权）+ `-ary` → 具有自由裁量权的 → **自主决定的**

6.  **近义词辨析**
    | 单词              | 侧重点                                     | 例句对比                                                     |
    | ----------------- | ------------------------------------------ | ------------------------------------------------------------ |
    | **Discretionary** | 强调“**由使用者自主决定**”，无强制规则约束 | The file owner has discretionary control over access rights. |
    | **Optional**      | 强调“**可选择的，非必须的**”               | Enabling this feature is optional for basic users.           |
    | **Voluntary**     | 强调“**自愿的，主动的**”                   | The team made a voluntary decision to adopt stricter security policies. |

我可以帮你整理一份**Linux 权限模型相关术语单词卡**，包含 DAC、MAC、SELinux 等核心词汇的音标和例句，需要吗？



# subject與object是主體與客體?

是的，在 Linux 权限控制（包括 DAC 和 MAC）的语境中，**subject** 就是**主体**，**object** 就是**客体**，二者是权限访问模型的核心概念。

### 一、核心定义与区分
| 概念         | **Subject（主体）**                                          | **Object（客体）**                                           |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **中文释义** | 主体                                                         | 客体                                                         |
| **技术含义** | 发起访问请求的**主动方**，通常是系统中的**进程/线程**，关联到具体的用户/用户组（UID/GID）或安全标签 | 被访问的**被动方**，是系统中的**资源**，常见类型包括文件、目录、设备、端口、管道等 |
| **核心作用** | 主动发起“读、写、执行”等访问操作                             | 作为被访问的目标，需要依据权限规则判断是否允许主体访问       |

### 二、技术场景示例
1.  **DAC 模型场景**
    当你执行 `cat /etc/passwd` 命令时：
    - **Subject（主体）**：`cat` 这个进程，关联的是你当前登录的用户 UID（比如 `1000`）。
    - **Object（客体）**：`/etc/passwd` 这个文件，拥有自己的权限位（比如 `rw-r--r--`）。
    - 系统检查：主体的 UID 是否匹配客体的所有者/同组权限，决定是否允许读取。

2.  **MAC 模型（SELinux）场景**
    当 Android 中的一个 App 进程试图读取其他 App 的数据文件时：
    - **Subject（主体）**：该 App 的进程，带有 SELinux 安全标签（比如 `u:r:untrusted_app:s0`）。
    - **Object（客体）**：其他 App 的数据文件，标签为 `u:object_r:app_data_file:s0:c123,c456`。
    - 系统检查：即使 DAC 权限允许，若 SELinux 策略中没有 `allow untrusted_app app_data_file:file read;` 这条规则，主体也无法访问客体。

### 三、补充说明
这个“主体-客体”的概念不仅适用于 Linux 权限控制，在整个**计算机安全领域**都是通用的，比如防火墙规则、数据库权限管理等场景，核心逻辑都是**“主体请求访问客体，系统依据策略判断是否允许”**。

我可以帮你整理一份**权限模型主体客体对应关系速查表**，结合 DAC/MAC 场景列出常见的主体和客体类型，需要吗？