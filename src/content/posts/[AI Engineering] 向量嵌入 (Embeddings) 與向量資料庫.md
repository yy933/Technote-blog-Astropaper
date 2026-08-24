---
title: "[AI Engineering] 向量嵌入 (Embeddings) 與向量資料庫"
pubDatetime: 2026-08-24T10:27:03.694Z
tags: ["AI Engineering","JavaScript","cheatsheet","Node.js","API","OpenAI"]
description: "Table of contents 什麼是 Embeddings（向量嵌入）？ Embeddings（嵌入） 是機器學..."
hackmd_id: "B1p8D9FDzg"
---

## Table of contents

## 什麼是 Embeddings（向量嵌入）？

**Embeddings（嵌入）** 是機器學習與 AI 演算法的核心基礎。簡單來說，它是將抽象的文字、句子或整篇文章轉換成一組**固定長度的高維度數字陣列（Vector / 向量）**的過程，也就是資料的「數值快照」。

傳統程式只能做「精確字詞比對」，無法理解文字背後的意涵；但透過 Embeddings，AI 能將文字映射到一個幾何的 **向量空間（Vector Space）** 中，進而掌握詞彙間的語意關聯。

`文字／句子 (Content Space) ───[ Embedding Model ]───> 高維度數字陣列 (Vector Space)`

## 為什麼需要 Embeddings？核心功能與優勢

### 1. **捕捉抽象語意（Semantic Understanding）**：
   * 語意相近的詞彙（例如 `cat` 與 `feline`），在向量空間中的距離會非常接近。
   * 能區分程度與情境（例如 `kitten` 與 `cat` 接近但帶有年齡語意；而 `building` 與兩者完全無關，距離極遠）。
### 2. **具備向量數值運算能力（Vector Arithmetic）**：
   * 經典公式：`King` - `Man` + `Woman` ≈ `Queen`。這展現了向量空間能精準量化性別、身分、階級等複合關係。
### 3. **高維度情境識別**：
   * 真實世界的模型（如 OpenAI 的 `text-embedding-ada-002`）輸出維度高達 1536 個浮點數，每一個維度代表不同的情境特徵（例如「皇室」、「歷史」、「西洋棋」等），讓 AI 能結合上下文精準判斷。

### 常見應用場景
* **語意搜尋（Semantic Search）**：超越關鍵字比對，直接理解使用者的搜尋意圖。
* **個人化推薦系統**：如 Spotify 推薦歌曲、Netflix 推薦影片、Amazon 推薦商品。
* **相似度分群（Clustering）與分類**：快速將大量文檔依主題歸類。
* **RAG 檢索增強生成**：作為 LLM 的長期記憶庫，減少 AI 幻覺。



## 什麼是向量資料庫（Vector Database）？

**向量資料庫**是專門用來高效儲存、檢索與管理高維度向量陣列的資料庫系統。

與傳統 SQL/NoSQL 資料庫搜尋「精確數值或字串」不同，向量資料庫透過**相似度指標（Similarity Metrics，如餘弦相似度 Cosine Similarity）**，能從海量向量中快速尋找與查詢字串「語意最接近」的結果。

### 向量資料庫四大 superpowers：
* **掌控私有資料**：讓 AI 只根據你的個人/企業專屬資料進行回答。
* **降低 Token 成本**：僅檢索最相關的片段餵給 LLM，避免傳送過多無關上下文。
* **建立長期記憶**：為聊天機器人儲存並摘要歷史對話。
* **抑制 AI 幻覺（Hallucination）**：提供明確的檢索依據，大幅提升回應精確度。

> **常見的向量資料庫選擇**：Pinecone、Chroma、Supabase (pgvector)、Qdrant 等。



## 實作：使用 OpenRouter 呼叫免費 Embedding 模型

雖然標準 OpenAI API 需要付費，但我們可以透過 OpenRouter 平台呼叫 NVIDIA 推出的免費嵌入模型 `nvidia/nemotron-3-embed-1b:free` 來練習實作。

### 1. 安裝套件  
使用標準的 `openai` SDK 即可相容：

```shell
npm install openai
```

### 2. 環境變數設定 (`.env`)  
在專案根目錄建立 `.env` 檔案，儲存從 [OpenRouter](https://openrouter.ai/) 取得的 API Key：

`VITE_OPENROUTER_API_KEY=sk-or-v1-your-key-here`

### 3. 設定配置文件 (config.js)  
在前端（Vite）環境中，記得使用 `import.meta.env` 讀取變數，並加上 `dangerouslyAllowBrowser: true` 放行：

```typescript
import OpenAI from "openai";

if (!import.meta.env.VITE_OPENROUTER_API_KEY) {
  throw new Error("OpenRouter API Key 未設定！");
}

const openai = new OpenAI({
  baseURL: "[https://openrouter.ai/api/v1](https://openrouter.ai/api/v1)",
  apiKey: import.meta.env.VITE_OPENROUTER_API_KEY,
  dangerouslyAllowBrowser: true, // 僅限本地開發測試使用
});

export default openai;
```

### 4. 批次轉換文字為向量 (Node.js / Vite)  
API 支援直接傳入陣列（Batch Processing），一次請求即可處理多筆資料：

```typescript
import openai from "./config.js";

const content = [
  "Beyond Mars: speculating life on distant planets.",
  "Jazz under stars: a night in New Orleans' music scene.",
  "Mysteries of the deep: exploring uncharted ocean caves.",
];

async function main(inputList) {
  // 呼叫 Embeddings API
  const response = await openai.embeddings.create({
    model: "nvidia/nemotron-3-embed-1b:free", // 免費 Embedding 模型
    input: inputList, // 支援陣列一次處理多筆
    encoding_format: "float",
  });

  // 將回傳的向量資料對應回原始文本
  const results = response.data.map((item, index) => ({
    content: inputList[index],
    embedding: item.embedding, // 取得該段文字的高維度向量陣列
  }));

  console.log(results);
}

main(content).catch(console.error);
```

## 解析 Embedding 回傳資料結構  
當發送請求後，API 回傳的 `response.data` 是一個陣列，資料結構如下：

```json
[
  {
    "object": "embedding",
    "index": 0, // 對應輸入陣列的索引值
    "embedding": [
      0.0123456,
      -0.0456789,
      0.0089123,
      // ...後面可能包含數百至數千個浮點數
    ]
  }
]
```

### 欄位說明：
* `object`：固定為 `"embedding"`，表示此物件格式。
* `index`：標示這筆向量對應到 `input` 陣列中的第幾個資料片段（從 `0` 開始）。
* `embedding`：最核心的向量資料，由無數個浮點數組成的 Array，代表這段文字在語意空間中的精準幾何座標。

## 關於純前端（Browser）開發的安全警告

:warning: 重要資安警告：  
在前端中使用 `dangerouslyAllowBrowser: true` 僅限於 `Localhost` 開發實驗！  
若將寫死 API Key 的前端程式碼部署至線上生產環境， API Key 將能被任何人從瀏覽器的 Network 頁籤中直接複製盜用。

- Production 環境的正確架構：  
必須建立一條後端 API（如 Next.js API Routes、Express 或 Cloudflare Workers）負責保管 API Key 並對外呼叫 Embeddings API，前端僅能向自己的後端發送請求。


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>