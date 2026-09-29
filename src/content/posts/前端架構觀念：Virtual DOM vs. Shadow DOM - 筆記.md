---
title: "前端架構觀念：Virtual DOM vs. Shadow DOM - 筆記.md"
pubDatetime: 2026-09-29T04:21:15.831Z
tags: ["Interview Preparation","React.js","React","Concepts","Frontend","Virtual DOM","DOM"]
description: "Tags: Interview Preparation React React.js Concepts Fronten..."
---

###### Tags: `Interview Preparation` `React` `React.js` `Concepts` `Frontend` `Virtual DOM` `DOM`


## Table of contents


## 什麼是 DOM（Document Object Model）？
 在深入比較兩者的差異之前，必須先理解最根本的 **DOM（檔案物件模型）**。

 DOM 是瀏覽器將 HTML 文件解析後所建立的樹狀結構（DOM Tree）。透過 JavaScript 操作 DOM，我們可以動態改變網頁的內容、結構與樣式。

 然而，直接且頻繁地操作真實 DOM 會帶來兩大痛點： 

1.**效能瓶頸**：直接修改 DOM 常觸發瀏覽器的重繪（Repaint）與重排（Reflow），當節點龐大時會導致畫面卡頓。   
2.  **樣式污染與元件耦合**：全域的 CSS 樣式容易互相干擾，無法做到真正的組件隔離（Encapsulation）。 為了解決這兩大不同的痛點，前端社群與 W3C 標準分別推出了 **Virtual DOM** 與 **Shadow DOM**。

## 核心差異總覽  
雖然兩者的名稱都帶有 DOM，但解決的問題與應用的層級完全不同： 
*  **Virtual DOM** 是一種**效能優化機制**，主要用來減少直接操作真實 DOM 的開銷。 
*   **Shadow DOM** 則是一種**封裝機制**，是 Web Components 標準的一部分，用來實現 CSS 與 HTML 結構的隔離。

## Virtual DOM（虛擬 DOM）  
Virtual DOM（VDOM）是用 JavaScript 物件在記憶體中模擬真實 DOM 的技術。它是 **應用層（框架層）** 的解決方案，並非瀏覽器原生的功能。 

### 運作機制（Reconciliation）  
1.  **渲染**：當元件的資料（State）發生變更時，框架會在記憶體中建立一棵全新的 Virtual DOM 樹。   
2.   **比對（Diffing）**：透過 **Diff 演算法**，高效地比較「新舊兩棵 Virtual DOM 樹」的差異。   
3.   **更新（Patching）**：將計算出來的最小差異，以「批次」的方式一次性更新到真實 DOM 上。 

### 優點
*  **提升渲染效能**：將多次連續的 DOM 操作合併，避免不必要的重排與重繪。 
*   **跨平台能力（Cross-platform）**：因為 Virtual DOM 只是純粹的 JS 物件，它可以被渲染至瀏覽器以外的地方（例如 React Native 渲染成手機原生 UI）。 
### 代表技術  
React、Vue 等現代前端框架。

## Shadow DOM（影子 DOM）  
Shadow DOM 是瀏覽器**原生支援**的技術，屬於 W3C 標準中 **Web Components** 規範的重要組成部分。它允許將一棵「隱藏的 DOM 樹」附加到傳統的 DOM 節點上。 
### 運作機制（Scope Isolation）  
Shadow DOM 建立了一個獨立的 DOM 作用域（Shadow Tree），這個作用域與主文件（Light DOM）完全隔離： 
*  **CSS 隔離（Style Scoping）**：在 Shadow DOM 內部撰寫的 CSS 樣式不會洩漏到外部，外部的全域 CSS 也無法直接影響內部。
*    **DOM 隔離**：主文件的 `document.querySelector` 預設無法存取或選取到 Shadow DOM 內部的元素節點。 
### 優點
 *  **完美的樣式封裝**：開發者在打造大型 UI 組件庫時，不必再擔心 class 名稱衝突或全域樣式污染。 
 *   **原生 UI 的隱藏實作**：瀏覽器原生的 `<video>` 控制列、`<input type="date">` 或 `<input type="range">` 滑桿，內部都是透過 Shadow DOM 封裝了複雜的 HTML 與 CSS。 
 ### 代表技術
 Web Components、Lit、Stencil，以及瀏覽器原生 HTML 標籤。

