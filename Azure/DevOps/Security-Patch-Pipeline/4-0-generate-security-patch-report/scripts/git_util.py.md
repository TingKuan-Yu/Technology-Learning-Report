# scripts/code_integration/utils/git_util.py



## import git

- 這是一個**第三方套件**，名稱為 **GitPython**，需要額外安裝：

  ```bash
  pip install GitPython
  ```

  或者在這個專案中，因為使用 pipenv，可能在 `Pipfile` 或 `requirements.txt` 中定義。

  - scripts/code_integration/requirements.txt

    ```
    GitPython>=3.1.27
    ```

    

## from git import Repo, TagReference

- `Repo` 是 GitPython 中最核心的類別，代表一個 **Git 倉庫物件**，提供了操作 Git 倉庫的所有功能。

- Repo 的主要功能
  1. 初始化/開啟 Git 倉庫

     ```python
     # 開啟現有倉庫
     repo = Repo('/path/to/repo')
     
     # 複製遠端倉庫
     repo = Repo.clone_from(url='https://github.com/...', to_path='/path')
     ```

  2. . 存取 Git 基本資訊
     repo.remotes - 所有遠端倉庫 (origin, upstream 等)
     repo.branches - 所有分支
     repo.tags - 所有標籤
     repo.git_dir - Git 目錄路徑
     repo.working_dir - 工作目錄路徑

  3. 執行 Git 命令

     ```python
     # 低階命令執行
     repo.git.execute(['git', 'checkout', 'main'])
     
     # 高階 API
     repo.git.checkout('main')
     repo.git.push('origin', 'main')
     ```

  4. 提交與歷史操作

     - `repo.commits()` - 取得提交歷史
     - `repo.commit()` - 建立提交
     - `repo.index` - 操作暫存區 (staging area)

  5. 遠端倉庫管理

     ```python
     # 建立新遠端
     repo.create_remote('upstream', 'https://...')
     
     # 存取遠端
     origin = repo.remotes.origin
     origin.fetch()
     origin.pull()
     ```

     

## from git.exc import GitCommandError, BadName

- `GitCommandError` 是 GitPython 套件中的**異常類別**，用於捕捉和處理 **Git 命令執行失敗**時拋出的錯誤。

- GitCommandError 的作用
  當執行 Git 命令失敗時（如 checkout 不存在的分支、push 被拒絕、merge 衝突等），GitPython 會拋出此異常。

  異常包含的資訊

  ```python
  try:
      repo.git.execute(['git', 'checkout', 'non-existent-branch'])
  except GitCommandError as ex:
      print(ex.status)      # 錯誤狀態碼（通常是 128）
      print(ex.stderr)      # Git 的錯誤訊息（標準錯誤輸出）
      print(ex.stdout)      # Git 的標準輸出
      print(ex.command)     # 執行的命令
  ```



## from utils import error_util

- from utils import error_util 是從專案內部的 utils 模組導入 error_util 子模組，用於定義和管理錯誤常數。

- 這不是 Python 內建或第三方套件，而是專案自定義的模組：

  - scripts/code_integration/**utils**/**error_util**.py

- 相對導入關係：

  當前檔案：git_util.py
  導入模組：error_util.py
  兩者在同一個 utils/ 目錄下

- error_util 的內容
  這個模組定義了錯誤訊息常數，用於統一管理錯誤狀態：

  ```python
  # Tag 相關錯誤
  ERROR_MULTI_TAG = "[error] more than one tag"
  ERROR_NO_TAG = "[error] no tag"
  ERROR_UNKNOWN = "[error] unknown"
  ERROR_TAG_MESSAGE_LIST = [ERROR_MULTI_TAG, ERROR_NO_TAG, ERROR_UNKNOWN]
  
  # Clone 相關錯誤
  ERROR_CANNOT_CLONE = "[error] The repo cannot clone"
  
  # Commit 相關錯誤
  ERROR_NOTTHING_TO_COMMIT = "[error] nothing to commit"
  ```

  

## from utils.logger import SimpleLogger

- 這是 Python 的 從模組中導入特定物件 的語法。

  語法結構拆解

  ```
  from   utils.logger   import   SimpleLogger
    ↓         ↓            ↓           ↓
  關鍵字   模組路徑      關鍵字    要導入的物件
  ```

  - 各部分說明
    1. from - 關鍵字

    表示「從...導入」
    2. utils.logger - 模組路徑

    utils = 套件名稱（**目錄**）
    . = **路徑分隔符**
    **logger = 模組名稱（檔案 logger.py）**
    完整路徑：logger.py

    3. import - 關鍵字

    表示「導入」
    4. SimpleLogger - 要導入的**物件**

    **從 logger.py 中導入 SimpleLogger 類別**
    現在可以直接使用 SimpleLogger，不需要前綴

- 使用方式對比

  - 方法 1：使用 from...import（當前用法）

    ```
    from utils.logger import SimpleLogger
    
    SimpleLogger.logger.info("訊息")  # ✅ 簡潔
    ```

    - 可以導入多個物件

      ```
      # 導入多個
      from utils.logger import SimpleLogger, SomeOtherClass
      
      # 導入全部（不推薦）
      from utils.logger import *
      ```

      

  - **方法 2：使用 `import`**

    ```
    import utils.logger
    
    utils.logger.SimpleLogger.logger.info("訊息")  # ❌ 冗長
    ```

  - **方法 3：使用別名**

    ```
    from utils.logger import SimpleLogger as Logger
    
    Logger.logger.info("訊息")  # ✅ 可自訂名稱
    ```

    

