# AI-WIKI Github open project

# Idea

- 建一個WIKI Web 跑在local ubuntu

- 此WIKI提供interface 可用Github copilot CLI AI去幫忙修改WIKI

- 此wiki web可跑在docker內

- 此docker可被local kubernates管理

  

## Copilot - 我有個idea, 幫忙看一下是否可在公司內的ubuntu電腦上實作完成. 之後可以布置在azure上

你的這個 idea **在公司內 Ubuntu 電腦上做 PoC（先 local 跑），再搬到 Azure**，技術上是**可行**的；而且你列的三個條件（Wiki Web、用 GitHub Copilot CLI 協助修改、Docker 化、再由 local Kubernetes 管）都能串得起來。下面我用「可行性評估 + 建議架構 + PoC 步驟藍圖 + 風險/限制」幫你快速掃一遍，讓你可以直接拿去規劃。  
（我也找到一些內部/外部資料可佐證：內部有 Kubernetes/Minikube 的簡報與文件，GitHub Copilot 有公司內入口與 Devices org 指引；外部則有 Copilot CLI 安裝與政策控管的官方說明。） [\[Azure-Kube...Pitch-Deck \| PowerPoint\]](https://microsoft.sharepoint.com/teams/ChinaAzureBGplaybook/_layouts/15/Doc.aspx?sourcedoc=%7B72C0CC4D-BE0D-448E-A626-23F2044E2375%7D&file=Azure-Kubernetes-Pitch-Deck.pptx&action=edit&mobileredirect=true&DefaultItemOpen=1), [\[Kubernetes Deep Dive \| PowerPoint\]](https://microsoft.sharepoint.com/teams/OMA/_layouts/15/Doc.aspx?sourcedoc=%7B8A0E04DD-508A-44FF-9DB7-D9058EDD1855%7D&file=Kubernetes%20Deep%20Dive.pptx&action=edit&mobileredirect=true&DefaultItemOpen=1), [\[DES \| Devi...Mar  2025! \| Outlook\]](https://outlook.office365.com/owa/?ItemID=AAMkAGE1Nzk3MjEzLTAzMDktNDAwZC05MGRiLWI4ZmJmYmEwYmZmNgBGAAAAAACmpSRZDz05QIRlp7rWFWUaBwBN5aFiz7LUSaCkA2AobrKuAAAAAAEMAABN5aFiz7LUSaCkA2AobrKuAAPJAYNaAAA%3d&exvsurl=1&viewmodel=ReadMessageItem), [\[docs.github.com\]](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli), [\[docs.github.com\]](https://docs.github.com/en/copilot/concepts/policies)

***

## 1) 你要的能力，逐條「可行性」對照

### A.「建一個 Wiki Web 跑在 local Ubuntu」

✅ 可行。常見自架 wiki（Wiki.js / BookStack / DokuWiki）都可以在 Ubuntu 上直接跑，也都有成熟的 Docker 部署方式。外部文章也直接給了「用 Docker 部署 Wiki.js」的例子。 [\[dasroot.net\]](https://dasroot.net/posts/2026/02/running-your-own-wiki-wikijs-bookstack-dokuwiki/), [\[xtom.com\]](https://xtom.com/blog/self-hosted-wiki/), [\[geekflare.com\]](https://geekflare.com/software/self-hosted-wiki-software/)

**選型建議（偏工程團隊/可 Git 化）**

*   **Wiki.js**：偏現代化、支援多種 DB、功能完整；適合要做 auth/整合與未來上 Azure。 [\[dasroot.net\]](https://dasroot.net/posts/2026/02/running-your-own-wiki-wikijs-bookstack-dokuwiki/), [\[geekflare.com\]](https://geekflare.com/software/self-hosted-wiki-software/)
*   **DokuWiki**：檔案式、不需要 DB、維護簡單；適合「先做起來」但整合性較傳統。 [\[xtom.com\]](https://xtom.com/blog/self-hosted-wiki/), [\[wikimatrix.org\]](https://www.wikimatrix.org/compare/dokuwiki+bookstack+wiki-js)
*   **BookStack**：UI 友善、書本結構；偏知識庫而非工程 Git 工作流。 [\[dasroot.net\]](https://dasroot.net/posts/2026/02/running-your-own-wiki-wikijs-bookstack-dokuwiki/), [\[wikimatrix.org\]](https://www.wikimatrix.org/compare/dokuwiki+bookstack+wiki-js)

> 如果你很在意「內容可用 Git 管 + PR flow」，也可以考慮 **Gollum**（Git wiki），不過你目前需求是「Web wiki + AI 幫改」，Wiki.js 類更像企業內 KB。

***

- Tony
  - [gollum](https://github.com/gollum)/**[gollum](https://github.com/gollum/gollum)**Public

### B.「Wiki 提供 interface，可用 GitHub Copilot CLI AI 幫忙修改 Wiki」

✅ 可行，但要注意：**Copilot CLI 是偏 terminal agent**，它不是「直接內嵌在 Wiki 介面」的產品能力；你要的是「Wiki Web UI 觸發 → 後端跑 Copilot CLI → 對 wiki 內容做 patch → 回寫」。這個模式可以做到，通常做法是：

*   Wiki 內容存在 **Git repo** 或檔案目錄（Markdown）
*   Wiki Web 提供「提交修改需求」的 UI（例如：一句 prompt + 目標頁面）
*   後端 service（受控環境）去呼叫 Copilot CLI 產生 diff / patch
*   由後端將 patch 以 PR / commit 的方式回寫（或產生建議讓人審核再套用）

**關鍵限制：組織政策**  
GitHub 官方文件明確提到：**如果 Copilot 是由組織提供，必須在 org policy 啟用 Copilot CLI，否則不能用**。  
你們內部也有 Copilot 的入口與 org 指引連結（aka.ms/copilot、DevicesWiki 指引）。 [\[docs.github.com\]](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli), [\[docs.github.com\]](https://docs.github.com/en/copilot/concepts/policies) [\[DES \| Devi...Mar  2025! \| Outlook\]](https://outlook.office365.com/owa/?ItemID=AAMkAGE1Nzk3MjEzLTAzMDktNDAwZC05MGRiLWI4ZmJmYmEwYmZmNgBGAAAAAACmpSRZDz05QIRlp7rWFWUaBwBN5aFiz7LUSaCkA2AobrKuAAAAAAEMAABN5aFiz7LUSaCkA2AobrKuAAPJAYNaAAA%3d&exvsurl=1&viewmodel=ReadMessageItem)

***

### C.「Wiki web 可跑在 Docker 內」

✅ 完全可行。上述 wiki 系統都有現成 container image/compose 範例。 [\[dasroot.net\]](https://dasroot.net/posts/2026/02/running-your-own-wiki-wikijs-bookstack-dokuwiki/), [\[geekflare.com\]](https://geekflare.com/software/self-hosted-wiki-software/)

***

### D.「Docker 可被 local Kubernetes 管理」

✅ 可行。內部簡報也直接提到 local dev 用 **Minikube**（也提到 K3s 是另一種 distro）  
外部也整理了 Minikube / K3s / Kind / MicroK8s 等選項在 Ubuntu 上的定位與差異。 [\[Kubernetes Deep Dive \| PowerPoint\]](https://microsoft.sharepoint.com/teams/OMA/_layouts/15/Doc.aspx?sourcedoc=%7B8A0E04DD-508A-44FF-9DB7-D9058EDD1855%7D&file=Kubernetes%20Deep%20Dive.pptx&action=edit&mobileredirect=true&DefaultItemOpen=1), [\[Azure-Kube...Pitch-Deck \| PowerPoint\]](https://microsoft.sharepoint.com/teams/ChinaAzureBGplaybook/_layouts/15/Doc.aspx?sourcedoc=%7B72C0CC4D-BE0D-448E-A626-23F2044E2375%7D&file=Azure-Kubernetes-Pitch-Deck.pptx&action=edit&mobileredirect=true&DefaultItemOpen=1) [\[shyam.kubeify.com\]](https://shyam.kubeify.com/2025/02/lightweight-kubernetes-options-for.html), [\[devzero.io\]](https://www.devzero.io/blog/minikube-vs-kind-vs-k3s), [\[betterstack.com\]](https://betterstack.com/community/guides/scaling-docker/minikube-vs-kubernetes/)

***

## 2) 建議的整體架構（最符合你需求的「安全 + 可搬 Azure」）

我建議把系統拆成 3 個 container（或 3 個 k8s deployment）：

### (1) Wiki Web（主服務）

*   Wiki.js / BookStack / DokuWiki 任一
*   若是 Wiki.js/BookStack 通常需要 DB（Postgres/MariaDB 等）

### (2) Git Storage（內容來源）

兩種路線擇一：

*   **路線 A（偏工程）**：Wiki content 以 **Git repo**（Markdown）為真實來源
*   **路線 B（偏傳統）**：Wiki 自己存 DB/檔案，但提供 export/import（後面做 AI patch 會比較繞）

若你想要 Copilot CLI 幫你改 wiki 最順：**路線 A**會省很多事（因為 Copilot CLI 很擅長對檔案/Repo 產生修改）。

### (3) “Copilot Patch Service”（你要的 AI 修改介面）

*   提供 REST API：`POST /propose-edit`（輸入：page、prompt）
*   在容器內執行 Copilot CLI（或 `gh copilot` 入口）
*   產出：
    *   `git diff` / patch
    *   commit 到 branch
    *   開 PR（如果你 Git 平台允許）
*   **重要：永遠讓人審核後才合併**（避免 AI 直接改壞）

> GitHub 也提到 `gh copilot` 可以協助安裝/呼叫 Copilot CLI（偏整合 GitHub CLI 生態）。 [\[github.blog\]](https://github.blog/changelog/2026-01-21-install-and-use-github-copilot-cli-directly-from-the-github-cli/), [\[docs.github.com\]](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli)

***

## 3) PoC 的落地路徑（在公司 Ubuntu 上最省力的一套）

### Step 1：先用 Docker Compose 讓 Wiki 跑起來（不先碰 k8s）

*   好處：最快驗證「Wiki + 內容儲存」可用
*   先選 Wiki：若你想未來 Azure AD/企業 auth，Wiki.js 比較對味（外部比較文也強調它的 enterprise auth 整合面向）。 [\[dasroot.net\]](https://dasroot.net/posts/2026/02/running-your-own-wiki-wikijs-bookstack-dokuwiki/), [\[geekflare.com\]](https://geekflare.com/software/self-hosted-wiki-software/)

### Step 2：把 Wiki 內容走向「可被 Git 管」

*   即使你最後選 Wiki.js，你也可以把主要內容（Markdown）放 repo
*   你可以定義：每個 page 對應一個 md 檔案（或資料夾）
*   這樣 Copilot CLI 才能自然地對檔案做 patch

### Step 3：在同一台 Ubuntu 上做 Copilot CLI 可用性驗證

官方安裝文件指出：Copilot CLI 可以用 npm / Homebrew / install script 等方式安裝，且需要有效 Copilot 訂閱；若由組織提供且 org policy 禁用 CLI 會無法使用。 [\[docs.github.com\]](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli), [\[docs.github.com\]](https://docs.github.com/en/copilot/concepts/policies)

> 這一步在公司環境常見阻礙：
>
> *   Node.js 版本限制（文件提到 npm 安裝需要 Node.js 22+） [\[docs.github.com\]](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli)
> *   公司端點政策（能不能裝 npm global / curl pipe bash）
> *   Copilot CLI policy 是否已開（組織控管） [\[docs.github.com\]](https://docs.github.com/en/copilot/concepts/policies)

### Step 4：做一個最小的 “Copilot Patch Service”

最小 MVP 就做兩件事：

1.  收 prompt + 檔名
2.  產生 patch（diff）回傳給前端（先不自動 commit）

這樣你就能在 Wiki 上做 UI：「產生修改建議」→ 顯示 diff → 人點同意才套用。

### Step 5：再搬到 local k8s（Minikube / K3s / Kind）

內部資料與外部比較都指出 Minikube 是最常見 local dev 選項；K3s 更輕量但更像 edge。你如果目標是未來上 AKS，Minikube / Kind 會比較貼近「開發測試」節奏。 [\[Kubernetes Deep Dive \| PowerPoint\]](https://microsoft.sharepoint.com/teams/OMA/_layouts/15/Doc.aspx?sourcedoc=%7B8A0E04DD-508A-44FF-9DB7-D9058EDD1855%7D&file=Kubernetes%20Deep%20Dive.pptx&action=edit&mobileredirect=true&DefaultItemOpen=1), [\[Azure-Kube...Pitch-Deck \| PowerPoint\]](https://microsoft.sharepoint.com/teams/ChinaAzureBGplaybook/_layouts/15/Doc.aspx?sourcedoc=%7B72C0CC4D-BE0D-448E-A626-23F2044E2375%7D&file=Azure-Kubernetes-Pitch-Deck.pptx&action=edit&mobileredirect=true&DefaultItemOpen=1), [\[shyam.kubeify.com\]](https://shyam.kubeify.com/2025/02/lightweight-kubernetes-options-for.html), [\[devzero.io\]](https://www.devzero.io/blog/minikube-vs-kind-vs-k3s)

***

## 4) 你需要特別注意的「公司環境風險點」（這些常決定能不能做成）

### (1) Copilot CLI 是否允許（Policy）

官方清楚寫：若組織/enterprise 管理者禁用 Copilot CLI，你就不能用。  
→ 這會是 PoC 第一個 gating item。 [\[docs.github.com\]](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli), [\[docs.github.com\]](https://docs.github.com/en/copilot/concepts/policies)

### (2) 認證與 token 保存方式

Copilot CLI 需要登入/驗證（官方安裝文件提到首次啟動會要求 /login）。  
若你把它放在 service container 裡，就會涉及「token 怎麼存、怎麼輪替、怎麼避免被別人用」。  
**建議**：PoC 階段先用「個人開發者帳號」在本機跑通；正式化再設計服務帳號/受控 identity。 [\[docs.github.com\]](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli)

### (3) 不能讓 AI 直接對內部機密亂掃

Copilot CLI 是 agentic（能讀檔/提議修改），你需要在 service 層加上：

*   只允許操作特定 repo/path（白名單）
*   只回傳 diff，不直接執行任意命令
*   強制人工 review

（你們內部推 Copilot adoption 的信件也在談 agent mode 的多步任務能力，代表它確實能「讀檔、提議 edits、跑指令」；越強就越要管控邊界。） [\[DES \| Devi...Mar  2025! \| Outlook\]](https://outlook.office365.com/owa/?ItemID=AAMkAGE1Nzk3MjEzLTAzMDktNDAwZC05MGRiLWI4ZmJmYmEwYmZmNgBGAAAAAACmpSRZDz05QIRlp7rWFWUaBwBN5aFiz7LUSaCkA2AobrKuAAAAAAEMAABN5aFiz7LUSaCkA2AobrKuAAPJAYNaAAA%3d&exvsurl=1&viewmodel=ReadMessageItem)

***

## 5) 我會怎麼幫你把這個 idea 變成「可提案的一頁架構」(你可以直接貼到 wiki/email)

**PoC 目標**  
在公司 Ubuntu 開發機上建立一套「可本機運行的 Wiki 平台」，並提供一個受控的 AI 修改流程（Copilot CLI 產生 diff → 人工審核 → 合併），最後可一鍵搬到 Azure（AKS 或 Container Apps）。

**PoC 成功標準**

1.  Wiki 服務可用（登入、建立/瀏覽頁面）
2.  任選一頁輸入 prompt，可產生對應 Markdown 的 diff
3.  Diff 可被人工套用並反映到 Wiki
4.  全部服務容器化，並可在 local Kubernetes 部署

***

## 我只問你 1 個關鍵問題（用來直接幫你定「最短 PoC 路徑」）

你希望 Wiki 的內容「**以 Git repo (Markdown) 當唯一真相**」嗎？

*   如果 **是**：我會建議走「Git-backed + Copilot Patch Service」這條最順、也最像工程團隊工作流。
*   如果 **否**：那就要看你選的 Wiki（DB/檔案）怎麼把內容抽出來給 Copilot CLI 改，流程會比較繞。

如果你回「是/否」，我可以直接給你一份更精準的 **PoC 元件清單（含建議的 Wiki 方案、k8s 選擇、以及 service 的 API 規格草案）**。



# 用gollum當wiki solution可行嗎?

結論先給：**✅ 可行，而且「非常適合你現在這個 PoC / 工程導向的 Wiki + AI（Copilot CLI）修改內容」的想法**。
 但它也有**明確取捨**，適合的情境要說清楚。

下面我用「**可行性 → 為什麼適合你 → 風險 / 不適合的點 → 是否建議採用**」來講。

------

## 一、Gollum 當 Wiki solution「技術上完全可行」

### 1️⃣ Gollum 本質

- **Gollum 是一個 Git‑based wiki**
- Wiki 內容 **就是一個 Git repository（Markdown / AsciiDoc 等純文字）**
- Web UI 只是「對 Git repo 的一層編輯/瀏覽介面」
- 原本就是 **GitHub / GitLab Wiki 的核心精神** [[github.com\]](https://github.com/gollum/gollum), [[en.wikipedia.org\]](https://en.wikipedia.org/wiki/Gollum_(software))

✅ 這點和你前面提的

> 「Wiki 提供 interface，用 GitHub Copilot CLI AI 幫忙修改 Wiki」

是**高度對齊**的。

------

### 2️⃣ Docker / Kubernetes 完全沒問題

- **官方提供 Docker image：`gollumwiki/gollum`**
- 明確支援 Docker、Volume 掛 Git repo
- 很多文件示範在 container / K8s 跑 [[hub.docker.com\]](https://hub.docker.com/r/gollumwiki/gollum), [[github-wiki-see.page\]](https://github-wiki-see.page/m/gollum/gollum/wiki/Gollum-via-Docker)

➡️ 你要：

- local Ubuntu 跑
- 包成 Docker
- 再交給 local Kubernetes（minikube / k3s / kind）
- 未來搬 Azure (AKS)

✅ **Gollum 完全 OK**

------

## 二、為什麼 Gollum「特別適合你這個 idea」

### ✅ 1. Git = Single Source of Truth（超關鍵）

Gollum 的 Wiki：

```
wiki/
 ├─ README.md
 ├─ usb/
 │   ├─ usb-architecture.md
 │   └─ usb-known-issues.md
 └─ images/
```

- 每一頁就是一個檔案
- **Git diff / history / revert 天然存在**
- 不需要從 DB dump 內容給 AI

➡️ **GitHub Copilot CLI 最擅長的就是：**

- 讀檔
- 產生 diff
- 建議 patch

✅ 這是「天作之合」 [[github.com\]](https://github.com/gollum/gollum)

------

### ✅ 2. AI 修改 Wiki 的 flow 非常乾淨

你可以做成這樣的 PoC flow（不需要 hack Gollum）：

```
[Wiki Web]
   ↓ 使用者輸入 prompt
[AI Patch Service]
   ↓ 呼叫 Copilot CLI
   ↓ 對 *.md 產生 diff
[Review UI]
   ↓ 人工確認
[git commit / PR]
   ↓
Gollum 自動顯示更新後內容
```

**沒有 DB schema** **沒有匯出 / 匯入** **沒有 API 反序列化問題**

➡️ 這一點比 Wiki.js / BookStack **簡單很多**

------

### ✅ 3. 工程師友善（你是目標使用者）

- 可直接用 VS Code / vim 改 Wiki
- PR / code review 模式天然存在
- 和你熟的 Git / DevOps flow 完全一致

📌 對 **Devices / MDEP / BSP / platform 團隊**
 👉 這是加分，不是缺點

------

## 三、但 Gollum 也有「明確限制」（一定要知道）

### ⚠️ 1. **它不是「企業知識庫型 Wiki」**

Gollum **沒有內建**：

- 漂亮的所見即所得（WYSIWYG）
- 強大的全文搜尋
- 複雜的 RBAC / page-level ACL（要自己接 auth）
- Non‑engineer 友善 UX

📌 官方定位就是「simple, git-powered wiki」 [[github.com\]](https://github.com/gollum/gollum)

➡️ 若目標是：

- PM / non-tech / 大量一般員工編輯
- 想取代 Confluence

❌ **Gollum 不適合**

------

### ⚠️ 2. Auth / SSO 需要自己處理

- 預設沒有 Azure AD / Entra ID
- 可以透過：
  - reverse proxy
  - omniauth / LDAP
  - Git 層權限

📌 可行，但不是 out-of-box（PoC OK，產品化要算成本） [[decovar.dev\]](https://decovar.dev/blog/2021/01/07/gollum-markdown-wiki/)

------

### ⚠️ 3. UI 擴充性有限

- 不適合做複雜 workflow UI
- AI prompt / diff review 建議 **獨立一個 side service**

➡️ 但你原本的構想就不是「全部塞進 Wiki UI」，這點其實沒衝突。

------

## 四、和你前面幾個方案的「定位比較」

| 方案       | 適合度（你這個 idea） | 原因                         |
| ---------- | --------------------- | ---------------------------- |
| **Gollum** | ⭐⭐⭐⭐⭐                 | Git‑native，AI patch 最乾淨  |
| Wiki.js    | ⭐⭐⭐                   | UI 強，但 DB + API + AI 複雜 |
| BookStack  | ⭐⭐                    | 偏知識庫，不是工程流         |
| DokuWiki   | ⭐⭐⭐                   | 檔案型，但 Git 整合較弱      |

------

## 五、給你的結論建議（很明確）

### ✅ 我會建議你這樣說服別人：

> **「如果目標是 PoC：Git‑based Wiki + Copilot CLI AI 改文件，Gollum 是最小、最乾淨、工程成本最低的方案。」**

### ✅ 非常適合：

- Ubuntu local PoC
- 工程團隊
- 文件即 code
- AI 產 diff、人工審核

### ❌ 不適合：

- 想做公司級 KB
- 非工程師大量編輯
- 強權限 / UI workflow

------

如果你要，我可以下一步直接幫你做其中一個：

- ✅ **「Gollum + Copilot CLI」PoC 架構圖（文字版 / wiki 可貼）**
- ✅ PoC scope checklist（一天能跑起來的版本）
- ✅ 幫你寫一段 **proposal / email**，用來跟主管或同事提案

你想往 **PoC 提案** 還是 **技術實作路線圖** 走？



