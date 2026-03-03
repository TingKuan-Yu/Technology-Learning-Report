

[TOC]



# PowerShell 的**自動變數**

你想了解的是 PowerShell 中的**自动变量（Automatic Variables）**，它是 PowerShell 预定义、自动创建并维护的一组特殊变量，无需你手动声明，就能在脚本/命令中直接使用，用来获取运行时的环境、状态、输入输出等关键信息。

简单来说，自动变量就像 PowerShell 给你准备的“系统状态仪表盘”，能让你随时查看当前的执行环境、命令结果、错误信息等，是编写灵活脚本的核心工具。

### 一、自动变量的核心特点
1. **预定义且只读**：大部分自动变量由 PowerShell 自动创建和更新，你只能读取其值，无法手动修改（少数如 `$ErrorActionPreference` 可修改）；
2. **上下文相关**：值会随执行环境、命令状态动态变化（比如 `$PWD` 会随当前目录切换而更新）；
3. **全局可用**：在控制台、脚本、函数中都能直接使用，无需导入或声明。

### 二、常用自动变量分类及示例
我按使用场景把最常用的自动变量分类，方便你理解和记忆：

#### 1. 环境/路径相关（最基础）
| 变量名    | 作用                                                         | 示例值                                       |
| --------- | ------------------------------------------------------------ | -------------------------------------------- |
| `$PWD`    | 当前工作目录（Print Working Directory），等价于 `Get-Location` | `C:\Users\你的用户名`                        |
| `$HOME`   | 当前用户的主目录                                             | `C:\Users\你的用户名`                        |
| `$PSHome` | PowerShell 的安装目录                                        | `C:\Windows\System32\WindowsPowerShell\v1.0` |
| `$Env:`   | 环境变量集合（不是单个变量，是变量驱动器），如 `$Env:PATH` 查看系统PATH | `C:\Windows\system32;...`                    |

**示例**：
```powershell
# 查看当前目录
Write-Host "当前目录：$PWD"

# 查看系统PATH环境变量
$Env:PATH

# 切换到用户主目录
cd $HOME
```

#### 2. 命令执行/结果相关（最常用）
| 变量名          | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| `$?`            | 上一条命令的执行状态：`$true` 表示成功，`$false` 表示失败    |
| `$LASTEXITCODE` | 上一个外部程序（如 EXE、CMD 命令）的退出码（0=成功，非0=失败） |
| `$^`            | 上一条命令中第一个标记（单词）                               |
| `$$`            | 上一条命令中最后一个标记（单词）                             |

**示例**：
```powershell
# 执行一个成功的命令
Get-ChildItem $HOME
$?  # 输出 True

# 执行一个失败的命令
Get-ChildItem "不存在的路径"
$?  # 输出 False

# 执行外部程序（如 ping），查看退出码
ping 127.0.0.1
$LASTEXITCODE  # 成功则输出 0，失败则输出非0
```

#### 3. 输入/参数相关（脚本/函数必备）
| 变量名               | 作用                                                         |
| -------------------- | ------------------------------------------------------------ |
| `$args`              | 脚本/函数未声明参数时，接收所有传入的参数（数组）            |
| `$PSBoundParameters` | 脚本/函数中已绑定的参数（键值对哈希表），仅高级函数/脚本可用 |
| `$Input`             | 管道输入的内容集合（仅在脚本/函数的 `process` 块中生效）     |
| `$MyInvocation`      | 当前脚本/函数的调用信息（如脚本路径、调用参数、运行位置）    |

**示例**：
```powershell
# 脚本 test.ps1 内容
Write-Host "传入的所有参数：$args"
Write-Host "第一个参数：$($args[0])"
Write-Host "脚本路径：$($MyInvocation.MyCommand.Path)"

# 调用脚本：.\test.ps1 测试1 测试2
# 输出：
# 传入的所有参数：测试1 测试2
# 第一个参数：测试1
# 脚本路径：C:\Temp\test.ps1
```

#### 4. 错误/调试相关（排错必备）
| 变量名                   | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `$Error`                 | 错误集合，存储当前会话中所有的错误信息（数组，`$Error[0]` 是最新错误） |
| `$ErrorActionPreference` | 默认错误处理策略（可修改）：`SilentlyContinue`（忽略）、`Stop`（终止）、`Continue`（继续） |
| `$PSCmdlet`              | 仅高级函数/脚本可用，指向当前 Cmdlet/脚本的执行对象，用于 `ShouldProcess` 等操作 |

