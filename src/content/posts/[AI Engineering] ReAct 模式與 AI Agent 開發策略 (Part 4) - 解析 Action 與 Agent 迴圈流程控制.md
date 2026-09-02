---
title: "[AI Engineering] ReAct 模式與 AI Agent 開發策略 (Part 4) - 解析 Action 與 Agent 迴圈流程控制"
pubDatetime: 2026-09-02T08:41:25.330Z
tags: ["AI Engineering","Prompt Engineering","Concepts","AI Agent","LLM","ReAct(AI Agent)"]
description: "Table of contents 前言：如何讓 Agent 自動決定是否繼續迭代？ 在上一篇中，我們解析了 Agen..."
hackmd_id: "Sk0hw8SdGg"
---

## Table of contents

## 前言：如何讓 Agent 自動決定是否繼續迭代？  
在上一篇中，我們解析了 Agent 的無狀態（Stateless）特性與對話歷史陣列（Messages Array）的運作方式。

本篇將聚焦在如何從 LLM 回傳的純文字中，精準抓取它想要執行的工具名稱與參數，並藉此控制迴圈是否繼續運作？

## 迴圈控制的核心：`Action:` 關鍵字  
在 ReAct 模式中，**LLM 的輸出文字是決定 Agent 是否繼續執行（Loop）的開關**：

- 存在 `Action:` 關鍵字：代表任务尚未完成，需要後端解析 Action，執行 JavaScript 函式，並把結果包成 Observation 輸入給 LLM，觸發下一次迴圈。
- 不存在` Action:` 關鍵字：代表 LLM 已經獲得足夠資訊並產出了最終答案（`Answer:`），跳出迴圈並回傳結果。

## 文字分隔符（Delimiter）與 LLM 的換行機制 (`\n`)  
LLM 輸出的文字並非亂無章法，而是透過換行符號 (`\n`) 進行段落切割：

```
Thought: I need to get the user's location.\n
Action: getLocation: null\n
PAUSE
```

### 為什麼 LLM 會自動使用 `\n` 作為分隔符？  
在 System Prompt 的 Few-Shot 範例中，我們使用了換行來分隔 `Thought:`、`Action:` 和 `PAUSE`。LLM 會模仿這個格式，將各個指令獨立放置在單獨的行（Line）中。

(備註：雖然也有開發者會在 Prompt 中強制定義特殊分隔符如 `####`，但實務測試顯示使用預設換行符 `\n`即可精準運作。)

## Action 字串解析與處理 5 大步驟 (Iteration Plan)  
在 Agent Loop 的每一次迭代中，後端程式碼將執行以下 5 個步驟來處理 LLM 的輸出：

```
[ LLM 回傳文字 Response ]
           │
           ▼
1. 依據 `\n` 切割字串 ➔ 轉為 Array of Strings
           │
           ▼
2. 搜尋包含 "Action:" 的目標字串
           │
 ┌─────────┴─────────┐
 │ (找不到 Action)   │ (找到 Action)
 ▼                   ▼
[跳出迴圈，結束]     3. 剖析出 Function 名稱與參數 (Argument)
                     │
                     ▼
                    4. 後端執行對應的 JavaScript 函式
                     │
                     ▼
                    5. 將執行結果包裝成 Observation Message 餵回 LLM Array
                     │
                     ▼
                    [觸發下一次 Loop 迭代]
```

### 5 步驟詳細說明  
1. **切割字串 (`split('\n')`)**：將 LLM 回傳的完整 Response 依據換行符號 `\n` 拆解成字串陣列。  
1. **尋找 `Action` 行**：遍歷陣列，搜尋開頭包含 `"Action:"` 的字串。  
1. **剖析 `Action`（Parse Function & Parameter）**：從小於標記或特定格式中提取出要呼叫的 JavaScript 函式名稱（如 `getCurrentWeather`）與傳入參數（如 `New York City`）。  
1. **執行真實函式**：在 Node.js 端發動對應的本地函式或 API。  
1. **注入 Observation**：將函式回傳的真實資料組裝成 `{ role: 'user', content: 'Observation: ...' }`，推入 Messages 陣列中，提供給 LLM 進行下一次推理。 

### 5 步驟處理流程對照表

| 步驟 (Step) | 操作內容 | 文字範例 / 執行結果 |  
| :--- | :--- | :--- |  
| **1. Split** | 依據 `\n` 將字串切成陣列 | `["Thought: ...", "Action: getCurrentWeather: New York City", "PAUSE"]` |  
| **2. Search** | 搜尋包含 `Action:` 的字串行 | `"Action: getCurrentWeather: New York City"` |  
| **3. Parse** | 提取函式名稱與參數 | **Function**: `getCurrentWeather`<br>**Param**: `"New York City"` |  
| **4. Execute** | 後端執行真實工具函式 | `getCurrentWeather("New York City")` |  
| **5. Observe** | 將結果封裝並推入 Messages | `Observation: { location: "New York City", forecast: ["sunny"] }` |