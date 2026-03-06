[TOC]



# 文件标签 + 域转换规则（type_transition） + 启动上下文

在 Android SELinux 体系中，一个可执行文件被标记（label）为 `abc`，其运行后的进程 **subject（主体）不一定等于 `abc`**——最终的 subject 由「文件标签 + 域转换规则（type_transition） + 启动上下文」共同决定，存在多种不同情况。以下是详细拆解：

### 一、核心概念先理清
- **文件标签（file label）**：可执行文件本身的 SELinux 标签（如你说的 `abc`），用 `ls -Z` 查看：
  ```bash
  ls -Z /data/local/tmp/your_executable
  # 输出示例：u:object_r:abc:s0  your_executable → 文件标签是 abc
  ```
- **进程主体（process subject/domain）**：进程运行时的 SELinux 域（subject），用 `ps -Z` 查看：
  ```bash
  ps -Z | grep your_executable
  # 输出示例：u:r:untrusted_app:s0  xxx  your_executable → 进程域是 untrusted_app（而非 abc）
  ```

### 二、进程 subject 是否等于文件 label 的 3 种核心场景
#### 场景 1：无域转换规则 → subject = 文件 label（仅特殊场景）
只有满足以下所有条件，进程 subject 才会等于文件 label `abc`：
1. 可执行文件的 label 是 `abc`；
2. 启动该文件的父进程域（如 `init`）有 `type_transition` 规则允许转换到 `abc`；
3. 无其他优先级更高的域转换规则覆盖；
4. 内核/SELinux 策略未限制该 label 作为进程域。

**示例（仅 init 启动时可能生效）**：
```bash
# 策略中定义 abc 是进程域（domain）
type abc, domain, mlstrustedobject;

# 定义 init 启动 abc 文件时，进程域转为 abc
type_transition init_t abc:process abc;

# 允许 init 启动 abc
allow init_t abc:file execute;
allow init_t abc:process transition;
```
此时 `init` 启动 `/data/local/tmp/your_executable`（label=abc），进程 subject 就是 `abc`。

#### 场景 2：有默认域转换规则 → subject ≠ 文件 label（最常见）
Android 有大量**默认域转换规则**，会覆盖文件 label，导致进程 subject 不等于 `abc`。
**典型案例**：
- 普通 App（`untrusted_app_t` 域）启动 label=abc 的可执行文件 → 进程 subject 仍为 `untrusted_app_t`；
- `root`（`su_t` 域）启动 label=abc 的可执行文件 → 进程 subject 仍为 `su_t`；
- `cameraserver`（`cameraserver_t` 域）启动 label=abc 的可执行文件 → 进程 subject 仍为 `cameraserver_t`。

**底层原因**：
Android 策略中，普通进程启动可执行文件时，默认继承父进程的域，而非文件 label：
```bash
# 策略中无针对 abc 的专属转换规则，触发默认规则
# 父进程 untrusted_app_t 启动任意文件，进程域仍为 untrusted_app_t
type_transition untrusted_app_t file_type:process untrusted_app_t;
```

#### 场景 3：有显式域转换规则 → subject = 规则指定的域（而非 abc）
如果策略中定义了「启动 abc 文件时转换到其他域」，则进程 subject 是规则指定的域：
```bash
# 策略定义：untrusted_app_t 启动 label=abc 的文件时，进程域转为 my_custom_domain
type_transition untrusted_app_t abc:process my_custom_domain;

# 此时进程 subject = my_custom_domain，而非 abc
```

### 三、关键验证方法（确认进程实际 subject）
无论文件 label 是什么，都可以通过以下命令直接查看进程的真实 subject：
```bash
# 方法 1：ps -Z 查看（最直接）
ps -Z | grep <可执行文件名>

# 方法 2：查看进程的 selinux 上下文文件
cat /proc/<PID>/attr/current
# 输出示例：u:r:untrusted_app:s0 → 进程 subject 是 untrusted_app

# 方法 3：strace 跟踪域转换（调试用）
strace -e avc,execve ./your_executable
# 查看是否有 avc: denied 或 transition 相关日志
```

### 四、实操：如何让进程 subject 等于文件 label `abc`
如果需要强制进程 subject 为 `abc`，需在 SELinux 策略中添加 3 类规则：
```bash
# 1. 定义 abc 为进程域
type abc, domain;

# 2. 定义父进程（如 untrusted_app_t）启动 abc 文件时，转换到 abc 域
type_transition untrusted_app_t abc:process abc;

# 3. 允许父进程启动并转换到 abc 域
allow untrusted_app_t abc:file { execute execute_no_trans };
allow untrusted_app_t abc:process transition;

# 4. 允许 abc 域的基础权限（如访问文件、网络等）
allow abc self:process { fork exec };
allow abc tmpfs:file { read write execute };
```

### 总结
1. **核心结论**：可执行文件 label 为 `abc`，运行后的进程 subject 不一定是 `abc`，仅当有显式 SELinux 域转换规则时才会相等；
2. **常见情况**：普通进程（App/root/cameraserver）启动该文件，subject 继承父进程域（如 `untrusted_app_t`/`su_t`），与 `abc` 无关；
3. **验证方法**：用 `ps -Z` 或 `cat /proc/<PID>/attr/current` 查看进程真实 subject。

如果需要针对你的场景（如 `abc` 是摄像头相关 label，想让进程 subject 匹配）调整 SELinux 策略，可以补充具体的启动进程（如 cameraserver/init/App），我会给出完整的策略规则模板。