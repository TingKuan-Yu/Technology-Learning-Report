[TOC]



# Android.bp 中genrule的作用

在 Android 构建系统（Soong）中，`genrule` 是一个 **自定义构建规则模块**，核心作用是：**执行用户自定义的命令（如脚本、编译、转换等），生成构建过程中需要的中间文件或最终产物**，弥补 Soong 内置模块（如 `cc_library`、`android_app`、`filegroup`）的灵活性不足——当内置模块无法满足特定构建需求时，`genrule` 允许你直接嵌入 shell 命令、脚本等，实现个性化的文件生成/处理逻辑。


### 一、genrule 的核心定位
Soong 的内置模块（如 `cc_library` 用于编译 C/C++ 库、`android_app` 用于打包 APK）是“声明式”的（只需配置参数，Soong 自动处理编译/打包流程），而 `genrule` 是“命令式”的（你需要明确告诉 Soong 执行什么命令、输入什么文件、输出什么文件）。

简单说：**内置模块解决“常规构建”，genrule 解决“非常规构建”**。


### 二、genrule 的典型使用场景
当你需要在构建过程中做以下事情时，就该用 `genrule`：
1. **文件格式转换**：比如将 XML 转换为 JSON、将 ProtoBuf 定义文件（.proto）编译为 Java/C++ 代码、将模板文件（.tpl）填充变量后生成配置文件；
2. **脚本辅助构建**：执行 Python/Shell 脚本处理源文件（如批量修改代码、生成版本号文件、合并资源）；
3. **自定义编译/打包**：内置模块不支持的编译逻辑（如特殊编译器、自定义链接脚本）；
4. **分版本生成文件**：结合之前的需求，根据 Android 版本（API Level）生成不同的配置文件、脚本输出；
5. **中间文件预处理**：构建前对源文件做预处理（如代码混淆模板生成、静态资源压缩）。


### 三、genrule 的核心属性（必须掌握）
`genrule` 的配置需明确“输入-命令-输出”三个核心环节，以下是最常用的属性（全部为 Soong 规范属性，无自定义扩展）：

| 属性名       | 作用                                                         | 必选 | 示例                                            |
| ------------ | ------------------------------------------------------------ | ---- | ----------------------------------------------- |
| `name`       | 模块名称（唯一标识，其他模块可通过该名称依赖此 genrule 的输出） | 是   | `name: "generate_my_config"`                    |
| `srcs`       | 构建的输入文件（依赖文件），Soong 会监控这些文件变化，变化后重新执行 | 否   | `srcs: ["config.tpl", "version.txt"]`           |
| `out`        | 构建的输出文件（必须明确声明，Soong 会管理输出路径、避免重复生成） | 是   | `out: ["my_config.json"]`                       |
| `cmd`        | 核心执行命令（shell 命令，用于处理 `srcs` 生成 `out`），支持变量引用 | 是   | `cmd: "python3 gen_config.py $(SRCS) > $(OUT)"` |
| `cmd_bash`   | 用 bash 执行命令（兼容复杂 shell 语法，如管道、循环）        | 否   | `cmd_bash: "cat $(SRCS) | grep 'key' > $(OUT)"` |
| `enabled`    | 控制模块是否参与构建（结合条件编译，如分 Android 版本启用）  | 否   | `enabled: android_api_level == 33`              |
| `visibility` | 控制其他模块是否能依赖此 genrule 的输出（默认仅当前目录可见） | 否   | `visibility: ["//vendor/my:__subpackages__"]`   |


### 四、genrule 的关键变量（cmd 中常用）
`cmd`/`cmd_bash` 中可使用 Soong 预定义变量，简化路径引用（无需写绝对路径）：
- `$(SRCDIR)`：当前 `Android.bp` 所在的源文件目录路径；
- `$(SRCS)`：`srcs` 中声明的所有输入文件（空格分隔，含路径）；
- `$(OUT)`：`out` 中声明的所有输出文件（空格分隔，输出路径由 Soong 自动管理，默认在 `out/soong/.intermediates/` 下）；
- `$(OUT_DIR)`：构建输出的根目录（即 `out/` 目录）；
- `$(TARGET_ARCH)`：目标架构（如 `arm64`、`x86_64`）；
- 自定义变量：通过 `variables` 属性声明，如 `variables: { MY_VERSION: "1.0" }`，在 `cmd` 中用 `$(MY_VERSION)` 引用。


