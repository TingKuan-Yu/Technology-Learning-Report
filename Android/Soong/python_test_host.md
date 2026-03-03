[TOC]



#　python_test_host

在 Android 构建系统（Soong）中，`python_test_host` 是一个 **专门用于构建和执行「主机侧 Python 测试」的内置模块**——简单说，它的核心作用是：定义运行在 **构建主机（如你的开发电脑、CI 服务器，通常是 Linux 系统）** 上的 Python 自动化测试，而非运行在 Android 设备/模拟器（目标侧）上的测试。

### 一、核心定位：主机侧 vs 目标侧 Python 测试
Android 构建系统中，Python 测试分两类，`python_test_host` 负责「主机侧」：
| 测试类型           | 运行环境                | 对应的 Soong 模块            | 核心用途                                                     |
| ------------------ | ----------------------- | ---------------------------- | ------------------------------------------------------------ |
| 主机侧 Python 测试 | 构建主机（Linux/macOS） | `python_test_host`           | 测试构建系统依赖的 Python 脚本/工具（如编译辅助脚本、配置生成脚本）、验证主机侧逻辑正确性 |
| 目标侧 Python 测试 | Android 设备/模拟器     | `python_test`（或 `pytest`） | 测试运行在 Android 系统上的 Python 应用/服务（需设备支持 Python 环境） |

简单理解：`python_test_host` 是给「构建过程本身」写的 Python 测试（验证构建工具靠谱），而非给「Android 应用」写的测试。


### 二、`python_test_host` 的典型使用场景
当你需要以下需求时，会用到 `python_test_host`：
1. 测试 Android 构建流程中依赖的 Python 脚本（如 `genrule` 中调用的 Python 生成脚本、自定义构建工具）；
2. 验证主机侧工具的逻辑正确性（如版本号处理、文件格式转换、配置解析等）；
3. 自动化检查构建相关的规则（如 Android.bp 语法校验、资源文件合法性检查）；
4. 构建系统的单元测试/集成测试（如 Soong 模块配置的有效性验证）。


### 三、核心属性（必须掌握）
`python_test_host` 是 Soong 预定义模块，支持以下关键属性（按常用程度排序）：
| 属性名           | 作用                                                         | 必选 | 示例                                                         |
| ---------------- | ------------------------------------------------------------ | ---- | ------------------------------------------------------------ |
| `name`           | 测试模块名称（唯一标识，用于执行测试时指定）                 | 是   | `name: "my_python_host_test"`                                |
| `srcs`           | 测试源文件（Python 测试脚本，如 `test_xxx.py`）              | 是   | `srcs: ["test_script.py", "test_utils.py"]`                  |
| `main`           | 测试入口文件（若多个 `srcs`，指定哪个文件作为执行入口）      | 否   | `main: "test_script.py"`                                     |
| `deps`           | 依赖的 Python 库（主机侧可用的 Python 依赖，如 `pytest`、自定义 Python 模块） | 否   | `deps: [":my_python_lib", "//external/pytest:python3-pytest"]` |
| `args`           | 执行测试时传递给 Python 脚本的命令行参数                     | 否   | `args: ["--verbose", "--config=$(SRCDIR)/test_config.json"]` |
| `data`           | 测试所需的辅助文件（如测试数据、配置文件，会被复制到测试执行目录） | 否   | `data: ["test_data.csv", ":generated_test_config"]`          |
| `enabled`        | 控制测试是否启用（可结合条件编译，如分 Android 版本启用）    | 否   | `enabled: android_api_level >= 33`                           |
| `python_version` | 指定 Python 版本（如 `python3`，默认使用主机环境的 Python 3） | 否   | `python_version: "python3"`                                  |
| `visibility`     | 控制其他模块是否能依赖此测试模块（默认仅当前目录可见）       | 否   | `visibility: ["//vendor/my:__subpackages__"]`                |


