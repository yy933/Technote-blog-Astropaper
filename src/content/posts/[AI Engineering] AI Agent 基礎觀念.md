---
title: "[AI Engineering] AI Agent 基礎觀念"
pubDatetime: 2026-09-01T08:42:23.913Z
tags: ["AI Engineering","Concepts","AI Agent","LLM","Prompt Engineering"]
description: "Table of contents Prompt Engineering Prompt Engineering 是讓..."
hackmd_id: "rJTPnZV_Gx"
---

## Table of contents

## Prompt Engineering
>Prompt Engineering 是讓 AI 依照你的意願行事的藝術 (The art of getting AI to do what you want)。

比起嚴謹的硬科學，Prompt Engineering更偏向與 AI 溝通的「軟實力 (Soft skills)」。如何精準表達需求，將直接決定 AI 回應的品質。

`需求構想 (User Intent) ───[ Prompt Engineering ]───> 精準回應 (Optimal Output)`

## 高效率 Prompt 的四大核心原則
### 1. 極盡具體 (Be Specific)  
過去使用搜尋引擎時，我們被訓練成將問題簡化為關鍵字。

但大型語言模型 (LLM) 擁有極強的上下文理解能力，提供資訊越具體，模型表現越好。使用 LLM 需要打破舊有的關鍵字搜尋習慣。

### 2. 使用專業術語 (Use Technical Terms)  
在提示詞中直接使用領域內的專有名詞（如 CSS、Flexbox、Div），能大幅提升提示詞的專一度與回應品質。

### 3. 提供額外上下文與範例 (Provide Context & Examples)  
附帶背景資訊、限制條件或現有的程式碼範例，能幫助模型精準縮小預測範圍，產出可直接應用的結果。

### 4. 持續迭代優化 (Iterate)  
第一次提問未必能得到完美的答案，根據模型的輸出結果持續調整與優化 Prompt 是必然的開發過程。


## Prompt 優化案例示範：以網頁元件置中為例  
透過逐步增加具體度與技術術語，觀察模型回應品質的差異：

| 提示詞等級 | 輸入的 Prompt 範例 | LLM 回應品質與問題分析 |  
| :--- | :--- | :--- |  
| **較差 (Poor)** | *"How do I center something?"* | **缺乏上下文**：模型無法判斷情境，只能試圖涵蓋所有可能（包含 Photoshop 圖片置中、Word 內文置中、HTML/CSS 等），無法給出針對性解答。 |  
| **尚可 (Better)** | *"How do I center an element on a web page?"* | **縮減範圍**：排除無關領域（如 Photoshop），提供 Flexbox、Margin 等網頁置中方法，但仍不夠精確。 |  
| **優秀 (Much Better)** | *"How do I center a div element horizontally and vertically using CSS Flexbox?"* | **高度聚焦**：精準包含需求（水平/垂直）、對象（div）與技術（Flexbox），AI 能直接給出深度程式碼與詳細解釋。 |  
| **極致 (Best/Ultimate)** | *"這是我的 HTML...請用上述 Class，透過 Flexbox 將此 div 水平垂直置中"* | **提供範例與語境**：附上現有程式碼上下文（Context），AI 回傳的結果通常可直接複製貼上至專案中使用。 |

## 控制 LLM 的輸出長度與格式  
除了提升具體度外，精準控制模型的 **長度 (Length) 與 格式 (Format)** 也是提示工程的核心技巧：

### 控制輸出長度 (Controlling Length)
- 限制篇幅：直接指定字數或段落（例如：Explain in two paragraphs...），避免回應冗長。

- 擴充範例：要求列出指定數量的例子（例如：Give me 20 examples of...）。
  - 注意事項：要求過多範例時，模型可能會開始出現較天馬行空、偏離常態的極端範例（Out there suggestions），適合用於廣泛發想，但需人工篩選。

### 指定輸出格式與風格 (Controlling Format & Style)
- **表格化比較 (Tabular Format)**：要求以表格呈現（如：Compare X and Y in a tabular format），方便進行 Side-by-side 的直觀對照。

- **類比說明 (Analogy)**：要求使用比喻解釋概念（如：Explain using an analogy），非常適合教學或理解抽象概念。

- **自動註解程式碼 (Annotate Code)**：貼上程式碼並要求在每行加上註解說明（如：Add a comment over each line explaining what it does），可作為自動化編寫程式文件的利器。

## 廣義 AI 視角  
在進入實作之前，我們需要拉廣視角 (Zoom out)：**大型語言模型 (LLM) 只是龐大 AI 工具箱中的其中一項工具。**

```
+-------------------------------------------------------------+
| 人工智慧 (Artificial Intelligence, AI)                       |
|                                                             |
|   +-----------------------------------------------------+   |
|   | AI Agents (AI 代理系統)                              |   |
|   |                                                     |   |
|   |  可調用的工具庫 (Tools):                              |   |
|   |  - LLMs (GPT-4, Mistral, Llama 2)                   |   |
|   |  - 語音模型 (Text-to-Speech / Speech-to-Text)        |   |
|   |  - 電腦視覺 (Computer Vision)                        |   |
|   |  - 圖像生成 (DALL-E 3, Midjourney)                   |   | 
|   |  - 感測器數據處理器 (Sensor Processors)               |   |
|   +-----------------------------------------------------+   |
+-------------------------------------------------------------+
```

### 什麼是 AI Agent？  
定義： **AI Agent 的本質是整合多種工具 (Tools)，並透過「迴圈迭代 (Iteration)」持續執行任務，直到目標完成為止。**

### AI Agent 的複雜度光譜
- **簡單級別 (Simple Agent)**：利用 LLM 作為「推理引擎 (Reasoning tool)」，配合程式碼迴圈 (Loop) 解決特定單一任務（例如：檢索即時資訊並摘要）。

- **複雜級別 (Complex Agent)**：如自動駕駛汽車 (Autonomous Vehicle)。

  - 最終目標：安全地從 A 點抵達 B 點。
  - 運作機制：將任務拆解為數百萬個微小決策，同時即時整合鏡頭、Lidar 雷達等感測器數據，動態調整並推進至目標。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>