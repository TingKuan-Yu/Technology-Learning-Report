[TOC]



# soong_config_set

`soong_config_set` 是一個在 Android 建置系統 (Android Build System) 中，特別是在 **Make 檔案 (`.mk`)** 中使用的函數。它的主要目的是將一個變數從傳統的 Make 系統（Kati/Make）**傳遞**或**暴露**給現代的 **Soong 建置系統**（即處理 `Android.bp` 檔案的系統）。

這是實現 Make 和 Soong 兩種建置系統之間**溝通橋樑**的關鍵工具。



### `soong_config_set` 的基本語法



這個函數通常定義在 `build/make/core/soong_config.mk` 中，並以 Make 語法使用：

Makefile

```makefile
$(call soong_config_set, <Namespace>, <Variable_Name>, <Value>)
```

| **參數**          | **說明**                                                     |
| ----------------- | ------------------------------------------------------------ |
| `<Namespace>`     | 命名空間。這是您在 Soong (`Android.bp`) 中引用該變數時需要使用的前綴。通常是 OEM 或硬體名稱，例如 `vendor_nvidia`。 |
| `<Variable_Name>` | 變數名稱。您要傳遞給 Soong 的變數名稱。                      |
| `<Value>`         | 變數的值。可以是布林值 (`true`/`false`)、字串或數值。        |

------



### 使用情境與範例

假設您想根據您的硬體配置，在 Make 檔案中設定一個標誌（Flag），並讓 `Android.bp` 中的 C++ 庫根據這個標誌來編譯特定的源檔案或設定特定的編譯選項。

#### 步驟 1: 在 Make 檔案中設定變數 (`BoardConfig.mk` 或 `AndroidProducts.mk`)

您在 Make 檔案中定義變數，並使用 `soong_config_set` 將其傳遞給 Soong。

Makefile

```makefile
# 假設我們根據是否啟用高階音訊功能來決定編譯
ENABLE_ADVANCED_AUDIO := true

# 使用 soong_config_set 將變數傳遞給 Soong
# Namespace: vendor_mydevice
# Variable_Name: AdvancedAudio
# Value: true

$(call soong_config_set, vendor_mydevice, AdvancedAudio, $(ENABLE_ADVANCED_AUDIO))
```

#### 步驟 2: 在 `Android.bp` 中讀取變數 (`soong_config_module_type`)



在 `Android.bp` 檔案中，您需要使用 `soong_config_module_type` 相關的屬性來讀取這個變數。

程式碼片段

```
cc_library_shared {
    name: "lib_audio_driver",
    srcs: ["audio_base.cpp"],
    
    // 1. 設置配置命名空間
    soong_config_namespace: "vendor_mydevice",

    // 2. 根據變數的值設定條件編譯
    soong_config_variables: ["AdvancedAudio"],

    // 3. 根據變數 AdvancedAudio 的布林值來執行條件邏輯
    target: {
        android: {
            // 如果 AdvancedAudio 為 true，執行此區塊
            soong_config_bool: {
                AdvancedAudio: {
                    srcs: ["audio_advanced.cpp"], // 添加進階音訊的源檔案
                    cflags: ["-DADVANCED_AUDIO_SUPPORT"], // 添加宏定義
                },
            },
        },
    },
}
```



### 總結



`soong_config_set` 的主要功能就是：

- **橋接 (Bridging)：** 解決 Android Build System 從 Make 過渡到 Soong 期間的兼容性問題。
- **客製化 (Customization)：** 允許 OEM 和 BSP 開發者在 Make 檔案中定義特定的硬體配置，並將這些配置有效地應用到現代的 `Android.bp` 模組中。