**示例**：
```powershell
# 执行错误命令
Get-ChildItem "不存在的路径"

# 查看最新错误详情
$Error[0]

# 查看错误消息（简化版）
$Error[0].Message

# 修改默认错误处理策略（忽略所有错误）
$ErrorActionPreference = "SilentlyContinue"
Get-ChildItem "不存在的路径"  # 不再报错
```

#### 5. 数组/循环相关（脚本逻辑常用）
| 变量名           | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| `$null`          | 空值，用于判断变量是否为空（PowerShell 中 `$null` 是单例）   |
| `$true`/`$false` | 布尔值（True/False），注意区分大小写（PowerShell 中 `$TRUE` 也等效） |
| `$foreach`       | `foreach` 循环中，指向当前循环的枚举器对象                   |
| `$matches`       | 正则表达式匹配后，存储匹配结果的哈希表                       |

**示例**：
```powershell
# 判断变量是否为空
$name = $null
if ($name -eq $null) {
    Write-Host "变量为空"
}

# 正则匹配后获取结果
"Hello123World" -match "(\d+)"
$matches[1]  # 输出 123（匹配到的数字）
```

#### 6. 其他高频变量
| 变量名         | 作用                                              |
| -------------- | ------------------------------------------------- |
| `$Host`        | 当前 PowerShell 主机的信息（如版本、名称）        |
| `$PID`         | 当前 PowerShell 进程的 ID                         |
| `$PROFILE`     | PowerShell 配置文件的路径（启动时自动加载的脚本） |
| `$LASTCOMMAND` | 上一条执行的命令文本（PowerShell 7+ 支持）        |

**示例**：
```powershell
# 查看当前 PowerShell 进程ID
Write-Host "当前进程ID：$PID"

# 查看配置文件路径（如果不存在可创建）
$PROFILE

# 查看 PowerShell 版本信息
$Host.Version
```

### 三、如何查看所有自动变量？
作为新手，你可以用以下命令快速查看所有自动变量及其值，方便学习和排查：
```powershell
# 列出所有自动变量（按名称排序）
Get-Variable -Scope Global | Where-Object { $_.Options -match "ReadOnly|Constant" } | Sort-Object Name

# 查看单个自动变量的详情（比如 $Error）
Get-Variable Error | Format-List *
```

### 四、使用注意事项
1. **不要重命名/覆盖**：避免创建和自动变量同名的自定义变量（比如 `$PWD = "test"`），会导致系统状态读取异常；
2. **区分作用域**：部分自动变量（如 `$args`）在函数内和脚本内的作用域不同，函数内的 `$args` 仅接收函数参数；
3. **版本差异**：少数自动变量（如 `$LASTCOMMAND`）仅在 PowerShell 7+ 支持，PowerShell 5.1 中无此变量。

---

### 总结
1. **自动变量是 PowerShell 预定义的“系统状态变量”**，无需声明即可直接使用，核心用于获取环境、命令、错误等信息；
2. **高频常用的有**：`$PWD`（当前目录）、`$?`（命令执行状态）、`$Error`（错误信息）、`$args`（传入参数）、`$null`（空值）；
3. **核心价值**：让你无需额外代码就能获取运行时上下文，大幅提升脚本的灵活性和健壮性（比如用 `$?` 判断命令是否成功，用 `$Error` 排查问题）。

记住：编写 PowerShell 脚本时，合理利用自动变量能避免重复造轮子，比如用 `$PWD` 代替手动写路径，用 `$?` 代替复杂的错误判断，是提升脚本效率的关键。

# 查到自動變量如 $MyInvocation 有MyCommand 又有Path

你想知道的是如何查看自动变量（比如 `$MyInvocation`）的层级结构——比如它包含 `MyCommand` 属性，而 `MyCommand` 又包含 `Path` 属性，核心方法是通过 PowerShell 的内置命令查看变量的**属性和值**，我会教你几种实用方法，让你能清晰看到这类嵌套属性的完整结构。

