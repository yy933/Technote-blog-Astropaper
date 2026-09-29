---
title: "前端元件觀念：Function Component vs. Class Component - 筆記.md"
pubDatetime: 2026-09-29T07:02:48.185Z
tags: ["Frontend","Concepts","React","React.js","Interview Preparation"]
description: "Tags: Interview Preparation React React.js Concepts Fronten..."
---

###### Tags: `Interview Preparation` `React` `React.js` `Concepts` `Frontend`


## Table of contents

## 觀念演進  
在 React 的發展歷程中，UI 元件的寫法經歷了從 **Class Component（類別元件）** 轉向 **Function Component（函式元件）** 的重大演進。理解兩者的差異不僅是面試必考題，更是掌握現代 React 心智模型（Mental Model）的關鍵。



## 核心差異  
兩者最根本的差別在於 **程式設計思維（Programming Paradigm）** 的不同：

* **Class Component**： **物件導向（OOP）**，依賴 `this` 指標、類別實例（Instance）與特定時間點的生命週期方法。
* **Function Component**：基於 **宣告式/函數式編程（Functional）**，強調 **「UI 是資料狀態（State）的映射」** ，透過 **Hooks** 處理副作用與狀態。



## 1. 狀態與副作用處理（State & Side Effects）

### Class Component
* **狀態管理**：在 `constructor` 中宣告 `this.state` 物件，並透過 `this.setState()` 進行非同步狀態更新。
* **副作用（Side Effects）**：依賴明確的生命週期方法（Lifecycle Methods），例如：
  * `componentDidMount`：組件掛載後（發送 API 請求）。
  * `componentDidUpdate`：組件更新後（監聽 State/Props 變化）。
  * `componentWillUnmount`：組件卸載前（清除 Timer 或 Event Listener）。

### Function Component (with Hooks)
* **狀態管理**：使用 `useState` 或 `useReducer` 獨立宣告與管理狀態。
* **副作用（Side Effects）**：使用 `useEffect` 同步副作用。它不再以「掛載/更新/卸載的時間點」思考，而是以 **「資料狀態（Dependencies）的變化」** 為核心：
  * 傳入空陣列 `[]` 相當於 `componentDidMount`。
  * 回傳 Cleanup 函式 `return () => {}` 相當於 `componentWillUnmount`。



## 2. `this` 指標與程式碼簡潔度

* **`this` 的缺點**：Class Component 必須經常處理 JavaScript `this` 的指向綁定（如在 constructor 中 `this.handleClick = this.handleClick.bind(this)` 或寫成箭頭函式），初學者極易犯錯。
* **邏輯分散**：Class Component 的相關邏輯常被強行拆散在不同生命週期（例如在 `componentDidMount` 開啟監聽，在 `componentWillUnmount` 移除監聽）。
* **Hooks 的解法**：Function Component 完全沒有 `this` 指標問題，且能透過 **Custom Hooks（自訂 Hook）** 將相關邏輯封裝在一起並跨元件複用。



## 3. Capture Value（快照特性）與閉包

由於 JavaScript 的 **閉包（Closure）** 特性，Function Component 在每一次 Render 時，都擁有當時獨立的 `props` 與 `state` 快照（Capture Value）：

* **Class Component**：`this.props` 是可變的（Mutable），當非同步行為（如 `setTimeout` 或非同步 API）完成時，`this.props` 讀取的是「當下最新的實例屬性」，可能導致非預期的 UI 狀態改變。
* **Function Component**：渲染當時傳入的 `props` 就會被鎖定在當次渲染的閉包中，UI 行為更加穩定且可預測。

---

## 對照表

| 比較面向 | Class Component | Function Component (with Hooks) |  
| :--- | :--- | :--- |  
| **程式思維** | 物件導向 (OOP) | 宣告式 / 函數式 (Functional) |  
| **語法複雜度** | 較繁瑣，需宣告 `class` 與 `render()` | 極簡，純 JS 函式回傳 JSX |  
| **`this` 綁定** | 需要處理 `this` 指向問題 | 無 `this` 問題 |  
| **狀態宣告** | `this.state` / `this.setState` | `useState` / `useReducer` |  
| **副作用處理** | 生命週期（`componentDidMount` 等） | `useEffect` 監聽依賴變化 |  
| **邏輯複用** | 靠 HOC (高階組件) 或 Render Props，易造成 Component Tree 嵌套過深 | 透過 **Custom Hooks**，邏輯拍平且高度複用 |  
| **打包體積** | 較難進行程式碼壓縮與 Tree-shaking | 極易被打包工具壓縮與優化 |  
| **官方趨勢** | 舊專案維護使用 | **現代 React 的標準與首選** |

---

# FAQ

### Q1: Class Component 未來會被 React 官方完全廢棄（Deprecate）嗎？
**目前不會，但已經不推薦寫新的 Class Component。**

* **官方立場**：React 官方並無計畫移除 Class Component，以確保龐大的舊有生態系專案能正常運作。
* **實務建議**：在開發新功能或新專案時，應 100% 採用 Function Component + Hooks。舊專案若運作正常則無須強行重構，只需在維護時逐漸替換即可。



### Q2: 什麼是 Rules of Hooks（Hooks 的使用規則）？為什麼不能在條件式（if）裡面呼叫 Hooks？  
React Hooks 有兩大核心規則：  
1. **只能在 React Function Component 或 Custom Hooks 的最頂層呼叫 Hooks**（不能在迴圈、條件判斷 `if` 或巢狀函式中呼叫）。  
2. **只能在 React Function Component 中呼叫 Hooks**（普通的 JS 函式不能呼叫）。

**原因**：  
React 底層是依賴 **呼叫順序（Order of Calls）** 來紀錄與配對每個 Hook 的 State。如果將 Hook 放在 `if` 條件句中，一旦條件改變導致 Hook 執行順序錯亂，React 就無法精準將 State 覆加到正確的位置上。


### Q3: 為什麼 Function Component 的邏輯複用能力（Custom Hooks）超越 Class Component？  
在 Class Component 時代，要在元件之間複用邏輯（例如「監聽視窗尺寸改變」），必須使用 **HOC (Higher-Order Component)** 或 **Render Props**。這會造成：
* **Wrapper Hell（嵌套地獄）**：DOM/Component 樹上會疊加一層又一層的包裹元件。
* **命名衝突**：HOC 注入的 Props 可能會互相覆蓋。

而 Function Component 的 **Custom Hooks** 只是純粹的 JavaScript 函式呼叫，能在不改變元件樹結構的前提下，抽出任何狀態與副作用邏輯。