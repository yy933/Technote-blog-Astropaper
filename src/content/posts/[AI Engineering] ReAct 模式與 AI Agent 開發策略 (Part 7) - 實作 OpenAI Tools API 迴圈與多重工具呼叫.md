---
title: "[AI Engineering] ReAct 模式與 AI Agent 開發策略 (Part 7) - 實作 OpenAI Tools API 迴圈與多重工具呼叫"
pubDatetime: 2026-09-03T06:19:35.281Z
tags: ["AI Engineering","Prompt Engineering","Concepts","AI Agent","LLM","ReAct(AI Agent)","OpenAI","API"]
description: "Table of contents 1. Agent 迭代邏輯規劃 使用 OpenAI Tools API 後，我們不..."
hackmd_id: "BkZ-tOU_Ge"
---

## Table of contents

## 1. Agent 迭代邏輯規劃  
使用 OpenAI Tools API 後，我們不再需要透過正則表達式（Regex）去剖析回應字串，而是直接依據 API 回傳的 `finish_reason` 來決定下一步：

```
Check finish_reason
 ├── "stop"        ➔ AI 已給出最終解答，直接 return 結束迴圈
 └── "tool_calls"  ➔ AI 要求呼叫工具
                    ├── 1. 將 Assistant 的請求訊息推入 messages
                    ├── 2. 遍歷 tool_calls 陣列
                    ├── 3. 執行對應的本地 JS 函式
                    └── 4. 將執行結果包裝成 role: "tool" 推入 messages，進入下一輪迴圈
```

## 2. 完整實作程式碼

```javascript
import OpenAI from "openai"
import { getCurrentWeather, getLocation, tools } from "./tools"

export const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
    dangerouslyAllowBrowser: true // 只在本地開發練習時使用!
})

const availableFunctions = {
    getCurrentWeather,
    getLocation
}

async function agent(query) {
    const messages = [
        { role: "system", content: "You are a helpful AI agent. Give highly specific answers based on the information you're provided. Prefer to gather information with the tools provided to you rather than giving basic, generic answers." },
        { role: "user", content: query }
    ]

    const MAX_ITERATIONS = 5

    for (let i = 0; i < MAX_ITERATIONS; i++) {
        console.log(`Iteration #${i + 1}`)
        
        const response = await openai.chat.completions.create({
            model: "gpt-3.5-turbo-1106",
            messages,
            tools
        })

        const { finish_reason: finishReason, message } = response.choices[0]
        
        // 【關鍵順序 1】先將 Assistant 發起請求的訊息放入歷史紀錄
        messages.push(message)

        // 【分支 1】任務完成，跳出迴圈
        if (finishReason === "stop") {
            console.log(message.content)
            console.log("AGENT ENDING")
            return message.content
        } 
        // 【分支 2】處理工具呼叫
        else if (finishReason === "tool_calls") {
            const { tool_calls: toolCalls } = message
            
            // 遍歷 toolCalls 陣列（支援多重/平行工具呼叫）
            for (const toolCall of toolCalls) {
                const functionName = toolCall.function.name
                const functionToCall = availableFunctions[functionName]
                
                // 執行本地非同步函式 (暫不帶參數)
                const functionResponse = await functionToCall()
                console.log(`Executed ${functionName}:`, functionResponse)

                // 【關鍵順序 2】將工具執行結果包裝成 role: "tool" 推入歷史紀錄
                messages.push({
                    tool_call_id: toolCall.id, // 綁定對應的呼叫 ID
                    role: "tool",
                    name: functionName,
                    content: functionResponse
                })
            }
        }
    }
}

// 測試：一次詢問多個城市的天氣
await agent("What's the current weather in Tokyo and New York City and Oslo?")
```

## 3. 注意事項
### 為什麼 `tool_calls` 是一個陣列？  
當使用者一次提出複雜問題（例如：「查詢東京、紐約與奧斯陸的天氣」）時，**LLM 可以在單一輪次中發起多重工具呼叫 (Parallel Tool Calls)。**

因此 `message.tool_calls` 是個陣列，必須使用 `for...of` 遍歷處理，確保每一個工具呼叫都有被執行並回傳結果。

### `role: "tool"` 訊息規格  
將工具結果寫回 messages 陣列時，格式有嚴格要求：

```javascript
messages.push({
    tool_call_id: toolCall.id, // 必須與 LLM 發出的 ID 一致
    role: "tool",              // 專用角色名稱
    name: functionName,        // 工具名稱
    content: functionResponse  // 執行結果 (字串格式)
})
```

- `tool_call_id `的作用：當 LLM 同時呼叫 3 次相同的工具時，`tool_call_id` 是唯一的憑證，用來將回傳結果與特定參數的工具呼叫精確綁定。

### 最容易犯錯的順序（Missing Assistant Message）  
如果在推入 `role: "tool"` 之前，忘記先 `messages.push(message)`，OpenAI API 會拋出以下錯誤：

```
Invalid parameter messages: a message with role 'tool' must be a response to a preceding message with 'tool_calls'.
```


正確對話歷史堆疊順序：

```
1. [User]      "What's the weather in Tokyo and NYC?"
2. [Assistant] { content: null, tool_calls: [ { id: "call_123", ... } ] }  <-- 必須先 push 這筆！
3. [Tool]      { role: "tool", tool_call_id: "call_123", content: "75°F" }  <-- 才能 push 這筆！
```

## 4. 執行流程日誌  
測試執行 `"What's the current weather in Tokyo and New York City and Oslo?"` 時的運作

```
Iteration #1
- LLM 回傳 finish_reason: "tool_calls"
- tool_calls 包含 3 個叫用請求 (Tokyo, NYC, Oslo)
- 程式遍歷執行 3 次 getCurrentWeather()
- 將 3 筆 role: "tool" 的結果推入 messages 陣列

Iteration #2
- LLM 讀取包含工具結果的對話歷史
- LLM 判斷資訊已足夠，回傳 finish_reason: "stop"
- 輸出內容："The weather is 75°F and sunny in Tokyo, New York City, and Oslo."
- AGENT ENDING
```

## 總結與後續規劃   
1. 結構化驅動：利用 `finish_reason`（`stop` vs `tool_calls`）能打造出乾淨、可靠且不易出錯的 Agent 迴圈。

2. 多重呼叫支援：透過遍歷 `tool_calls` 陣列，Agent 具備了一次處理多個工具呼叫的能力。

3. 下一步優化：

* 支援工具動態參數解析 (`toolCall.function.arguments`)。
* 將 Hardcoded 工具資料替換為真實第三方 API。
* 打造前端視覺化介面。