### 一、核心方法：查看自动变量的完整结构
`$MyInvocation` 是一个**对象类型**的自动变量，包含多层嵌套属性（如 `MyCommand`、`ScriptLineNumber` 等），要查到它的所有属性（包括嵌套的 `MyCommand.Path`），最常用的有 3 种方法：

#### 方法 1：用 `Get-Member` 查看属性/方法（最基础）
`Get-Member`（简写 `gm`）能列出对象的所有属性、方法，是查看变量结构的核心命令。
```powershell
# 查看 $MyInvocation 的所有属性（只看属性，排除方法）
$MyInvocation | Get-Member -MemberType Property

# 输出示例（关键部分）：
Name           MemberType Definition
----           ---------- ----------
BoundParameters Property   System.Collections.Generic.Dictionary[string,object] BoundParameters {get;}
MyCommand      Property   System.Management.Automation.CommandInfo MyCommand {get;}
ScriptLineNumber Property   int ScriptLineNumber {get;}
# ... 其他属性
```

如果想进一步查看 `MyCommand` 的属性（比如 `Path`），只需对嵌套属性再用 `Get-Member`：
```powershell
# 查看 $MyInvocation.MyCommand 的所有属性
$MyInvocation.MyCommand | Get-Member -MemberType Property

# 输出示例（关键部分）：
Name       MemberType Definition
----       ---------- ----------
Path       Property   string Path {get;}
Name       Property   string Name {get;}
CommandType Property   System.Management.Automation.CommandTypes CommandType {get;}
# ... 其他属性
```

#### 方法 2：用 `Format-List *` 查看属性值（最直观）
`Format-List *`（简写 `fl *`）会列出对象**所有属性的名称和对应值**，能直接看到 `MyCommand.Path` 的具体值，适合快速查看实际内容：
```powershell
# 查看 $MyInvocation 的所有属性及值（包括嵌套的 MyCommand）
$MyInvocation | Format-List *

# 输出示例（关键部分）：
MyCommand             : test.ps1
BoundParameters       : {}
ScriptLineNumber      : 1
InvocationName        : .\test.ps1
PipelineLength        : 0
PipelinePosition      : 0
MyCommand | Format-List *  # 展开 MyCommand 的详情：
    Name               : test.ps1
    Path               : C:\Temp\test.ps1  # 这就是你要找的 Path 属性值
    CommandType        : ExternalScript
    # ... 其他属性
```

如果只想直接获取 `$MyInvocation.MyCommand.Path` 的值，可直接访问：
```powershell
# 直接获取脚本的完整路径（脚本内执行）
Write-Host "当前脚本路径：$($MyInvocation.MyCommand.Path)"
```

#### 方法 3：用 `Select-Object` 精准查看嵌套属性
如果只想看指定的嵌套属性（比如只看 `MyCommand.Path` 和 `ScriptLineNumber`），用 `Select-Object` 更简洁：
```powershell
# 只查看 $MyInvocation 的 ScriptLineNumber 和 MyCommand.Path
$MyInvocation | Select-Object ScriptLineNumber, @{
    Name = "ScriptPath"
    Expression = { $_.MyCommand.Path }
}

# 输出示例：
ScriptLineNumber ScriptPath
---------------- ----------
1                C:\Temp\test.ps1
```

### 二、实战示例：在脚本中查看 `$MyInvocation.MyCommand.Path`
创建一个测试脚本 `test.ps1`，内容如下：
```powershell
# 查看 $MyInvocation 的完整结构
Write-Host "=== 完整的 MyInvocation 结构 ==="
$MyInvocation | Format-List *

# 直接获取脚本路径
Write-Host "`n=== 脚本的完整路径 ==="
$scriptPath = $MyInvocation.MyCommand.Path
Write-Host "当前脚本路径：$scriptPath"

# 若脚本未保存（仅在控制台执行），Path 会为空，可补充判断
if ([string]::IsNullOrEmpty($scriptPath)) {
    Write-Host "注意：当前未执行脚本文件，Path 属性为空"
}
```

执行脚本：
```powershell
.\test.ps1
```

输出效果（关键部分）：
```
=== 完整的 MyInvocation 结构 ===
MyCommand             : test.ps1
BoundParameters       : {}
ScriptLineNumber      : 1
InvocationName        : .\test.ps1
...
MyCommand             :
    Name               : test.ps1
    Path               : C:\Temp\test.ps1
    ...

