---
title: "[React.js - Next.js] Next.js 的渲染策略（Rendering Strategies）：SSG、ISR 與 SSR - 筆記"
pubDatetime: 2026-08-11T09:22:16.076Z
tags: ["React.js","Next.js","App Router","Frontend","Concepts"]
description: "Table of contents 什麼是渲染策略（Rendering Strategies）？ 前端應用程式的核心任..."
hackmd_id: "HyHDHwu8Gg"
---

## Table of contents

## 什麼是渲染策略（Rendering Strategies）？

前端應用程式的核心任務之一是 **「編譯 HTML 並交付給瀏覽器」**。而渲染策略（Rendering Strategies）就是用來決定這個 HTML **「在何時（When）」** 以及 **「在何地（Where）」** 被編譯出來的機制。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡 **核心觀念**：Next.js 透過彈性的渲染策略，讓我們能根據頁面內容的「即時性需求」與「訪問頻率」，選擇在 **建置時（Build Time）** 或 **請求時（Request Time）** 生成頁面，達到極速載入與資料最新的平衡。

</blockquote>



## 1. 頁面生成的三個關鍵時間點

在理解策略之前，必須先區分 HTML 是在哪個階段被構建出來的：

1. **Build Time（建置時）**：
   在將專案部署上線前執行（例如執行 `npm run build`）。此時會把 TypeScript 轉譯、剔除未使用的程式碼，並將頁面預先渲染（Pre-render）成靜態 HTML。
   * **特點**：構建只發生一次，產出的 HTML 會被快取起來，訪問速度極快。  
2. **Request Time（請求時）**：
   使用者在瀏覽器輸入網址或點擊連結發出 Request 的當下，伺服器收到請求才「動態/即時」生成 HTML 送回瀏覽器。
   * **特點**：每次請求都會重新算一次，能保證拿到的資料絕對是最新的。  
3. **In Browser（瀏覽器端 / Client Side）**：
   傳統 Single Page Application (SPA) 的做法，伺服器只傳回極簡的 HTML 外殼，等 JavaScript 檔案載入完畢後，由瀏覽器在 Client 端構建 DOM。
   * *註： Next.js 預設使用 Server Components，頁面層級主要是在伺服器端完成渲染。*


## 2. Next.js 三大核心渲染策略解析

### 1. Static Site Generation (SSG) - 靜態頁面生成

* **何時渲染**：**Build Time**（僅在執行 `next build` 建置時渲染一次）。
* **何處渲染**：Server（建置環境/打包階段）。
* **運作機制**：在部署時就將頁面的 HTML 預先渲染完成，並寫入快取（Cache）或 CDN。當使用者訪問該頁面時，伺服器直接傳回快取好的 HTML。
* **適用情境**：內容不常變動的頁面，例如：關於我們（About）、隱私權條款、部落格文章。

---

### 2. Incremental Static Regeneration (ISR) - 漸進式靜態再生成

* **何時渲染**：**Build Time** + 設定時間視窗後的 **背景觸發更新**。
* **何處渲染**：Server（由背景 Request 觸發生成）。
* **運作機制**：建置時先生成靜態頁面，並設定一個重新驗證時間（Revalidation time window，如 60 秒）。超過設定時間後，若有使用者發出請求，伺服器會先傳回舊的快取頁面，同時在 **背景（Background）** 重新生成最新的靜態 HTML 並更新快取。
* **適用情境**：資料會變動但不需要秒級即時更新的頁面，例如：商品列表、文章目錄、社群熱門貼文。

---

### 3. Server-Side Rendering (SSR / Dynamic Rendering) - 伺服器端渲染

* **何時渲染**：**Request Time**（每次使用者發送 HTTP 請求時）。
* **何處渲染**：Server（每收到一次請求就即時執行與渲染）。
* **運作機制**：伺服器不使用預先建置好的快取，而是針對每一次進入頁面的 Request，即時抓取最新資料並渲染成 HTML 送回瀏覽器。
* **適用情境**：與使用者個人身份高度相關、或資料必須 100% 即時呈現的頁面，例如：個人儀表板（Dashboard）、購物車、實時股票資訊。



## 3. 三大渲染策略比較

