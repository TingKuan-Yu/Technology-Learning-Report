# Personal AI Wiki System – PoC



## Purpose

This PoC explores a **personal, engineering-oriented AI Wiki system**

that combines:

- Git-based Wiki (Markdown as source of truth)

- AI-assisted content editing using GitHub Copilot CLI

- Containerized deployment (Docker + local Kubernetes)

- Clear human-in-the-loop review model

The goal is **not** to replace enterprise Wiki solutions,

but to validate a **low-friction, engineer-friendly documentation workflow**.



## AI Integration Concept

- 使用 GitHub Copilot CLI 讀取 Wiki Markdown 檔案

- 由使用者輸入 prompt（例如：

  > “Refine this page to be more concise and technical”
  >  “Add troubleshooting steps for USB enumeration issue”）

- Copilot CLI 產生 **git diff**

- 使用者 review diff，手動決定是否套用並 commit

  - 確保 AI 不會在未經審核下修改內容
  - 完全符合工程師既有 Git 工作習慣

##  Architecture

- Gollum 提供 Wiki Web UI，實際內容來自 Git repo
- AI 僅負責 **產生建議修改(diff)**，由人決定是否套用
- Git history 天然保留變更紀錄與回溯能力

```


┌──────────────────────────────┐
│    Web Browser               │
│ (User / Engineer)            │
└─────────────┬────────────────┘
              │ HTTP
              ▼
┌──────────────────────────────┐
│    Gollum Wiki               │
│  (Web UI + Markdown View)    │
│  - Read / Browse pages       │
│  - Basic edit (optional)     │
└─────────────┬────────────────┘
              │ mount volume
              ▼
┌──────────────────────────────┐
│   Wiki Git Repository        │
│ (Markdown files, images)     │
│ ✅ Single Source of Truth    │
└─────────────┬────────────────┘
              │ read/write
              ▼
┌──────────────────────────────┐
│  AI Patch Service (PoC)      │
│ - REST / CLI trigger         │
│ - Calls GitHub Copilot CLI   │
│ - Generates git diff         │
└─────────────┬────────────────┘
              │
              ▼
┌──────────────────────────────┐
│   GitHub Copilot CLI         │
│ - Read Markdown files        │ 
│ - Propose content changes    │
│ - Output diff (NOT commit)   │
└──────────────────────────────┘
```

