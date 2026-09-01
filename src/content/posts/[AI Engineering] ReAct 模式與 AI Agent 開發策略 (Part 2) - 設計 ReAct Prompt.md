---
title: "[AI Engineering] ReAct 模式與 AI Agent 開發策略 (Part 2) - 設計 ReAct Prompt"
pubDatetime: 2026-09-01T11:10:29.132Z
tags: ["AI Engineering","Prompt Engineering","Concepts","AI Agent","LLM","ReAct(AI Agent)"]
description: "Table of contents 前言：Prompt 是 AI Agent 的靈魂 在上篇筆記中，我們介紹了 ReA..."
hackmd_id: "SkF5yN4OMx"
---

## Table of contents

## 前言：Prompt 是 AI Agent 的靈魂  
在上篇筆記中，我們介紹了 ReAct (Reasoning + Acting) 模式的觀念與 4 個實作步驟。

本篇將說明如何設計 ReAct Prompt。AI Agent 本身並不具備真正的思考或程式執行能力，因此我們需要透過精心設計的 System Prompt，規範 LLM 按照 **「思考 ➔ 行動 ➔ 暫停 ➔ 觀察 ➔ 解答」** 的固定思維鏈進行回應。

## 4步驟實作地圖

```
┌─────────────────────────────────────────────────────────┐
│ [Plan 1] Design a well-written ReAct prompt             │ ◄─ 本篇重點
└────────────────────────────┬────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────┐
│ [Plan 2] Build a loop for my agent to run in            │
└────────────────────────────┬────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────┐
│ [Plan 3] Parse any actions that the LLM determines      │
└────────────────────────────┬────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────┐
│ [Plan 4] End condition - final Answer is given          │
└─────────────────────────────────────────────────────────┘
```

## 設計 ReAct System Prompt 關鍵要素

一個合格的 ReAct System Prompt 需要具備以下四大關鍵區塊：

### 1. 步驟與循環流程規範 (Cycle Workflow)  
明確定義思考流程，並強調 LLM 輸出 `PAUSE` 標記的時機。

* Thought：描述對問題的分析與理解（思維鏈）。
* Action：選擇並格式化要執行的工具，隨後強制回傳 `PAUSE`。
* PAUSE：**關鍵暫停標記！** 告訴後端程式碼「**LLM 已完成本次推理，正在等待工具執行結果**」。
* Observation：由後端程式注入工具執行的結果。
* Answer：當資訊足夠時，產出最終解答並跳出循環。

### 2. 可用工具宣告 (Available Actions)  
條列 Agent 可以呼叫的外部 Function 列表、所需的傳參格式與功能描述。

### 3. 多樣本示範 (Few-Shot Prompting)  
提供一組完整的對話範例 (Example session)，展示 Agent 如何從收到提問、調用工具、接收 Observation，到最後產出 Answer 的完整歷程。

## 程式碼實作：`systemPrompt` 設計  
以下為在 Node.js / JavaScript 專案中針對位置與天氣情境設計的 `systemPrompt`：

```javascript
import OpenAI from "openai"
import { getCurrentWeather, getLocation } from "./tools"

export const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
    dangerouslyAllowBrowser: true
})

/**
 * Goal - Build an agent that can answer any questions 
 * that might require knowledge about my current location and the current weather at my location.
 */

const systemPrompt = `
You cycle through Thought, Action, PAUSE, Observation. At the end of the loop you output a final Answer. Your final answer should be highly specific to the observations you have from running the actions.

1. Thought: Describe your thoughts about the question you have been asked.
2. Action: run one of the actions available to you - then return PAUSE.
3. PAUSE
4. Observation: will be the result of running those actions.

Available actions:
- getCurrentWeather: 
    E.g. getCurrentWeather: Salt Lake City
    Returns the current weather of the location specified.
- getLocation:
    E.g. getLocation: null
    Returns user's location details. No arguments needed.

Example session:
Question: Please give me some ideas for activities to do this afternoon.
Thought: I should look up the user's location so I can give location-specific activity ideas.
Action: getLocation: null
PAUSE

You will be called again with something like this:
Observation: "New York City, NY"

Then you loop again:
Thought: To get even more specific activity ideas, I should get the current weather at the user's location.
Action: getCurrentWeather: New York City
PAUSE

You'll then be called again with something like this:
Observation: { location: "New York City, NY", forecast: ["sunny"] }

You then output:
Answer: <Suggested activities based on sunny weather that are highly specific to New York City and surrounding areas.>
`
```

### 設計細節解析

| 提示詞區塊 | 運作機制與設計目的 |  
| :--- | :--- |  
| **`PAUSE` 標記** | 讓 LLM 「主動暫停」。當 LLM 輸出 `PAUSE` 時，後端程式碼會截斷輸出並執行對應的 JavaScript Function。 |  
| **`Available actions` 格式** | 採用 `ActionName: Argument` 的簡潔格式，方便後端使用簡單的字串分割或正則表達式進行 Parsing。 |  
| **`Few-Shot` 範例** | 引導 LLM 模仿正確的 `Thought` 邏輯與格式，大幅降低 LLM 在中途格式跑偏或忘記輸出 `PAUSE` 的機率。 |




<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>