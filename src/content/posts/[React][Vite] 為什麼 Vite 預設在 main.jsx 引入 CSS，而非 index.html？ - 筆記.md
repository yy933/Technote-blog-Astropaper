---
title: "[React][Vite] 為什麼 Vite 預設在 main.jsx 引入 CSS，而非 index.html？ - 筆記"
pubDatetime: 2026-06-11T10:07:38.065Z
tags: ["React.js","Vite","Frontend","CSS","Bundler","Performance","Issue"]
description: "Table of contents :memo: 問題 使用 Vite 新開啟一個 React 專案時，注意到預設的..."
hackmd_id: "BklfrbObGe"
---

## Table of contents

## :memo: 問題
使用 Vite 新開啟一個 React 專案時，注意到預設的 CSS 樣式是在 `main.jsx` 中透過 `import './index.css'` 引入，而非傳統網頁開發中，在 `index.html` 的 `<head>` 中使用 `<link rel="stylesheet" href="/index.css" />`。
這樣做有什麼好處？真的能讓速度更快、減少檔案大小或載入時間嗎？


## :memo: 核心原因
在 JavaScript/JSX 中 `import` CSS，本質上**並不單純為了「讓原始檔案變小」**，而是現代前端建構工具為了解決大型專案的**開發體驗（DX, Developer Experience）**、**模組化管理**以及**生產環境的自動化優化**所演進出的最佳實踐（Best Practice）。

## :memo: 好處與優勢

### 1. 模組化管理 (Scoping & Bundling)
傳統做法將 CSS 寫在 `index.html` 中，意味著所有樣式都是全域（Global）的。當專案規模擴大，容易發生樣式衝突、覆蓋（Specificity 戰爭）或「**改了 A 元件卻意外弄壞 B 元件**」的慘劇。

#### 透過 Vite 在 JS 中引入 CSS：
* **元件與樣式綁定：** 可以實踐「一個元件資料夾，同時包含 `Button.jsx` 與 `Button.css`」的專案結構。在 `Button.jsx` 內部 `import './Button.css'`，讓樣式生命週期與元件同步，搬移或刪除元件時極其直覺。

* **原生支援 CSS Modules：** Vite 內建支援 CSS Modules（例如將檔名改為 `Button.module.css` 並透過 `import styles from './Button.module.css'` 引入）。Vite 會自動為類名加上獨一無二的雜湊值（Hash），**從根本上徹底解決全域命名衝突問題**。

### 2. 良好的開發體驗：HMR - Hot Module Replacement
這是開發時最感同身受的優勢：
* **傳統 `<link>` 做法：** 修改 CSS 後，瀏覽器通常需要重新整理整個頁面（Full Refresh），這會導致開發到一半的狀態（如：打開到一半的 Modal、輸入到一半的表單資料）全部消失。

* **Vite 的 HMR 做法：** 因為 CSS 進入了 Vite 的Dependency Graph，當修改 `index.css` 時，Vite 的 HMR 引擎可以**只把修改後的樣式動態注入到瀏覽器中，網頁完全不需重新整理**，所有元件狀態依然完好保留，存檔瞬間畫面即同步更新。

### 3. 生產環境的極致優化 (Production Optimization)
在執行 `npm run build` 打包專案時，Vite（背後使用 Rollup）會對這些 `import` 的 CSS 進行多項自動化處理：

* **程式碼分割 (Code Splitting) 與按照需求載入：** 如果 React 專案很大並採用了動態載入（Lazy Loading / `React.lazy`），Vite 會自動將該路由或元件專屬的 CSS 拆分成獨立小檔案。**只有當使用者瀏覽到該頁面時，才會非同步下載該頁面的 CSS**。
若寫在 `index.html`，則不論使用者看哪一頁，都必須在首頁第一時間下載全站完整的 CSS。

* **快取雜湊值優化 (Cache Busting)：**
  Vite 打包時會自動壓縮 CSS，並在副檔名前加上內容雜湊值（例如 `index-d8a3f4b.css`）。**若內容未變動，瀏覽器便會直接讀取強快取；若樣式有修改，產出的雜湊值就會改變，強制瀏覽器抓取最新版本，免去手動管理版本號（Query String）的麻煩**。



## Recap


| 特性 | 在 `main.jsx` 中 `import` (Vite 做法) | 在 `index.html` 中 `<link>` (傳統做法) |
| :--- | :--- | :--- |
| **開發時熱更新 (HMR)** | **極快**，只更新樣式，不刷新頁面、不丟失狀態 | 較慢，通常需要整頁刷新 |
| **檔案與樣式組織** | **模組化**，CSS 可緊跟 React 元件或啟用 CSS Modules | 全域管理，專案規模大時極易混亂、衝突 |
| **效能優化 (打包後)** | **自動化優化**：自動壓縮、按需載入（Code Splitting）、自動加雜湊快取 | 需手動配置工具處理，否則只能一次性載入全域樣式 |


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>