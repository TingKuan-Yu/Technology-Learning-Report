# 帳號管理

注意database是在進入wikijs第一次選的

- PostgreSQL
  - Ubuntu root password
  - CREATE USER wikiuser WITH ENCRYPTED PASSWORD '你的密碼';
    - GRANT ALL PRIVILEGES ON DATABASE wikidb TO wikiuser;
  - 當你啟動 Wiki.js 並進入瀏覽器設定頁面（通常是 `http://localhost:3000`）時，填入以下資訊：

    - **Database Type:** PostgreSQL
- Wiki.js Admin
  - Account & Password
- Wiki.js user
  - Account & Password



## 快速安裝建議 

如果你是在自己的 Ubuntu Desktop 測試，最快的方法是使用 **Docker Compose**。這會幫你自動把 Wiki.js 和 PostgreSQL 一次打包跑起來，省去配置資料庫權限的麻煩：

YAML

```
# docker-compose.yml 片段範例
services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: wiki
      POSTGRES_USER: wikijs
      POSTGRES_PASSWORD: your_password
    volumes:
      - ./db-data:/var/lib/postgresql/data
  wiki:
    image: requarks/wiki:2
    depends_on:
      - db
    ports:
      - "80:3000"
```

------

> **我的建議：** 既然你都在 Ubuntu 上跑了，除非你有非常特殊的理由（例如公司強迫只能用 MySQL），否則**請直接安裝 PostgreSQL**。這能避免你在設定 Wiki.js 搜尋功能時遇到莫名的錯誤。

需要我提供在 Ubuntu 上手動安裝 PostgreSQL 並設定給 Wiki.js 使用的詳細指令嗎？



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

### 第三步：最重要的一步：調整 PostgreSQL 設定

預設情況下，PostgreSQL 可能不允許 Docker 容器連線。請檢查以下兩點：

1. **修改 `postgresql.conf`：** 確認 `listen_addresses` 有開放。

   Bash

   ```
   sudo nano /etc/postgresql/14/main/postgresql.conf 
   # (版本號 14 可能隨你安裝的有所不同)
   ```

   找這一行：`listen_addresses = 'localhost'` 改成 `listen_addresses = '*'`。

2. **修改 `pg_hba.conf`：** 允許連線。

   Bash

   ```
   sudo nano /etc/postgresql/14/main/pg_hba.conf
   ```

   - 允許本地連線 - 在檔案末尾添加：`host all all 0.0.0.0/0 md5`。
     - 目前沒設

   - 目前default

     ```
     # "local" is for Unix domain socket connections only
     local   all             all                                     peer
     # IPv4 local connections:
     host    all             all             127.0.0.1/32            scram-sha-256
     # IPv6 local connections:
     host    all             all             ::1/128                 scram-sha-256
     ```

   - **wiki.js是裝在docker內** -> 加wiki.js Docker 虛擬網段（172.17.0.1）

     host    all             all             172.17.0.0/16           md5

3. **重啟資料庫：**

   Bash

   ```bash
   sudo systemctl restart postgresql
   ```





# Run wiki.js

