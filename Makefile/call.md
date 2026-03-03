[TOC]



#　Makefile call

在傳統的 GNU Make（或 Android 建置系統中的 Make/Kati）中，`call` 是一個非常基本且強大的**內建函數 (Built-in Function)**。

它的主要用途是：**呼叫一個帶有參數的自定義函數**，並傳回該函數執行後的結果。這讓您可以將重複的邏輯封裝成一個可重複使用的程式碼塊（Macro）。

### `call` 函數的基本語法

`call` 函數的語法非常直接：

Makefile

```
$(call <函數名稱>, <參數1>, <參數2>, <參數3>...)
```

| **參數**     | **說明**                                                     |
| ------------ | ------------------------------------------------------------ |
| `<函數名稱>` | 在 Makefile 中定義的變數名稱。該變數的值就是整個函數的定義主體。 |
| `<參數N>`    | 傳遞給該函數的參數，從 `$1` 開始依序編號。                   |



### 步驟 1：定義一個帶參數的函數 (Macro)

您需要將函數邏輯定義為一個普通的 Make 變數。在函數體內，您使用特殊變數 `$1`, `$2`, `$3` 等來代表傳入的參數。

**範例：定義一個用於新增編譯標誌 (CFLAGS) 的函數**

Makefile

```makefile
# 定義函數：ADD_CFLAGS
# $1: 要新增的 CFLAGS 列表
# $2: 模組名稱
ADD_CFLAGS = $(info Adding CFLAGS "$1" to module $2) \
             -Wno-unused-variable -DMODULE_$(shell echo $2 | tr a-z A-Z)_SUPPORT
```

- **注意：** 函數定義中的**換行符**通常需要使用反斜線 `\` 進行跳脫，以便 Make 將整個定義視為一個單行指令。



### 步驟 2：使用 `call` 函數呼叫它

在您的規則（Rule）或變數賦值中使用 `call` 來執行這個函數。

Makefile

```makefile
TARGET_MODULE = MyDriver

# 呼叫 ADD_CFLAGS 函數
# 參數 $1 = "-O2 -g"
# 參數 $2 = "MyDriver"
# 執行結果會被賦值給 CFLAGS_RESULT
CFLAGS_RESULT := $(call ADD_CFLAGS, "-O2 -g", $(TARGET_MODULE))

# 打印結果
$(info Final CFLAGS: $(CFLAGS_RESULT))
```

**輸出結果 (當執行 Make 時):**

```
Adding CFLAGS "-O2 -g" to module MyDriver
Final CFLAGS: -Wno-unused-variable -DMODULE_MYDRIVER_SUPPORT
```



### 應用情境：在 Android Make 系統中的常見用法

在 Android 的 `BoardConfig.mk` 或 `Android.mk` 中，`call` 函數非常常用，主要用於呼叫 Build System 定義的**核心宏**。

#### 範例：呼叫 `soong_config_set`

正如我們之前討論的，`soong_config_set` 其實就是一個定義在 Build System 核心檔案中的 Make 函數：

Makefile

```makefile
# 呼叫 soong_config_set 函數，傳遞三個參數
$(call soong_config_set, vendor_nvidia, HasGPU, true)
```

#### 範例：呼叫標準模組定義宏

在舊版 Android 或特定 Vendor 的 `Android.mk` 中，您會看到呼叫宏來定義模組：

Makefile

```makefile
# 呼叫 BUILD_EXECUTABLE 宏，並將一堆屬性作為參數傳遞
# 這是舊版編譯系統定義可執行檔的方式
include $(call BUILD_EXECUTABLE)
```



### 總結



`call` 函數讓 Make 具備了程式語言中**函數 (Function)** 或**巨集 (Macro)** 的功能，大幅提高了 Makefiles 的**模組化、可讀性和重用性**。只要記住：

1. **定義**函數時使用 `$1`, `$2` 來接收參數。
2. **呼叫**函數時使用 `$(call FunctionName, param1, param2)`。