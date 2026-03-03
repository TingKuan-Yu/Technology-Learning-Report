

# PostgreSQL的安裝與設定步驟

既然決定要在 Ubuntu 上使用 **PostgreSQL**，這裡幫你整理了最精簡、不繞路的安裝與設定步驟。

跟著這幾個指令走，大約 3 分鐘就能搞定資料庫部分：

------

### 第一步：安裝 PostgreSQL

在終端機輸入以下指令來安裝主程式：

Bash

```
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```

### 第二步：建立 Wiki.js 專用的資料庫與帳號

PostgreSQL 安裝後會自帶一個 `postgres` 系統用戶，我們切換過去執行管理指令：

1. **進入 PostgreSQL 控制台：**

   Bash

   ```
   sudo -i -u postgres psql
   ```

2. **建立資料庫（假設名稱叫 `wikidb`）：**

   SQL

   ```
   CREATE DATABASE wikidb;
   ```

3. **建立使用者並設定密碼（請把 `你的密碼` 換掉）：**

   SQL

   ```
   CREATE USER wikiuser WITH ENCRYPTED PASSWORD '你的密碼';
   ```

4. **賦予權限：**

   SQL

   ```
   GRANT ALL PRIVILEGES ON DATABASE wikidb TO wikiuser;
   ```

5. **退出：**

   SQL

   ```
   \q
   exit
   ```