=== 脚本的完整路径 ===
当前脚本路径：C:\Temp\test.ps1
```

### 三、注意事项
1. **`$MyInvocation.MyCommand.Path` 的生效场景**：
   - 仅在**脚本文件中执行**时才有值（指向脚本的完整路径）；
   - 若在 PowerShell 控制台直接输入 `$MyInvocation.MyCommand.Path`，值为空（因为没有脚本文件）；
2. **PowerShell 7+ 替代方案**：
   新版 PowerShell 提供了更简洁的 `$PSCommandPath` 自动变量，直接等价于 `$MyInvocation.MyCommand.Path`，推荐优先使用：
   ```powershell
   Write-Host "当前脚本路径：$PSCommandPath"  # 效果和 $MyInvocation.MyCommand.Path 一致
   ```
3. **属性大小写**：PowerShell 对属性名不区分大小写（如 `$MyInvocation.mycommand.path` 也生效），但建议按规范写驼峰式（`MyCommand.Path`）。

---

### 总结
1. 查看嵌套属性（如 `$MyInvocation.MyCommand.Path`）的核心方法：
   - 用 `Get-Member` 查**属性名称**（结构）；
   - 用 `Format-List *` 查**属性值**（内容）；
   - 直接访问 `$变量.嵌套属性.子属性` 取具体值；
2. `$MyInvocation.MyCommand.Path` 用于获取**当前执行脚本的完整路径**，仅在脚本文件中生效；
3. PowerShell 7+ 可直接用 `$PSCommandPath` 替代 `$MyInvocation.MyCommand.Path`，更简洁。

通过这几种方法，你可以轻松查看任何自动变量的嵌套属性，比如 `$Error[0].Exception.Message`、`$PSBoundParameters.Keys` 等，是分析 PowerShell 对象的必备技巧。

# 自動變量為何可以向cmdlet一樣執行, 如$MyInvocation就可直接印出

你好奇的是为什么 PowerShell 的自动变量（比如 `$MyInvocation`）能像 Cmdlet 一样直接输入就输出内容，核心原因是：**PowerShell 对「变量引用」的执行逻辑是「获取变量值并自动格式化输出」，而非自动变量本身是 Cmdlet**——你看到的“直接印出”，是 PowerShell 对所有变量（包括自动变量）的默认处理行为，和 Cmdlet 的执行逻辑本质不同。

下面我会拆解这个过程，帮你理清「变量输出」和「Cmdlet 执行」的核心区别，以及为什么自动变量能“直接印出”。

### 一、先明确：自动变量 ≠ Cmdlet
首先要纠正一个认知：
- **Cmdlet**（如 `Get-ChildItem`）是「可执行的命令」，需要 PowerShell 引擎解析并执行其逻辑（比如读取文件、返回对象）；
- **自动变量**（如 `$MyInvocation`）是「存储数据的容器」，本身不包含可执行逻辑，只是存储了 PowerShell 运行时的状态数据（对象/值）。

你输入 `$MyInvocation` 能直接输出内容，不是因为 `$MyInvocation` 是“可执行命令”，而是 PowerShell 的「默认输出机制」在起作用。

### 二、PowerShell 变量“直接印出”的核心逻辑
PowerShell 是「面向对象的壳」，其核心规则是：**任何未被重定向/赋值的“值引用”，都会被自动发送到「输出流（Success Stream）」，并由 PowerShell 自动格式化显示**。

这个过程分为 3 步（以 `$MyInvocation` 为例）：
1. **变量解析**：PowerShell 看到 `$MyInvocation`，首先解析为「获取该变量存储的对象值」（比如一个 `InvocationInfo` 类型的对象）；
2. **发送到输出流**：因为这个值没有被赋值给其他变量（如 `$x = $MyInvocation`），也没有被管道传递（如 `$MyInvocation | Format-Table`），所以会被自动发送到“成功输出流”；
3. **自动格式化显示**：PowerShell 会根据对象的类型，选择合适的格式（表格/列表）展示其属性和值（比如 `$MyInvocation` 会以列表形式展示 `MyCommand`、`ScriptLineNumber` 等属性）。

#### 对比验证：普通变量也能“直接印出”
自动变量的这个特性，和普通自定义变量完全一致——只要你引用变量的值且不做其他处理，就会自动输出：
```powershell
# 自定义普通变量
$myVar = @{Name="测试"; Value=123}

