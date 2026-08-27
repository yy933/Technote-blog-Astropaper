---
title: "[AI Engineering] 使用 Google Gemini API 生成 Text Embedding (向量)"
pubDatetime: 2026-08-27T04:18:26.949Z
tags: ["AI Engineering","JavaScript","cheatsheet","Node.js","API","Gemini","Supabase","Embedding","Database"]
description: "Table of contents Gemini Embedding 是什麼? Google Gemini 提供高效能..."
hackmd_id: "S1Gn_QTDMl"
---

## Table of contents

## Gemini Embedding 是什麼?  
Google Gemini 提供高效能的文字向量化（Text Embedding）服務，能將非結構化的文字（如電影簡介、文章、產品描述）轉換為高維度的數值向量（Array of numbers）。

轉換後的向量能夠捕捉文字背後的**語意資訊（Semantic Meaning）**，讓電腦可以透過計算向量之間的餘弦相似度（Cosine Similarity），輕鬆實現語意搜尋、RAG（檢索增強生成）、電影與商品推薦系統等功能。

## 為什麼選擇 Gemini Embedding?
* **極高免費額度（Free Tier）**：  
相較於 OpenAI 的免費模型(例如`text-embedding-3-small`)，Google Gemini API 提供相當寬鬆且大方的免費試用額度，非常適合個人專案、MVP 開發或學習驗證。
* **高彈性的輸出維度（Output Dimensionality）**：  
最新一代的模型（如 `gemini-embedding-001`）預設輸出為 3072 維度，但支援**自訂維度裁切（如降至 768 維）**，完美相容於大部分預設 768 維度的向量資料庫（例如 Supabase pgvector、Pinecone 等）。
* **極佳的多語言語意理解**：  
承襲 Google 強大的語言模型底蘊，對繁體中文及多國語言的語意捕捉能力相當突出。


## 實作
### Prerequisite  
1. 在Supabase建立資料表  
2. 開啟 vector extension  
3. 建立Vector index (HNSW Index) 加速相似度語意搜尋  
4. 建立RPC function(比對相似度)

```sql
-- 1. 開啟 vector 擴充套件
create extension if not exists vector;

-- 2. 建立 movies 資料表 (設定 768 維度)
create table movies (
  id bigint primary key generated always as identity,
  title text not null,
  release_year int,
  content text,
  embedding vector(768)
);

-- 3. 建立 HNSW Index 加速搜尋
create index on movies using hnsw (embedding vector_cosine_ops);

-- 4. 建立 RPC 函式 (match_movies)
CREATE OR REPLACE FUNCTION match_movies (
  query_embedding VECTOR(768),
  match_threshold FLOAT DEFAULT 0.0,
  match_count INT DEFAULT 1
)
RETURNS TABLE (
  id BIGINT,
  title TEXT,
  release_year INT2,
  content TEXT,
  similarity FLOAT
)
LANGUAGE plpgsql
AS $$
BEGIN
  RETURN QUERY
  SELECT
    movies.id,
    movies.title,
    movies.release_year,
    movies.content,
    -- Calculate Cosine Similarity (1 - Cosine Distance)
    1 - (movies.embedding <=> query_embedding) AS similarity
  FROM movies
  WHERE 1 - (movies.embedding <=> query_embedding) > match_threshold
  ORDER BY movies.embedding <=> query_embedding ASC
  LIMIT match_count;
END;
$$;
```

