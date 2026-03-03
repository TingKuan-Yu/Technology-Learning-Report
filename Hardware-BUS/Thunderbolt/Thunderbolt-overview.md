[TOC]



# Thunderbolt (雷電) 技術全解析

Thunderbolt 是一種高速硬體介面標準，其核心優勢在於**「頻寬極高」**且**「協定整合」**。它不僅僅是一個連接埠，而是一個能同時承載 PCIe 資料、DisplayPort 影像與 USB 信號的傳輸通道。

### 1. 核心技術原理

Thunderbolt 的強大在於它使用了 **Daisy Chaining (菊鏈式連接)** 與 **Protocol Tunneling (協定隧道)** 技術。

- **協定隧道 (Tunneling)：** Thunderbolt 將 PCIe 和 DisplayPort 的封裝在同一個傳輸層中。這意味著你可以透過一個 Type-C 接口連接顯示卡 (eGPU)、高速 NVMe 磁碟陣列與 8K 螢幕。
- **整合電源：** 支援 USB Power Delivery (USB-PD)，允許單線供電、資料、影像同步完成。

### 2. 各代技術規格對比

| 規格              | 介面類型         | 最大頻寬 (單向)   | 主要特性                                                |
| ----------------- | ---------------- | ----------------- | ------------------------------------------------------- |
| **Thunderbolt 1** | Mini DisplayPort | 10 Gbps           | 首創資料與影像整合。                                    |
| **Thunderbolt 2** | Mini DisplayPort | 20 Gbps           | 支援 4K 影像傳輸。                                      |
| **Thunderbolt 3** | **USB Type-C**   | 40 Gbps           | 開始與 USB-C 融合，支援 eGPU。                          |
| **Thunderbolt 4** | **USB Type-C**   | 40 Gbps           | **保底規格提升：** 支援雙 4K 螢幕、提升 PCIe 最小頻寬。 |
| **Thunderbolt 5** | **USB Type-C**   | **80 - 120 Gbps** | 支援 Bandwidth Boost，鎖定 8K 與高更新率電競需求。      |

### 3. Thunderbolt 4 vs. USB4：有什麼不同？

這是最常被混淆的部分。簡單來說：**Thunderbolt 4 是一套嚴格的「認證標準」，而 USB4 則是底層的「技術協定」。**

- **認證門檻：** 所有的 Thunderbolt 4 設備必須通過 Intel 的嚴格測試，保證至少能輸出兩個 4K 畫面，且 PCIe 傳輸速率必須達到 32Gbps。
- **相容性：** Thunderbolt 4 設備一定相容 USB4，但 USB4 設備不一定具備 Thunderbolt 的完整性能。

### 4. 為什麼專業人士需要 Thunderbolt？

1. **外接顯示卡 (eGPU)：** 讓輕薄筆電透過 PCIe 通道連接桌面級顯卡，進行 3D 渲染或遊戲。
2. **極速儲存：** 搭配 NVMe SSD 外接盒，讀寫速度可破 3000MB/s，幾乎等同內置硬碟。
3. **單線作業 (Single-Cable Setup)：** 透過一個 Dock (擴充塢) 同時連接網路線、鍵盤、滑鼠、音響、雙螢幕並為筆電充電。
4. **音訊工程：** 極低延遲的資料傳輸對錄音介面 (Audio Interface) 而言至關重要。

### 5. 辨識方法

要辨識你的接口是否支援 Thunderbolt，最簡單的方法是看接口旁邊是否有**「閃電圖示」**：

- **⚡️ 圖示：** 代表支援 Thunderbolt。
- **⚡️ 旁邊有數字 4：** 代表是最新的 Thunderbolt 4。
- **僅有 SS 或 D 圖示：** 通常只是普通的 USB 或支援 DisplayPort 的 USB-C。

### 6. 未來展望：Thunderbolt 5

最新的 Thunderbolt 5 引入了 **PAM3 調變技術**，在特定情況下可以透過「非對稱傳輸」達到 **120 Gbps**。這對於追求極致性能的創作者（如 8K 影片剪輯）與高階電競玩家（360Hz+ 更新率）將是革命性的改變。