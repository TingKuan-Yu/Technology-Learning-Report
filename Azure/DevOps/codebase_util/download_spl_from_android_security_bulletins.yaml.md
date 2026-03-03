# download_spl_from_android_security_bulletins.yaml



## Pipeline

- [(0-0) download-spl-package](https://dev.azure.com/ampx/tooling/_build?definitionId=8106&_a=summary)

- [(0-1) download-spl-from-maniests](https://dev.azure.com/ampx/tooling/_build?definitionId=9587&_a=summary)



## Feeds

- [security-patch](https://dev.azure.com/ampx/mdep/_artifacts/feed/security-patch)
  - spl-from-android-security-bulletins
    - [2025.1028.1](https://dev.azure.com/ampx/mdep/_artifacts/feed/security-patch/UPack/spl-from-android-security-bulletins/overview/2025.1028.1)
      - [(0-0) download-spl-package](https://dev.azure.com/ampx/415ef6b4-62b1-4640-87ac-20f7c497e53f/_build?definitionId=8106&_a=summary)/[2025.1028.1](https://dev.azure.com/ampx/415ef6b4-62b1-4640-87ac-20f7c497e53f/_build/results?buildId=1798505)



## security-patch/spl-from-android-security-bulletins/[2025.1028.1](https://dev.azure.com/ampx/mdep/_artifacts/feed/security-patch/UPack/spl-from-android-security-bulletins/overview/2025.1028.1)

- (0-0) download-spl-package/[2025.1028.1](https://dev.azure.com/ampx/tooling/_build/results?buildId=1798505&view=results)





## Explain download_spl_from_android_security_bulletins.yaml

### 程式說明

這是一個 **Azure DevOps Pipeline**，用於從 Android 安全公告網站下載並打包安全補丁（SPL - Security Patch Level）。

### **主要功能**

自動化從 Google Android 安全公告網站爬取、解析和打包安全補丁的流程。

### **輸入參數**

- **`securitypatchdate`**: 安全補丁日期，格式為 `YYYY-MM`（例如：`2025-11`）

### **工作流程**

#### **步驟 1: 解析安全公告網站**

```
android_security_bulletin_parser.py --security_patch_date "2025-11-01"
```

- 爬取 Android 安全公告網站（`https://source.android.com/security/bulletin`）
  - **會比由google drive內下載的晚一個月.** 
- 解析指定月份的安全漏洞資訊
- **生成** JSON 文件：`security_patch_2025-11-01.json`
- 內容包含：CVE 編號、受影響的倉庫、commit hash 等

#### **步驟 2: 生成補丁文件**

```
generate_android_security_patch.py --security_patch_json "security_patch_2025-11-01.json"
```

- 讀取 JSON 文件中的 commit 資訊
- 從 AOSP Git 倉庫下載對應的補丁
  - https://android.googlesource.com/
    - Note: Partner preview版由https://partner-android.googlesource.com下載.
- 生成 `patches/` 目錄，包含所有安全修復的 `.patch` 文件

#### **步驟 3: 建立補丁索引**

```
generate_android_patch_index.py --security_patch_json "security_patch_2025-11-01.json"
```

- 生成 `android_patch_path.json`
- 記錄每個補丁的路徑和元數據，便於後續管理和應用

#### **步驟 4: 生成公告 JSON**

```
generate_android_bulletin_json.py --patch_path_file android_patch_path.json --security_patch_file "security_patch_2025-11-01.json"
```

- 整合補丁路徑和安全資訊
- 生成完整的安全公告 JSON 文件

#### **步驟 5: 打包和壓縮**

```
mkdir "2025-11-android-bulletin-partner-preview-patches"
mv patches "2025-11-android-bulletin-partner-preview-patches"
zip -r "2025-11-android-bulletin-partner-preview-patches.zip" ...
```

- 將所有補丁文件打包成 ZIP
- 包含所有 JSON 元數據文件

#### **步驟 6: 發布到 Azure Artifacts**

```
- task: UniversalPackages@0
  vstsFeedPackagePublish: "spl-from-android-security-bulletins"
```

- 將打包好的補丁發布到 Azure Artifacts Feed
- Package 名稱：`spl-from-android-security-bulletins`
- 版本號：使用 Build.BuildNumber（例如：`2025.1201.1`）

### **輸出產物**

```
2025-11-android-bulletin-partner-preview-patches.zip
├── patches/
│   ├── android-13.0.0_r1/
│   │   └── platform/
│   │       ├── frameworks/base/
│   │       └── system/core/
│   └── android-12.0.0_r1/
│       └── platform/
│           └── ...
├── security_patch_2025-11-01.json
├── android_patch_path.json
└── android_bulletin.json
```

### **與其他 Pipeline 的關係**

此 pipeline 產出的 package (`spl-from-android-security-bulletins`) 會被 `download_spl_from_android_manifest.yaml` 下載並使用，用於與從 manifest 生成的補丁進行比較和整合。

### **使用場景**

每月 Google 發布新的 Android 安全公告時，運行此 pipeline 自動下載和打包該月份的所有安全補丁，供設備製造商使用。



