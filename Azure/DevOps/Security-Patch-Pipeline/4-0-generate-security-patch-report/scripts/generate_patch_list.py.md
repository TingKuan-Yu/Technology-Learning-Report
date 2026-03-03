# scripts/security_patch/generate_patch_list.py



## Overview

這是一個 **Android 安全性補丁管理工具**，用於自動生成補丁列表、檢查補丁應用狀態，並在 Azure DevOps (ADO) 上建立 Pull Request。

- **核心功能**
  - **補丁狀態檢查與分類**
    掃描安全性補丁目錄
    檢查每個補丁是否已應用到 PR 分支
    分類為「已應用」和「未應用」
  2. **自動建立 Pull Request**
  為已應用補丁的倉庫自動建立 PR
  設定標題、描述、審核者、工作項目
  支援草稿模式和自動完成
  3. **生成補丁清單 JSON**
  輸出完整的補丁資訊（路徑、連結、狀態、PR 連結等）



## 主要函數解析

- is_patch_on_pr_branch() - 檢查補丁是否在 PR 分支上

  ```python
  def is_patch_on_pr_branch(repo_path, patch_path, codebase_branch, pr_branch):
  ```

  - 解析補丁檔案，取得標題和作者
  - 找到 PR 分支與程式碼庫分支的共同祖先
  - 在 PR 分支上搜尋符合條件的提交
  - 返回 True/False

- generate_patch_list()

  ```python
  def generate_patch_list(security_patch_path, repo_path, generate_type, ado_token, ...)
  ```

  ```
  1. 初始化補丁列表容器
     ↓
  2. 載入排除的補丁 (excluded patches)
     - 加入到補丁列表，標記為 should_apply = false
     ↓
  3. 載入包含的補丁 (included patches)
     - 對每個倉庫遍歷所有 .patch 檔案
     ↓
  4. 檢查補丁是否已應用
     - 呼叫 is_patch_on_pr_branch()
     ↓
  5. 分類補丁
     - 已應用：暫存到 applied_patch_list
     - 未應用：直接加入 patch_info_list
     ↓
  6. 為已應用補丁的倉庫建立 PR
     - 呼叫 create_pr()
     - 將 PR 連結填入補丁資訊
     ↓
  7. 輸出補丁列表 JSON
  ```

  

## 檔案結構說明

- 輸入目錄結構

  ```
  security_patch_path/
  ├── included_patches/
  │   ├── repo_info.json           ← 包含的倉庫資訊
  │   ├── excluded_patch_info.json ← kernel 模式額外資訊
  │   └── <repo_name>/
  │       └── *.patch              ← 補丁檔案
  └── excluded_patches/
      ├── excluded_patch_info.json ← 排除的補丁資訊
      └── *.patch
  ```

- 輸出檔案

  ```
  security_patch_path/
  └── patch_list.json  ← 最終補丁清單
  ```



## 兩種模式差異

### - TYPE_KERNEL

- 使用補丁**連結**作為唯一識別 (patch_key = patch_link)
- 需要額外的 `excluded_patch_info.json` 來建立補丁連結對應

### - TYPE_FRAMEWORK

- 使用補丁**路徑**作為唯一識別 (patch_key = patch_path)
- 不需要額外的連結對應



## 命令列參數

```bash
python generate_patch_list.py \
  --security_patch_path /path/to/patches \    # 補丁目錄
  --repo_path /path/to/repos \                # 倉庫根目錄
  --generate_type kernel \                    # kernel 或 framework
  --ado_token <PAT_TOKEN> \                   # Azure DevOps Token
  --reviewers email1,email2 \                 # 審核者
  --work_item 12345,67890 \                   # 工作項目編號
  --pr_title "Security Patch 2024-12" \       # PR 標題（可選）
  --pr_description "Description" \            # PR 描述（可選）
  --auto_complete \                           # 自動完成（可選）
  --draft                                     # 草稿模式（可選）
```



## Function Overview

- is_patch_on_pr_branch()

  ```
  def is_patch_on_pr_branch(repo_path, patch_path, codebase_branch, pr_branch):
  ```

  - 個函數用於**檢查某個補丁是否已經應用到 PR 分支上**，通過比對 git 提交記錄來判斷。
    - repo_path - Git 倉庫路徑
    - patch_path - 補丁檔案路徑（.patch 檔案）
    - codebase_branch - 程式碼庫主分支（如 main）
    - pr_branch - PR 分支（如 security/2024-12/kernel）