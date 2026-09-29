---
title: "前端框架觀念：React 的特性與限制 - 筆記.md"
pubDatetime: 2026-09-29T07:02:25.161Z
tags: ["Frontend","Concepts","React","React.js","Interview Preparation"]
description: "Tags: Interview Preparation React React.js Concepts Fronten..."
---

###### Tags: `Interview Preparation` `React` `React.js` `Concepts` `Frontend`


## Table of contents

## 認識限制的本質  
在前端技術選擇或面試討論中，了解一項技術的「限制（Limitations）」並非代表該技術不好，而是代表你清楚它的適用邊界與代價。掌握 React 的特性與限制，有助於在面對專案架構設計時作出合適的決策。



## 特性與限制總覽  
React 雖然是目前最熱門的前端工具之一，但開發者必須意識到以下四項主要限制：

* **定位為 Library 而非 Framework**：專注於 UI 渲染，缺乏強制性的專案架構規範。
* **Bundle Size 體積偏大**：完整載入對初次載入速度有一定開銷。
* **由企業（Meta）高度主導**：享有豐富維護資源的同時，也伴隨決策封閉的疑慮。
* **官方文件過時與學習曲線陡峭**：技術演進迅速，初學者容易遇到教材斷層。



## 1. Library vs. Framework  
React 官方將自己定義為「用於建構使用者介面的 JavaScript 函式庫（Library）」，而非完整的框架（Framework）。

### 運作機制與特性
* **專注於 View 層**：React 只負責 MVC 架構中的 View（視圖層）。
* **漸進式引入（Progressive Adoption）**：可以輕鬆將 React 掛載到現有專案的單一 `<div>` 中，極適合漸進式重構（如從傳統 jQuery/Backbone 轉型）。
* **自由度極高**：不限制資料夾結構、路由（Routing）或狀態管理（State Management）方式。

### 限制與影響  
自由度是一把雙刃劍。因為沒有統一規範，大型團隊若沒有制定好 Code Review 規範，容易導致程式碼結構混亂。

### 替代方案與延伸  
如果專案需要完整的框架規範（如檔案路由、SSR 等），可搭配基於 React 的全棧框架，例如 **Next.js** 或 **Gatsby**。


## 2. 包大小與效能（Bundle Size）  
React 包含 Virtual DOM 比對機制與完整的合成事件系統（Synthetic Event System），導致其基本檔案體積相對較大。

### 限制與影響  
對於需要極致優化首頁載入時間（First Load Time）或網路頻寬受限的應用，React 的基本載入開銷可能成為負擔。

### 應對機制
* **程式碼拆分（Code Splitting）**：透過 `React.lazy` 與 Dynamic Import 實現按需載入。
* **輕量化替代方案**：可改用 API 與 React 高度兼容但體積極小的 **Preact**。



## 3. 企業（Meta / Facebook）主導權  
React 是由 Meta（Facebook）內部團隊開發並開源維護的專案。

### 優缺點
* **優勢（資源保障）**：有大型科技公司資助，核心開發團隊為支薪員工，專案不會面臨無人維護或突然中斷的風險。
* **限制（決策封閉性）**：雖然屬於開源社群，但許多重大架構方向（如 React Server Components）依然由內部團隊閉門決定，外部社群的溝通成本與透明度較受限。


## 4. 官方文件與學習曲線  
React 的生態系演進非常迅速（從 Class Components 轉向 Function Components + Hooks）。

### 限制與影響
* **文件過時與混亂**：歷史文件中仍保留大量 Class-based 的教學，導引路線較為分散，容易讓初學者迷失。
* **學習陡峭**：除了學習 React 本身，開發者還需要自行選擇並學習路由、狀態管理、打包工具等周邊生態。



## 四大面向總覽

| 比較面向 | 特性說明 | 產生的限制 / 影響 | 建議應對方案 |  
| :--- | :--- | :--- | :--- |  
| **定位 (Type)** | 僅負責 UI 視圖的 Library | 缺乏目錄結構與全域架構規範 | 引入 Next.js / Remix 等上層框架 |  
| **體積 (Size)** | 包含 VDOM 與 Synthetic Events | Bundle 較大，影響首頁載入 | 實施 Code Splitting 或改用 Preact |  
| **維護 (Vendor)** | 由 Meta 專職團隊資助維護 | 社群溝通成本高，決策主導權在企業 | 關注官方 RFC 討論，評估升級風險 |  
| **生態 (Docs)** | 技術迭代迅速（Hooks/RSC） | 文件混亂，新手學習曲線陡峭 | 透過系統化課程學習，避開舊版 API |



