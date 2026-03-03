# js.wiki overview

[TOC]



# Reference

- https://js.wiki/



# Overview

Wiki.js 與 js.wiki 實際上指的是同一個產品：基於 Node.js、Markdown 和 Git 的現代化開源 Wiki 系統。js.wiki 是其官方網站域名，而 Wiki.js 是產品名稱。它具有高權限控制、現代化UI、支援 Markdown 和 Git 同步等特色。 

- **Wiki.js (產品/軟體名稱):** 指的是一個開源的、現代化的 Wiki 軟體，旨在簡化知識管理，通常用於企業內部文檔或個人知識庫。
- **js.wiki (官方網站):** 這是 Wiki.js 的官方域名，用戶可以在此下載、查看文檔和取得該軟體的資訊。 

**關鍵差異總結**
其實沒有技術或功能上的差異，兩者是「同一個東西的產品名與官方網址」的關係。

**Wiki.js 核心特色：**

- **技術棧:** 基於 Node.js 和 Vue.js 建構。
- **內容管理:** 支援 Markdown、HTML 或 WikiText，並能將內容存儲為 .MD 檔案。
- **Git 同步:** 支援與遠端 Git 儲存庫自動同步。
- **權限控制:** 具有複雜的權限組管理，支援 OAuth2、Google、GitHub 等第三方登入。
- **特色:** 現代化 UI，適用於技術文檔、團隊知識庫。 



# Database for Wiki.js

在 Ubuntu Desktop 上安裝 Wiki.js 時，雖然 Wiki.js 理論上支援幾種資料庫，但官方的建議非常明確。

以下是你在 Ubuntu 上可以選擇的資料庫選項：

## 1. 官方強烈建議：PostgreSQL (首選)

這是 Wiki.js **唯一官方正式推薦**且完整支援的資料庫。

- **優勢：** 效能最好、全文檢索功能最完整、未來升級最不容易出錯。

- **版本需求：** 建議使用 **PostgreSQL 12** 或更高版本。

- **Ubuntu 安裝指令：**

  Bash

  ```
  sudo apt update
  sudo apt install postgresql postgresql-contrib
  ```

## 2. 其他相容選項 (不建議用於正式環境)

Wiki.js 使用了 *Knex.js* 資料庫查詢建構器，因此技術上可以連接以下資料庫，但官方文件明確指出**許多進階功能（如搜尋、權限過濾）可能無法正常運作**：

- **MySQL / MariaDB：** 雖然很多人習慣用，但 Wiki.js 對其支援度正在降低，許多新功能是以 PostgreSQL 為基準開發的。
- **SQLite：** 適合極小規模、單人測試使用。它的好處是不用安裝伺服器軟體，直接存成一個檔案。
- **MS SQL Server：** 通常用於企業內部已有現成環境的情況。

------

## 3. 為什麼在 Ubuntu 上選 PostgreSQL？

如果你在 Ubuntu Desktop 上運行，PostgreSQL 是最穩定的選擇，原因如下：

| **功能**              | **PostgreSQL**                | **MySQL / Others**         |
| --------------------- | ----------------------------- | -------------------------- |
| **全文檢索 (Search)** | 內建支援，速度極快            | 支援有限，可能需要額外引擎 |
| **JSON 處理**         | 優異（Wiki.js 大量使用 JSON） | 較弱                       |
| **官方測試**          | 每一版開發都優先測試          | 僅作基本相容測試           |
| **安裝難度**          | 在 Ubuntu 上非常簡單          | 相同                       |

------

- - - 



# 方案 B：直接掛載到本地資料夾 (Sideload)

如果你不想用 Git，只想把電腦裡的資料夾直接「塞」進去：

1. **準備資料夾：** 把你的 `.md` 檔案通通放進剛才 Docker 掛載的 `./wiki-data` 資料夾中。
2. **設定 Wiki.js：**
   - 進入 **Administration** > **Storage**。
   - 選擇 **Local File System**。
   - 設定你的路徑（在 Docker 容器內通常是 `/wiki/storage`）。
3. **觸發匯入：**
   - 同樣點擊 **Import**。Wiki.js 會偵測到新檔案並將其索引化。



# 在第一次匯入後, 之後若想要再匯入, 但是改之前匯入的md file, 那要怎麼做.

如果你想徹底清除 Wiki.js 及其所有的資料庫內容，讓 Ubuntu 回到乾淨的狀態，請依照以下步驟執行。這會刪除 **Docker 容器**、**掛載的檔案** 以及 **實機上的 PostgreSQL 資料庫**。

------

## 第一步：刪除 Wiki.js Docker 容器

