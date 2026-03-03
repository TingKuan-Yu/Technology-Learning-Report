# generate_security_patch_by_manifest.py



## Overview

這是一個用於從 Android 安全標籤（security tags）生成安全補丁的 Python 腳本。

## **主要功能**

從兩個 Android 安全標籤之間的差異自動生成補丁文件，用於 Android 安全更新流程。

## **工作流程**

### **1. 輸入驗證**

- 接收兩個安全標籤（例如：`android-security-13.0.0_r10` 和 `android-security-13.0.0_r9`）
- 驗證標籤格式是否符合 `android-security-{major}.{minor}.{patch}_r{rev}` 格式
- 確認兩個標籤屬於相同的 Android 版本（major.minor.patch 必須相同）
- 確定哪個是較舊版本（previous）和較新版本（current）

### **2. 下載並解析 Manifest**

- 從 AOSP（Android Open Source Project）克隆 manifest 倉庫
- 切換到 current_tag 分支
- 解析 `default.xml` 獲取所有專案列表

### **3. 解析每個專案的 Git Hash**

- 對 manifest 中的每個專案（repo）：
  - 查詢兩個標籤對應的 Git commit hash
  - 記錄有差異的專案（hash 不同表示有更新）
  - 記錄無法解析的專案

### **4. 生成補丁文件**

- 對每個有差異的專案：
  - 克隆該專案的倉庫到本地
  - 使用 `git format-patch` 生成從 previous_hash 到 current_hash 的所有補丁
  - 將補丁保存到結構化目錄：`patches/android-{version}_r1/platform/{repo_path}/`

### **5. 特殊處理**

- ```
  handle_special_patches()
  ```

   

  處理特定倉庫的特殊情況

  - 例如：`platform/build` 會移除版本號升級相關的補丁（避免不必要的版本衝突）

### **6. 清理空目錄**

- `prune_dirs_without_files()` 遞迴刪除不包含任何文件的空目錄
- 保持輸出結構清晰

### **輸出結果**

```
output_dir/
├── results/
│   ├── patches/
│   │   └── android-13.0.0_r1/
│   │       └── platform/
│   │           ├── frameworks/base/
│   │           │   ├── 0001-xxx.patch
│   │           │   └── 0002-xxx.patch
│   │           └── system/core/
│   │               └── 0001-xxx.patch
│   └── logs/
└── repos/  (臨時克隆的倉庫)
```



## **關鍵變數設定**

在 Azure Pipeline 中設定環境變數：

```
print(f"##vso[task.setvariable variable=ANDROID_SECURITY_PATCH_DIR]{android_security_patch_dir}")
```

這讓後續的 pipeline 步驟知道補丁目錄名稱。

## **使用場景**

當 Google 發布新的 Android 安全更新時，這個腳本可以自動提取兩個安全標籤之間的所有變更，生成可應用的補丁文件，供設備製造商整合到自己的 Android 版本中。