# FAQ

### Q1: React 是 Library 還是 Framework？
**React 是 Library。**

* **理由**：它只解決 View（UI 呈現）的問題，不強制規定 Routing、State Management、API 呼叫或檔案資料夾結構。
* **補充說明**：雖然它是 Library，但我們可以透過結合生態系（如 React Router、Redux）或是使用上層框架（如 Next.js），將其擴展為完整的 Framework 架構。

---

### Q2: 為什麼 React Bundle 偏大？有什麼解決辦法？  
React 需要包含 **Virtual DOM 協調演算法（Reconciler）** 以及 **合成事件（Synthetic Event）** 系統，因此基礎體積較大。

**解決方式：**  
1. 使用 **Code Splitting（懶載入）**，將不同頁面的元件拆分成小包（Chunks）。  
2. 使用 **Tree-shaking** 清理未使用的套件。  
3. 對於效能與體積極度敏感的專案（如嵌入式系統或極簡 Web App），可以直接替換為 **Preact**。

---

### Q3: React 官方文件過時會影響開發嗎？初學者該如何調整學習方向？
**會影響學習效率，但不會影響現代開發實踐。**

* **現狀**：現代 React 開發（React 16.8+）已全面轉向 **Function Components 與 Hooks**，舊有的 Class Components 與生命週期（如 `componentDidMount`）多用於維護舊專案。
* **建議**：學習時應優先掌握 Function Components、`useState`、`useEffect` 與自訂 Hooks（Custom Hooks），忽略舊文件中的 Class 語法，並關注 React 官方推出的新版文件（react.dev）。

### Q4: 為何選擇React而不是Vue或Angular?  
選擇工具時應從**架構控制權**、**漸進式採納**與**生態系資源**三個面向來思考：   
1.  **架構控制權與彈性（Library vs. Framework）**： 
*  **Angular**：為大而全的強規範框架（Batteries-included），開箱即用但學習曲線陡峭、包袱較重。 
*   **Vue**：屬於漸進式框架，上手最快，但官方約束力高於 React。 
*   **React**：作為純粹的 Library，遵循控制反轉（IoC），讓開發者能完全根據專案需求決定資料夾結構與周邊套件（如路由或狀態管理）。   
2.   **漸進式採納（Progressive Adoption）**：
* React 僅專注於 UI 視圖，能以最低成本局部嵌入舊專案（如 jQuery 或舊 MVC 系統）的單一 `<div>` 中進行逐步重構，無須砍掉重練。   
3.   **生態系與跨平台發展**： 
* React 擁有全前端最龐大的社群與第三方套件庫（Shadcn UI, Zustand 等）。 
* 具備良好的跨平台能力（如透過 React Native 延伸至手機 App 開發，或是搭配 Next.js 升級為全端 SSR 應用程式）。

### Q5: 什麼時候建議選擇 Vue 或 Angular？   
技術與工具選擇應根據「團隊成員背景」、「交付時間（Time-to-Market）」與「維護成本」來決定：   
1.  **建議選擇 Vue 的情境**： 
*  **追求極速交付（MVP）**：官方維護完整的路由（Vue Router）與狀態管理（Pinia），減少套件選型時間，能極快產出產品。 
*  **成員背景多元 / 上手時間有限**：HTML/CSS/JS 三者分離的 Template 語法學習曲線平緩，適合包含實習生、Junior 或傳統後端工程師的團隊。 

2.  **建議選擇 Angular 的情境**： 
*  **大型企業級系統（Enterprise App）**：需要維護長達 5~10 年，極度重視程式碼結構的嚴謹度與統一性。 
*  **強大 OOP / 後端背景團隊**：團隊熟悉 C#、Java 或 .NET 物件導向概念，Angular 內建的 TypeScript 強型別、Dependency Injection (DI) 與 RxJS 能無縫對接開發思維。 
*  **降低第三方套件風險**：內建 HTTP Client、表單驗證、路由等所有模組，完全依賴官方維護，適合對資安與相容性極度要求的大型企業。