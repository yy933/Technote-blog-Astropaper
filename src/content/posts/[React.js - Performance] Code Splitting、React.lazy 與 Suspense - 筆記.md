---
title: "[React.js - Performance] Code Splitting、React.lazy 與 Suspense - 筆記"
pubDatetime: 2026-07-14T07:18:25.271Z
tags: ["React.js","Performance","Frontend"]
description: "Table of contents Code Splitting 核心觀念 在現代大型單頁應用程式（SPA）中，打包工..."
hackmd_id: "H1xAT8XEfe"
---

## Table of contents

## Code Splitting 核心觀念  
在現代大型單頁應用程式（SPA）中，打包工具（Vite / Webpack）預設會將所有程式碼打包成單一的巨大 `bundle.js`，導致首頁載入過慢。

**Code Splitting（程式碼拆分）** 的核心思想是：**「將單一巨大產物拆分為多個非同步載入的小積木（Chunks）。當下用不到的元件先不下載，等使用者觸發特定條件時，才經由網路動態載入。」**



## 1. 核心三要素：Dynamic Import、lazy、Suspense

### 1. Dynamic Import & `React.lazy`  
React 提供用來動態載入組件的內建函式。它接受一個回傳 `import()` Promise 的函式。

```javascript
// 傳統打包：不管用不用得到，一律載入
// import ProductsList from "./ProductsList"

// 載入優化：拆分獨立檔案，觸發渲染時才動態下載
const ProductsList = React.lazy(() => import("./ProductsList"))
```

### 2. `React.Suspense`  
由於動態下載組件需要網路傳輸時間，**在檔案尚未抵達的「空窗期」，Suspense 容器可以捕獲這個等待狀態，並藉由 fallback 屬性渲染出替代的 Loading 畫面**。

```javascript
<React.Suspense fallback="{<h2">Loading...</h2>}>
  <div className="products-list">
    {showProducts && <ProductsList/>}
  </div>
</React.Suspense>
```

## 2. 效能優化指標好處 (Web Vitals)
- **降低 Total Blocking Time (TBT) 與 Time to Interactive (TTI)**：  
瀏覽器不用在首頁加載時耗費大量 CPU 資源去解析（Parse）與執行（Execute）那些還沒用到的深層組件，主執行緒（Main Thread）得以保持空閒，網頁能更流暢地回應使用者的點擊輸入。
- **優化 First Contentful Paint (FCP)**：  
Bundle 體積變小意味著網路下載變快，核心 UI 結構能以最快速度呈現在使用者眼前。

## 常見觀念問答
**Q: 實務上什麼時候該做 Code Splitting？每個元件都套 React.lazy 是好操作嗎？**

A: 絕對不是！過度拆分反而會造成反效果。  
因為每次呼叫 React.lazy 都代表瀏覽器要多發送一次 HTTP 請求，如果把太小的元件也拆分，會導致網路請求過於碎片化，且頻繁觸發 Suspense 的 Loading 閃爍，使用者體驗反而會變差。

實務上最推薦的兩大拆分時機：
**1. Route-based：**  
這是最標準的做法。不同的頁面（如：`/admin` 後台管理頁、`/profile` 個人頁）拆成獨立 Chunk。使**用者拜訪首頁時，絕對不需要下載後台管理介面的程式碼。**

**2. Interaction-based:**  
某些Modal、圖表（如 ECharts、G6）、或是隱藏在點擊按鈕底下的區塊，非常適合使用 `React.lazy` 延遲載入。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>