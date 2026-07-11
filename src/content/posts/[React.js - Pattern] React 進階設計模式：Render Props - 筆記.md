---
title: "[React.js - Pattern] React 進階設計模式：Render Props - 筆記"
pubDatetime: 2026-07-11T08:40:10.607Z
tags: ["React.js","Design Pattern","Compound Components","Advanced React","Frontend","Context API"]
description: "Table of contents Render Props 核心觀念 Render Props 是 React 中一..."
hackmd_id: "HknnqdyEGg"
---

## Table of contents

## Render Props 核心觀念
**Render Props** 是 React 中一種強大的程式碼複用（Code Reuse）設計模式。  
它的核心思想是：**「元件自己不決定畫面（UI）長什麼樣子，而是接受一個『會回傳 JSX 的函式』作為屬性（Prop），並在內部呼叫它、把資料傳出去。」**

簡單來說， **元件本身只負責管理狀態邏輯的處理，不負責決定畫面；畫面要如何渲染，則交由使用該元件的開發者自行決定**。

一種常見的 Render Props 寫法，是直接把 children 當成函式來使用，這種模式通常稱為 **Function as Children**（FaCC） 或 **Children as a Function**。 不透過自訂的 Prop 名稱，而是直接把夾在元件中間的 `children` 當作函式來執行。



## 1. 為什麼我們需要 Render Props？

在開發複雜的 UI 組件（如：Toggle 開關、Modal 彈窗、Dropdown 下拉選單）時，我們常常會遇到一個情況：**邏輯完全一樣，但每個專案/頁面想要的畫面長得完全不同。**

### 對比一：傳統 Compound Components 的極限  
傳統的複合元件（Compound Components）雖然優雅，但有時會造成程式碼冗餘：

```javascript
<Toggle.On>
  <div className="box filled">已開啟</div>
</Toggle.On>
<Toggle.Off>
  <div className="box">已關閉</div>
</Toggle.Off>
```

在某些需要依據同一份狀態切換 UI 的情境中，使用 Compound Components 可能需要為不同狀態撰寫兩份相似的 JSX，因此會產生一些重複程式碼。

### 對比二：Render Props 的終極解法  
透過 Render Props，子元件只負責把狀態吐出來，父元件只需要寫一次 HTML，再利用拿到的變數做動態切換：

```javascript
// 優點：結構只需寫一次，利用參數動態調整 class 與文字
<Toggle.Display>
  {(on) => (
    <div className={`box ${on ? "filled" : ""}`}>
      {on ? "已開啟" : "已關閉"}
    </div>
  )}
</Toggle.Display>
```

## 2. 資料傳遞與核心機制  
Render Props 利用了 JavaScript 中函式是First-class Function的特性。父元件先把一個函式交給子元件，子元件在適當時機執行這個函式，並把自己的資料當作參數傳入，因此父元件便能依據這些資料決定要渲染什麼畫面。

### Render Props 運作三大步驟：  
1. 父元件提供一個會回傳 JSX 的函式（Render Function）：在 `App.jsx` 中寫下一個箭頭函式，這個函式默默等待著 `on` 參數被帶入。  
1. `ToggleDisplay` 從 Context 取得目前的狀態：透過 `useContext` 拿到最外層的 `on` 狀態。  
1. 執行並渲染：`ToggleDisplay` 執行 `children(on)`，將狀態傳入 Render Function，取得 JSX 並回傳。。

```
[最外層 Toggle] ──儲存狀態 (on: true)
      │
      ▼ (透過 Context API 廣播)
[中間層 ToggleDisplay] ──經由 useContext 拿到 (on: true)
      │
      ▼ 關鍵一擊：執行 children(true)
[最內層 App 匿名函式] ──接收到 true 參數 ──► 渲染出對應的 JSX
```

以下以實際程式碼說明。

## 範例程式碼架構  
以下是結合 Context API 與 Render Props (Children as a Function) 的實作：

### 子元件：`ToggleDisplay.jsx`  
此時 `children` 並不是 JSX，而是一個函式物件。React 不會自動執行它，因此需要手動呼叫 `children(on)`，才能取得真正要渲染的 JSX。

```javascript
// ToggleDisplay.jsx
import React from "react"
import { ToggleContext } from "./Toggle"

export default function ToggleDisplay({ children }) {
    // 1. 從 Context 領出目前的開關狀態
    const { on } = React.useContext(ToggleContext)
    
    // 2. 把狀態當作參數，直接執行 children 函式！
    return children(on)
}
```

### 父元件：`App.jsx`

```javascript
import React from 'react';
import Toggle from "./components/Toggle"

function App() {
  return (
    <Toggle>
      <Toggle.Button>切換狀態</Toggle.Button>
      
      {/* 傳入一個箭頭函式作為 children */}
      <Toggle.Display>
        {(on) => {
          return <div className={`box ${on ? "filled" : ""}`}></div>
        }}
      </Toggle.Display>
    </Toggle>
  )
}
```

## 常見觀念問答
**Q: 既然有了 React Hooks (如自訂 Hook)，Render Props 是不是過時了？**

A: 沒有過時，兩者各司其職！

* **React Hooks**：**自訂 Hook 擅長抽離與共享狀態邏輯**，例如資料請求、視窗大小監聽或表單處理。**Hook 本身不負責渲染 UI，而是提供資料與操作方法，由元件自行決定如何呈現。**
* **Render Props**：**擅長處理「元件架構的封裝」**。當需要把狀態控制、DOM 結構、鍵盤事件（A11y）緊密綁定在一起（例如建立一個可重用的下拉選單元件），並開放畫面客製化給別人時，Render Props 依然是不可取代的模式。  
Headless UI 等 Headless Component Library 經常提供 Render Props（或 Function as Children）等 API，以便讓開發者自由控制畫面；同時也會搭配 Context、Compound Components 等模式來封裝元件邏輯。

**Q: 為什麼在 `ToggleDisplay` 裡面是寫 `children(on)`，而不是寫 `children`？**

A: 因為這時的 `children` 是一個函式。如果直接寫 `{children}`，React 只會拿到一個函式本體，並在畫面上顯示空白或噴出錯誤；寫成 `children(on)` 才是 **「執行這個函式並傳入參數」**，React 才能順利拿到函式執行後回傳的 JSX 畫面結果。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>