## 兩者對照與適用場景

| 比較維度 | Virtual DOM | Shadow DOM |   
| :--- | :--- | :--- |  
| **主要定位** | 效能優化 (Performance) | 樣式與結構封裝 (Encapsulation) |   
| **實現層級** | 應用層（由 React/Vue 等框架實現） | 瀏覽器原生層（W3C Web Components 標準） | | **解決痛點** | 避免頻繁操作真實 DOM 導致畫面重繪與卡頓 | 避免 CSS 樣式衝突與元件間的互相干擾 |   
| **核心技術** | JS 物件、Diff 演算法、Reconciliation | Shadow Tree、Scoped CSS、Web Components |   
| **適用場景** | 需要頻繁更新 UI 資料的複雜 SPA 應用 | 需要發布到跨框架專案、樣式需完全獨立的元件庫 | 

可以把 **Virtual DOM** 想成是「畫面的緩衝暫存區」，幫我們算怎樣改畫面最划算；而 **Shadow DOM** 則像是「元件的防護罩」，確保裡面的樣式與結構不會被外面的程式碼干擾。兩者並非競爭關係，甚至可以在同一套專案中同時並存。


# FAQ

### Q1: Virtual DOM 一定比直接操作真實 DOM 還要快嗎？
**不一定。** Virtual DOM 的優勢在於「當應用程式規模大、資料變化複雜時，保證提供『足夠好（Reasonably Fast）』的效能與開發體驗」，而非在所有情境下絕對最快。 
*  **簡單/小型操作**：直接使用原生 JS 操作 DOM（例如 `element.textContent = 'hello'`）絕對比 Virtual DOM 快，因為 Virtual DOM 需要額外的記憶體開銷與 Diff 運算成本。 
*  **複雜/高頻更新**：當畫面有大量動態變更時，Virtual DOM 能透過 Diff 計算出最小差異並「批次更新」，避免頻繁觸發 Layout/Reflow，此時效能就會超越未經優化的原生操作。 

### Q2: 在 React 列表渲染中，為什麼 key 不能用 index？  
`key` 是 React 在進行 Virtual DOM Diff 比對時用來識別節點身份的唯一標籤。 
*  **使用 index 的問題**：當陣列順序發生變更（如新增、刪除、排序）時，index 會重新編號。這會導致 React 誤以為舊節點被複用，引發不必要的重新渲染（Re-render）。 
*  **最嚴重的 Bug（State 錯亂）**：若列表包含 Uncontrolled Component（如 `<input>`），DOM 節點內部原生的輸入狀態不會被銷毀，會導致文字留在原本的位置，與變更後的資料不匹配。 
*  **解決方式**：使用資料中唯一且穩定的 ID 作為 `key`（例如 `item.id`）。 

### Q3: Shadow DOM 裡的樣式完全無法被外部修改嗎？  
預設情況下，Shadow DOM 外部的 CSS 無法覆蓋內部的樣式，達到完美的樣式隔離。但 Web Components 規範提供了**受控的自訂介面**：   
1.  **CSS Custom Properties（CSS 變數）**：內部樣式可以使用 `var(--main-color)`，外部則可以透過宣告 `--main-color: red;` 來穿透傳遞變數值。   
2.   **`::part()` 虛擬元素**：如果組件內部將特定節點標記為 `part="button"`，外部 CSS 就可以透過 `my-element::part(button)` 直接設定該節點的樣式。 

### Q4: React / Vue 的 Component 也有 CSS 隔離功能，這和 Shadow DOM 的隔離有何不同？
####  **React/Vue 的隔離（例如 CSS Modules、Scoped CSS）**： 
*   **實現方式**：屬於「編譯期」或「應用層」的模擬。打包工具（如 Webpack/Vite）會在編譯時自動為 CSS 加上雜湊值（Hash，如 `.btn[data-v-123]`）。 
*   **本質**：本質上依然是全域 CSS，只是透過 class 名稱唯一化來避免衝突。 *

#### **Shadow DOM 的隔離**：
*    **實現方式**：屬於「瀏覽器原生層」的真隔離。 
*   **本質**：內部 styles 獨立於主文件，即使外部寫了 `button { background: red !important; }` 也完全無法影響 Shadow DOM 內部的 `button`。