- use docker

  你需要加入資料庫的連線參數，並使用 `--network="host"` 讓容器能存取實機的資料庫：

  ```bash
  docker run -d \
    -p 3000:3000 \
    --name wiki \
    --network="host" \
    -e "DB_TYPE=postgres" \
    -e "DB_HOST=127.0.0.1" \
    -e "DB_PORT=5432" \
    -e "DB_USER=wikiuser" \
    -e "DB_PASS=你的密碼" \
    -e "DB_NAME=wikidb" \
    -v ./wiki-data:/wiki \
    requarks/wiki:2
     
  docker run -d \
    -p 3000:3000 \
    --name wiki \
    -e "DB_TYPE=postgres" \
    -e "DB_HOST=172.17.0.1" \
    -e "DB_PORT=5432" \
    -e "DB_USER=wikiuser" \
    -e "DB_PASS=Happy-Tony01" \
    -e "DB_NAME=wikidb" \
    -v /home/tony/ws/Kubernates-Study/Wiki.js-poc/my-git-wiki:/wiki/storage \
    requarks/wiki:2
    
  
  #   -e "DB_HOST=127.0.0.1" \ -> -e "DB_HOST=172.17.0.1" \
  
  docker run -d \
    -p 3000:3000 \
    --name wiki \
    --network="host" \
    # 原有数据库配置（保留）
    -e "DB_TYPE=postgres" \
    -e "DB_HOST=172.17.0.1" \
    -e "DB_PORT=5432" \
    -e "DB_USER=wikiuser" \
    -e "DB_PASS=你的密碼" \
    -e "DB_NAME=wikidb" \
    # 新增 Git 仓库配置（核心）
    -e "GIT_REPOSITORY_URL=https://github.com/你的用户名/你的wiki仓库.git" \
    -e "GIT_BRANCH=main" \
    -e "GIT_USER_NAME=你的GitHub用户名" \
    -e "GIT_USER_EMAIL=你的GitHub邮箱" \
    -e "GIT_AUTH_TYPE=token" \
    -e "GIT_AUTH_TOKEN=你的GitHub个人访问令牌" \
    # 数据卷（保留）
    -v ./wiki-data:/wiki \
    requarks/wiki:2  
    
    
  docker run -d \
    -p 3000:3000 \
    --name wiki \
    --network="host" \
    # 原有数据库配置（保留）
    -e "DB_TYPE=postgres" \
    -e "DB_HOST=172.17.0.1" \
    -e "DB_PORT=5432" \
    -e "DB_USER=wikiuser" \
    -e "DB_PASS=你的密碼" \
    -e "DB_NAME=wikidb" \
    # 新增 Git 仓库配置（核心）
    -e "GIT_REPOSITORY_URL=https://github.com/你的用户名/你的wiki仓库.git" \
    -e "GIT_BRANCH=main" \
    -e "GIT_USER_NAME=你的GitHub用户名" \
    -e "GIT_USER_EMAIL=你的GitHub邮箱" \
    -e "GIT_AUTH_TYPE=token" \
    -e "GIT_AUTH_TOKEN=你的GitHub个人访问令牌" \
    # 数据卷（保留）
    -v ./wiki-data:/wiki \
    requarks/wiki:2    
    
    
  docker run -d \
    -p 3000:3000 \
    --name wiki \
    -e "DB_TYPE=postgres" \
    -e "DB_HOST=172.17.0.1" \
    -e "DB_PORT=5432" \
    -e "DB_USER=wikiuser" \
    -e "DB_PASS=Happy-Tony01" \
    -e "DB_NAME=wikidb" \
    -e "GIT_REPOSITORY_URL=https://github.com/TingKuan-Yu/wiki.js-wiki-pages.git" \
    -e "GIT_BRANCH=main" \
    -e "GIT_USER_NAME=TingKuan-Yu" \
    -e "GIT_USER_EMAIL=tonyyui@hotmail.com" \
    -e "GIT_AUTH_TYPE=token" \
    -e "GIT_AUTH_TOKEN= " \
    -v /home/tony/ws/github/dev-helper-project/wiki-data:/wiki-data \
    requarks/wiki:2      
  ```
  
  - `DB_HOST` 配置 `127.0.0.1` 和 `172.17.0.1` 的区别
    - 当 PostgreSQL 安装在 **Ubuntu 本地**（非 Docker）：**Wiki.js 容器内**必须用 172.17.0.1（或**宿主机内网 IP**），用 127.0.0.1 会连接失败；

    - **注意：如果用了 --network="host"，则 DB_HOST 用 127.0.0.1**
  
    - 当 PostgreSQL 运行在另一个 Docker 容器：需用该容器的 IP / 容器名，而非上述两个地址；
  
    - 只有 Wiki.js 和 PostgreSQL 运行在同一个容器时，127.0.0.1 才有效（极少用）。
    
    - 为什么两者有区别？
      Docker 容器有自己的「独立网络空间」，你可以把每个容器理解成一台独立的虚拟机：
      127.0.0.1：在 Wiki.js 容器内，这个地址指向容器自己，而非宿主机（你的 Ubuntu 系统）；
      172.17.0.1：这是 Docker 默认桥接网络（bridge）给宿主机分配的固定内网 IP，容器通过这个地址能访问到宿主机上的服务（如本地的 PostgreSQL）。
  
