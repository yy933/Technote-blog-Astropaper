---
title: "[React.js] 掌握 React 中的資料流：Props 與 State 核心概念 - 筆記"
pubDatetime: 2026-06-18T09:11:47.556Z
tags: ["React.js","Frontend","JavaScript","React Hook"]
description: "Table of contents :memo: 簡介 在開發 React 應用程式時，網頁畫面的呈現與互動完全仰賴於..."
hackmd_id: "HJoGM4bfGl"
---

## Table of contents

## :memo: 簡介  
在開發 React 應用程式時，網頁畫面的呈現與互動完全仰賴於資料的流動。而控制元件（Component）資料流與畫面渲染的兩大核心概念，就是 **Props** 與 **State**。

簡單來說，它們就像是元件的 **「外在輸入」** 與 **「內在記憶」**。網頁畫面的更新（UI Render）就是透過這兩者資料的改變來觸發。


## :memo: 核心概念
### 1. 什麼是 Props (屬性)？
**Props（Properties 的縮寫）** 是從**外部（通常是父元件）傳遞進元件的資料**。它們就像是 HTML 標籤上的屬性，或者是呼叫函式時所傳入的「參數」。**透過帶入不同的 props，我們可以讓同一個元件呈現不同的外觀或行為，大幅提升元件的複用性（Reusability）**。

* **唯讀性（Read-Only / Immutable）：元件絕對不能、也無法修改自己接收到的 props。** React 嚴格遵守「單向資料流」，資料只能從上游（父）往下游（子）傳遞，子元件只能被動接收。
* **觸發重新渲染（Re-render）：** 當父元件傳進來的 props 改變時，子元件會自動被重新呼叫與渲染，以呈現最新的資料。

### 2. 什麼是 State (狀態)？
**State** 是元件**內部自己管理的私有資料（狀態）**。它用來記錄元件隨時間過去，或是因為使用者互動（例如：點擊按鈕、輸入表單、API 非同步請求回傳）而產生的內部數據變更。

* **可變性（Mutable，但須透過特定方法）：** 元件可以自由修改自己的 state，但**絕對不能直接進行修改**（例如 `state.count = 1` 是無效的寫法），必須使用 React 提供的更新控制函式（如 Function Component Hook 中的 `useState` 所提供的 `set` 函式）。
* **元件私有（封閉性）：** State 僅存在於宣告它的元件內部，外部的其他元件或父元件是無法直接存取或修改它的。
* **觸發重新渲染：** 當元件的 state 被成功更新時，該元件以及它底下的所有子元件都會被排入重新渲染的排程中。

## :memo: 觀念對比 (Props vs State)

為了更容易區分這兩個概念，以下將兩者的特性整理成對比表格：

| 特性 | Props (屬性) | State (狀態) |  
| :--- | :--- | :--- |  
| **資料來源** | 由父元件（外部）傳入 | 元件內部自行建立與管理 |  
| **能在元件內修改嗎** | ❌ 不行（唯讀） | ⭕ 可以（但必須透過相對應的設定函式） |  
| **主要用途** | 接收外部設定、提高元件的可複用性 | 紀錄與追蹤元件當前的互動狀態與資料變更 |  
| **是否觸發重新渲染** | ⭕ 是（當傳入的 Props 改變時） | ⭕ 是（當 State 經由專用函式被更新時） |  
| **生活化比喻** | 像是一台車的 **外殼顏色、型號**（出廠時由工廠決定，買家拿來使用） | 像是一台車的 **時速、剩餘油量**（在行駛與互動過程中會不斷動態變化） |



## :memo: 實作分享

這裡我們用一個簡單的「計數器（Counter）」專案，來示範父元件如何透過 **State** 管理狀態，並將其作為 **Props** 傳遞給子元件。

### 1. **建立 React 專案環境**
 這裡我們可以使用 Vite 來快速建立一個 React 專案：
 
 ```bash
 npm create vite@latest react-props-state-demo -- --template react
 cd react-props-state-demo
 npm install
 npm run dev
   ```

### 2. 專案元件架構  
專案的核心邏輯與架構非常精簡：

```
src
├─ App.jsx            (父元件：管理 State 並傳遞 Props)
└─ CounterDisplay.jsx (子元件：接收 Props 並負責畫面顯示)
```
### 3. 實作負責顯示畫面的子元件 (接收 Props)  
建立 `CounterDisplay.jsx`，此元件不擁有任何狀態，純粹依賴外部傳進來的 `props` 來顯示內容。

```jsx
// CounterDisplay.jsx
import React from 'react';

function CounterDisplay(props) {
  // props.title 和 props.count 都是由外部決定的，此元件內部唯讀
  return (
    <div style={{ border: '1px solid #ccc', padding: '15px', borderRadius: '8px' }}>
      <h2>{props.title}</h2>
      <p style={{ fontSize: '20px' }}>目前數字：<strong>{props.count}</strong></p>
    </div>
  );
}

export default CounterDisplay;
```


### 4. 實作負責管理狀態的父元件 (管理 State)  
在 `App.jsx` 中，我們使用 React Hook `useState` 來宣告計數器的核心資料，並將其傳遞給子元件。

```jsx
// App.jsx
import React, { useState } from 'react';
import CounterDisplay from './CounterDisplay';

function App() {
  // 1. 定義一個內部 state 叫做 count，預設值為 0
  const [count, setCount] = useState(0); 

  // 2. 定義點擊按鈕後的更新邏輯
  const handleIncrement = () => {
    setCount(count + 1); // 透過專用函式修改 state，React 會自動重新渲染畫面
  };

  return (
    <div style={{ padding: '20px', fontFamily: 'Arial, sans-serif' }}>
      <h1>React 狀態與屬性實作示範</h1>

      {/* 3. 將 title 字串和 count 狀態作為 props 傳遞給子元件 */}
      <CounterDisplay title="我的計數器元件" count={count} />

      <br />
      {/* 點擊按鈕觸發狀態變更 */}
      <button 
        onClick={handleIncrement}
        style={{ padding: '10px 20px', fontSize: '16px', cursor: 'pointer' }}
      >
        數字加 1
      </button>
    </div>
  );
}

export default App;
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

:pushpin: Points：
* 在 `App.jsx` 中，當使用者點擊按鈕呼叫 `setCount(count + 1)` 時，App 元件的 state 發生改變，觸發 App 重新渲染。
* 當 App 重新渲染時，傳遞給 `CounterDisplay` 的 `props.count` 也隨之改變，於是 `CounterDisplay` 也跟著重新渲染，使用者便能立即在畫面上看到最新的數字。

</blockquote>


## 小結  
簡單總結 Props 與 State 的關係：

* Props 是外部塞給元件的設定，元件自己不能改。
* State 是元件內部的私人記憶，元件可以自己控制。

在設計 React 元件時，**一個良好的習慣是盡可能讓元件保持「無狀態（Stateless）」，多利用 Props 來負責渲染畫面**；**而將「有狀態（Stateful）」的資料邏輯集中在少數的父元件中管理。這樣不僅能維持資料的單向清晰流動，也能大幅提升程式碼的可維護性**！

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>