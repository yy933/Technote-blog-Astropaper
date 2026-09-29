---
title: "前端框架觀念：React State 與 Lifecycle（生命週期）- 筆記.md"
pubDatetime: 2026-09-29T02:59:20.930Z
tags: ["Interview Preparation","React","React.js","Concepts","State Management","Frontend"]
description: "Tags: Interview Preparation React React.js Concepts State M..."
---

###### Tags: `Interview Preparation` `React` `React.js` `Concepts` `State Management` `Frontend`


## Table of contents


## 核心觀念總覽

* **Props vs. State**：Props 像傳給函式的**參數（外部傳入）**；State 像宣告在函式內部的**區域變數（內部管理）**。
* **State 的運作本質**：Class Component 的 State 附著於**持續存在的物件實例（Persistent Object Instance）**；Function Component 則是在每次 State 改變時**重新執行整個函式（Re-called / Re-instantiated）**。
* **三大生命週期階段**：**Mounting（掛載）**、**Updating（更新）**、**Unmounting（卸載）**。
* **現代替代方案**：Function Component 透過 **`useEffect` Hook** 來統一處理並涵蓋傳統生命週期的行為。


## 1. Props vs. State 的比喻與差異

**兩者本質上都是 JavaScript 物件（Plain JS Objects）**，也都會觸發 UI 渲染，但運作範圍與主導權完全不同：

| 比較面向 | Props (Properties) | State |  
| :--- | :--- | :--- |  
| **類比概念** | 像是傳入函式的 **參數 (Parameters)** | 像是宣告在函式內的 **區域變數 (Variables)** |  
| **作用域 (Scope)** | 從元件外部傳入，控制權在父元件 | 完全封閉於元件內部（Local Scope） |  
| **存取與修改** | **唯讀（Read-Only）**，內部不可直接修改 | 可透過 State 更新函式（如 `setState` / `useState`）修改 |

---

## 2. Class vs. Function Component 的 State 

### Class Component
* **機制**：State 附著在 Class 建立的 **元件物件實例（Class Instance）** 上。
* **特性**：這個物件在元件的生命週期中是 **持續存在（Persists）** 的，狀態的變更是在同一個物件上透過 `this.setState()` 進行修改與維護。

### Function Component
* **機制**：每次 State 發生改變時，整個 Function Component 會被**重新呼叫（Re-called）與重新實例化**。
* **特性**：它並非一個持續存在的實例物件，而是透過 React 底層的 Hooks 機制，在每一次函式重新執行時重新取得當前狀態的快照（Snapshot）。

---

## 3. 元件生命週期（Component Lifecycle）三大階段

Class Component 透過特定的生命週期方法（Lifecycle Methods）來捕捉元件的狀態變化：

```text
【Mounting 掛載】      ➔   【Updating 更新】   ➔   【Unmounting 卸載】
（進入 DOM / 第一次渲染)   (State / Props 發生改變)    (從 DOM 中移除)
```

1. **Mounting（掛載階段）**：
   * **觸發時機**：元件第一次被建立並放入 DOM 頁面上時。
   * **關鍵方法**：`render()` ➔ `componentDidMount()`（常用於發送 API 請求、訂閱事件）。

2. **Updating（更新階段）**：
   * **觸發時機**：當 State 或 Props 發生改變，元件進行重新渲染時。
   * **關鍵方法**：`render()` ➔ `componentDidUpdate()`（常用於回應資料改變後的處理）。

3. **Unmounting（卸載階段）**：
   * **觸發時機**：元件即將從頁面中被移除/銷毀時。
   * **關鍵方法**：`componentWillUnmount()`（常用於移除 Event Listener、清除 Timer 或取消 API 訂閱，防止記憶體洩漏）。

---

## 4. 在 Function Component 中替代生命週期：`useEffect`

在現代 React 開發中，不再需要撰寫分散的生命週期方法，而是統一透過 **`useEffect` Hook** 來達成：

* **對應 Mounting**：帶入空的依賴陣列 `useEffect(() => {}, [])`。
* **對應 Updating**：帶入特定依賴項 `useEffect(() => {}, [state])`。
* **對應 Unmounting**：在 Effect 中回傳 Cleanup 函式 `useEffect(() => { return () => { /* 清除邏輯 */ }; }, [])`。



# FAQ

### Q1: Props 與 State 有什麼差別？
**回答**：  
1. **傳遞方向與控制權**：Props 是從外部（父元件）傳入元件的資料，就像傳入函式的參數，元件內部不能直接修改它（唯讀）；State 是元件內部自行宣告與管理的狀態，就像函式內部的區域變數。  
2. **共同點**：兩者都是 JavaScript 物件，且資料改變時都會觸發 UI 的重新渲染（Re-render）。

---

### Q2: 為什麼 Function Component 在 State 更新時會重新執行整個函式，卻不會遺失 State？  
因為 React 底層使用 **Hooks（例如 `useState`）與閉包機制**。  
當 State 改變觸發 Re-render 時，雖然 Function 會被重新呼叫執行，但 `useState` 會向 React 底層的 Fiber 節點索取並回傳「最新更新後的 State 值」，從而實現狀態的跨渲染保存。

---

### Q3: 傳統 Class 的 `componentWillUnmount` 主要用在什麼情境？在 Function Component 要怎麼寫？
* **主要情境**：用於執行**資源清除（Cleanup）**，例如移除 `window.addEventListener`、`clearInterval` 或取消未完成的非同步請求，避免記憶體洩漏（Memory Leak）。
* **Function Component 對應寫法**：在 `useEffect` 中**回傳一個清理函式（Cleanup Function）**：

```javascript
useEffect(() => {
  const handleScroll = () => console.log('scrolling...');
  window.addEventListener('scroll', handleScroll);

  // 回傳的函式等同於 componentWillUnmount 的清除作用
  return () => {
    window.removeEventListener('scroll', handleScroll);
  };
}, []); // 空陣列確保只在 Unmount 時執行清除
```