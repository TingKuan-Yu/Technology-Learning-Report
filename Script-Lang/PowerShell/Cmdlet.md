[TOC]



# Cmdlet 

你想了解的是 PowerShell 中的 Cmdlet（发音类似 “command-let”），它是 PowerShell 最核心、最基础的命令单元，也是你使用 PowerShell 时打交道最多的对象。

### 一、Cmdlet 是什么？
简单来说，**Cmdlet 是 PowerShell 中最小的、可执行的命令单元**，它不是传统的 EXE 程序，而是基于 .NET 框架构建的轻量级命令，专门为 PowerShell 的管道（Pipeline）和对象化数据处理设计。

可以把它理解成：
- 传统 CMD 里的 `dir`、`copy` 是“简单命令”，而 PowerShell 的 `Get-ChildItem`（对应 dir）、`Copy-Item`（对应 copy）就是 Cmdlet；
- 每一个 Cmdlet 都只专注做一件具体的事，比如“获取文件”“创建文件夹”“停止进程”，通过组合多个 Cmdlet 能完成复杂任务。

### 二、Cmdlet 的核心特点
1. **命名规则：动词-名词（Verb-Noun）**
   这是 Cmdlet 最标志性的特征，格式固定为 `动词-名词`，且名词首字母大写，比如：
   - `Get-ChildItem`：获取子项（查看文件/文件夹）
   - `Set-Content`：设置文件内容
   - `Start-Process`：启动进程
   - `Stop-Service`：停止服务
   这种命名方式让你一眼就能看懂命令的作用，比如 `Get-Process` 就是“获取进程”，`New-Item` 就是“新建项目（文件/文件夹）”。

2. **面向对象，而非纯文本**
   这是 Cmdlet 和传统 CMD 命令最核心的区别：
   - CMD 命令输出的是**纯文本**（比如 `dir` 输出的是字符串）；
   - Cmdlet 输出的是**.NET 对象**（包含属性和方法），你可以直接对这些对象的属性进行筛选、修改，无需像 CMD 那样解析文本。

3. **轻量级、可组合**
   Cmdlet 本身体积小，且天生支持“管道（|）”，可以把一个 Cmdlet 的输出直接传给另一个 Cmdlet 作为输入，比如：
   ```powershell
   # 获取所有进程，筛选出内存占用大于 100MB 的，只显示进程名和内存占用
   Get-Process | Where-Object { $_.WorkingSetMB -gt 100 } | Select-Object Name, WorkingSetMB
   ```

### 三、常用 Cmdlet 示例（新手必知）
下面是几个最常用的 Cmdlet，帮你直观理解它的用法：

| Cmdlet 命令     | 作用               | 等价 CMD 命令 |
| --------------- | ------------------ | ------------- |
| `Get-ChildItem` | 查看文件/文件夹    | `dir`         |
| `Get-Content`   | 读取文件内容       | `type`        |
| `Copy-Item`     | 复制文件/文件夹    | `copy`        |
| `Move-Item`     | 移动文件/文件夹    | `move`        |
| `Remove-Item`   | 删除文件/文件夹    | `del`/`rd`    |
| `Get-Process`   | 查看正在运行的进程 | `tasklist`    |
| `Stop-Process`  | 终止进程           | `taskkill`    |

#### 实用示例：
```powershell
# 1. 查看当前目录下的所有文件（简写：gci）
Get-ChildItem

# 2. 读取 test.txt 文件的内容（简写：gc）
Get-Content test.txt

# 3. 筛选出名称包含 "Chrome" 的进程，并终止它
Get-Process -Name *Chrome* | Stop-Process -Force
```

### 四、如何查看/学习 Cmdlet？
作为新手，你可以用以下 Cmdlet 快速查询和学习其他 Cmdlet：
1. `Get-Command`：列出所有可用的 Cmdlet（比如 `Get-Command -Verb Get` 列出所有以 Get 开头的 Cmdlet）；
2. `Get-Help`：查看某个 Cmdlet 的详细帮助（比如 `Get-Help Get-ChildItem -Detailed`）；
3. `Get-Alias`：查看 Cmdlet 的简写别名（比如 `Get-Alias gci` 会显示对应的全称是 Get-ChildItem）。

---

### 总结
1. **Cmdlet 是 PowerShell 的核心命令单元**，基于 .NET 构建，而非传统 EXE 程序；
2. **命名规则固定为「动词-名词」**（如 Get-Item），语义清晰，易记易用；
3. **面向对象输出**，支持管道组合，能高效完成复杂的自动化任务，这是它和传统 CMD 命令的核心区别。

记住：PowerShell 的核心优势就是通过组合简单的 Cmdlet，完成原本需要写复杂脚本的任务，而理解 Cmdlet 是用好 PowerShell 的第一步。