# 直接引用变量，会自动输出内容（和 $MyInvocation 行为一致）
$myVar

# 输出：
Name                           Value
----                           -----
Name                           测试
Value                          123
```

#### 反例：变量值被处理时，不会“直接印出”
如果变量值被赋值/管道传递/重定向，就不会触发自动输出：
```powershell
# 赋值给其他变量 → 无输出
$x = $MyInvocation

# 管道传递给格式化命令 → 按指定格式输出（而非默认格式）
$MyInvocation | Format-Table MyCommand, ScriptLineNumber

# 重定向到文件 → 内容写入文件，控制台无输出
$MyInvocation > invocation.log
```

### 三、自动变量“输出内容有意义”的额外原因
你觉得 `$MyInvocation` 直接输出“有用”，还因为：
1. **自动变量存储的是「结构化对象」**：比如 `$MyInvocation` 存储的是 `InvocationInfo` 对象（包含 `MyCommand`、`Path` 等属性），`$PWD` 存储的是 `DirectoryInfo` 对象，PowerShell 格式化这些对象时会展示关键属性，看起来“有意义”；
2. **自动变量的值是「动态更新的」**：PowerShell 会实时维护自动变量的值（比如 `$PWD` 随目录切换更新），所以你任何时候引用，都能得到当前的有效状态，而非固定值。

#### 对比：存储简单值的自动变量输出更“简洁”
如果自动变量存储的是简单值（而非复杂对象），输出也会更简单，本质逻辑不变：
```powershell
# $PID 存储的是数字（进程ID），直接输出就是数字
$PID
# 输出：12345（你的PowerShell进程ID）

# $true 存储的是布尔值，直接输出就是 True
$true
# 输出：True
```

### 四、和 Cmdlet 执行的核心区别
为了更清晰，我用表格对比「输入自动变量」和「执行 Cmdlet」的本质差异：

| 行为             | `$MyInvocation`（变量引用）      | `Get-Location`（Cmdlet 执行）                              |
| ---------------- | -------------------------------- | ---------------------------------------------------------- |
| 本质             | 获取变量存储的**已有值**         | 执行命令的**逻辑**，生成新值                               |
| 触发逻辑         | 变量解析 → 发送到输出流 → 格式化 | 解析命令 → 执行内置逻辑 → 返回对象 → 发送到输出流 → 格式化 |
| 是否消耗计算资源 | 几乎无（仅读取内存中的值）       | 有（执行命令逻辑，如读取系统状态）                         |
| 能否无参数执行   | 是（变量本身就是值）             | 是（多数 Cmdlet 有默认参数）                               |
| 能否接收参数     | 否（变量是值容器，不是命令）     | 是（如 `Get-Location -PSProvider FileSystem`）             |

### 五、实战验证：手动关闭“自动输出”
你可以通过 `Out-Null` 关闭自动输出，验证“输出”是流的行为，而非变量本身的特性：
```powershell
# 直接引用 $MyInvocation → 输出内容
$MyInvocation

# 引用 $MyInvocation 并发送到 Out-Null → 无输出（值仍存在，只是不显示）
$MyInvocation | Out-Null

# 验证值仍存在：赋值后输出
$x = $MyInvocation
$x  # 依然会输出内容
```

---

### 总结
1. 自动变量能“直接印出”，核心是 **PowerShell 对「未处理的变量值」会自动发送到输出流并格式化显示**，而非自动变量是 Cmdlet；
2. 自动变量本身是「存储状态数据的容器」，无执行逻辑，和 Cmdlet（可执行命令）本质不同；
3. 所有变量（包括自定义变量）都有这个“直接印出”的特性，自动变量只是因为存储了系统状态数据，输出内容更有实际意义。

简单说：你输入 `$MyInvocation` 时，PowerShell 做的是“把这个变量里存的东西拿出来给你看”，而输入 `Get-Location` 时，PowerShell 做的是“执行这个命令，获取当前位置，再把结果给你看”——前者是“读数据”，后者是“执行+读数据”，这就是核心区别。