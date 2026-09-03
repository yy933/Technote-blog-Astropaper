---
title: "[AI Engineering] ReAct 模式與 AI Agent 開發策略 (Part 5) - 封裝 Agent 自動化迭代迴圈與終止條件"
pubDatetime: 2026-09-02T11:30:27.440Z
tags: ["AI Engineering","Prompt Engineering","Concepts","AI Agent","LLM","ReAct(AI Agent)"]
description: "Table of contents 前言與目標 本篇作為手動實作 ReAct Agent 系列文的最後一篇文，將著重於..."
hackmd_id: "HkN_nuHOfl"
---

## Table of contents

## 前言與目標  
本篇作為手動實作 ReAct Agent 系列文的最後一篇文，將著重於 Step 2 (建立 Agent 迴圈) 與 Step 4 (迴圈中止與呈現結果)。我們將把所有組件串聯起來，封裝成一個能夠自主思考、感測環境、調用工具，並自動終止的完整 AI Agent！

## 實作程式碼

```javascript
import OpenAI from "openai"
import { getCurrentWeather, getLocation } from "./tools"

export const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
    dangerouslyAllowBrowser: true
})

// 定義可用的本地工具映射表
const availableFunctions = {
    getCurrentWeather,
    getLocation
}

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

async function agent(query) {
    const messages = [
        { role: "system", content: systemPrompt },
        { role: "user", content: query }
    ]
    
    // 設定最大迭代次數，避免 API 無限迴圈耗盡預算
    const MAX_ITERATIONS = 5
    // 將 Regex 移至迴圈外，優化效能
    const actionRegex = /^Action: (\w+): (.*)$/
    
    // 【Step 2: 建立 Agent 迴圈】
    for (let i = 0; i < MAX_ITERATIONS; i++) {
        console.log(`Iteration #${i + 1}`)
        
        const response = await openai.chat.completions.create({
            model: "gpt-3.5-turbo",
            messages
        })

        const responseText = response.choices[0].message.content
        console.log(responseText)
        
        // 1. 將 LLM 的回應（Thought + Action）推入歷史對話
        messages.push({ role: "assistant", content: responseText })
        
        // 2. 解析字串
        const responseLines = responseText.split("\n")
        const foundActionStr = responseLines.find(str => actionRegex.test(str))
        
        // 【Step 3 & Step 4 邏輯判定】
        if (foundActionStr) {
            // A. 解析並執行工具
            const actions = actionRegex.exec(foundActionStr)
            const [_, action, actionArg] = actions
            
            if (!availableFunctions.hasOwnProperty(action)) {
                throw new Error(`Unknown action: ${action}: ${actionArg}`)
            }
            
            console.log(`Calling function ${action} with argument ${actionArg}`)
            const observation = await availableFunctions[action](actionArg)
            
            // 將工具執行的結果 (Observation) 推入歷史對話，繼續下一輪迴圈
            messages.push({ role: "assistant", content: `Observation: ${observation}` })
        } else {
            // 【Step 4: 迴圈中止】沒有找到 Action，代表 LLM 已給出最終 Answer
            console.log("Agent finished with task")
            return responseText
        }
    }
}

// 測試呼叫 Agent
console.log(await agent("What are some activity ideas that I can do this afternoon based on my location and weather?"))
```

## 機制解析
### 1. 為何選擇有限的 for 迴圈而不是 while(true)？
- **預算與資安防護**：呼叫 LLM API 是需要計費的。若採用無窮 while 迴圈，當 LLM 輸出格式失控（例如忘記輸出 Answer: 或陷入邏輯死鎖）時，可能導致程式無限調用 API，造成巨大的財務損失。

- 設定 `MAX_ITERATIONS`（如 5 次）**為 Agent 提供了明確的安全邊界 (Safety Net)。**

### 2. 狀態管理 (Context Window / Message History)  
Agent 能實現多輪推理與感知環境的關鍵，在於每次迭代都將對話狀態寫回 `messages` 陣列：

- LLM 回應：`messages.push({ role: "assistant", content: responseText })`
- 環境觀測：`messages.push({ role: "assistant", content: Observation: ${observation} })`

這種漸進式的對話注入，確保了 LLM 在下一輪迭代時能獲得完整的歷史 context，進而做出正確的下一步決策。

### 3. 終止條件（Exit Condition）
- 判斷邏輯：當解析陣列時 `foundActionStr` 為 `undefined`，代表 LLM 認定目前擁有的情報已足夠解決問題，不再輸出 `Action:` 語法，而是輸出最終的 `Answer:`。

此時觸發 `else` 區塊，列印完成訊息並回傳最後結果，優雅跳出迴圈。

## Agent 運作全流程

```
[ 使用者提問 ] ➔ "What can I do this afternoon based on my location & weather?"
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│ Iteration #1                                             │
│ 1. LLM 推理 ➔ Thought: 需要先取得使用者位置               │
│ 2. LLM 輸出 ➔ Action: getLocation: null                  │
│ 3. 程式解析 ➔ 執行 getLocation()                          │
│ 4. 狀態寫回 ➔ Observation: "New York City, NY"           │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│ Iteration #2                                             │
│ 1. LLM 推理 ➔ Thought: 知道位置了，接著查詢該地天氣       │
│ 2. LLM 輸出 ➔ Action: getCurrentWeather: New York City   │
│ 3. 程式解析 ➔ 執行 getCurrentWeather()                   │
│ 4. 狀態寫回 ➔ Observation: "2°F, snowy"                  │
└────────────────────────────┬─────────────────────────────┘
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│ Iteration #3                                             │
│ 1. LLM 推理 ➔ 資訊已齊全，可給出客製化活動建議             │
│ 2. LLM 輸出 ➔ Answer: 根據紐約下雪天氣，建議去中央公園... │
│ 3. 程式解析 ➔ 查無 Action! (觸發 Else 終止條件)          │
│ 4. 跳出迴圈 ➔ Return 最終解答給使用者                     │
└──────────────────────────────────────────────────────────┘
```

## 總結與潛在問題  
AI Agent 本質上就是 **「LLM 推理 + 正則解析 + 狀態維護 + 動態工具呼叫 + 控制迴圈」** 的組合。

### 潛在的問題  
透過字串切分（`split("\n")`）與正則表達式（Regex）抓取指令非常脆弱（Fragile），**只要 LLM 少打一個空格或格式微幅跑偏就可能導致解析崩潰**。

因此，為了解決手動 Parsing 的不穩定性，**現代 AI Agent 開發會轉向採用 OpenAI 原生支援的 Function Calling (Tools API) 或 LangChain / LlamaIndex 等框架，交由官方 Schema 規範結構化數據**，提高生產環境的穩定度與安全性！