首先停止並移除運行中的 Wiki.js 容器：

Bash

```
# 停止並刪除名為 wiki 的容器
docker rm -f wiki

# 刪除 Wiki.js 的 Docker 鏡像 (選做，節省空間)
docker rmi requarks/wiki:2
```

## 第二步：刪除掛載的本地資料 (Markdown 檔案等)

刪除你之前用 `-v` 掛載在本地的資料夾：

Bash

```
# 假設你在當前目錄下的 wiki-data
rm -rf ./wiki-data
```

------

## 第三步：刪除 PostgreSQL 中的資料庫與帳號

由於資料庫是裝在 Ubuntu 實機上，我們需要進去手動刪除：

1. **進入 PostgreSQL 控制台：**

   Bash

   ```
   sudo -i -u postgres psql
   ```

2. **刪除資料庫：**

   SQL

   ```
   DROP DATABASE wikidb;
   ```

3. **刪除使用者：**

   SQL

   ```
   DROP USER wikiuser;
   ```

4. **退出：**

   SQL

   ```
   \q
   exit
   ```

------

## 第四步：(選做) 完全解除安裝 PostgreSQL

如果你以後完全不需要 PostgreSQL 了，可以將整個軟體移除：

Bash

```
# 移除軟體
sudo apt purge postgresql postgresql-contrib -y

# 刪除剩餘的設定檔與資料夾
sudo rm -rf /etc/postgresql/
sudo rm -rf /var/lib/postgresql/
```

------

## 總結清單

執行完上述步驟後，你的系統中將不會留下任何 Wiki.js 的痕跡：

- [x] **Docker 容器** 已清除
- [x] **Wiki 圖片與緩存檔案** 已刪除
- [x] **PostgreSQL 內容與帳號** 已刪除

> **⚠️ 警告：** 這些動作是**不可逆**的。在執行 `rm -rf` 或 `DROP DATABASE` 之前，請確認你已經備份了任何重要的 Markdown 檔案或筆記。

你是打算換成其他 Wiki 工具（如 BookStack 或 Obsidian），還是想重新嘗試用 Docker Compose 這種更整潔的方式來安裝呢？



# wiki.js刪除, 包含db

如果你想徹底清除 Wiki.js 及其所有的資料庫內容，讓 Ubuntu 回到乾淨的狀態，請依照以下步驟執行。這會刪除 **Docker 容器**、**掛載的檔案** 以及 **實機上的 PostgreSQL 資料庫**。

------

## 第一步：刪除 Wiki.js Docker 容器

首先停止並移除運行中的 Wiki.js 容器：

Bash

```
# 停止並刪除名為 wiki 的容器
docker rm -f wiki

# 刪除 Wiki.js 的 Docker 鏡像 (選做，節省空間)
docker rmi requarks/wiki:2
```

## 第二步：刪除掛載的本地資料 (Markdown 檔案等)

刪除你之前用 `-v` 掛載在本地的資料夾：

Bash

```
# 假設你在當前目錄下的 wiki-data
rm -rf ./wiki-data
```

------

## 第三步：刪除 PostgreSQL 中的資料庫與帳號

由於資料庫是裝在 Ubuntu 實機上，我們需要進去手動刪除：

1. **進入 PostgreSQL 控制台：**

   Bash

   ```
   sudo -i -u postgres psql
   ```

2. **刪除資料庫：**

   SQL

   ```
   DROP DATABASE wikidb;
   ```

3. **刪除使用者：**

   SQL

   ```
   DROP USER wikiuser;
   ```

4. **退出：**

   SQL

   ```
   \q
   exit
   ```

------

## 第四步：(選做) 完全解除安裝 PostgreSQL

如果你以後完全不需要 PostgreSQL 了，可以將整個軟體移除：

Bash

```
# 移除軟體
sudo apt purge postgresql postgresql-contrib -y

# 刪除剩餘的設定檔與資料夾
sudo rm -rf /etc/postgresql/
sudo rm -rf /var/lib/postgresql/
```

------

## 總結清單

執行完上述步驟後，你的系統中將不會留下任何 Wiki.js 的痕跡：

- [x] **Docker 容器** 已清除
- [x] **Wiki 圖片與緩存檔案** 已刪除
- [x] **PostgreSQL 內容與帳號** 已刪除

> **⚠️ 警告：** 這些動作是**不可逆**的。在執行 `rm -rf` 或 `DROP DATABASE` 之前，請確認你已經備份了任何重要的 Markdown 檔案或筆記。

你是打算換成其他 Wiki 工具（如 BookStack 或 Obsidian），還是想重新嘗試用 Docker Compose 這種更整潔的方式來安裝呢？