- ### 4. 驗證是否成功

  啟動容器後，打開瀏覽器輸入： http://localhost:3000

  你應該會看到 Wiki.js 的初始化頁面，要求你設定管理員信箱與密碼。





### 第三步：在 Wiki.js 設定畫面輸入資訊

當你啟動 Wiki.js 並進入瀏覽器設定頁面（通常是 `http://localhost:3000`）時，填入以下資訊：

- **Database Type:** PostgreSQL
- **Host:** `localhost` (如果你是在同一台電腦跑)
- **Port:** `5432` (預設值)
- **User:** `wikiuser`
- **Password:** `你的密碼`
- **Database Name:** `wikidb`

------

### 小提醒

- **防火牆：** 如果你的 Wiki.js 跟資料庫在**不同台**電腦，記得要在 Ubuntu 上開啟 5432 埠口：`sudo ufw allow 5432/tcp`。
- **備份：** 之後如果要備份，只需要執行 `pg_dump wikidb > backup.sql` 就能把整台 Wiki 的內容（除了圖片檔案）帶走。

**資料庫準備好後，你接下來是要用 Docker 跑 Wiki.js，還是直接用 Node.js 跑呢？**

# Issue fixing

- docker: Error response from daemon: Conflict. The container name "/wiki" is already in use by container "6320ebf06ba526f4b75fcb4563b1195adc58d792639ec25694d50c55bcfb54f4". You have to remove (or rename) that container to be able to reuse that name.

  - 這是一個非常經典的 Docker 錯誤。這代表你剛才執行的指令雖然報錯了（或是沒跑起來），但 Docker 已經「佔用」了 `/wiki` 這個容器名稱。

    在 Docker 中，容器名稱是唯一的。即使容器是「停止」狀態（Exited），你也無法用同一個名稱啟動新的容器。

    ### 解決方法：三部曲

    請依序執行以下指令：

    #### 1. 刪除舊的容器

    先強制刪除那個佔位但不工作的舊容器：

    Bash

    ```
    docker rm -f wiki
    ```

    *(註：`-f` 會強制停止並刪除，即使它還在背景跑。)*

    #### 2. (選做) 清理掛載資料夾

    如果你之前的指令建立了一個空資料夾 `./wiki-data`，建議先確認它是否存在，或是把它清空，讓 Wiki.js 重新初始化：

    Bash

    ```
    # 確保資料夾存在且權限正確
    mkdir -p ./wiki-data
    ```

    #### 3. 重新執行啟動指令 (建議方案)

    為了避開之前的 `Published ports are discarded` 警告，同時確保能連到你實機上的資料庫，我們這次改用 **方案 B**（Bridge 模式），這是最標準的做法：

    Bash

    ```
    docker run -d \
      -p 3000:3000 \
      --name wiki \
      -e "DB_TYPE=postgres" \
      -e "DB_HOST=172.17.0.1" \
      -e "DB_PORT=5432" \
      -e "DB_USER=wikiuser" \
      -e "DB_PASS=你的密碼" \
      -e "DB_NAME=wikidb" \
      -v $(pwd)/wiki-data:/wiki \
      requarks/wiki:2
    ```

    ------

    ### 常見的「卡關」檢查點

    如果執行完後 `http://localhost:3000` 還是打不開，請檢查：

    1. **查看 Log 報錯：**

       執行 `docker logs wiki`。

       - 如果看到 `Connection refused`：代表 Docker 找不到你的資料庫。
       - 如果看到 `Ident authentication failed`：代表 PostgreSQL 不認識這個 Docker 連線。

    2. **Postgres 的白名單 (最重要！)：**

       請務必確認你的 `/etc/postgresql/14/main/pg_hba.conf` 檔案最後一行有加上：

       `host all all 0.0.0.0/0 md5`

       並且記得重啟 Postgres：`sudo systemctl restart postgresql`。

    **現在請試著執行 `docker rm -f wiki` 後再跑一次指令。如果有噴出任何 Log 錯誤訊息，直接貼給我看，我幫你診斷！**

