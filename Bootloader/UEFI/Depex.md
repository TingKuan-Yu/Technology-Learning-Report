[TOC]



# Depex

`[Depex]` 是 **Dependency Expression（依賴表達式）** 的縮寫，是 UEFI EDK2 INF 檔中用來宣告這個 driver **在被 DXE Dispatcher 載入執行前，必須先滿足哪些條件**的區段。

在 [ColorbarsDxe.inf](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 中：

表示 DXE Dispatcher 必須等到 `gPcdProtocolGuid` **和** `gEfiPcdProtocolGuid` 這兩個 Protocol 都已經被其他 driver 安裝之後，才會載入並執行 ColorbarsDxe 的 entry point。

**運作原理**：

- DXE Dispatcher 負責排程所有 DXE driver 的載入順序
- 每個 driver 的 depex 會被編譯成一段二進位的依賴表達式（`.depex` 檔）
- Dispatcher 會持續掃描尚未載入的 driver，檢查其 depex 是否已滿足
- 一旦所有依賴的 Protocol/GUID 都存在於系統中，driver 就會被 dispatch

**額外注意**：除了 INF 檔中手動寫的 depex 外，**所使用的 library 也會自動注入額外的 depex**。如前面分析所示，`UefiDriverEntryPoint` library 會自動把 12 個 Architectural Protocol（包含 `gEfiBdsArchProtocolGuid` 等）加到最終的 depex 中，使得實際依賴比 INF 檔中寫的要多得多。