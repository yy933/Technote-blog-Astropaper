---
title: "[AI Engineering] ReAct 模式與 AI Agent 開發策略 (Part 3) - Agent Loop 運作機制與無狀態特性"
pubDatetime: 2026-09-02T04:13:33.428Z
tags: ["AI Engineering","Prompt Engineering","Concepts","AI Agent","LLM","ReAct(AI Agent)"]
description: "Table of contents 前言：理解 Agent 迴圈的本質 在設計好 ReAct System Promp..."
hackmd_id: "Syf00Mr_Gx"
---

## Table of contents

## 前言：理解 Agent 迴圈的本質  
在設計好 ReAct System Prompt之後，我們即將進入第2步驟，也就是建立一個agent能運作的loop。

但在動手寫程式碼之前，必須先釐清一個關鍵問題：「所謂的 Agent 迴圈（Agent Loop）在底層到底是如何運作的？」

**LLM 本質上只是一個無狀態 (Stateless) 的 API Endpoint。** 本篇將從 LLM 的視角切入，剖析 Agent 如何透過「對話歷史陣列（Messages Array）」在多次 API 呼叫之間傳遞狀態。

## LLM 的無狀態特性 (Stateless Nature)  
LLM 就像一個完全沒有長期記憶的 API：

* **無狀態 (Stateless)**：每一次呼叫 `openai.chat.completions.create` 時，LLM 都不會記得上一次跟你聊了什麼。
* **對話歷史陣列 (Messages Array)**：為了讓 LLM 「記憶」上下文，每次發送請求時，我們**必須將整串歷史訊息（包含 System Prompt、User Question、過去的 Thought/Action、以及 Observation）封裝成陣列一次性餵給 LLM。**
* **重新審視 (Start from Scratch)**：LLM 每次被喚醒，都是從頭閱讀這份訊息陣列，並根據最新的資訊決定產出下一個 Message。

## 圖解 Agent Loop 執行生命週期 (Agent Lifecycle)  
理解 Agent 運作最好的方式，是從 LLM 的「被喚醒 (Awake)」與「休眠 (Sleep)」過程來看：

```
使用者提問 (User Query)
       │
       ▼
┌────────────────────────────────────────────────────────┐
│ [Loop 1] LLM 被喚醒 (Awake)                             │
│ 1. 接收 Messages: [System Prompt, User Query]          │
│ 2. 推理與決定: 輸出 Thought + Action (要求取得位置)    │
│ 3. 輸出 PAUSE 標記 ───► LLM 休眠 (Sleep / Stop)        │
└────────────────────────┬───────────────────────────────┘
                         │
                         ▼ (後端 JavaScript 接手)
               [執行工具: getLocation()]
                         │
                         ▼ (獲得結果: New York City)
┌────────────────────────────────────────────────────────┐
│ [Loop 2] LLM 再次被喚醒 (Awake)                         │
│ 1. 接收完整歷史 Messages:                               │
│    [System Prompt, User Query, Thought+Action, Observation]│
│ 2. 推理與決定: 輸出 Thought + Action (查詢紐約天氣)     │
│ 3. 輸出 PAUSE 標記 ───► LLM 休眠 (Sleep / Stop)        │
└────────────────────────┬───────────────────────────────┘
                         │
                         ▼ (後端 JavaScript 接手)
               [執行工具: getCurrentWeather()]
                         │
                         ▼ (獲得結果: Sunny)
┌────────────────────────────────────────────────────────┐
│ [Loop 3] LLM 再次被喚醒 (Awake)                         │
│ 1. 接收完整歷史 Messages (包含最新天氣 Observation)     │
│ 2. 評估資訊足夠，產出最終解答: Answer                   │
│ 3. 任務結束，跳出 Agent Loop                             │
└────────────────────────────────────────────────────────┘
```

## Agent Loop 運作細節表格

| 階段 / Loop | 傳入 LLM 的 Messages 陣列內容 | LLM 產出內容 (Output) | LLM 狀態 / 後端行為 |  
| :--- | :--- | :--- | :--- |  
| **Loop 1** | 1. `System Prompt`<br>2. `User Question` | - `Thought`: 需要先取得使用者位置<br>- Action: `getLocation: null`<br>`PAUSE` | **LLM 休眠**<br>後端執行 `getLocation()` 取得 `"New York City, NY"` |  
| **Loop 2** | 1. `System Prompt`<br>2. `User Question`<br>3. `Thought + Action`<br>4. `Observation: "New York City, NY"` | - `Thought`: 需查詢紐約天氣<br>- `Action`: `getCurrentWeather: New York City`<br>`PAUSE` | **LLM 休眠**<br>後端執行 `getCurrentWeather()` 取得 `晴天` |  
| **Loop 3** | 1. `System Prompt`<br>2. `User Question`<br>3. `Thought` + `Action` (位置)<br>4. `Observation` (位置)<br>5. `Thought` + `Action` (天氣)<br>6. `Observation` (天氣) | `Answer`: 根據紐約晴天推薦具體戶外活動... | **任務完成**<br>偵測到 `Answer`:，跳出 Agent Loop |


## 總結：Agent 的雙重角色分工
**ReAct Agent 的運作本質是 LLM 與後端程式碼的協同接力賽：**

* **LLM (Reasoning Engine)**：負責大腦的思考與決策（決定「做什麼 Action」與「何時結束 Answer」）。
* **後端程式碼 (Execution & Loop Control)**：負責維持 `while` 迴圈、解析 Action 手腳發動、執行 JS 函式，並把結果包裝成 Observation 推進給 LLM。


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>