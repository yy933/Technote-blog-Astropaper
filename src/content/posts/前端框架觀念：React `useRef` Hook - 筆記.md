---
title: "前端框架觀念：React `useRef` Hook - 筆記.md"
pubDatetime: 2026-09-29T04:23:24.137Z
tags: ["Concepts","React.js","React","Interview Preparation","React Hook","Frontend"]
description: "Tags: Interview Preparation React React.js Concepts React H..."
---

###### Tags: `Interview Preparation` `React` `React.js` `Concepts` `React Hook` `Frontend`


## Table of contents

## 前言  
在 React 中，資料與 UI 的同步主要依賴 State 驅動。然而，當我們需要**跨渲染保存資料但不希望觸發 Re-render**，或是需要**直接操作真實 DOM 元素**時，就會使用到 **`useRef`** Hook。官方文件特別提醒「**切勿過度使用 Ref**」，但在特定的情境（如焦點控制、整合第三方 DOM 函式庫）下，它是不可或缺的工具。



## 核心特性與觀念總覽

* **跨渲染保存資料**：Ref 物件的值在元件多次 Re-render 之間會被保留（Persist）。
* **不觸發重新渲染**：修改 `ref.current` 的值**不會**像修改 State 一樣觸發元件重新渲染。
* **主要用途**：聚焦（Focus）、媒體播放控制、觸發命令式動畫，以及作為第三方非 React DOM 函式庫的附著點（Attach Point）。
* **更新時機**：若要在渲染後更新 Ref 或操作其對應的 DOM，通常配合 **`useEffect`** 執行。



## 1. State 與 Ref 的差異比較


| 特性 / 項目 | State (`useState`) | Ref (`useRef`) |  
| :--- | :--- | :--- |  
| **數值更新時** | ➔ **會觸發** 元件重新渲染（Re-render） | ➔ **不會觸發** 元件重新渲染 |  
| **資料持久性** | 可跨渲染保存資料 | 可跨渲染保存資料 |  
| **存取與修改方式** | 透過 Setter 函式（如 `setState(newValue)`） | 直接讀取或修改 `ref.current` 屬性 |  
| **核心定位** | 驅動 UI 視覺變化的 **宣告式（Declarative）** 狀態 | 保存不影響 UI 渲染的資料，或進行 **命令式（Imperative）** 操作 |



## 2. 何時該使用 Ref？（應用情境）

React 官方強調「不要濫用 Ref」，但在以下情境中，使用 Ref 是最佳做法：

### 1. 管理焦點（Focus）或媒體播放（Media Playback）
* **最常見（99% 的 Ref 用途）**：當開啟 Modal（彈窗）時，利用 Ref 自動將焦點（Focus）對焦至輸入框（`<input>`）。
* **範例**：控制影片或音訊的 `play()` / `pause()` 方法。

### 2. 觸發命令式動畫（Imperative Animations）
* 當動畫不需要或不適合由 State 狀態驅動，而是需要透過 JavaScript 進行命令式控制時。

### 3. 整合第三方 DOM 函式庫（Third-party DOM Libraries）
* 因為 React 使用 **虛擬 DOM（Virtual DOM）**，傳統 JS 函式庫（如 D3.js、Chart.js 或特定 jQuery 外掛）無法直接抓取元素。
* 透過 Ref 提供真實 DOM 的引用，讓第三方函式庫有可以「掛載（Attach）」的實體 DOM 節點。



## 3. 在 Function Component 中正確更新 Ref 的時機

若需要在元件掛載或更新後更新 Ref 或存取其 DOM 節點，**正確做法是放在 `useEffect` 內部處理**。

### 程式碼範例：開啟 Modal 自動對焦輸入框

```javascript
import { useEffect, useRef } from 'react';

function SearchModal() {
  // 1. 建立 Ref 物件
  const inputRef = useRef(null);

  useEffect(() => {
    // 3. 在 Effect 中（DOM 掛載完成後）存取並更新 DOM
    if (inputRef.current) {
      inputRef.current.focus(); // 讓輸入框自動取得焦點
    }
  }, []);

  // 2. 將 Ref 綁定到 JSX DOM 元素上
  return (
    <div className="modal">
      <input ref={inputRef} type="text" placeholder="搜尋..." />
    </div>
  );
}
```

# FAQ

### Q1: `useRef`  與一般在元件外部宣告的全域變數有什麼差別？

-   **全域變數**：會在該元件的所有實體（Instances）之間**共享**，如果同一個頁面渲染了兩次該元件，它們會互相干擾與覆蓋資料。
    
-   **`useRef`**：每一個元件實體都有自己**獨立**的 Ref 物件，不會跨實體共享，且能在元件生命週期內持續存在。
    

### Q2: 為什麼修改  `ref.current`  不會觸發 Re-render？

因為  `useRef`  回傳的是一個單純的 JavaScript 物件，結構如  `{ current: value }`。React 並沒有為  `current`  屬性設定底層的監聽機制（Setter），因此改動其值只是單純修改記憶體中的物件屬性，不會通知 React 執行 Fiber 節點的更新與重新渲染。