---
title: "[AI Engineering] ReAct 模式與 AI Agent 開發策略 (Part 1)"
pubDatetime: 2026-09-01T10:15:17.027Z
tags: ["LLM","AI Agent","Concepts","Prompt Engineering","AI Engineering","ReAct(AI Agent)"]
description: "Table of contents 什麼是 ReAct 模式？(Reasoning + Acting) 傳統的 LLM..."
hackmd_id: "Bk3gZmN_Gl"
---

## Table of contents

## 什麼是 ReAct 模式？(Reasoning + Acting)  
傳統的 LLM 開發模式多為「單向鏈結 (Hardcoded Chains)」，例如在程式碼中寫死：`取得位置 → 取得天氣 → 餵給 LLM → 輸出解答`。這種做法缺乏彈性，一旦使用者提問變更，整套流程就會失效。

為了讓 AI 能靈活處理解決開放性問題，我們需要引進 ReAct 框架。
**ReAct 是由 Reasoning（推理） 與 Acting（行動） 組合而成，核心概念是將 LLM 作為「推理引擎」，讓它能夠自主思考、調用工具、觀察環境結果，並持續迭代直到完成目標。**

```
使用者提問 (Query) ───> [ 思考 (Thought) ───> 行動 (Action) ───> 觀察 (Observation) ] (Loop) ───> 最終解答 (Answer)
```

## 為什麼需要 ReAct？鏈式寫法 vs. ReAct 模式

### 1. 傳統寫死鏈結 (Hardcoded Chains) 的缺點
* 功能單一化：預先在程式碼中硬編碼固定順序，AI 只能做特定任務。
* 無法應變：若使用者問「今天需要帶傘嗎？」，程式碼若寫死「先抓位置再抓新聞」，就會浪費 Token 且無法精準回應。

### 2. ReAct 模式的核心優勢
* 工具自主調配：AI 根據當前情境，自己決定「要不要用工具」、「何時用工具」、「用什麼工具」。
* 環境觀察與動態修正：獲得外部工具回傳結果（Observation）後，AI 會評估資訊是否足夠，若不足可繼續發起第二次行動。

## AI Agent 運作機制：Reason、Act、Observe 三階段  
ReAct 模式的運作是由三個主要步驟構成的Loop：

```
[使用者提問 Query]
       │
       ▼
┌──────────────┐
│  Reasoning   │ ◄────────────────────────┐
│  (思考/推理)  │                          │
└──────┬───────┘                          │
       │                                  │
       ├──► (若資料已充足) ──────────────┐ │
       │                                │ │
       ▼ (若需要更多資料)                 │ │
┌──────────────┐                        │ │
│    Acting    │                        │ │
│ (決定呼叫函式)│                        │ │
└──────┬───────┘                        │ │
       │                                │ │
       ▼ (由後端執行 Function)           │ │
┌──────────────┐                        │ │
│  Observing   │                        │ │
│  (觀察回傳結果)│ ───────────────────────┘ │
└──────────────┘                          │
                                          ▼
                                 [回答使用者 Response]
```

### Reasoning Phase (推理階段)：

* LLM 分析使用者的問題，進行 Chain-of-Thought（思維鏈）思考。
* **評估目前現有資訊是否足以回答問題，若不足則規劃下一步需要調用的工具。**

### Acting Phase (行動階段)：

* **LLM 輸出特定格式的文字**（例如指定 Function 名稱與參數），指出想要執行的動作，並主動標記 `PAUSE` 暫停。
* 重要概念：**LLM 本身不能直接運行程式碼，而是由後端程式接手解析並執行該 Function。**

### Observing Phase (觀察階段)：

* **後端程式將 Function 執行的真實數據回傳給 LLM（即為 Observation）**。
* LLM 觀察新數據，判斷是否準備好回答，或需進入下一次推理。

## 開發策略與模型成本控制 (Cost Management)

在開發自主 Agent 時，因為引入了「迴圈迭代機制」，**單次使用者提問可能會觸發 4 ~ 5 次以上的 LLM API 呼叫。**

- **開發測試期（使用輕量模型）**：建議優先選用輕量模型(例如`gpt-3.5-turbo`)。價格僅為 GPT-4 的 1/10 ~ 1/20，能大幅降低除錯與測試迴圈時的 Token 成本。
- **生產環境（使用強大模型）**：在正式上線或需要高度精確推理時，再切換回 gpt-4 或更高階模型，以獲得最優質的邏輯判斷與輸出。

## ReAct Agent 的開發藍圖 - 4 步驟  
為了從頭實作一個具備 ReAct 機制的 Agent，我們將開發流程拆解為 4 個步驟：

- 目標：建立一個能夠回答任何問題的Agent，Agent可能會需要使用者所在地與當地天氣的資訊。
- 4個步驟:
  - 設計一個優良的ReAct prompt
  - 建立迴圈(loop)讓agent進行多輪對話
  - 取得LLM輸出的action並撰寫字串解析邏輯
  - 迴圈中止，呈現最終結果

### 4 個步驟說明
#### 1. 設計一個優良的ReAct prompt  
設計良好的 System Prompt，規範 Thought, Action, PAUSE, Observation 的格式，並定義可用的外部工具（Available Actions）與提供 Few-Shot 範例。

#### 2. 建立迴圈(loop)讓agent進行多輪對話  
在 Node.js 端建立 `while` 或 `for` 迴圈，讓 Agent 能夠持續進行多輪對話與狀態推進。

#### 3. 取得LLM輸出的action並撰寫字串解析邏輯  
撰寫字串解析邏輯，捕獲 LLM 輸出的 Action 語法，並自動呼叫真實的 JavaScript 工具函式 。

#### 4. 迴圈中止，呈現最終結果  
設定跳出迴圈的終止條件，當 LLM 輸出 Answer: 標籤時停止迴圈，並將最終結果呈現給使用者。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>