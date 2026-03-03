# PRODUCT_APEX_SYSTEM_SERVER_JARS



## Android JARs

- Overview
  - 通常指的是包含 Android 框架（Framework）類別定義的 Java 歸檔文件（Java Archive）。

- 分類

  - `android.jar` (SDK Stub JAR)

    這是最常見的 Android JAR。當你在 Android Studio 中開發 App 時，專案會引用對應 API Level 的 `android.jar`（例如位於 `Android/Sdk/platforms/android-34/android.jar`）。

    - **它的作用**：它包含了 Android 公開 API 的 **存根（Stubs）**。也就是說，它裡面只有類別、方法名和參數的定義，而沒有具體的實作代碼（方法體內通常只有 `throw new RuntimeException("Stub!")`）。
    - **為什麼這樣設計**：
      - **編譯時**：讓你的編譯器知道有哪些 API 可以調用。
      - **運行時**：當 App 跑在手機上時，系統會**動態鏈接**到手機韌體（ROM）內真正的實作代碼。這確保了 App 體積不會過大，且能相容不同廠商的系統實作。

  - Framework JARs (內部/隱藏 API)

    對於系統開發者或 Vendor（如您提到的 Rockchip 開發環境）來說，常會提到「框架 JAR」。
  
    - **來源**：編譯 Android 原始碼（AOSP）後產出的檔案，如 `framework.jar`、`services.jar`、`telephony-common.jar`。
    - **特點**：
      - 包含了 **Hidden (@hide)** 和 **Internal** 的 API，這些是標準 SDK `android.jar` 中看不到的。
      - 如果您在開發系統應用（System App）或需要調用系統底層邏輯（如 USB 管理、電源管理），就需要引用這些完整的 JAR 檔。
  
  - Library JARs
  
    這指的是第三方或 Google 提供的 Java 程式庫。
  
    - 例如：早期版本的 `android-support-v4.jar` 或一些不包含資源（Resources）的純 Java 工具庫。
    - *注意：現在 Android 開發更傾向於使用 **.AAR** 文件，因為 AAR 可以包含 Android 資源檔（Layouts, Drawables），而 JAR 只能包含純代碼。*
  
  - GKI 與核心編譯中的 JAR
  
    在您之前的 GKI (Generic Kernel Image) 或內核編譯背景下，JAR 較少出現，但如果是涉及 Android 系統層的編譯：
  
    - **`core-oj.jar` / `core-libart.jar`**：這些是 Android Runtime (ART) 核心的 Java 類別庫，定義了 Java 基本語法物件（如 `java.lang.*`）。
  
  
  

## Overview

- This is a critical AOSP **build variable** used to define **which Java/Kotlin libraries (JARs) are loaded into the System Server process via APEX modules**