[TOC]



# std::string_view 怎麼用

在 C++17 之後，`std::string_view` 被稱為「現代 C++ 處理字串的最佳實踐」。它本質上是一個 **「唯讀的視窗」**，指向一個已經存在的字串序列（字元陣列或 `std::string`）。

它最大的價值在於：**避免不必要的記憶體拷貝（Copy）**。

---

### 1. 核心結構

`std::string_view` 內部非常輕量，只存儲了兩個東西：

1. 一個指向字串開頭的**指標** (`const char*`)。
2. 字串的**長度** (`size_t`)。

因為它不擁有（Own）記憶體，所以它不會觸發 `malloc` 或 `new`。

---

### 2. 基本用法

```cpp
#include <string_view>
#include <string>
#include <iostream>

void printName(std::string_view sv) {
    // sv 就像 string 一樣可以用
    std::cout << "Length: " << sv.length() << std::endl;
    std::cout << "Content: " << sv << std::endl;
}

int main() {
    // 1. 從 C 形式字串建立 (直接指向字串常數，無拷貝)
    printName("USB Camera"); 

    // 2. 從 std::string 建立 (無拷貝)
    std::string s = "Qualcomm 8550";
    printName(s);

    // 3. 切片功能 (非常強大，且完全無拷貝)
    std::string_view sub = s;
    sub.remove_prefix(9); // 變成 "8550"，只是移動指標位置而已
    return 0;
}

```

---

### 3. 為什麼要用它？（效能對比）

如果你寫一個函式 `void process(std::string s)`：

* 當你傳入一個大字串時，系統會**複製整個字串**。
* 如果改用 `const std::string& s`，雖然避開了拷貝，但如果你傳入的是 `"literal string"`，編譯器仍會建立一個臨時的 `std::string` 物件。

使用 `std::string_view`：

* 不論傳入什麼，都只會複製指標和長度（通常是 16 bytes），效能極高。

---

### 4. 關鍵限制與危險（重要！）

因為 `string_view` 是「借用」別人的記憶體，所以在使用時要遵守 **「活得比主體短」** 的原則。

#### **危險 A：懸空指標 (Dangling Pointer)**

```cpp
std::string_view getBrokenView() {
    std::string s = "Temporary";
    return std::string_view(s); // 錯誤！s 會在此函式結束後銷毀
} // 回傳的 view 會指向無效記憶體，導致崩潰

```

#### **危險 B：不保證以 `\0` 結尾**

傳統 C 函式（如 `printf` 或 `strlen`）依賴字串結尾的 `\0`。但 `string_view` 是靠長度來運作的，它指向的記憶體中間可能沒有 `\0`。

* **正確做法**：如果你要傳給底層 C API（如 `open()` 或 `ioctl()`），請使用 `std::string` 或確定 `sv.data()` 是以 null 結尾。

---

### 5. 在 Android 底層開發的應用場景

在你處理 **Qualcomm 8550 的 UVC 專案**中，`string_view` 非常適合以下情境：

1. **解析 XML/JSON 標籤**：
當你從大塊 Buffer 中尋找 `<name>Camera</name>` 時，你可以用 `string_view` 指向中間的 `Camera`，而不需要把這 6 個字元另外存成一個 `std::string`。
2. **Log 處理**：
就像你之前寫的 `escapeJsonString(std::string_view in)`。這讓你的轉義函式可以接受任何來源的字串（常數、全域變數、Local string）而不產生額外的負載。

---

### 💡 總結：什麼時候用？

* **函式的輸入參數**：如果該參數是「唯讀」的，請優先使用 `std::string_view` 取代 `const std::string&`。
* **字串切片/過濾**：當你需要頻繁取子字串時，它是效能神器。
* **不要用來當成員變數**：除非你能保證主體字串的生命週期比類別更長，否則很容易造成 Dangling 指標。

**這能解釋為什麼你之前看到的 `escapeJsonString` 範例要選用 `string_view` 嗎？**



# 在 `std::string_view` 中，`rfind` 的用法

在 `std::string_view` 中，`rfind` 的用法與 `std::string` 幾乎完全一樣。它的全名是 **Reverse Find**，顧名思義是「從右往左（從後往前）」搜尋指定的子字串或字元。

在 Android 底層開發（如解析檔案路徑 `/dev/video4`）時，這是一個非常有效率的工具。

---

### 1. 基本語法

```cpp
size_t rfind(string_view s, size_t pos = npos) const noexcept;
size_t rfind(char c, size_t pos = npos) const noexcept;

```

* **搜尋目標**：可以是另一個 `string_view`、C 字串或單一字元。
* **pos**：搜尋的起點（從這個位置開始往左找）。預設是 `npos`，代表從最後一個字元開始找。
* **回傳值**：找到的起始索引（index）。如果找不到，回傳 `std::string_view::npos`。

---

### 2. 實戰範例：解析 UVC 裝置路徑

假設你拿到一個路徑字串，你想找出最後一個斜線後的名稱：

```cpp
#include <string_view>
#include <iostream>

void parsePath(std::string_view path) {
    // 找出最後一個 '/' 的位置
    size_t last_slash = path.rfind('/');

    if (last_slash != std::string_view::npos) {
        // 使用 substr 切出名稱 (從 last_slash + 1 開始到結尾)
        std::string_view fileName = path.substr(last_slash + 1);
        std::cout << "Device Node: " << fileName << std::endl;
    }
}

int main() {
    parsePath("/dev/video2"); // 輸出: video2
    parsePath("video4");      // 找不到 /，不執行輸出
    return 0;
}

```

---

### 3. 指定起始位置 (`pos`) 的進階用法

如果你想找「倒數第二個」某種字元，可以利用 `pos` 參數：

```cpp
std::string_view path = "/sys/class/video4linux/video4";

// 1. 先找最後一個 '/'
size_t last = path.rfind('/'); // 指向 video4 前面的 '/'

// 2. 從 last - 1 的位置開始往左找倒數第二個 '/'
size_t second_last = path.rfind('/', last - 1); // 指向 video4linux 前面的 '/'

```

---

### 4. 為什麼在 `string_view` 用 `rfind` 效能更好？

當你對 `std::string` 呼叫 `rfind` 時，如果你傳入的是一個 C 字串常數（例如 `path.rfind("/dev/")`），系統**可能**會建立一個臨時的 `std::string` 物件。

但在 `std::string_view` 中：

1. **完全無拷貝**：所有操作都在原始記憶體位址上進行。
2. **Noexcept**：`string_view` 的 `rfind` 被標記為 `noexcept`，這對編譯器優化非常有幫助。

---

### 5. 注意事項：`npos` 的檢查

這是在使用任何 `find` 家族函式時最容易出 Bug 的地方。務必檢查結果是否為 `npos`。

```cpp
size_t pos = sv.rfind(':');
if (pos == std::string_view::npos) {
    // 沒找到，直接取值 sv[pos] 會導致崩潰
}

```

---

### 💡 總結建議

在你目前處理 **Qualcomm 8550** 的 USB 偵測邏輯中，如果你需要從 `dmesg` 的輸出或是 `/sys/bus/usb/devices/` 的長字串中提取特定的 ID（例如提取 `1-1.1:1.0` 中的最後一碼），使用 `rfind` 搭配 `substr` 是最乾淨且效能最高的寫法。

**需要我幫你寫一段「從完整路徑中自動提取 video 序號」的完整 `string_view` 工具函式嗎？**