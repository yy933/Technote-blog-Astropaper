---
title: "[AI Engineering] ReAct 模式與 AI Agent 開發策略 (Part 8) - 工具參數解析 (Function Arguments) 與真實 API 整合"
pubDatetime: 2026-09-03T09:37:23.302Z
tags: ["AI Engineering","Prompt Engineering","Concepts","AI Agent","LLM","ReAct(AI Agent)","OpenAI","API"]
description: "Table of contents 前言 在上篇筆記中，我們完成了 Agent 處理多重工具呼叫（Parallel T..."
hackmd_id: "S19MRoU_Gx"
---

## Table of contents

## 前言  
在上篇筆記中，我們完成了 Agent 處理多重工具呼叫（Parallel Tool Calls）的基礎架構。然而，先前的工具都是「無參數」的假資料。

本篇筆記重點：

1. 工具 Schema 定義：如何在 tools 定義中為工具添加 `parameters` (型別、描述、Enum 與 Required 欄位)。  
1. 參數解析與帶入：從 `toolCall.function.arguments` 中解析出 JSON 物件並帶入本地 JS 函式。  
1. 實例擴充：整合 `ip-api.com` 打造真實的 IP 定位工具，測試 Agent 鏈式呼叫能力。

## 1. 定義帶有參數的 Tool Schema  
為了讓 LLM 知道工具需要哪些參數，我們必須使用 JSON Schema 在 `tools` 陣列中定義 `parameters` 結構：

```javascript
// tools.js
export const tools = [
    {
        type: "function",
        function: {
            name: "getCurrentWeather",
            description: "Get the current weather for a given location",
            parameters: {
                type: "object",
                properties: {
                    location: {
                        type: "string",
                        description: "The location from where to get the weather, e.g. San Francisco, CA"
                    },
                    unit: {
                        type: "string",
                        enum: ["celsius", "fahrenheit"] // 限定可填入的值
                    }
                },
                required: ["location"] // 指定必填欄位
            }
        }
    },
    {
        type: "function",
        function: {
            name: "getLocation",
            description: "Get the user's current location based on their IP address",
            parameters: {
                type: "object",
                properties: {} // 不需要參數時留空物件
            }
        }
    }
]
```

### Schema 設定小撇步：
* `properties`：定義每個參數的名稱、型別 (`string`, `number`, `boolean` 等) 與提示描述。
* `enum`：限制 LLM 只能從指定的列舉值中做選擇。
* `required`：提示模型哪些欄位是呼叫工具時絕對不能缺少的。

## 2. 程式碼實作：解析與傳遞 function.arguments  
LLM 回傳的 `toolCall.function.arguments` 是一個 JSON 格式的字串，必須使用 `JSON.parse()` 將其解析為 JavaScript 物件後傳入本地函式：

```javascript
import OpenAI from "openai"
import { getCurrentWeather, getLocation, tools } from "./tools"

export const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
    dangerouslyAllowBrowser: true
})

const availableFunctions = {
    getCurrentWeather,
    getLocation
}

async function agent(query) {
    const messages = [
        { 
            role: "system", 
            content: "You are a helpful AI agent. Give highly specific answers based on the information you're provided. Prefer to gather information with the tools provided to you rather than giving basic, generic answers." 
        },
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
        
        // 必須先將 Assistant 的請求訊息推入對話歷史
        messages.push(message)
        
        if (finishReason === "stop") {
            console.log(message.content)
            console.log("AGENT ENDING")
            return message.content
        } else if (finishReason === "tool_calls") {
            const { tool_calls: toolCalls } = message
            
            for (const toolCall of toolCalls) {
                const functionName = toolCall.function.name
                const functionToCall = availableFunctions[functionName]
                
                // 【重點】解析 JSON 字串為 JS 物件
                const functionArgs = JSON.parse(toolCall.function.arguments)
                
                // 呼叫函式並帶入參數
                const functionResponse = await functionToCall(functionArgs)
                // 將帶有tool_call_id的message推進對話歷史
                messages.push({
                    tool_call_id: toolCall.id,
                    role: "tool",
                    name: functionName,
                    content: functionResponse
                })
            }
        }
    }
}

// 測試鏈式工具呼叫：Agent 會先呼叫 getLocation，取得地點後再呼叫 getCurrentWeather
await agent("What's the current weather in my current location?")
```

## 3. 擴充真實工具：網路 IP 自動定位 (`getLocation`)  
為了讓 Agent 的實用度提升，我們使用 [http://ip-api.com/json](http://ip-api.com/json) 來實現根據使用者當前 IP 獲取地理位置的工具：

```javascript
// tools.js 裡面的 getLocation 實作
export async function getLocation() {
    try {
        const response = await fetch("http://ip-api.com/json")
        const data = await response.json()
        
        // 直接將 API 回傳的完整 JSON 物件序列化為字串回傳給 LLM
        return JSON.stringify(data)
    } catch (error) {
        console.error("Error fetching location:", error)
        return JSON.stringify({ error: "Failed to fetch location" })
    }
}
```

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

**💡為什麼可以直接把整個 JSON 回傳給 LLM？**  
現代大語言模型（如 GPT-3.5/GPT-4）具備極佳的 JSON 解析能力。我們不需要手動過濾出 city 或 country 欄位，直接回傳原始 JSON 字串，LLM 就能自動汲取它需要的欄位資訊。

</blockquote>

## 4. 鏈式呼叫執行流程分析 (Chain of Thought)  
當使用者提問：`"What's the current weather in my current location?"` 時，Agent 的多輪思考脈絡如下：

```
[Iteration 1]
1. User 問：「我目前位置的天氣如何？」
2. LLM 發現不知道 User 位置，發出 tool_call: getLocation()
3. Agent 執行 getLocation() ➔ 回傳 IP 定位資料: { city: "New York", country: "USA", ... }

[Iteration 2]
1. LLM 讀取到歷史紀錄中的 IP 資料，獲得城市「New York」
2. LLM 發出 tool_call: getCurrentWeather({ location: "New York" })
3. Agent 執行 getCurrentWeather() ➔ 回傳天氣資料: "75°F and sunny"

[Iteration 3]
1. LLM 讀取到天氣資料，判斷資訊已齊全
2. finish_reason 轉為 "stop"
3. LLM 輸出最終解答：「The current weather in New York is sunny with a temperature of 75°F.」
```

## Recap & Takeaways

* **`JSON.parse()` 的重要性**：`toolCall.function.arguments` 始終為 `string` 格式，呼叫本地函式前務必先進行 JSON 解析。
* **LLM 的結構化能力**：LLM 能精準將自然語言（如 `"Tokyo and NYC"`）轉譯為符合 JSON Schema 的參數物件。
* **鏈式工具協同 (Tool Chaining)**：Agent 能根據第一個工具的回傳結果（定位資料），動態生成第二個工具需要的參數（地點名稱），自動完成多步驟任務。