## from typing import List

- 這是從 Python **內建模組 `typing`** 導入 `List` 類型提示（Type Hint）。
- 基本說明
  - typing - Python 3.5+ 的內建模組，用於類型標註（Type Annotations）
  - List - 表示「列表」類型的泛型類別



## class GitUtil:

- def __init__(self, work_path, git_url=None, reference=None, branch=None):
  - 這是 `GitUtil` 類別的**建構子（Constructor）**，用於初始化 Git 倉庫物件。支援兩種模式：**開啟現有倉庫**或**從遠端clone倉庫**。
  - def __init__(self, work_path, git_url=None, reference=None, branch=None):
    - `work_path` - 工作目錄路徑（必需）
    - `git_url` - Git 倉庫 URL（可選，決定是否克隆）
    - `reference` - 參考倉庫路徑（可選，用於加速克隆）
    - `branch` - 指定分支名稱（可選）







## search_commits_by_info()

- Git 提交搜尋工具，可以根據作者、標題、時間、描述等多個條件組合搜尋提交記錄

  ```
  def search_commits_by_info(self, author, title=None, time=None, desc=None, range=None) -> List[GitCommitInfo]:
  ```

  - author - 作者名稱（必需）
  - title - 提交標題/主旨（可選）
  - time - 提交時間（可選）
  - desc - 提交描述/訊息內容（可選）
  - range - Git 範圍（可選，如 commit1..commit2）
  - 返回值：List[GitCommitInfo] - GitCommitInfo 物件列表

- 執行流程詳解

  - 步驟 1：初始化（第 381-383 行）
    定義分隔符：

    LOG_SEP - 欄位分隔符（用於分隔 commit hash, author, date, subject）
    END_SEP - 提交分隔符（用於分隔不同的提交記錄）
    pf - Git log 格式字串
    格式說明：

    %H - 完整 commit hash
    %an - 作者名稱
    %ad - 作者日期
    %s - 提交主旨（標題）

  - 步驟 2：建構搜尋參數（第 384-394 行）

    ```
    if author is not None:
        param += [f"--author={author}"]
    
    if desc is not None:
        param += [f"--grep={desc}", "--fixed-strings"]
    elif title is not None:
        # title is also included in description
        param += [f"--grep={title}", "--fixed-strings"]
    
    if range is not None:
        param += [range]
    ```

    - author - 總是添加（必需參數）
    - desc 或 title - 互斥，desc 優先
      使用 --fixed-strings 確保字面匹配（不使用正則表達式）
    - range - 限制搜尋範圍
    - 注意：title 和 desc 都使用 --grep，因為 Git 中標題也是提交訊息的一部分。

  - 步驟 3：執行 Git 命令（第 396-399 行）

    ```
    try:
        SimpleLogger.logger.info(f"Start to search commits by author: {author}, title: {title}, time: {time}")
        # example: ['598165f1e48a;;Hangyu Hua;;Sat Jan 1 01:21:38 2022 +0800']
        cmd = ['git', 'log' ] + param + [f"--pretty=format:{pf}"]
        output_list = [result.strip() for result in self.repo.git.execute(cmd).strip().split(END_SEP)]
    ```

    - **構建的命令範例**：

      ```bash
      git log \
        --author="John Doe" \
        --grep="Fix security issue" \
        --fixed-strings \
        main..feature-branch \
        --pretty=format:"%H;;[LOGSEP];;%an;;[LOGSEP];;%ad;;[LOGSEP];;%s;;[ENDSEP];;"
      ```

      - **輸出格式範例**：

        ```
        598165f1e48a;;[LOGSEP];;Hangyu Hua;;[LOGSEP];;Sat Jan 1 01:21:38 2022 +0800;;[LOGSEP];;Fix CVE-2024-1234;;[ENDSEP];;
        abc123def456;;[LOGSEP];;Jane Smith;;[LOGSEP];;Mon Feb 5 10:30:00 2022 +0800;;[LOGSEP];;Another fix;;[ENDSEP];;
        ```

    - 按時間過濾（第 404-405 行）

      ```python
      if time is not None:
          output_list = [c for c in list(filter(lambda output: output.split(LOG_SEP)[2] == time, output_list))]
      ```

      - 如果指定了 `time` 參數，進行精確時間匹配。

    - 步驟 7：按標題過濾（第 407-408 行）

      ```
      if title is not None:
          output_list = [c for c in list(filter(lambda output: title in output.split(LOG_SEP)[3], output_list))]
      ```

      - 如果指定了 `title`，確保標題**包含**指定文字。

    - 步驟 8：提取 Commit ID 並獲取完整資訊（第 410-411 行）

      ```
      result_commit_id = [output.split(LOG_SEP)[0] for output in output_list]
      return [self.get_commit_info(c_id) for c_id in result_commit_id]
      ```

      - 從搜尋結果中提取所有 commit hash
        對每個 commit hash 呼叫 get_commit_info() 獲取完整資訊
        包括完整的 diff
        包括完整的描述

  - 使用範例

    - 搜尋特定範圍

      ```
      # 搜尋 PR 分支上的新增提交
      commits = git_util.search_commits_by_info(
          author="Jane Smith",
          title="Fix CVE-2024",
          range="main..feature/security-patch"
      )
      ```

      













