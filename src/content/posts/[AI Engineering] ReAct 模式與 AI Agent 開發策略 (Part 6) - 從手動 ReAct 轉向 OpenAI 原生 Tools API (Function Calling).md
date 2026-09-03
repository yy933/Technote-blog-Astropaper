---
title: "[AI Engineering] ReAct 模式與 AI Agent 開發策略 (Part 6) - 從手動 ReAct 轉向 OpenAI 原生 Tools API (Function Calling)"
pubDatetime: 2026-09-03T03:56:59.510Z
tags: ["AI Engineering","Prompt Engineering","Concepts","AI Agent","LLM","ReAct(AI Agent)","OpenAI","API"]
description: "Table of contents 前言 在之前的實作中，我們透過 Prompt 規範 LLM 輸出 Action:..."
hackmd_id: "H1KB3DUOMe"
---

## Table of contents

## 前言

在之前的實作中，我們透過 Prompt 規範 LLM 輸出 `Action: functionName: arg`，並用正則表達式解析。但這種做法脆弱且容易出錯。

OpenAI 提供了原生的 `tools` 參數，讓 LLM 能夠直接回傳結構化的 JSON，精準告知我們「需要呼叫什麼函式」以及「傳入什麼參數」，徹底擺脫手動解析字串的困境。

## 1. 定義 `tools` 陣列結構  
在呼叫 OpenAI Chat Completions API 時，可傳入一個 `tools` 陣列。每個 Tool 的基本結構如下：

```javascript
// tools.js
export const tools = [
    {
        type: "function",
        function: {
            name: "getCurrentWeather",
            description: "Gets the current weather for a location.",
            parameters: {
                type: "object",
                properties: {}, // 無參數時設為空物件
            }
        }
    },
    {
        type: "function",
        function: {
            name: "getLocation",
            description: "Get the user's current location.",
            parameters: {
                type: "object",
                properties: {}, // 無參數時設為空物件
            }
        }
    }
]
```

說明：
* `type`：目前固定填寫 `"function"`。
* `name`：必須與本地 JavaScript 函式名稱精確匹配。
* `description`：**極度重要**！LLM 會閱讀此描述來判斷「何時」該使用這個工具。
* `parameters`：遵循 JSON Schema 格式。即使函式不需要參數，也必須明確宣告 `{ type: "object", properties: {} }`。

## 2. 在 API 請求中啟用 `tools`  
呼叫 `openai.chat.completions.create` 時，直接將 `tools` 陣列傳入：

```javascript
import { getCurrentWeather, getLocation, tools } from "./tools"

const response = await openai.chat.completions.create({
    model: "gpt-3.5-turbo",
    messages,
    tools // 帶入工具宣告
})
```

## 3. 解析 Response 物件的轉變  
使用 `tools` API 後，OpenAI 的回傳格式會根據是否需要呼叫工具而有所不同：

### 情境 A：一般對答（不需要調用工具）  
當使用者問「How are you today?」時，LLM 不需要調用工具，回應與過去相同：

* `finish_reason`: `"stop"`
* `message.content`: 包含正常回答字串。
* `message.tool_calls`: `undefined`

```json
{
    "index": 0,
    "message": {
        "role": "assistant",
        "content": "As an AI, I don't have feelings, but I'm here to assist you..."
    },
    "finish_reason": "stop"
}
```

## 情境 B：觸發工具呼叫（需要調用工具）  
當使用者問「What is my current location?」時，LLM 決定調用工具：

* `finish_reason`: 變成 `"tool_calls"`（代表 LLM 暫停回答，等待工具執行結果）。
* `message.content`: 為 `null`（LLM 暫時不輸出文字）。
* `message.tool_calls`: 產生一個陣列，包含 LLM 要求執行的工具細節（包含唯一 `id`、工具名稱 `name` 與傳入參數 `arguments`）。

```json
{
    "index": 0,
    "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
            {
                "id": "call_SDhXnJbvxSWwy1m1R1J43EmQ",
                "type": "function",
                "function": {
                    "name": "getLocation",
                    "arguments": "{}"
                }
            }
        ]
    },
    "finish_reason": "tool_calls"
}
```

## 總結與後續步驟   
1. **判斷依據改變**：過去我們檢查字串中是否有 `Action:`；現在改為檢查 `response.choices[0].message.tool_calls` 是否存在。  
1. **更高可靠性**：OpenAI 原生 API 確保了工具名稱與 JSON 參數的結構正確性，大幅降低手動 Parsing 的錯誤率。  
1. **下一步**：接下來我們將根據 `tool_calls` 陣列中的內容，自動執行本地對應的 JavaScript 函式，並將工具執行的結果回傳給 OpenAI！