---
title: "[AI Engineering]  使用 Groq 串接LLM API"
pubDatetime: 2026-05-29T04:02:25.533Z
tags: ["JavaScript","cheatsheet","AI Engineering","Node.js","API"]
description: "Table of contents Groq是什麼? [Groq](https://groq.com)是一個極速的 A..."
hackmd_id: "H1UuA_UgMx"
---

## Table of contents

## Groq是什麼?  
[Groq](https://groq.com)是一個極速的 AI 推理（Inference）平台。它並非自己訓練模型，而是透過自主研發的 LPU（Language Processing Unit） 晶片，專門用來加速開源模型（如 Meta 的 Llama、Google 的 Gemma、Mistral 等）的推論速度。

對開發者而言，它整合了多個開源 LLM，只需連接到一個端點、進行一次身份驗證，即可自由切換並存取多個頂尖的開源模型。

## 為什麼選擇Groq?
* **無與倫比的極致速度**：  
傳統 GPU（如 NVIDIA H100）在處理 LLM 推理時會受到記憶體頻寬的限制。Groq 的 LPU 架構突破了這個瓶頸，每秒可以輸出數百個 Token（Tokens Per Second, TPS），幾乎是市面上最快的推理服務。
* **高性價比（Cost-Effective）：**  
提供非常大方的免費額度（Free Tier），且付費方案的每百萬 Token 價格（Pricing perM tokens）通常顯著低於 OpenAI 或 Anthropic 的同等級模型。
* **相容 OpenAI API 規範：**  
Groq SDK 的設計與呼叫方式與 openai 套件高度相似，這讓開發者在更換底層服務時，幾乎可以無痛切換程式碼。


## 實作

### 取得API Key  
先到[Groq](https://groq.com)官網取得API Key。產生key之後一定要馬上複製保存，因為關閉視窗之後就不會再出現了。  
在 Node.js 環境中，建議將 Key 存放在 `.env` 檔案中：

`GROQ_API_KEY=gsk_your_key_here`

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡Tips：Groq SDK 預設會自動讀取環境變數中的 `GROQ_API_KEY`，因此在實例化 `new Groq()` 時不需手動傳入參數。

</blockquote>


### 選擇model  
接著依專案需求選擇model，這裡選擇了免費方案text-to-text中極為便宜的llama-3.1-8b-instant作為範例。可以到[官方文件](https://console.groq.com/docs/models)比較不同模型的速度、價格、[Rate Limits](https://console.groq.com/docs/rate-limits)。

常見的熱門選擇包括：

* llama-3.1-8b-instant：速度極快、成本極低，適合處理簡單任務或需要即時回應的場景。
* llama-3.3-70b-versatile：推理能力強大，適合處理複雜邏輯、程式碼生成或長文本分析。

### 串接API - 一般模式  
進入[官方文件說明頁面](https://console.groq.com/docs/model/llama-3.1-8b-instant)，這裡有很清楚的程式碼範例：

1. 安裝groq-sdk

```shell
npm install groq-sdk
```

2. 串接API (Node.js)

```javascript
import Groq from "groq-sdk";

// 建立 Groq instance 物件（會自動讀取 process.env.GROQ_API_KEY）
const groq = new Groq();

async function main() {
  // 建立對話文本
  const completion = await groq.chat.completions.create({
    model: "llama-3.1-8b-instant",
    messages: [
      {
        role: "user",
        content: "Explain why fast inference is critical for reasoning models",
      },
    ],
    temperature: 0.7, // 控制隨機性 (0-2)，越低越理性精準
    max_completion_tokens: 1024 // 限制模型單次輸出的最大 Token 數量
  });
    
  // 取得 AI 回傳的文字內容
  console.log(completion.choices[0]?.message?.content);
}

main().catch(console.error);
```

參數與欄位說明：
- `groq.chat.completions.create({...})`：用來向模型發送對話請求的主要方法。
  - `model`: `"llama-3.1-8b-instant"` 指定要使用的模型名稱
  - `messages`: 歷史對話陣列。每個物件包含：
    - `role`：角色。`"system"`（設定 AI 底層人格/規則）、`"user"`（使用者輸入的 Prompt）、`"assistant"`（AI 回應的內容）。
    - `content`：該角色的具體對話文字。
  - `temperature`：隨機性（創意度）。設為 0 回答最固定理性；設為 1 較有變化。
  - `max_completion_tokens`：模型輸出的最大限制，防止無限生成。
  - `top_p` (Number)：核採樣（Nucleus Sampling）。另一種控制隨機性的方式。設為 `1` 代表考慮所有可能的字詞。通常與 `temperature` 二選一調整即可（若調整 `temperature`，這裡就維持 `1`）。
  - `stream (Boolean)`：是否開啟「串流」模式。 設為 `true` 時，AI 不會等整段話全部生完才一口氣回傳，而是像打字機一樣，做出一點字就馬上傳回前端。
  - stop (String / Array / null)：停止訊號。可以設定特定的字串（例如 `"\n"` 或 `["END"]`），當模型生成到這個字串時，就會強制停止輸出。設為 `null` 代表不特別指定。
- `completion.choices[0]?.message?.content`:API 回傳的標準 JSON 結構中，最核心的文字回覆所在路徑。


### 串接 API - 串流模式 Stream (Node.js)

如果希望 AI 能像打字機一樣，做出一點字就馬上回傳，而不是等整段話生完才一口氣顯示，可以開啟 `stream: true`：

```javascript
import Groq from "groq-sdk";
const groq = new Groq();

async function main() {
  const chatCompletion = await groq.chat.completions.create({
    model: "llama-3.1-8b-instant",
    messages: [
      {
        role: "user",
        content: "請用繁體中文寫一首關於寫程式的短詩。",
      },
    ],
    stream: true, // 開啟串流模式
  });

  // 使用 for await...of 異步迭代器接收每一個資料碎片 (Chunk)
  for await (const chunk of chatCompletion) {
    // 串流模式下的文字增量存放在 delta.content 中
    const content = chunk.choices[0]?.delta?.content || '';
    process.stdout.write(content); 
  }
}

main().catch(console.error);
```

**Stream 模式說明：**
* **資料結構改變**：一般模式回傳的是 `message` 物件（完整內容）；**串流模式回傳的是複數個 chunk 物件**，**核心文字改存放在 delta（增量）物件中，如 `chunk.choices[0]?.delta?.content`。** 

* **安全取值**：使用可選鏈（`?.`）與空字串保底（`|| ''`），**是因為最後一個結束的 chunk 通常不包含 content，而是包含 `"finish_reason": "stop"` 來宣告生成結束**。

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

**Chunk的資料結構**  
當開啟 `stream: true` 時，Groq 回傳的不是一個完整的 JSON 大物件，而是好幾十個、甚至上百個小碎片（Chunks）。每個 chunk 其實都是一個 JavaScript 物件。  
在 `for await (const chunk of chatCompletion)` 迴圈中，每次迭代拿到的 chunk 結構大致如下：

```json
{
  "id": "chatcmpl-7b89f81a-6377-4b72-8876-b631d8c11bf8",
  "object": "chat.completion.chunk",
  "created": 1716382000,
  "model": "llama-3.1-8b-instant",
  "system_fingerprint": "fp_87cbf7b192",
  "choices": [
    {
      "index": 0,
      "delta": {
        "content": "你"
      },
      "logprobs": null,
      "finish_reason": null
    }
  ]
}
```

**Chunk重要欄位詳細解析**  
雖然欄位很多，但在前端或後端開發時，通常只會把目光聚焦在 choices 裡面：
* **choices (Array)：**  
這是一個陣列。因為 API 允許你一次生成多個不同的回答（透過 n 參數設定，但預設是 1），所以它用陣列儲存。通常我們只拿第一個，也就是 `choices[0]`。

* **delta (Object)：**  
這是與非串流（Non-stream）最大的不同點！  
在非串流模式下，這裡的欄位叫 message（包含完整的對話）；但在串流模式下，它叫 delta（代表「增量」或「變化量」），裡面只包含這一個碎片新蹦出來的字。

* **content (String)：**  
這次串流傳過來的文字片段（例如："你"、"好"、"嗎"，有時候甚至是半個中文字的編碼，或者是換行符號 \n）。

* **finish_reason (String / null)：**  
代表模型這段話生成完了沒有。  
在前面的 99% 的 chunks 裡面，它都會是 null（代表還沒生完，後面還有）。  
當收到最後一個 chunk 時，`content` 會是空的，而 `finish_reason` 會變成` "stop"`（代表正常結束）或 `"length"`（代表達到設定的 `max_completion_tokens` 上限而強制中斷）。


</blockquote>

## 關於純前端（Browser）開發的安全警告  
groq-sdk 預設只允許在 Node.js 等後端環境執行。**若直接在純前端（如 React、Vue 或純 HTML 檔案）中使用並寫死 API Key，Key 將會完全暴露在瀏覽器的 Network 中，極易被他人竊取盜刷！**(除了在本地端練習以外，完全不建議把key放在純前端的環境中)

若僅在 Localhost 本地端實驗，可透過加入 `dangerouslyAllowBrowser: true` 來強制放行：

```javascript
const groq = new Groq({
  apiKey: "gsk_...",
  dangerouslyAllowBrowser: true // 允許在瀏覽器環境執行，線上環境請勿使用！
});
```
實務上部署至線上環境時，**正確做法應是建立一條後端 API（如 Next.js API Routes、Express 或 Cloudflare Workers）來隱藏 API Key，再由前端發送 fetch 請求呼叫該後端**。


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>