### 取得 API Key  
先到 [Google AI Studio](https://aistudio.google.com/) 點擊 **Get API key** 免費申請 API Key。產生 Key 之後請妥善保存。

在 Node.js / Next.js 環境中，請將 Key 存放在 `.env.local` 或 `.env` 檔案中：

`GEMINI_API_KEY=your_gemini_api_key_here`

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡Tips：Gemini SDK (`@google/genai`) 預設會自動讀取環境變數中的 `GEMINI_API_KEY`，因此在實例化時無需手動傳入 Key。

</blockquote>


### 選擇 Model 與維度注意事項  
Google 的 Embedding 模型目前熱門選擇為：

* `gemini-embedding-2`：Gemini API 中第一個多模態嵌入模型。這項技術會將文字、圖片、影片、音訊和文件對應到統一的嵌入空間，支援超過 100 種語言的跨模態搜尋、分類和叢集。
* `gemini-embedding-001`：處理純文字內容的標準文字向量模型，語意精準且回應快速。



<blockquote class="my-6 p-4 bg-red-50 dark:bg-red-950/30 border-l-4 border-red-500 rounded-r-md text-red-900 dark:text-red-200 blocknoted-fix">

⚠️ **注意事項：維度（Dimensions）陷阱**
* Gemini Embedding 模型預設輸出的向量長度為 **3072 維**。
* 若向量資料庫（如 Supabase `vector` 欄位）設定的是 **768 維**，未加設定直接寫入會觸發資料庫錯誤：`expected 768 dimensions, not 3072` (Error code `22000`)。
* 解決方式：在呼叫 API 時傳入 `config: { outputDimensionality: 768 }` 即可！

</blockquote>


### 串接 API - 生成向量

1. 安裝 Google Gen AI Official SDK

```shell
npm install @google/genai
```

2. 撰寫向量生成函式 `(lib/embeddings.ts)`

```typescript
import { GoogleGenAI } from "@google/genai";

// 建立 Gemini SDK instance（會自動讀取 process.env.GEMINI_API_KEY）
const ai = new GoogleGenAI(); 
// 若環境變數名稱不同，亦可顯式傳入：const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

export async function generateEmbedding(text: string): Promise<number[]> {
  try {
    const response = await ai.models.embedContent({
      model: "gemini-embedding-001", 
      contents: text,
      config: {
        outputDimensionality: 768, // 👈 關鍵設定：配合向量資料庫縮減至 768 維度
      },
    });

    // 取得產出的向量陣列 (Array of floats)
    const embedding = response.embeddings?.[0]?.values;

    if (!embedding || embedding.length === 0) {
      throw new Error("Gemini API 未回傳有效的向量資料");
    }

    return embedding;
  } catch (error) {
    console.error("Gemini Embedding API 發生錯誤:", error);
    throw new Error("Failed to generate embedding");
  }
}
```

3. 參數與欄位說明：

- `ai.models.embedContent({...})`：Gemini SDK 用於產生文本向量的主要方法。
  - `model: "gemini-embedding-001"` 指定要使用的向量模型名稱。
  - `contents`: 準備要進行向量化的純文字內容（如：「`Interstellar is a 2014 sci-fi film directed by Christopher Nolan...`」）。
  - `config.outputDimensionality`: 自訂向量輸出維度。傳入 `768` 即可讓 Gemini 自動將 `3072` 維度降維至 `768` 維度且不失關鍵語意。
- `response.embeddings[0].values`: API 回傳的向量陣列，型態為 `number[]`（例如：`[-0.012, 0.045, -0.089, ...]`）。


### 整合至 Vector Database (以 Supabase pgvector 為例)  
將生成的向量資料批次寫入 Supabase pgvector 資料庫的範例：

```typescript
// app/api/ingest/route.ts
import { supabase } from "@/lib/supabase";
import { movies } from "@/data/content";
import { generateEmbedding } from "@/lib/embeddings";
import { NextResponse } from "next/server";

// insert data into supabase
export async function GET() {
  try {
    // 1. generate embeddings altogether
    const moviesToInsert = await Promise.all(
      movies.map(async (movie) => {
        const textToEmbed = `Title: ${movie.title} (${movie.releaseYear}). ${movie.content}`;
        const embedding = await generateEmbedding(textToEmbed);
        return {
          title: movie.title,
          release_year: parseInt(movie.releaseYear, 10),
          content: movie.content,
          embedding: embedding,
        };
      }),
    );

    // 2. Batch insert data into Supabase (only sending HTTP Request once)
    const { data, error } = await supabase
      .from("movies")
      .insert(moviesToInsert)
      .select();

    if (error) {
      console.error("Supabase error. Failed to ingest movies:", error);
      throw error;
    }

    return NextResponse.json({
      message: "Successfully ingested movies data into Supabase.",
      ingestedCount: data.length,
      movies: data,
    });
  } catch (error: any) {
    console.error("Ingestion process error:", error);
    return NextResponse.json(
      { error: error.message || "Internal Server Error" },
      { status: 500 },
    );
  }
}

```

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

💡 小提示 (Rate Limit)：  
如果 ingest 的資料量較大（例如超過 20 筆以上），使用 `Promise.all` 一次性發送平行請求可能會觸發 Gemini API 的 Rate Limit (429)。實務上建議使用 分組 (Chunk/Batch) 或 逐筆 (Sequential/For-loop) 的方式處理。

</blockquote>

## 關於純前端（Browser）開發的安全警告  
`@google/genai` 預設適合在 Node.js、Next.js App Router / API Routes 等後端環境中執行。請勿直接在純前端（如 React client component、Vue 或純 HTML 檔案）中使用並hard-coded API Key，這會導致 `GEMINI_API_KEY` 完全暴露在瀏覽器的 Network 頁面中！

實務上部署至線上環境時，正確做法應是建立一條後端 API（如 Next.js API Routes 或 Express）來隱藏 API Key，再由前端發送 `fetch` 請求呼叫該後端。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>