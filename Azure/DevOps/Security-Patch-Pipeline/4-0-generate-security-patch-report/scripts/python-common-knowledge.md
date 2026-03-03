# Python common knowledge



## `__init__.py` 的作用

- 這個檔案是空的，但它的存在具有重要意義。

- 主要功能

  - 將目錄標記為 Python 套件（Package）

    - __init__.py 的存在使得 utils/ 目錄成為一個 Python 套件，允許其他模組導入它：

      ```python
      from utils import error_util      # ✅ 可以這樣導入
      from utils import git_util        # ✅ 可以這樣導入
      from utils.logger import SimpleLogger  # ✅ 可以這樣導入
      ```

    - **如果沒有 `__init__.py`**：

      ```python
      from utils import error_util      # ❌ 會報錯：ModuleNotFoundError
      ```

- 目錄結構說明

  ```
  utils/
  ├── __init__.py         ← 將 utils/ 標記為套件
  ├── error_util.py       ← 可被導入的模組
  ├── git_util.py         ← 可被導入的模組
  └── logger.py           ← 可被導入的模組
  ```

- `__init__.py` 的三種使用方式

  - 空檔案（目前狀態）
    - 僅將目錄標記為套件
    - 不執行任何初始化程式碼

  - 控制套件導出（可選）

    ```python
    # 如果檔案內容是這樣：
    from .error_util import *
    from .git_util import GitUtil
    
    # 則可以直接這樣導入：
    from utils import GitUtil
    ```

    - from utils import git_util   --> from utils import GitUtil

  - 套件初始化（可選）

    ```python
    # 可以在這裡執行初始化程式碼
    print("utils package loaded")
    
    # 或設定套件層級的變數
    __version__ = "1.0.0"
    ```

- 在你的專案中
  scripts/code_integration/utils/__init__.py 是空的，所以它的唯一作用就是：

  告訴 Python "utils 是一個套件，可以從中導入模組"

  這就是為什麼 git_util.py:4 中可以使用 from utils import error_util 的原因。

## Python 的 `logging`

- Python 的 `logging` 模块是官方推荐的日志处理工具（替代 `print` 语句），支持**分级输出、多目标输出（控制台 / 文件）、日志格式化、模块级隔离**等核心能力，是项目中记录运行状态、调试问题、追踪流程的标准方案。

  | 概念          | 作用                                                         |
  | ------------- | ------------------------------------------------------------ |
  | **日志级别**  | 按重要性划分日志（DEBUG < INFO < WARNING < ERROR < CRITICAL），可过滤低级别日志 |
  | **Logger**    | 日志入口，每个模块 / 项目可创建独立 Logger，负责产生日志     |
  | **Handler**   | 日志输出目标（如控制台 `StreamHandler`、文件 `FileHandler`），可多个 Handler 共存 |
  | **Formatter** | 日志格式（如包含时间、模块、级别、内容）                     |
  | **Filter**    | 日志过滤规则（如仅记录特定模块 / 级别 / 关键词的日志）       |