---
title: "[AI Engineering] ReAct 模式與 AI Agent 開發策略 (Part 9) - OpenAI Beta 簡化工具呼叫 (runTools)"
pubDatetime: 2026-09-03T11:12:22.342Z
tags: ["AI Engineering","Prompt Engineering","Concepts","AI Agent","LLM","ReAct(AI Agent)","OpenAI","API"]
description: "Table of contents 前言 在前面的系列文中，我們為了實現完整的 Agent 互動，手動寫了 for 迴..."
hackmd_id: "BJmwkAIdfl"
---

## Table of contents

## 前言  
在前面的系列文中，我們為了實現完整的 Agent 互動，手動寫了 `for` 迴圈來檢查 `finish_reason`、解析工具參數、執行 JS 函式，並將結果以 `role: "tool"` 寫回 `messages` 陣列。

為了減輕開發者撰寫這些重複性（Boilerplate）命令式程式碼（Imperative Code）的負擔，OpenAI 在 SDK（如 Node.js Beta 版）中推出了便利的封裝工具——`runTools`。這代表 AI 框架正從命令式開發邁向更加宣告式（Declarative）的開發模式。

## 1. 架構對比：傳統迴圈 vs. Beta `runTools`

| 比較項目 | 手動 Agent 迴圈 (`chat.completions.create`) | Beta 輔助函式 (`openai.beta.chat.completions.runTools`) |  
| :--- | :--- | :--- |  
| **迴圈管理** | 須自行撰寫 `for` 或 `while` 迴圈控制迭代次數 | **自動化**：SDK 自動在背景持續輪詢與呼叫，直到取得最終回答 |  
| **函式執行** | 需解析 `JSON.parse(arguments)` 並手動呼叫本地函式 | **自動化**：直接傳入 JS 函式引用，SDK 會自動解析參數並執行 |  
| **多工具呼叫** | 須自行遍歷 `tool_calls` 陣列並處理非同步執行 | **自動化**：原生支援平行工具呼叫（Parallel Tool Calls） |  
| **對話歷史更新** | 須手動將 Assistant 與 Tool 回傳訊息 `push` 進 `messages` | **自動化**：SDK 自動維持與維護完整對話歷史紀錄 |  
| **狀態監聽** | 透過 `console.log` 或手動印出每一次迭代資料 | 提供 `.on("message", ...)` 等事件監聽器（Event Listeners） |


## 2. Tool 定義的結構調整  
在使用 `runTools` 時，`tools` 陣列遵循 OpenAI 標準的 Tools Schema 格式，但可以直接將 JavaScript 函式實體綁定在 `function` 屬性中:

```javascript
// tools.js
import { getCurrentWeather, getLocation } from "./api"

export const tools = [
    {
        type: "function",
        function: {
            // 1. 直接將 JS 函式物件傳給 function 屬性
            function: getCurrentWeather, 
            // 2. 提供名稱與描述供 LLM 判讀
            name: "getCurrentWeather",
            description: "Get the current weather for a location",
            // 3. 提供 parameters JSON Schema
            parameters: {
                type: "object",
                properties: {
                    location: {
                        type: "string",
                        description: "The location from where to get the weather"
                    }
                },
                required: ["location"]
            }
        }
    },
    {
        type: "function",
        function: {
            function: getLocation,
            name: "getLocation",
            description: "Get the user's current location based on IP address",
            parameters: {
                type: "object",
                properties: {}
            }
        }
    }
]
```

## 3. 完整實作程式碼  
使用 `openai.beta.chat.completions.runTools` 後，原本動輒數十行的 Agent 迴圈精簡為短短數行：

```javascript
import OpenAI from "openai"
import { getCurrentWeather, getLocation, tools } from "./tools"

export const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
    dangerouslyAllowBrowser: true
})

async function agent(query) {
    const messages = [
        { 
            role: "system", 
            content: "You are a helpful AI agent. Give highly specific answers based on the information you're provided. Prefer to gather information with the tools provided to you rather than giving basic, generic answers." 
        },
        { role: "user", content: query }
    ]

    // 1. 呼叫 Beta API，自動處理工具連鎖與平行執行
    const runner = openai.beta.chat.completions.runTools({
        model: "gpt-4o",
        messages,
        tools
    })
    // 2. 透過事件監聽器查看背景執行的訊息傳遞
    .on("message", (message) => {
        console.log("[Message Event]:", message)
    })

    // 3. 異步等待 SDK 執行完所有必要函式並生成最終結果
    const finalContent = await runner.finalContent()
    
    console.log("\n=== Final Answer ===")
    console.log(finalContent)
}

await agent("What's the current weather in my current location?")
```

## 4. 關鍵機制解析
### 1. `runner` 監聽器與事件 (`.on`)  
`runTools` 會回傳一個 `RunnableCompletion` 物件（簡稱 `runner`），它採用 Event Emitter 模式，讓開發者仍能監控內部的互動細節：

- `on("message", callback)`：只要對話歷史產生新訊息（包含 System, User, Assistant, Function/Tool 訊息），就會觸發此事件。
- `on("functionCall", callback)`：當 SDK 開始執行特定函式時觸發。

### 2. 平行工具呼叫支援 (Parallel Tool Calls)  
與舊版 `runFunctions` 相比，`runTools` 在處理多重任務（例如一次詢問東京與紐約的天氣）時，會在背景平行執行這些 JS 函式，並自動將多筆帶有 `tool_call_id` 的執行結果塞回 `messages` 後再次發送給模型。

### 3. `await runner.finalContent()`  
這是一個非同步方法，SDK 會在背景自動完成所有的「模型思考 ➔ 工具呼叫 ➔ 帶入結果 ➔ 模型再思考」迴圈，直到模型不再要求呼叫任何工具並生成最終純文字回答時，此 Promise 才會 Resolve 並回傳文字內容。

## Recap and Takeaways
* **從 Functions 遷移至 Tools**：OpenAI 已將 API 標準統一切換至 `Tools`，使用 `runTools` 能完整支援平行呼叫並相容未來的工具型態。

* **抽象化趨勢**：AI 底層基礎設施演進極快，許多原本需要手動處理的 ReAct 迴圈、解析與狀態維護，正逐漸被官方 SDK 與高級 API（如 Assistants API / Beta Helpers）原生吸收。

* **理解底層依然重要**：雖然 `runTools` 大幅簡化了程式碼，但先掌握手動處理 `finish_reason`、`tool_calls` 與 `tool_call_id` 的基本原理，才能在遇到複雜排錯、客製化迴圈或跨平台框架（如 LangChain、LlamaIndex）時游刃有餘。

* **持續關注官方文件**：AI 技術迭代頻繁，建議隨時查閱 GitHub 儲存庫與最新官方文件，掌握 API 格式的最新變化。