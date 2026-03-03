[TOC]



# 個人 AI WIKI system

好，下面是一份**可以直接拿去貼 Wiki / 寄給主管 / 當 PoC 提案用的「個人 AI WIKI system」PoC 提案稿**。
 內容結構也刻意對齊公司內常見的 PoC guideline（目標、範圍、成功標準、風險），方便被接受與 review，風格參考內部 PoC 範本的寫法。 [[WIKI | Cre...OC in team | ADO Work Item (PowerBI)\]](https://dev.azure.com/powerbi/3a3467dc-0814-4e9d-8eec-555851655f69/_workitems/edit/1986550)

------

## PoC Proposal: Personal AI Wiki System

### 1. Background / Motivation

在日常工程工作中，技術筆記、設計說明、debug 紀錄、平台知識大多以 Markdown / 文件方式分散在不同 repo 或個人筆記中，存在以下痛點：

- 文件分散、搜尋與維護成本高
- 文件更新仰賴人工，容易過期
- 缺乏 AI 輔助將「想法 → 可讀文件」快速落地
- 現有企業 Wiki 偏向知識庫，對工程師 Git / PR workflow 支援不足

因此提出 **Personal AI Wiki System** PoC，驗證「**Git-based Wiki + AI (Copilot CLI) 協助修改內容**」是否能成為工程師友善、可維護、可擴充的文件解決方案。

------

### 2. PoC Objective

本 PoC 目標是驗證以下能力是否可行且實用：

1. 在 **本地 Ubuntu 環境** 建立個人 Wiki 系統
2. Wiki 內容以 **Git repository (Markdown)** 作為 single source of truth
3. 透過 **AI（GitHub Copilot CLI）** 輔助產生 / 修改 Wiki 內容
4. 內容變更以 **diff / review / commit** 流程進行，避免 AI 直接寫入
5. 系統可 **Docker 化並由 local Kubernetes 管理**，具備未來搬移至 Azure（AKS）的可行性

------

### 3. Scope (PoC 範圍)

#### In Scope

- Wiki solution：**Gollum (Git-powered Wiki)**
- 作業環境：Company Ubuntu development machine
- 內容格式：Markdown
- Wiki deployment：Docker container
- Orchestration：Local Kubernetes（Minikube / K3s 擇一）
- AI 輔助：GitHub Copilot CLI（產生 diff，不自動 commit）

#### Out of Scope

- 企業級權限管理（RBAC / Azure AD SSO）
- 多使用者協作與審批流程
- 取代現有公司 Wiki / Confluence
- Production 等級 SLA / 高可用設計

------

### 4. Proposed Architecture (High Level)

```
[ Browser ]
     |
     v
[ Gollum Wiki Web ]
     |
     |  (Markdown files)
     v
[ Git Repository ]  <---- single source of truth
     |
     | (read / diff)
     v
[ AI Patch Service ]
     |
     | (Copilot CLI generates diff)
     v
[ Review / Apply Patch ]
     |
     v
[ Git Commit ]
```

- Gollum 提供 Wiki Web UI，實際內容來自 Git repo
- AI 僅負責 **產生建議修改(diff)**，由人決定是否套用
- Git history 天然保留變更紀錄與回溯能力

------

### 5. Why Gollum?

選擇 **Gollum** 作為 PoC Wiki solution 的原因：

- Git-native：Wiki 本質即 Git repository
- Markdown 為一等公民，工程師友善
- 官方提供 Docker image，易於 container 化
- 與 GitHub / GitLab Wiki 格式高度相容
- 適合「個人 / 工程導向」Wiki，而非企業 KM

（此 PoC 不追求 fancy UI，而是驗證 **workflow + AI integration**）

------

### 6. AI Integration Concept

- 使用 GitHub Copilot CLI 讀取 Wiki Markdown 檔案

- 由使用者輸入 prompt（例如：

  > “Refine this page to be more concise and technical”
  >  “Add troubleshooting steps for USB enumeration issue”）

- Copilot CLI 產生 **git diff**

- 使用者 review diff，手動決定是否套用並 commit

✅ 確保 AI 不會在未經審核下修改內容
 ✅ 完全符合工程師既有 Git 工作習慣

------

### 7. Success Criteria

PoC 成功的判定標準：

- ✅ 能在 Ubuntu 上成功啟動 Gollum Wiki（Docker）
- ✅ Wiki 內容可透過 Git clone / edit / commit 正常運作
- ✅ Copilot CLI 能針對 Wiki 內容產生有意義的修改建議
- ✅ 修改流程以 diff + human review 為主
- ✅ 系統可在 local Kubernetes 上穩定運行

------

### 8. Risks & Mitigations

| Risk                           | Mitigation               |
| ------------------------------ | ------------------------ |
| Copilot CLI 被組織 policy 禁用 | PoC 初期僅驗證技術可行性 |
| AI 修改品質不穩定              | 僅產生 diff，不自動寫入  |
| Gollum UI 不夠友善             | PoC 目標為工程使用者     |
| 未來擴充性                     | PoC 僅驗證核心概念       |

------

### 9. Expected Outcome

透過此 PoC，我們可以回答以下問題：

- Git-based Wiki 是否適合工程文件長期維護？
- AI 輔助「寫 / 改文件」是否能實際提升效率？
- 是否值得進一步發展成 team-level 或 project-level solution？
- 是否有潛力作為內部文件 AI tooling 的基礎原型？

------

### 10. Summary

本 PoC 聚焦在 **最小可行、工程導向、低風險** 的方式，驗證「**個人 AI Wiki system**」的實際價值，而非嘗試打造完整企業產品。

> **Git is the source of truth.
>  AI assists, humans decide.**

------

如果你要，我可以再幫你：

- 精簡成 **1-page proposal**（給主管快速看）
- 改成 **email / Teams 貼文版本**
- 畫一張 **簡單 architecture 圖**
- 或直接產生 **Gollum + Docker + PoC README**

你比較想拿這份提案用在「email / wiki / review meeting」哪個情境？



# architecture

好，這裡我**直接給你兩個成品**，都可直接拿去用（不用再加工）：

------

## ✅ 一、簡單 Architecture 圖（可直接貼在 Wiki / README）



┌──────────────────────────────┐

│    Web Browser      │

│ (User / Engineer)      │

└─────────────┬────────────────┘

​       │ HTTP

​       ▼

┌──────────────────────────────┐

│    Gollum Wiki      │

│  (Web UI + Markdown View)  │

│  - Read / Browse pages   │

│  - Basic edit (optional)  │

└─────────────┬────────────────┘

​       │ mount volume

​       ▼

┌──────────────────────────────┐

│   Wiki Git Repository    │

│ (Markdown files, images)  │

│ ✅ Single Source of Truth  │

└─────────────┬────────────────┘

​       │ read/write

​       ▼

┌──────────────────────────────┐

│  AI Patch Service (PoC)   │

│ - REST / CLI trigger    │

│ - Calls GitHub Copilot CLI │

│ - Generates git diff    │

└─────────────┬────────────────┘

​       │

​       ▼

┌──────────────────────────────┐

│   GitHub Copilot CLI    │

│ - Read Markdown files    │

│ - Propose content changes  │

│ - Output diff (NOT commit) │

└──────────────────────────────┘

Deployment:

\- All components run in Docker

\- Managed by local Kubernetes (Minikube / K3s)

\- Future: migrate to Azure (AKS)



> ✅ 核心設計重點：
>  **AI 只產生 diff，不直接寫入 Wiki**，所有變更仍走 Git / review。

------

## ✅ 二、PoC README（可直接放 repo）

你可以直接建立 `README.md`，內容如下：



\# Personal AI Wiki System – PoC

\## 1. Purpose

This PoC explores a **personal, engineering-oriented AI Wiki system**

that combines:

\- Git-based Wiki (Markdown as source of truth)

\- AI-assisted content editing using GitHub Copilot CLI

\- Containerized deployment (Docker + local Kubernetes)

\- Clear human-in-the-loop review model

The goal is **not** to replace enterprise Wiki solutions,

but to validate a **low-friction, engineer-friendly documentation workflow**.

\---

\## 2. Problem Statement

\- Technical knowledge is scattered across Markdown files, notes, and repos

\- Documentation becomes outdated quickly

\- Existing Wiki tools focus on UI, not engineering workflows

\- AI can help, but direct auto-editing is risky

\---

\## 3. Solution Overview

This PoC uses:

\- **Gollum** as a lightweight, Git-powered Wiki

\- **GitHub Copilot CLI** to propose documentation changes

\- **Git diff + review** as the control mechanism

Markdown files stored in Git are the **single source of truth**.

\---

\## 4. Architecture



Browser ↓ Gollum Wiki (Web UI) ↓ Git Repository (Markdown) ↓ AI Patch Service ↓ GitHub Copilot CLI

```
All components run in Docker and are orchestrated by local Kubernetes
(Minikube / K3s).

---

## 5. Key Design Principles

- ✅ Git is the source of truth
- ✅ AI proposes changes, humans decide
- ✅ No database lock-in
- ✅ Simple, reproducible deployment
- ✅ Engineer-first workflow (PR, diff, history)

---

## 6. In Scope (PoC)

- Gollum Wiki running in Docker
- Markdown-based Wiki content
- GitHub Copilot CLI generating diffs for Wiki pages
- Manual review and apply of AI-generated changes
- Local Kubernetes deployment

---

## 7. Out of Scope

- Enterprise RBAC / Azure AD SSO
- Multi-user collaboration workflows
- Production-grade availability / SLA
- Replacement of existing enterprise Wiki systems

---

## 8. Success Criteria

The PoC is considered successful if:

1. Gollum Wiki runs locally in Docker
2. Wiki content is fully Git-managed
3. Copilot CLI can generate meaningful Markdown diffs
4. AI changes can be reviewed and applied manually
5. System runs under local Kubernetes
6. Architecture is portable to Azure (AKS)

---

## 9. Risks & Mitigations

| Risk | Mitigation |
|-----|-----------|
| AI produces low-quality content | Human review via diff |
| Copilot CLI policy restrictions | Validate in PoC phase |
| Limited Wiki UI | Acceptable for engineer-focused use |
| Token / credential handling | Local PoC only |

---

## 10. Expected Outcome

This PoC validates that:

- Git-based Wiki + AI is a viable documentation model
- AI can significantly reduce documentation maintenance cost
- A human-in-the-loop approach keeps quality and safety

It provides a foundation for future team-level or org-level extensions.
```

------

## ✅ 建議你下一步（超實用）

1. 建一個 repo：`personal-ai-wiki-poc`
2. 放：
   - `README.md`（上面那份）
   - `wiki/`（Markdown pages）
   - `docker-compose.yml`
3. 用這份直接跟主管 / 同事介紹（**非常清楚、工程味夠**）

如果你要，我也可以幫你：

- ✅ 把這份縮成 **一頁 proposal（manager-friendly）**
- ✅ 補 `docker-compose.yml`
- ✅ 寫 **PoC demo 操作流程（5 分鐘 demo）**

你想往哪個方向補？



# PoC demo 操作流程（5 分鐘 demo）

好，以下是一個**「5 分鐘 PoC Demo 操作流程」**，是**Demo Script / Show Flow（示範用）**，不是宣稱產品既有流程，而是你在簡報或 live demo 時「照著跑」的節奏。
 設計目標：**5 分鐘內讓主管/同事一眼看懂：這個 AI Wiki 在幹嘛、價值在哪、為什麼合理。**

------

## 🎬 PoC Demo：Personal AI Wiki System（5 分鐘）

### Demo 前準備（不算在 5 分鐘內）

- Ubuntu 已啟動

- Docker / Minikube 已跑

- Gollum Wiki 已啟動（[http://localhost:4567）](http://localhost:4567）/)

- Wiki repo 內已有 1～2 篇 Markdown 頁面

  例如：

  - `USB-Issues.md`
  - `Bringup-Notes.md`

- GitHub Copilot CLI 已可用（登入完成）

------

## ⏱ 0:00 – 0:30｜開場（Why）

**你說的話（重點）**

> 我想 demo 一個 *工程導向* 的個人 AI Wiki。
>  核心想法是：**Wiki 內容本身就是 Git repo，AI 只負責「建議修改」，人負責決定。**

✅ 傳達三件事：

- 不是要做 Confluence replacement
- Git 是 source of truth
- AI = 助手，不是自動寫手

------

## ⏱ 0:30 – 1:30｜Step 1：打開 Wiki（What）

**操作**

- Browser 開 `http://localhost:4567`
- 點開一篇頁面，例如 `USB-Issues`

**畫面重點**

- 看到 Markdown render 的技術文件
- 明顯是工程筆記（log、steps、root cause）

**你說的話**

> 這個 Wiki 實際上就是一個 Git repo
>  每一頁 = 一個 Markdown 檔案
>  不需要 DB，也沒有 proprietary format

------

## ⏱ 1:30 – 2:30｜Step 2：展示 Git source of truth（Trust）

**操作**

- 切到 Terminal
- `git log --oneline`
- `ls *.md`
- `git show HEAD:USB-Issues.md`（或用 VS Code 開）

**你說的話**

> Wiki 沒有隱藏資料
>  所有內容都在 Git
>  你可以用 VS Code、vim、PR、code review 管它

✅ 這一步是「讓工程主管安心」的關鍵

------

## ⏱ 2:30 – 3:30｜Step 3：AI 幫忙「建議修改」（Magic）

**操作（Terminal）**

copilot

**示範 prompt（事先準備好）**

```
Please improve USB-Issues.md:
- Add a clear problem statement
- Add a troubleshooting checklist
- Keep the tone technical and concise
```

**接著**

- Copilot CLI 產生建議
- 產出 **git diff**（或你手動執行 `git diff`）

**你說的話**

> AI 不會直接改 Wiki
>  它只產生 diff
>  跟 code review 一樣，我可以看、選、或丟掉

------

## ⏱ 3:30 – 4:30｜Step 4：Human-in-the-loop（Control）

**操作**

- 顯示 `git diff`
- 手動 `git commit -am "Improve USB troubleshooting section"`

**你說的話**

> 關鍵點在這裡
>  **AI 沒有 write permission，只有 suggestion permission**
>  品質跟 ownership 還是在人

✅ 這句話非常加分（安全 + 工程文化）

------

## ⏱ 4:30 – 5:00｜Step 5：回到 Wiki 看結果（Impact）

**操作**

- Browser refresh Wiki page
- 顯示剛剛 AI 協助補強的內容

**你說的話（收尾）**

> 同一份內容
>  同一個 Git repo
>  AI 幫忙「降低寫文件的痛苦」
>  但沒有破壞工程流程

------

## ✅ Demo 結束後你已經證明的 5 件事

1. ✅ Wiki 是 Git-based、可控、可回溯
2. ✅ AI 有幫助，但不越權
3. ✅ 流程符合工程師習慣（diff / review / commit）
4. ✅ Docker + local k8s，可搬到 Azure
5. ✅ 很適合「個人 / 小團隊工程文件」

------

## 🧩 一句話總結（給主管）

> 這不是一個 AI 取代 Wiki 的系統，
>  而是一個 **AI 降低文件維護成本、但不破壞工程治理的 Wiki PoC**。

------

如果你要，我可以下一步幫你：

- ✅ 把這個 demo 流程**縮成一頁簡報**
- ✅ 幫你寫 **Demo README.md**
- ✅ 幫你準備 **5 行 demo 講稿（manager-friendly）**
- ✅ 或直接幫你做 **docker-compose + demo repo 結構**

你想接下來用在哪一種情境？（主管 review / tech sharing / Garage PoC）



# WIKI Solution

Yes — **there are solid alternatives to Gollum** that avoid Ruby and fit **C#, Python, Java, or JavaScript** very well. Below is a **clean, engineering‑oriented comparison**, focused on what you want for your **Personal AI Wiki PoC** (Git‑based, Markdown, Docker/K8s‑friendly, AI‑patchable).

------

## ✅ Best‑fit alternatives by language

### 🟨 **JavaScript / Node.js (Strongly recommended)**

These align closest to your original Gollum + Copilot CLI idea.

### **1) Wiki.js (Node.js) — closest full replacement**

**Language**: JavaScript (Node.js)
 **Storage**: Markdown + Git sync (or DB)
 **Docker/K8s**: ✅ official images
 **Why it fits**:

- Modern wiki engine written in Node.js
- Supports **Markdown stored in Git repositories**
- Has REST APIs → easier to integrate **AI patch service**
- Enterprise‑friendly (auth, extensible)

**Trade‑off**:

- Heavier than Gollum
- Uses DB by default (Git‑only mode still OK, but more moving parts)

📌 Good if you want: *“Gollum, but modern, JS‑based, and extensible”*    [[github.com\]](https://github.com/Requarks/wiki), [[en.ubunlog.com\]](https://en.ubunlog.com/wiki-js-node-git-markdown/)

------

### **2) Docsify (Node.js) — ultra‑light, PoC‑friendly**

**Language**: JavaScript
 **Storage**: Markdown files (Git repo)
 **Docker/K8s**: ✅ trivial
 **Why it fits**:

- No backend DB at all
- Markdown rendered **directly from Git**
- Perfect for **AI‑generated Markdown diffs**
- Extremely fast to demo

**Trade‑off**:

- Read‑only by design (editing via Git, not web UI)
- No built‑in auth or workflow

📌 Ideal for **5‑minute PoC demos** and personal engineering notes    [[docsify.js.org\]](https://docsify.js.org/)

------

### **3) Raneto (Node.js) — middle ground**

**Language**: JavaScript (Node.js)
 **Storage**: Markdown files
 **Docker/K8s**: ✅ official images
 **Why it fits**:

- Simple wiki / knowledge base
- Built‑in web editor (optional)
- Git‑friendly Markdown

**Trade‑off**:

- Smaller ecosystem than Wiki.js
- Less “enterprise polish”

[[raneto.com\]](https://raneto.com/)

------

## 🟦 **Python**

Great if you want **documentation‑first + Git + AI**.

### **4) MkDocs (Python)**

**Language**: Python
 **Storage**: Markdown + Git
 **Docker/K8s**: ✅ common
 **Why it fits**:

- Markdown is first‑class
- Clean structure for AI edits
- Very popular in engineering orgs

**Trade‑off**:

- Static site generator (build step)
- No live wiki editing UI

📌 Works very well with **Copilot CLI → git diff → rebuild**    [[mkdocs.org\]](https://www.mkdocs.org/)

------

## 🟩 **C# / .NET**

Best if you want **full control** or Microsoft‑native stack.

### **5) ASP.NET Core + Markdown (Custom Wiki)**

**Language**: C#
 **Storage**: Markdown files + Git
 **Why it fits**:

- Full control over workflow
- Easy to embed **AI review / diff UI**
- Natural Azure / Entra ID integration

**How**:

- ASP.NET Core
- Markdig / Westwind Markdown
- Git CLI or LibGit2Sharp

**Trade‑off**:

- You build more yourself
- Not turnkey

📌 Best if PoC may evolve into **internal product**    [[bing.com\]](https://bing.com/search?q=wiki+engine+ASP.NET+Core+markdown), [[github.com\]](https://github.com/RickStrahl/Westwind.AspNetCore.Markdown)

------

## 🟥 **Java**

Java options exist but are **weaker for Git‑Markdown wikis**. Most Java wiki engines are DB‑centric (Confluence‑like), not ideal for your Git‑AI flow.

------

## ✅ My **clear recommendation for your PoC**

Based on your stated goals:

> ✅ Git = source of truth
>  ✅ AI proposes diffs
>  ✅ Docker + K8s
>  ✅ Engineer‑friendly
>  ✅ Avoid Ruby

### **Top 2 choices**

| Choice                | When to use                             |
| --------------------- | --------------------------------------- |
| **Docsify (JS)**      | Fastest PoC, cleanest AI‑diff demo      |
| **Wiki.js (Node.js)** | If you want “real wiki” + future growth |

### **If you want C#**

- Build **minimal ASP.NET Core Markdown Wiki**
- Reuse same PoC architecture you already designed

------

## 🔁 Drop‑in replacement for your PoC architecture

```
Browser
  ↓
Docsify / Wiki.js (Markdown UI)
  ↓
Git Repository (Markdown = source of truth)
  ↓
AI Patch Service (Copilot CLI)
  → git diff
  → human review
  → commit
```

------

If you want, I can next:

- ✅ Replace your **Gollum PoC** with **Docsify‑based PoC**
- ✅ Rewrite your **5‑minute demo flow** using Docsify
- ✅ Provide a **Node.js / C# reference repo layout**
- ✅ Help you decide **Docsify vs Wiki.js** for your org context

Just tell me which direction you want to go.



# Node.js vs JavaScript 核心区别
## 1. 先给「身份定位」（一句话分清）
- **JavaScript (JS)**：是一门**编程语言**（就像「中文」这种语言本身）。
- **Node.js**：是基于 JS 语言的**运行环境**（就像「能说中文的人/设备」）。

---

## 2. 超直白对比表
| 维度     | JavaScript                     | Node.js                                   |
| -------- | ------------------------------ | ----------------------------------------- |
| 本质     | 脚本编程语言                   | JS 的运行环境（基于 Chrome V8 引擎）      |
| 运行位置 | 主要在**浏览器里**（前端）     | 主要在**服务器/电脑终端**（后端）         |
| 核心能力 | 操作网页DOM、和浏览器交互      | 操作文件、搭建服务器、处理后端逻辑        |
| 依赖     | 依赖浏览器环境                 | 独立运行，不依赖浏览器                    |
| 举例     | 点击按钮弹出弹窗、网页表单验证 | 搭建 API 接口、读写本地文件、运行后端服务 |

---

## 3. 通俗比喻（瞬间理解）
把 JavaScript 比作「烹饪方法」：
- 在浏览器里用 JS → 像「在厨房（浏览器）里用这套方法做菜」，只能用厨房的工具（DOM、浏览器API）；
- Node.js → 像「把这套烹饪方法搬到工厂（服务器）」，能用到工厂的工具（文件系统、网络模块），但没有厨房的工具（不能操作DOM）。

---

## 4. 补充关键细节
1. **JS 是 Node.js 的「基础」**：Node.js 完全兼容 JS 语法，你写的 JS 代码（除了浏览器专属API）都能在 Node.js 里跑；
2. **Node.js 扩展了 JS 的能力**：JS 原本只能在浏览器干活，Node.js 让 JS 能「走出浏览器」，做后端开发、写脚本、操作电脑硬件；
3. **使用场景差异**：
   - 写网页交互 → 用浏览器里的 JS；
   - 写后端接口、自动化脚本 → 用 Node.js 运行 JS 代码。

### 总结
1. **JavaScript 是语言**，Node.js 是「能让 JS 脱离浏览器运行的工具」；
2. JS 主打前端浏览器交互，Node.js 主打后端/终端操作；
3. Node.js 基于 JS 语法，但新增了文件、网络等后端专属功能。

如果需要，我可以给你**最简单的示例代码**：分别展示「浏览器 JS」和「Node.js 运行的 JS」的不同写法，帮你直观感受。