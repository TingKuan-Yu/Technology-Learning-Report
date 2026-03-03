# GIT_REPOSITORY_URL

你疑惑的是在 `docker run` 启动 Wiki.js 时配置的 `GIT_REPOSITORY_URL` 环境变量为什么好像没被读取，我会先解释这个参数的生效规则，再告诉你为什么可能没生效，以及如何确保它正常工作。

### 核心结论先说明
`GIT_REPOSITORY_URL` 这个环境变量**会被 Wiki.js 读取，但仅在 Wiki.js 「首次初始化」时生效**；如果你的 Wiki.js 已经完成过初始化（创建过管理员账号、配置过后台），这个环境变量就会被忽略，必须在后台手动配置 Git 仓库地址。

### 为什么环境变量可能「没被读取」？
#### 1. 生效时机：仅首次初始化有效
Wiki.js 的环境变量（包括 `GIT_REPOSITORY_URL`、`GIT_BRANCH` 等）的作用是**简化首次安装的配置**，流程如下：
- 首次启动 Wiki.js → 未检测到初始化配置 → 读取 `docker run` 中的 Git 环境变量 → 自动填充到后台 Git 配置中；
- 一旦完成初始化（创建了管理员账号、保存过任何配置）→ Wiki.js 会将配置写入 `/wiki/data/config.yml` 文件 → 后续启动会优先读取这个配置文件，不再读取 `docker run` 中的 Git 环境变量。

简单说：如果你的 Wiki.js 已经初始化过，`docker run` 里的 `GIT_REPOSITORY_URL` 就会失效，这是最常见的「没被读取」的原因。

#### 2. 环境变量不完整（缺少认证参数）
即使是首次初始化，仅配置 `GIT_REPOSITORY_URL` 和 `GIT_BRANCH` 也不够：
- 若仓库是**私有**：必须同时配置 `GIT_AUTH_TYPE=token`、`GIT_AUTH_TOKEN=你的令牌`，否则 Wiki.js 能读取 URL 但无法访问仓库；
- 若仓库是**公有**：需确保 `GIT_AUTH_TYPE=None`（默认可能不是），否则会因认证失败导致 URL 看似「没生效」。

完整的 Git 环境变量配置（首次初始化用）：
```bash
docker run -d \
  --name wiki \
  --network="host" \
  # 数据库配置（略）
  -e "DB_TYPE=postgres" \
  -e "DB_HOST=127.0.0.1" \
  # Git 完整环境变量（首次初始化生效）
  -e "GIT_REPOSITORY_URL=https://github.com/TingKuan-Yu/wiki.js-wiki-pages.git" \
  -e "GIT_BRANCH=main" \
  -e "GIT_AUTH_TYPE=token" \  # 私有仓库填 token，公有填 None
  -e "GIT_AUTH_TOKEN=你的GitHub个人访问令牌" \
  -e "GIT_USER_NAME=TingKuan-Yu" \
  -e "GIT_USER_EMAIL=你的邮箱" \
  -v /home/tony/.../wiki-data:/wiki/data \
  requarks/wiki:2
```

### 如何验证/确保 `GIT_REPOSITORY_URL` 生效？
#### 场景 1：Wiki.js 已初始化（环境变量失效）
此时需在后台手动配置 Git 仓库（环境变量不再生效）：
1. 登录 Wiki.js 后台 →「管理」→「集成」→「Git」；
2. 直接在页面中填写 `Repository URL`（即你的 GitHub 仓库地址），补全 Token/分支等信息；
3. 点击「测试连接」→「保存」，此时配置会写入 `/wiki/data/config.yml`，取代环境变量。

#### 场景 2：Wiki.js 未初始化（想让环境变量生效）
需先重置 Wiki.js 到首次初始化状态，再重新启动（带完整 Git 环境变量）：
```bash
# 1. 停止并删除现有容器
docker stop wiki && docker rm wiki

# 2. 清空数据目录（重置初始化状态）
rm -rf /home/tony/ws/github/dev-helper-project/wiki-data/*
sudo chown -R 1000:1000 /home/tony/ws/github/dev-helper-project/wiki-data

# 3. 重新启动容器（带完整 Git 环境变量）
docker run -d \
  --name wiki \
  --network="host" \
  -e "DB_TYPE=postgres" \
  -e "DB_HOST=127.0.0.1" \
  -e "DB_PORT=5432" \
  -e "DB_USER=wikiuser" \
  -e "DB_PASS=Happy-Tony01" \
  -e "DB_NAME=wikidb" \
  # 完整 Git 环境变量
  -e "GIT_REPOSITORY_URL=https://github.com/TingKuan-Yu/wiki.js-wiki-pages.git" \
  -e "GIT_BRANCH=main" \
  -e "GIT_AUTH_TYPE=token" \
  -e "GIT_AUTH_TOKEN=你的GitHub令牌" \
  -e "GIT_USER_NAME=TingKuan-Yu" \
  -e "GIT_USER_EMAIL=你的邮箱" \
  -v /home/tony/ws/github/dev-helper-project/wiki-data:/wiki/data \
  requarks/wiki:2
```
启动后访问 Wiki.js 初始化页面，完成管理员创建后，进入「集成→Git」会发现 URL 已自动填充，说明环境变量生效。

#### 场景 3：查看配置文件验证 URL
无论是否初始化，都可查看 Wiki.js 的配置文件，确认 URL 是否被写入：
```bash
# 查看 config.yml 中的 Git 配置
cat /home/tony/ws/github/dev-helper-project/wiki-data/config.yml | grep -A 10 "git"
```
若输出中能看到 `url: https://github.com/...`，说明 URL 已被读取（无论是环境变量还是后台配置写入）。

### 总结
1. `GIT_REPOSITORY_URL` 仅在 Wiki.js **首次初始化** 时生效，初始化后会被 `/wiki/data/config.yml` 中的配置覆盖；
2. 若环境变量看似「没被读取」，大概率是 Wiki.js 已完成初始化，需在后台手动配置 Git 仓库；
3. 即使是首次初始化，也需配置完整的 Git 环境变量（尤其是 `GIT_AUTH_TOKEN`），否则 URL 无法正常访问仓库。

关键点：环境变量是「首次初始化的快捷配置」，而非「永久生效的配置」；初始化后的 Git 配置需通过 Wiki.js 后台修改，或直接编辑 `config.yml` 文件。