# ingest_framework_security_patch.py



[TOC]



## Overview

### 程式功能

這支 Python 腳本負責從 Android Security Bulletin 套件中提取、分類與匯出 Framework 安全補丁（patches），並根據當前 codebase 的 manifest 篩選適用的儲存庫（repo）、生成後續流程所需的元資料。

### 主要流程

1. **複製與解壓**（[copy_security_patch](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)）

   - 掃描 [--security_patch_path](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)，找到唯一的 JSON 描述檔（[*-bulletin-*.json](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)）與 ZIP 封裝檔（`*[Bb]ulletin[-_]*.zip`）
   - 將 JSON 複製為 [SECURITY_PATCH_INFO_JSON](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)，ZIP 解壓到輸出目錄

2. **篩選作業系統版本**（[get_security_patch_for_os_ver](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)）

   - 依據 [--os_ver](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)（如 `12L`、`13`）用 glob 模式匹配 `bulletin_*/android-<os_ver>*/platform/*`
   - 將符合條件的補丁資料夾移至 `included_patches/`，建立 `excluded_patches/` 供後續存放排除項目
   - 刪除 bulletin 臨時資料夾

3. **映射補丁到專案**（[export_security_patch](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)）

   - 遍歷 `included_patches/` 下所有含 `.patch` 檔案的子資料夾

   - 以相對路徑從 [RepoManifest](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 物件（從 [MERGE_MANIFEST_NAME](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 載入）查找對應專案（[ManifestProject](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)）

   - 檢查專案是否屬於 [--target_ado_projects](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)（逗號分隔的 ADO project 清單）

   - 排除處理

     :

     - 若在 manifest 找不到專案 → 排除
     - **若專案不在目標 ADO project 清單 → 排除（並記錄警告）**
     - **排除的資料夾移至 `excluded_patches/` 並產生 [EXCLUDED_PATCH_INFO_JSON](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)**

   - **包含處理**: 收集 SSH URL、revision、路徑，產生 [INCLUDED_REPO_INFO_JSON](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

   - 若有未匯入的專案且 [--ignore_error](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 為 `false`，則中斷執行

4. **列出缺補丁 CVE**（[list_non_patch_cve](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)）

   **在2025-09-android-bulletin-partner-preview_v4.0.json內, 有"CVE": "CVE-2025-32332",, 但沒有patch, 如沒有patch_files` 欄位**

   - 讀取SECURITY_PATCH_INFO_JSON，篩選滿足以下條件的 CVE：

     - 缺少 `patch_level` 或 `aosp_versions` 或 `patch_files` 欄位
   - `patch_level` 結尾為 `-01`（framework patch）
     - 所屬 OS 版本包含當前 [--os_ver](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

   - 輸出 [CVE_missing_patch.json](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 供後續檢視

## 輸入參數

| 參數                                                         | 說明                                | 範例                                |
| ------------------------------------------------------------ | ----------------------------------- | ----------------------------------- |
| [--os_ver](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) | 目標 Android OS 版本                | `12L`, `13`                         |
| [--security_patch_path](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) | 下載的 security bulletin 套件路徑   | `$(System.DefaultWorkingDirectory)` |
| [--target_ado_projects](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) | 允許的 ADO project 清單（逗號分隔） | `mdep,device,ssi`                   |
| [--output](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) | 輸出目錄                            | `$(repoRoot)/$(artifactOutputPath)` |
| [--ignore_error](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) | 是否忽略專案匯入錯誤                | `true`/`false`                      |
| [--delete_existing_output](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) | 刪除既有輸出路徑（可選）            | 預設 `false`                        |

## 輸出檔案結構

## 錯誤處理

- 若找不到唯一 JSON/ZIP 檔案 → [ValueError](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
- 若沒有適合 OS 版本的補丁資料夾 → [ValueError](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
- 若有專案未匯入目標 ADO project 且 [ignore_error=false](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) → 記錄錯誤並 [exit(1)](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)
- 若 manifest 載入失敗 → [exit(1)](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)

## 與 Pipeline 整合

- 在 Azure Pipeline 的 [ingest_framework_security_patch.yaml:154-161](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 中被 `security_patch_script_launcher.sh` 呼叫
- 前置步驟：已下載 security patch 套件、已合併 manifest、已偵測 OS 版本
- 後續步驟：可選擇性生成 SPL 版本補丁、發布 artifact 至 feed



## Log Study -  error log related to ingest_framework_security_patch.py

- error log

  (1-0) ingest-framework-security-patch/[2025.1201.1](https://dev.azure.com/ampx/tooling/_build/results?buildId=1956295&view=logs&s=44e14aea-4d0b-5bc3-7109-b4f1cfe30b52)

  沒關係, 是預期的. 因叭能有一個-index.

  ```
  /mnt/vss/_work/1/s/codebase_util/scripts/security_patch/./ingest_framework_security_patch.py:32: SyntaxWarning: invalid escape sequence '\.'
  2025-12-02T09:16:04.0272522Z   cve_json_file = [ path for path in cve_json_file if not re.match(".*-index.*\.json", os.path.basename(path)) ]
  2025-12-02T09:16:04.0606916Z [2025-12-02 09:16:04][INFO] Start to ingest the framework patch
  ```

  - scripts/security_patch/ingest_framework_security_patch.py

    Download place **s/codebase_util**/scripts/security_patch/./ingest_framework_security_patch.py

    ```
    def copy_security_patch(security_patch_path, output):
        cve_json_file = list_files(security_patch_path, './*-bulletin-*.json')
        cve_json_file = [ path for path in cve_json_file if not re.match(".*-index.*\.json", os.path.basename(path)) ]
        
        
    if __name__ == '__main__':
    :
        copy_security_patch(args.security_patch_path, args.output)
    ```

    

## Log Study - Moving security patch folder

- error log

  (1-0) ingest-framework-security-patch/[2025.1201.1](https://dev.azure.com/ampx/tooling/_build/results?buildId=1956295&view=logs&s=44e14aea-4d0b-5bc3-7109-b4f1cfe30b52)

  ```
  2025-12-02T09:16:04.1495780Z [2025-12-02 09:16:04][INFO] Moving security patch folder: /mnt/vss/_work/1/s/codebase_util/artifact/2025-09-android-bulletin-partner-preview-patches/patches/android-13.0.0_r1/platform/art
  2025-12-02T09:16:04.1499708Z [2025-12-02 09:16:04][INFO] Moving security patch folder: /mnt/vss/_work/1/s/codebase_util/artifact/2025-09-android-bulletin-partner-preview-patches/patches/android-13.0.0_r1/platform/packages
  2025-12-02T09:16:04.1502476Z [2025-12-02 09:16:04][INFO] Moving security patch folder: /mnt/vss/_work/1/s/codebase_util/artifact/2025-09-android-bulletin-partner-preview-patches/patches/android-13.0.0_r1/platform/frameworks
  2025-12-02T09:16:04.1550553Z [2025-12-02 09:16:04][INFO] The folder does not contain patch file: /mnt/vss/_work/1/s/codebase_util/artifact/patch/
  ```

  - scripts/security_patch/ingest_framework_security_patch.py

    ```python
    
    def get_security_patch_for_os_ver(os_ver, output):
    :
        for folder in patch_folders:
            SimpleLogger.logger.info('Moving security patch folder: ' + folder)
            shutil.move(folder , patch_path)
    
    ```

    - in feed security-patch/mdep-framework-patch/2025.1201.1 folder

      ```
      $ tree patch -L 1
      patch
      ├── art
      ├── build
      ├── frameworks
      ├── packages
      └── repos.json
      
      4 directories, 1 file
      ```

      