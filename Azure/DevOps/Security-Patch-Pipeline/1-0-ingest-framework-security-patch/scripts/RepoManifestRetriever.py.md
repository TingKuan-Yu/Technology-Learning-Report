# RepoManifestRetriever.py

   

## Overview

- 目的: 以 Google `repo` 工具從指定 manifest 倉庫與分支產出「合併後的 manifest」檔案，並放到指定輸出位置。
- 主要流程:
  - 解析參數並檢查必填：`--working_path`、`--manifest_url`、`--manifest_branch`。
  - 建立或清空工作目錄（`--delete_existing_working_path` 為真時刪除既有路徑）。
  - 正規化 URL：移除形如 `xxx@dev.azure.com` 的前置字串。
  - 初始化 repo：執行 `repo init -u <url> -b <branch> [-m <manifest_name>]`。
  - 合**併匯出：執行 `repo manifest -o <merged_name>` 產生合併後的 manifest。**
  - 依需要移動輸出檔至 `--merged_manifest_path`，最後清理 `.repo` 目錄。
- 輸入/輸出:
  - 輸入參數：`--working_path`、`--manifest_url`、`--manifest_branch`、`--manifest_name`、`--merged_manifest_name`、`--merged_manifest_path`、`--delete_existing_working_path`、`--token`。
  - **輸出檔案**：預設檔名為 `constant.MERGE_MANIFEST_NAME`（若未覆寫 `--merged_manifest_name`），輸出於 **`--working_path`** 或指定的 `--merged_manifest_path`。
- 日誌與錯誤處理: 使用 `SimpleLogger` 記錄流程；任何步驟失敗會拋出 `ValueError` 並停止。
- 其他注意:
  - PAT 從 `--token` 或環境變數 `AZURE_DEVOPS_EXT_PAT` 取得，但目前未直接注入 `repo` 指令。
  - 在初始化時對 `manifest_url` 進行 `lower()`，若路徑大小寫敏感，可能造成潛在問題。



## Usage

- scripts/security_patch/yaml/ingest_framework_security_patch.yaml

  ```yaml
      stages:
      - stage: ingestFrameworkSecurityPatch
        displayName: "Ingest Framework Security Patch"
        :
            - script: |
                bash $(repoRoot)/$(securityPatchPipelineRootDir)/security_patch_script_launcher.sh "./util/RepoManifestRetriever.py" \
                    --working_path "$(repoRoot)/$(artifactOutputPath)" \
                    --manifest_url "${{ parameters.manifestUrl }}" \
                    --manifest_branch "${{ parameters.manifestBrnach }}" \
                    --manifest_name "${{ parameters.manifestName }}" \
                    --token "$AZURE_DEVOPS_EXT_PAT" \
                    --delete_existing_working_path
              displayName: "Download and Merge the Manifest"
  ```

  



## Logs - Download and Merge the Manifest

- log

  https://dev.azure.com/ampx/415ef6b4-62b1-4640-87ac-20f7c497e53f/_apis/build/builds/1956295/logs/95

  ```
  025-12-02T09:15:59.0810545Z ##[section]Starting: Download and Merge the Manifest
  2025-12-02T09:15:59.0815825Z ==============================================================================
  2025-12-02T09:15:59.0815932Z Task         : Command line
  2025-12-02T09:15:59.0815996Z Description  : Run a command line script using Bash on Linux and macOS and cmd.exe on Windows
  2025-12-02T09:15:59.0816096Z Version      : 2.250.1
  2025-12-02T09:15:59.0816159Z Author       : Microsoft Corporation
  2025-12-02T09:15:59.0816234Z Help         : https://docs.microsoft.com/azure/devops/pipelines/tasks/utility/command-line
  2025-12-02T09:15:59.0816326Z ==============================================================================
  2025-12-02T09:15:59.2257879Z Generating script.
  2025-12-02T09:15:59.2268366Z ========================== Starting Command Output ===========================
  2025-12-02T09:15:59.2295783Z [command]/usr/bin/bash --noprofile --norc /mnt/vss/_work/_temp/83340b8f-f9a8-4df3-b61d-048dc33cf46e.sh
  2025-12-02T09:15:59.7127642Z [2025-12-02 09:15:59][INFO] Start to retrieve the merged manifest
  2025-12-02T09:15:59.7132062Z [2025-12-02 09:15:59][INFO] Init repo with command: repo init -u https://dev.azure.com/ampx/mdep/_git/manifest.mdep -b mdep/13/1303.1/main
  2025-12-02T09:16:02.1958618Z [2025-12-02 09:16:02][INFO] Merge manifest with command: repo manifest -o merge.xml
  2025-12-02T09:16:02.7534453Z 
  2025-12-02T09:16:02.7596329Z ##[section]Finishing: Download and Merge the Manifest
  ```

  - -o merge.xml