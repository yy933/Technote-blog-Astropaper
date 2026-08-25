---
title: "[AI Engineering] 語意搜尋(Semantic Search)與 RAG (Retrieval-Augmented Generation)"
pubDatetime: 2026-08-25T08:29:06.384Z
tags: ["AI Engineering","JavaScript","cheatsheet","Node.js","API","Embedding","Project","Issue"]
description: "Table of contents 什麼是語意搜尋 (Semantic Search)？ 不同於傳統「關鍵字比對」，語..."
hackmd_id: "By8Q0Tcwfg"
---

## Table of contents

## 什麼是語意搜尋 (Semantic Search)？  
不同於傳統「關鍵字比對」，語意搜尋是透過 Embedding 模型 將文字轉化為高維度向量（數字座標）。資料庫透過計算向量間的「幾何距離（Cosine Similarity）」，找出含意最接近的資料。


## 什麼是 RAG (Retrieval-Augmented Generation)？  
將「向量語意搜尋（找資料）」與「LLM 生成（整理成人類對話）」結合的架構：

1. **Retrieval (檢索)**：將使用者問題轉向量，去向量資料庫撈出前 N 筆最相關的文字片段 (Context)。  
1. **Augmentation (增強)**：把捞出的 Context 拼接到 Prompt 中。  
1. **Generation (生成)**：讓 LLM 基於這些真實資料回答，避免 AI 產生「幻覺（Hallucination）」。

## 文字切塊 (Text Chunking) 實踐  
在將長文件存入向量資料庫前，必須先進行 Chunking（文本切割）：

### 為何要Chunking？

* 避開 Token 上限：模型無法一次吞下超長文件。
* 提高精準度：長文件生成的 Embedding 只有「大意」，切成短區塊才能捕捉關鍵細節。

### 核心參數設定

* chunkSize：單一區塊的最大字元數。
* chunkOverlap：相鄰區塊重疊的字數（建議設為 chunkSize 的 10% 左右），避免語意在斷句處丟失。

### 工具  
使用 LangChain 的 `RecursiveCharacterTextSplitter`，**它會按「段落 --> 句子 --> 單字」層層遞迴切分，最大程度保持語意完整。**

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

核心原則：「在不失去上下文的前提下，優化成最小字數。」
**如果單一區塊連人類抽出來看都能看懂，對 LLM 就是好區塊。**

</blockquote>


## 實作程式碼 (JavaScript / Node.js)
**結合 LangChain Text Chunking、OpenAI Embedding、Supabase 向量資料庫 與 OpenRouter LLM 對話**：

```typescript
import { openai, supabase } from "./config.js";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";

const EMBED_MODEL = "nvidia/nemotron-3-embed-1b:free";
const CHAT_MODEL = "meta-llama/llama-3.3-70b-instruct:free";

// 全域對話歷史紀錄（支援多輪對話）
const chatHistory = [
  {
    role: "system",
    content: `You are an enthusiastic podcast expert who loves recommending podcasts to people. 
You answer user questions naturally using ONLY the provided context. 
If you are unsure, say "Sorry, I don't know the answer." Do not make up facts.`,
  },
];

/**
 * 步驟一：長文字切塊與寫入資料庫 (Seed)
 */
async function seedDocument(rawText) {
  // 1. 使用 Recursive Splitter 切塊
  const splitter = new RecursiveCharacterTextSplitter({
    chunkSize: 150,
    chunkOverlap: 15,
  });
  const docs = await splitter.createDocuments([rawText]);
  const chunks = docs.map((d) => d.pageContent);

  // 2. 轉向量
  const embeddingResponse = await openai.embeddings.create({
    model: EMBED_MODEL,
    input: chunks,
  });

  const dbData = embeddingResponse.data.map((item, index) => ({
    content: chunks[index],
    embedding: item.embedding,
  }));

  // 3. 寫入 Supabase
  await supabase.from("documents").insert(dbData);
}

/**
 * 步驟二：語意搜尋 (Semantic Search)
 */
async function search(userQuery) {
  const embeddingResponse = await openai.embeddings.create({
    model: EMBED_MODEL,
    input: userQuery,
  });

  const { data, error } = await supabase.rpc("match_documents", {
    query_embedding: embeddingResponse.data[0].embedding,
    match_threshold: 0.30,
    match_count: 3, // 回傳分數最高的 3 筆
  });

  if (error || !data || data.length === 0) return "";
  return data.map((item) => item.content).join("\n- ");
}

/**
 * 步驟三：多輪對話控制與 LLM 回應 (Multi-turn RAG)
 */
async function chat(userQuery) {
  console.log(`\n👤 User: ${userQuery}`);

  // 1. 向量檢索
  const context = await search(userQuery);

  // 2. 組合 Context 與 Question
  const userContent = context
    ? `[Context]:\n- ${context}\n\n[Question]: ${userQuery}`
    : userQuery;

  // 3. 更新對話歷史
  chatHistory.push({ role: "user", content: userContent });

  // 記憶長度控制：避免超過 API Token 上限（維持最舊的 system + 最近 10 則）
  if (chatHistory.length > 11) {
    chatHistory.splice(1, 2);
  }

  // 4. 送出至 LLM
  try {
    const response = await openai.chat.completions.create({
      model: CHAT_MODEL,
      messages: chatHistory,
      temperature: 0.5,
    });

    const aiAnswer = response.choices[0].message.content;
    chatHistory.push({ role: "assistant", content: aiAnswer });
    console.log(`🤖 AI: ${aiAnswer}`);
  } catch (error) {
    if (error.status === 429) {
      console.error("⚠️ 觸發 API 流量限制 (Rate Limit)，請稍後再試。");
    } else {
      console.error("LLM Error:", error);
    }
  }
}
```

## 除錯記錄

### `HTTP 429 Too Many Requests`
- 免費 API（OpenRouter :free 模型）有嚴格的每分鐘請求數 (RPM) 限制。短時間內連續觸發多個 Embedding/Chat 請求。
- 解決方法:
  - 呼叫之間加上 `delay(3000)` 暫停時間。
  - 更換限流較鬆的模型。
  - 加上 `try...catch` 捕捉 `429` 錯誤。

### Context 相互污染 / 記憶混亂
- 全域宣告的 `chatMessages` 陣列直接不斷 `push` 新的 `Context`，導致第一輪的背景資料跑進第二輪。
- 解決方法：
  - 單輪獨立：每次 API 呼叫時在 function 內重新宣告全新的 messages。
  - 多輪連續：正確將 `user` 訊息與 `assistant` 回覆成對記錄，並加上歷史長度裁切（如限制 `splice` 陣列）。

### 搜尋結果不精準
- 整篇長文章只做成單一向量，喪失局部細節。	
- 解決方法：
  - 將文本先進行 Chunking（切塊），再針對每個 Small Chunk 生成向量存庫。