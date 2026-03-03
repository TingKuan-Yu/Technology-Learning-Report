



# ps1 file要如何在power shell內執行?

你想知道的是如何在 PowerShell 中执行 `.ps1` 脚本文件，这是使用 PowerShell 自动化的基础操作。执行 `.ps1` 文件的核心要点是：**解决 PowerShell 的执行策略限制** + **使用正确的路径/语法调用脚本**，下面我会一步步讲清楚完整的执行方法和注意事项。

### 一、前提：先了解 PowerShell 执行策略（关键）
PowerShell 默认有「执行策略」安全限制，默认不允许运行未签名的脚本（防止恶意脚本），所以第一步要确认/调整执行策略（仅需设置一次）。

#### 1. 查看当前执行策略
打开 PowerShell（建议以**管理员身份**运行），执行：
```powershell
Get-ExecutionPolicy
```
- 常见结果：
  - `Restricted`：默认值，禁止运行任何脚本（仅允许执行单个命令）；
  - `RemoteSigned`：允许运行本地编写的脚本，远程脚本需签名（推荐）；
  - `Unrestricted`：允许运行所有脚本（不推荐，安全性低）。

#### 2. 修改执行策略（解决“无法加载脚本”报错）
如果执行策略是 `Restricted`，需要修改为 `RemoteSigned`（最安全且够用）：
```powershell
# 以管理员身份运行 PowerShell，执行以下命令
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```
- 按提示输入 `Y` 确认即可；
- `-Scope CurrentUser` 表示仅对当前用户生效，无需系统级权限（更安全）。

### 二、执行 .ps1 脚本的 3 种常用方法
#### 方法 1：使用完整路径执行（最推荐、最不易出错）
无论脚本在哪个目录，直接用**完整路径**调用，避免路径解析问题：
```powershell
# 示例：脚本在 C:\Temp 目录下，文件名为 test.ps1
C:\Temp\test.ps1

# 如果路径包含空格，必须用引号包裹路径，且前面加 &（调用操作符）
& "C:\My Scripts\test.ps1"
```

#### 方法 2：切换到脚本目录后执行（常用）
先通过 `cd` 切换到脚本所在目录，再用相对路径执行：
```powershell
# 1. 切换到脚本目录（示例：C:\Temp）
cd C:\Temp

# 2. 执行脚本（必须加 .\ 表示“当前目录”，PowerShell 不默认搜索当前目录）
.\test.ps1
```
⚠️ 关键：**不能直接写 `test.ps1`**，PowerShell 会优先搜索系统 PATH 路径，而非当前目录，必须加 `.\` 明确指定“当前目录的脚本”。

#### 方法 3：通过 PowerShell 命令行参数执行（适合一次性运行）
可以直接在 PowerShell 启动时指定要执行的脚本，适合批处理/快捷方式：
```powershell
# 格式：powershell.exe -File "脚本完整路径"
powershell.exe -File "C:\Temp\test.ps1"

# 若需要隐藏窗口执行（比如后台运行），加 -WindowStyle Hidden
powershell.exe -WindowStyle Hidden -File "C:\Temp\test.ps1"
```

### 三、带参数执行脚本
如果你的脚本有自定义参数（比如 `test.ps1` 定义了 `-Path` 参数），执行时直接传参即可：
```powershell
# 示例1：当前目录脚本带参数
.\test.ps1 -Path "C:\" -Verbose

# 示例2：完整路径脚本带参数
C:\Temp\test.ps1 -Path "D:\Data" -Recurse

# 示例3：参数值包含空格（用引号包裹）
.\test.ps1 -Name "我的测试文件"
```

### 四、常见问题 & 解决方案
#### 问题 1：报错“无法加载文件 xxx.ps1，因为在此系统上禁止运行脚本”
- 原因：执行策略为 `Restricted`，禁止运行脚本；
- 解决：按前面的方法修改执行策略为 `RemoteSigned`。

#### 问题 2：输入 `test.ps1` 提示“test.ps1 不是内部或外部命令”
- 原因：未加 `.\`（当前目录）或完整路径，PowerShell 未找到脚本；
- 解决：改为 `.\test.ps1`（当前目录）或完整路径。

#### 问题 3：脚本路径含空格，执行时报错
- 原因：路径未加引号，PowerShell 解析错误；
- 解决：用 `&` 调用 + 引号包裹路径，比如 `& "C:\My Scripts\test.ps1"`。

#### 问题 4：脚本执行后一闪而过（双击脚本时）
- 原因：脚本执行完成后自动关闭窗口；
- 解决：在脚本末尾添加 `Pause` 命令（暂停窗口，按任意键关闭）：
  ```powershell
  # test.ps1 末尾添加
  Write-Host "脚本执行完成"
  Pause
  ```

### 五、进阶：在 ISE/VS Code 中执行脚本
如果你用编辑器编写脚本，执行更便捷：
1. **PowerShell ISE**：打开脚本后，按 `F5` 直接运行，或点击“运行脚本”按钮；
2. **VS Code**：安装 PowerShell 扩展后，打开脚本，按 `F5` 运行，或右键选择“Run Script”。

---

### 总结
1. 执行 `.ps1` 脚本的**前提**：先将 PowerShell 执行策略改为 `RemoteSigned`（解决权限限制）；
2. 核心语法：
   - 当前目录：`.\脚本名.ps1`（必须加 `.\`）；
   - 任意目录：`完整路径\脚本名.ps1`（路径含空格时用 `& "路径"`）；
3. 关键注意：路径含空格需用引号包裹 + `&` 调用，带参数时直接跟在脚本名后。

记住这几个核心规则，就能顺畅执行所有 PowerShell 脚本，避免90%的执行报错。