### 五、实用示例（覆盖核心场景）
#### 示例 1：用脚本生成配置文件
需求：通过 Python 脚本读取模板文件（`config.tpl`）和版本文件（`version.txt`），生成最终的 `app_config.json` 配置文件。
```bp
genrule {
    name: "generate_app_config",
    srcs: [
        "config.tpl",   // 模板文件（输入）
        "version.txt",  // 版本信息（输入）
    ],
    out: ["app_config.json"], // 生成的配置文件（输出）
    // 执行 Python 脚本，传入输入文件，输出到目标文件
    cmd: "python3 $(SRCDIR)/gen_config.py $(SRCS) > $(OUT)",
    // 允许其他模块依赖此输出（如 android_app 引用配置文件）
    visibility: ["//apps/myapp:__pkg__"],
}

// 其他模块依赖 genrule 的输出（例如 Android 应用）
android_app {
    name: "MyApp",
    srcs: ["src/**/*.java"],
    // 依赖 genrule 生成的配置文件（自动先执行 genrule，再打包 APK）
    srcs: [":generate_app_config"],
    // 将生成的配置文件打包到 APK 的 assets 目录
    assets: [":generate_app_config"],
    assets_dir: "config",
}
```

#### 示例 2：分 Android 版本生成不同文件（结合之前需求）
需求：Android 13（API 33）生成 `android13_config.xml`，Android 15（API 35）生成 `android15_config.xml`。
```bp
genrule {
    name: "generate_versioned_config",
    // 分版本输入（13 和 15 不同的模板）
    srcs: (android_api_level == 33) ? [
        "android13_config.tpl",
    ] : (android_api_level == 35) ? [
        "android15_config.tpl",
    ] : [],
    // 分版本输出
    out: (android_api_level == 33) ? [
        "android13_config.xml",
    ] : [
        "android15_config.xml",
    ],
    // 分版本执行命令（不同版本用不同脚本参数）
    cmd_bash: (android_api_level == 33) ?
        "sed 's/\\${API_LEVEL}/33/g' $(SRCS) > $(OUT)" :
        "sed 's/\\${API_LEVEL}/35/g' $(SRCS) > $(OUT)",
}
```

#### 示例 3：ProtoBuf 文件编译（生成 Java 代码）
需求：将 `.proto` 协议文件编译为 Java 代码，供 Java 库引用（内置模块未满足时，可用 genrule 自定义）。
```bp
genrule {
    name: "compile_proto_to_java",
    srcs: ["my_proto.proto"], // 输入：ProtoBuf 定义文件
    out: ["com/example/MyProto.java"], // 输出：生成的 Java 代码
    // 执行 protoc 编译器（假设 protoc 已在构建环境中）
    cmd: "protoc --java_out=$(dir $(OUT)) $(SRCS)",
}

// Java 库依赖生成的代码
java_library {
    name: "my_proto_lib",
    srcs: [":compile_proto_to_java"], // 依赖 genrule 输出
    shared_libs: ["libprotobuf-java"],
}
```


### 六、genrule 的使用注意事项
1. **输出文件必须明确声明**：`out` 必须列出所有生成的文件，Soong 会通过 `out` 管理构建缓存（仅当 `srcs` 或 `cmd` 变化时，才重新执行）；若漏写输出文件，可能导致构建缓存失效或文件丢失。
2. **避免硬编码路径**：优先使用 `$(SRCDIR)`、`$(OUT_DIR)` 等变量，而非 `/home/user/xxx` 这类绝对路径，确保构建环境兼容性（不同机器、不同构建目录都能正常执行）。
3. **依赖传递需显式**：若 `genrule` 的输入依赖其他模块（如另一个 `genrule` 的输出），直接在 `srcs` 中引用该模块名称即可（如 `srcs: [":other_genrule"]`），Soong 会自动处理执行顺序（先执行依赖模块，再执行当前 `genrule`）。
4. **性能考虑**：`genrule` 执行 shell 命令/脚本，效率低于 Soong 内置模块（内置模块有编译缓存、并行优化），非必要不滥用（例如简单的文件复制用 `filegroup`，而非 `genrule` 的 `cp` 命令）。
5. **调试技巧**：若 `genrule` 执行失败，可通过 `m <genrule_name> -j1 V=1` 查看详细执行日志（`V=1` 输出完整 `cmd` 命令，方便定位脚本/命令错误）。


### 总结
`genrule` 是 Android.bp 中“灵活兜底”的构建模块，核心价值是 **支持自定义命令生成文件**，解决内置模块无法覆盖的特殊构建需求。使用时只需明确“输入（srcs）- 命令（cmd）- 输出（out）”，再结合 Soong 变量和条件编译，即可实现复杂的构建逻辑（如分版本生成、脚本辅助、格式转换等）。