| 評估維度 | SSG (Static Site Generation) | ISR (Incremental Static Regeneration) | SSR (Server-Side Rendering) |  
| :--- | :--- | :--- | :--- |  
| **渲染時間 (When)** | Build Time（打包建置時） | Build Time + 視窗過期後背景更新 | Request Time（每次請求時） |  
| **渲染地點 (Where)** | Server (Build 階段) | Server (背景觸發) | Server (每次 Request 當下) |  
| **資料即時性** | 🔴 低（需重新部署才更新） | 🟡 中（過期後背景更新） | 🟢 極高（永遠是最新資料） |  
| **首字元回應時間 (TTFB)**| 🟢 極快（快取直接回應） | 🟢 極快（靜態快取回應） | 🟡 較慢（需等 Server 計算與 Fetch） |  
| **適合情境** | 內容固定的靜態頁面（About、FAQ）| 內容更新頻率中等的資料頁面（商品頁）| 個人化/需頻繁即時更新的頁面（Dashboard）|



## 4. 開發注意事項（Dev vs Prod）

* **`npm run dev` 的行為差異**：在開發環境（Development）下，為了支援熱重載（HMR），Next.js 通常每次請求都會重新渲染頁面。
* **正式上線（Production）驗證**：若要測試 SSG / ISR 的快取與預先渲染行為，必須執行 `npm run build` 並搭配 `npm start`，才能看到正式打包後的實際運作成果。

## 伺服器端渲染（Server Rendering）的好處
**SSG、ISR 與 SSR 這三種策略的共通點，就是 HTML 的構建與渲染都是在「伺服器端（Server）」或「打包階段（Build Time）」完成的，瀏覽器收到時已經是結構完整的 HTML 網頁。**

- **極佳的 SEO（搜尋引擎最佳化）**  
搜尋引擎爬蟲（如 Googlebot）抓取網頁時，可以直接拿到完整的 HTML 內文與 Meta 標籤，不需要等待 JavaScript 執行完畢才能索引內容。
- **首頁渲染速度快（Faster First Contentful Paint, FCP）**  
使用者開啟網址時，瀏覽器拿到 HTML 就能直接先印出畫面（白屏時間極短），不需要等整包巨大的 JS 檔下載完成並執行後才看到東西，這在行動網路或硬體效能較差的手機上特別明顯。
- **保護敏感資料與金鑰**  
資料庫連線（如 SQL 查詢）、私人 API 金鑰（API Secret Keys）或商業核心邏輯都只在 Server 端執行，完全不會暴露給 Client 端瀏覽器。
- **減少傳送到瀏覽器的 JavaScript 體積（Bundle Size）**  
如果頁面需要使用大型解析套件（例如將 Markdown 轉 HTML 的套件、複雜的資料處理庫），在 Server 端處理完後，這些套件就不需要打包傳給瀏覽器，能有效減輕 Client 端的負載。

## 瀏覽器端渲染（Client-Side Rendering, CSR）的例子  
雖然 Next.js 主打 Server Components，但在實際專案中，許多高度互動、即時或個人化的功能，依然必須在瀏覽器端渲染。

### 範例 1：傳統的 React 單頁應用程式（SPA，如 Vite + React）  
傳統沒有使用 SSR 框架的 React 專案，伺服器丟給瀏覽器的 HTML 只有這一行：

```htmlmixed
<div id="root"></div>
<script src="/main.js"></script>
```

所有的 HTML 標籤（`<h1>`、`<p>`、`<button>`）都是 JavaScript 在瀏覽器（Client 端）下單後才動態構建出來的。

### 範例 2：Next.js 中的 Client Component（使用 `'use client'`）  
在 Next.js 專案中，以下情境都會在瀏覽器端處理：
- 使用者互動與 Event Listeners
  - 點擊按鈕跳出彈窗（Modal / Dialog）
  - 表單輸入驗證、下拉選單開關
  - 深色/淺色模式（Theme）切換開關
- 複雜的前端繪圖與編輯工具
  - Canvas / 3D 繪圖：如 Three.js、Chart.js 圖表渲染。
  - 富文本編輯器（Rich Text Editor）：如 Tiptap、Draft.js，這些套件極度依賴瀏覽器的 `document` 與 `window` 物件。
  - 拖拉上傳與裁切圖片：檔案拖拉區域（Drag and Drop）與前端圖片預覽。
- 強即時性 / 輪詢（Polling）資料抓取
  - 使用 `swr` 或 `react-query` 在瀏覽器端定期 Fetch 的資料，例如：右上角未讀通知數、即時聊天室訊息、即時加密貨幣價格變化。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>