---
title: "前端框架觀念：React useEffect Hook 與運作機制 - 筆記.md"
pubDatetime: 2026-09-29T04:22:47.691Z
tags: ["Concepts","React.js","React","React Hook","Interview Preparation","useEffect","Frontend"]
description: "Tags: Interview Preparation React React.js Concepts React H..."
---

###### Tags: `Interview Preparation` `React` `React.js` `Concepts` `React Hook` `useEffect` `Frontend`


## Table of contents

## 前言  
在 Function Component 中，**`useEffect`** 是用來處理副作用（Side Effects，如發送 API 請求、訂閱事件、操作 DOM 等）的核心 Hook。它透過簡潔的語法，統一並取代了傳統 Class Component 分散的生命週期方法（`componentDidMount`、`componentDidUpdate`、`componentWillUnmount`）。



## 核心觀念

* **接收兩大參數**：第一個參數為 **Effect 回調函式**，第二個參數為 **依賴陣列（Dependency Array）**。
* **觸發時機由依賴陣列決定**：可控制為「僅 Mount 執行」、「 Mount + 特定變數更新時執行」或「每次 Render 皆執行」。
* **Cleanup 機制**：Effect 函式可回傳一個**清理函式**，用於取消訂閱、清除監聽或釋放資源，精準對應卸載（Unmount）情境。



## 1. `useEffect` 的兩大參數結構

`useEffect` 接收兩個參數：

```javascript
useEffect(() => {
  // 1. Effect Function (副作用邏輯)
  
  return () => {
    // Return Value: Cleanup Function (清理邏輯)
  };
}, [/* 2. Dependency Array (依賴陣列) */]);
```

1.  **第一個參數（Effect Function）**：必定是一個函式，放你需要執行的副作用程式碼。
      
2.  **第二個參數（Dependency Array）**：選擇性的陣列，用來決定第一個函式何時該被重新執行。
    

## 2. 依賴陣列的三種情境與執行時機

依賴陣列（Dependency Array）不同的傳入方式對執行時機的影響：

| 依賴陣列寫法 | 觸發時機（When it runs） | 觀念與用途 |  
| :--- | :--- | :--- |  
| **空陣列 `[]`** | 僅在 **Mount（掛載/第一次渲染）** 時執行一次 | 相當於 `componentDidMount`，用於初始發送 API 或註冊全域事件。 |  
| **有傳入變數 `[state]`** | **Mount 時** + 陣列內的**變數發生改變時**執行 | 相當於 `componentDidUpdate` 的特定變數監聽，變數更新即觸發。 |  
| **完全不傳第二參數** | **Mount 時** + 每次 **元件重新渲染（Re-render）** 時皆執行 | 任何 State 或 Props 變動都會觸發，極易造成效能問題或無窮迴圈，需謹慎使用。 |

> **提示**：依賴陣列中可以放多個變數（例如  `[a, b]`），只要其中**任意一個變數**改變，Effect 就會重新執行。

## 3. 回傳值：清理函式（Cleanup Function）

`useEffect`  的第一個函式可以**回傳另一個函式**，這個回傳值稱為  **Cleanup Function（清理函式）**。

### 運作原理與功能

-   **執行時機**：
    
    1.  當元件即將卸載（Unmount）時執行。
        
    2.  在下一次 Effect  **重新執行前**執行（用來清理上一次的副作用）。
        
-   **對應生命週期**：高度類似 Class Component 的  **`componentWillUnmount`**。
    
-   **未寫 Cleanup Function 時**：預設回傳值為  **`undefined`**，React 檢查到回傳非 Function 類型，即會跳過清理邏輯。
    
-   **主要用途**：
    
    -   移除視窗或 DOM 的 Event Listener（如  `window.removeEventListener`）。
        
    -   清除計時器（`clearInterval`  /  `clearTimeout`）。
        
    -   取消未完成的 API 請求（`AbortController`）。
# FAQ

### Q1: useEffect`  的第一個參數可以寫成  `async`  函式嗎？為什麼？

**答案是：不行。**

-   **原因**：`async`  函式預設會回傳一個  **Promise**  物件。但 React 規範  `useEffect`  的第一個函式只能回傳「一個用來清理資源的普通函式 (Cleanup Function)」**或**「什麼都不回傳 (`undefined`)」。若回傳 Promise，React 在卸載時呼叫該回傳值會出錯。
    
-   **正確寫法**：
- 寫法A: 在  `useEffect`  內部另外宣告一個  `async`  函式並呼叫它：
    

```javascript
// ❌ 錯誤寫法
useEffect(async () => {
  const data = await fetchData();
}, []);

// ✅ 正確寫法
useEffect(() => {
  const getData = async () => {
    const data = await fetchData();
  };
  getData();
}, []);
```

- 寫法B: 使用 IIFE（立即執行函式）

```javascript
useEffect(() => {
  (async () => {
    try {
      const response = await fetch('[https://api.example.com/data](https://api.example.com/data)');
      const data = await response.json();
    } catch (error) {
      console.error("Fetch error:", error);
    }
  })();
}, []);
```

#### 進階的最佳實務：結合 Cleanup Function 與 AbortController

在真實專案中，若 API 請求尚未完成元件就卸載，應透過  `AbortController`  取消請求，防止記憶體洩漏或 Race Condition：

```javascript
useEffect(() => {
  const controller = new AbortController();

  const fetchData = async () => {
    try {
      const response = await fetch('[https://api.example.com/data](https://api.example.com/data)', {
        signal: controller.signal // 將 signal 帶入 fetch
      });
      const data = await response.json();
    } catch (error) {
      if (error.name !== 'AbortError') {
        console.error("Fetch error:", error);
      }
    }
  };

  fetchData();

  // 回傳 Cleanup Function：當元件卸載時取消未完成的請求
  return () => {
    controller.abort();
  };
}, []);
```


### Q2: 如何用一句話總結  `useEffect`  的優勢？

`useEffect`  將原本分散在  `componentDidMount`、`componentDidUpdate`  與  `componentWillUnmount`  的程式邏輯，依據「資料依賴與功能相關性」整合在同一個地方，讓程式碼更簡潔、高度可讀且易於維護。

### Q3: 什麼時候會觸發 Cleanup Function（清理函式）？

1.  **元件被銷毀/卸載（Unmount）時**。
      
2.  **在下一次 Effect 被觸發前**：如果依賴變數改變引發 Effect 重新執行，React 會先執行上一次 Effect 回傳的 Cleanup 函式清除舊資源，再執行新一次的 Effect。