### 四、实用示例（覆盖核心场景）
#### 示例 1：基础主机侧 Python 测试（验证脚本逻辑）
需求：测试一个用于构建配置生成的 Python 脚本 `config_generator.py`，确保其输出格式正确。
```bp
// 1. 先定义被测试的 Python 库（若测试的是独立脚本，可省略此步）
python_library_host {
    name: "config_generator_lib",
    srcs: ["config_generator.py"], // 被测试的核心脚本
    python_version: "python3",
}

// 2. 定义主机侧 Python 测试
python_test_host {
    name: "test_config_generator",
    srcs: ["test_config_generator.py"], // 测试脚本
    main: "test_config_generator.py", // 测试入口
    deps: [
        ":config_generator_lib", // 依赖被测试的 Python 库
        "//external/pytest:python3-pytest", // 依赖 pytest 测试框架
    ],
    args: ["--verbose"], // 传递给 pytest 的参数
    data: [
        "test_input.json", // 测试用输入数据
        "expected_output.json", // 预期输出（用于断言）
    ],
    enabled: true, // 始终启用测试
}
```

#### 示例 2：分 Android 版本启用测试（结合之前需求）
需求：Android 15（API 35）新增的主机侧脚本，仅在 API 35 及以上版本执行测试。
```bp
python_test_host {
    name: "test_android15_host_script",
    srcs: ["test_android15_utils.py"],
    deps: [":android15_host_utils"],
    // 仅 Android 15+（API 35+）启用此测试
    enabled: android_api_level >= 35,
    args: [
        "--api-level=$(android_api_level)", // 传递当前构建的 API Level 给测试脚本
    ],
}
```

#### 示例 3：依赖 genrule 生成的测试数据
需求：测试数据是通过 `genrule` 动态生成的，测试脚本需使用该生成数据。
```bp
// 1. genrule 生成测试数据
genrule {
    name: "generate_test_data",
    srcs: ["test_data.tpl"],
    out: ["generated_test_data.json"],
    cmd: "python3 $(SRCDIR)/gen_test_data.py $(SRCS) > $(OUT)",
}

// 2. 测试模块依赖生成的测试数据
python_test_host {
    name: "test_with_generated_data",
    srcs: ["test_data_processor.py"],
    data: [":generate_test_data"], // 依赖 genrule 生成的文件
    args: ["--test-data=$(data_dir)/generated_test_data.json"], // 引用数据文件路径
}
```


### 五、如何执行 `python_test_host` 测试？
定义完成后，可通过 Android 构建命令（`m`/`mm`）执行测试：
```bash
# 1. 构建并执行指定的测试模块（推荐，仅运行目标测试）
mm test_config_generator -j1

# 2. 构建并执行当前目录下所有 python_test_host 测试
mm -j1

# 3. 执行所有主机侧 Python 测试（全量测试，耗时较长）
m python-tests-host

# 4. 调试模式执行（输出详细日志，方便定位失败原因）
mm test_config_generator -j1 V=1
```

测试执行结果会输出到终端，若测试失败（如断言不通过、脚本报错），会显示具体错误信息（如堆栈、失败用例）。


### 六、关键注意事项
1. **运行环境限制**：测试仅在「构建主机」执行，无法访问 Android 设备的资源（如设备文件、系统服务），若需测试设备相关逻辑，需用 `python_test`（目标侧测试）。
2. **Python 版本兼容性**：Android 构建系统默认使用 Python 3（推荐显式指定 `python_version: "python3"`），避免使用 Python 2 语法（已淘汰）。
3. **依赖可用性**：`deps` 中引用的 Python 库必须是「主机侧可用」的（如 `python_library_host` 模块、外部导入的主机侧库），不能引用 Android 设备侧的 Python 库。
4. **测试隔离性**：测试脚本应独立运行，避免依赖外部环境（如硬编码路径、主机本地文件），优先使用 `data` 属性传递辅助文件，确保测试可重复执行。
5. **与 `python_library_host` 的配合**：若测试的是复杂 Python 逻辑（而非单脚本），建议先将核心逻辑封装为 `python_library_host`（主机侧 Python 库），再用 `python_test_host` 依赖该库进行测试，提高代码复用性。


### 总结
`python_test_host` 是 Android.bp 中用于「主机侧 Python 测试」的专用模块，核心价值是自动化验证构建相关的 Python 脚本/工具的正确性，保障构建流程稳定。使用时需明确：测试运行在主机（而非设备）、依赖主机侧 Python 库、通过 `srcs`/`deps`/`data` 配置测试逻辑，最终通过 `mm` 等命令执行测试。

它与 `genrule` 常配合使用（如测试 `genrule` 生成的文件/脚本），也可结合条件编译（`enabled` 属性）实现分版本测试，是 Android 构建系统自动化测试的重要组成部分。