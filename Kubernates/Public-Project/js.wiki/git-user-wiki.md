# Git 用户认证



你希望在 Docker 部署的 Wiki.js 中接入 Git 用户认证，这样就能追踪是谁编辑了 Wiki 内容，我会一步步教你如何配置这个功能。

首先要明确：Wiki.js 本身不直接"加入 Git user"，而是通过**启用 Git 集成 + 配置用户认证（如 OAuth/GitHub/GitLab）** 实现两个核心目标：
1. 用户必须用 Git 平台（GitHub/GitLab）账号登录 Wiki.js
2. 所有 Wiki 编辑都会提交到 Git 仓库，提交记录会关联到登录用户的 Git 账号，从而追踪编辑者

### 一、前置准备
1. 确保你有一个 Git 仓库（GitHub/GitLab/Gitea 均可，这里以 GitHub 为例）

   - git@github.com:TingKuan-Yu/wiki.js-wiki-pages.git

2. 你的 Wiki.js 容器能访问外网（或内网 Git 服务）
3. 先停止并删除旧容器（保留数据卷）：
   ```bash
   docker stop wiki && docker rm wiki
   ```

### 二、步骤 1：创建 GitHub OAuth 应用（用于用户登录）
1. 登录 GitHub → 右上角头像 → `Settings` → `Developer settings` → `OAuth Apps` → `New OAuth App`
2. 填写信息：
   - `Application name`: Wiki.js Auth
   - `Homepage URL`: `http://你的服务器IP:3000`（Wiki.js 访问地址）
   - `Authorization callback URL`: `http://你的服务器IP:3000/auth/github/callback`
3. 点击 `Register application`，记录下 `Client ID` 和生成 `Client Secret`（保存好，后面要用）

### 三、步骤 2：重新部署 Wiki.js（保留数据，新增 Git 相关配置）
修改你的 Docker 启动命令，新增 Git 和认证相关环境变量，完整命令如下：
```bash
docker run -d \
  -p 3000:3000 \
  --name wiki \
  --network="host" \
  # 原有数据库配置（保留）
  -e "DB_TYPE=postgres" \
  -e "DB_HOST=127.0.0.1" \
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
```

**关键参数解释**：
- `GIT_REPOSITORY_URL`：你的 Git 仓库地址（HTTPS 格式，确保能访问）
- `GIT_AUTH_TOKEN`：GitHub 个人访问令牌（生成方式：GitHub → Settings → Developer settings → Personal access tokens → Generate new token，勾选 `repo` 权限）
- `GIT_USER_NAME/EMAIL`：提交 Wiki 内容时显示的 Git 用户名/邮箱（会关联到编辑记录）

### 四、步骤 3：在 Wiki.js 后台配置用户认证（GitHub 登录）
1. 访问 `http://你的服务器IP:3000`，登录 Wiki.js 管理员账号
2. 进入 `管理` → `认证` → `添加策略` → 选择 `GitHub`
3. 填写配置：
   - `Client ID`：步骤 2 中生成的 GitHub OAuth Client ID
   - `Client Secret`：步骤 2 中生成的 Client Secret
   - `作用域`：默认 `user:email` 即可
   - `启用`：勾选
   - `默认组`：根据需求设置（如 `Editors`）
4. 保存配置，退出管理员账号，刷新页面即可看到 `使用 GitHub 登录` 选项

### 五、验证效果（查看谁编辑了 Wiki）
1. 用 GitHub 账号登录 Wiki.js，编辑一篇文档并保存
2. 打开你的 Git 仓库（如 GitHub），查看提交记录：
   - 提交信息会显示编辑的文档名称
   - 提交者会显示你配置的 `GIT_USER_NAME`（如果是多用户，会显示对应登录用户的 Git 信息）
3. 在 Wiki.js 后台 → `活动日志` 中，也能直接看到：
   - 哪个用户（GitHub 账号名）
   - 编辑了哪个页面
   - 编辑时间/内容变更

### 六、适配其他 Git 平台（可选）
如果用 GitLab/Gitea，仅需调整两处：
1. OAuth 应用创建：在对应 Git 平台创建（流程和 GitHub 类似）
2. Git 配置参数：
   ```bash
   # GitLab 示例
   -e "GIT_REPOSITORY_URL=https://gitlab.com/你的用户名/你的wiki仓库.git"
   -e "GIT_AUTH_TOKEN=你的GitLab个人访问令牌"
   ```

### 总结
1. 实现 Wiki.js 追踪编辑者的核心是：**Git 仓库集成 + OAuth 认证（GitHub/GitLab）**，两者缺一不可。
2. 部署时需通过环境变量配置 Git 仓库地址、认证令牌，后台需配置 OAuth 登录策略。
3. 编辑后的内容会提交到 Git 仓库，提交记录和 Wiki.js 活动日志都会关联到用户的 Git 账号，从而明确谁编辑了 Wiki。

关键点：Git 个人访问令牌需勾选足够权限（如 `repo`），OAuth 回调地址必须和配置一致，否则登录会失败；多用户场景下，每个用户用自己的 Git 账号登录，提交记录会自动关联对应账号。