- error: Database Connection Error: ECONNREFUSED 172.17.0.1:5432

  - 看到 `ECONNREFUSED 172.17.0.1:5432`，這表示 Wiki.js 已經成功敲了你 Ubuntu 實機的大門，但 **PostgreSQL 把門鎖上了**，拒絕讓它進去。

    這是因為 PostgreSQL 預設只聽 `localhost`（127.0.0.1）的聲音，不理會來自 Docker 虛擬網段（172.17.0.1）的請求。

    我們來進行最後的「破門三部曲」：

    ------

    ### 1. 讓 PostgreSQL 監聽所有連線

    預設它只聽實機自己的請求，我們要讓它聽見 Docker 的聲音。

    1. 開啟設定檔（路徑中的 `14` 或 `15` 請根據你安裝的版本調整）：

       Bash

       ```
       sudo nano /etc/postgresql/14/main/postgresql.conf
       ```

    2. 找到 `#listen_addresses = 'localhost'`。

    3. 改成：**`listen_addresses = '\*'`** (記得去掉前面的 `#`)。

    ### 2. 允許 Docker 網段登入 (白名單)

    1. 開啟權限設定檔：

       Bash

       ```
       sudo nano /etc/postgresql/14/main/pg_hba.conf
       ```

    2. 捲到檔案最底部，加入這一行：

       Plaintext

       ```
       host    all             all             172.17.0.0/16           md5
       ```

       *這行的意思是：允許從 `172.17.x.x` 這個範圍進來的所有連線，使用密碼驗證。*

    ### 3. 重啟並觀察

    這步最重要，設定完一定要重啟才會生效：

    Bash

    ```
    sudo systemctl restart postgresql
    ```

    然後你可以觀察 Wiki.js 的 Log，它應該會自動重試並成功連上：

    Bash

    ```
    docker logs -f wiki
    ```

    ------

    ### 如果還是不行？（終極檢查）

    如果重啟了 PostgreSQL 還是拒絕連線，請檢查 **Ubuntu 的防火牆 (UFW)**：

    Bash

    ```
    # 暫時允許 Docker 存取 5432 埠口
    sudo ufw allow from 172.17.0.0/16 to any port 5432
    ```

    > **恭喜！** 一旦你在 Log 看到 `Database Connection [ OK ]` 和 `HTTP Server listening on port 3000`，你就可以打開瀏覽器輸入 `http://localhost:3000` 開始享受你的 Wiki.js 了。

    成功連上後，需要我教你如何設定第一個管理員帳號嗎？



# Check

- docker logs wiki

  ```
  tony@tony-dell:~/ws/Kubernates-Study/Wiki.js-poc/wiki-server$ docker logs wiki
  Loading configuration from /wiki/config.yml... OK
  2026-02-26T03:02:11.415Z [MASTER] info: =======================================
  2026-02-26T03:02:11.418Z [MASTER] info: = Wiki.js 2.5.312 =====================
  2026-02-26T03:02:11.418Z [MASTER] info: =======================================
  2026-02-26T03:02:11.418Z [MASTER] info: Initializing...
  2026-02-26T03:02:11.813Z [MASTER] info: Using database driver pg for postgres [ OK ]
  2026-02-26T03:02:11.816Z [MASTER] info: Connecting to database...
  2026-02-26T03:02:11.871Z [MASTER] info: Database Connection Successful [ OK ]
  2026-02-26T03:02:12.317Z [MASTER] warn: DB Configuration is empty or incomplete. Switching to Setup mode...
  2026-02-26T03:02:12.317Z [MASTER] info: Starting setup wizard...
  2026-02-26T03:02:12.464Z [MASTER] info: Starting HTTP server on port 3000...
  2026-02-26T03:02:12.464Z [MASTER] info: HTTP Server on port: [ 3000 ]
  2026-02-26T03:02:12.475Z [MASTER] info: HTTP Server: [ RUNNING ]
  2026-02-26T03:02:12.475Z [MASTER] info: 🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻🔻
  2026-02-26T03:02:12.475Z [MASTER] info: 
  2026-02-26T03:02:12.475Z [MASTER] info: Browse to http://YOUR-SERVER-IP:3000/ to complete setup!
  2026-02-26T03:02:12.475Z [MASTER] info: 
  2026-02-26T03:02:12.475Z [MASTER] info: 🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺🔺
  
  ```

- http://localhost:3000/

  - tony.yu.microsoft@gmail.com
    - P2Leader
    - wiki.tonyyu.com