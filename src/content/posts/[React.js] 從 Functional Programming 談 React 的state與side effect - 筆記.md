---
title: "[React.js] 從 Functional Programming 談 React 的state與side effect - 筆記"
pubDatetime: 2026-06-26T08:51:48.627Z
tags: ["JavaScript","Frontend","React.js","React Hook"]
description: "Table of contents :memo: 簡介 在學習 React 的過程中，可能曾遇過「網頁突然卡死、瀏覽器..."
hackmd_id: "BJhtuhoMfe"
---

## Table of contents


## :memo: 簡介  
在學習 React 的過程中，可能曾遇過「網頁突然卡死、瀏覽器瘋狂發送 API 請求」的崩潰情境，也就是**無限迴圈（Infinite Loop）**。

要真正理解為什麼會寫出無限迴圈，以及如何優雅地解決它，我們必須回到 React 的設計底層：**Functional Programming（函式編程，簡稱 FP）** 的思維核心。React 的畫面渲染本質上就是一個純粹的數學公式：

**UI = f(state)**

當我們破壞了這個公式的純粹性，React 的渲染機制就會失控。

## :memo: 核心觀念：什麼是 Functional Programming (FP)？  
Functional Programming 是一種程式設計的「風格」或「典範」。它將電腦運算視為數學上的函式計算，包含以下三大核心：

1. **純函式 (Pure Functions)**
   * **相同輸入，永遠得到相同輸出**。
   * **沒有Side Effects**：函式在執行過程中，不能去改變外部世界的狀態（例如：修改全域變數、直接修改傳進來的參數、讀寫檔案、或發送網路請求）。  
2. **不可變性 (Immutability)**
   * 資料一旦建立就不能被修改。若要變更，必須「複製一份舊的，並回傳一個全新的資料」，而非修改原資料（如：使用 `...spread` 展開運算子）。  
3. **宣告式程式碼 (Declarative)**
   * 專注於「要做什麼（What）」而非「怎麼做（How）」。例如在 React 中習慣使用 `.map()`、`.filter()` 處理陣列，而非傳統的 `for` 迴圈。

## :memo: 錯誤範例：無限迴圈  
假設我們要寫一個元件，在初始載入時透過 Star Wars API (`swapi.dev`) 取得角色資料並渲染在畫面上。

如果我們像寫傳統 JavaScript 一樣，直接把 `fetch` 寫在元件的主體內：

```jsx
import React from "react"

export default function App(props) {
    const [starWarsData, setStarWarsData] = React.useState(null)
    
    // ❌ 錯誤示範：將副作用直接寫在渲染主體中
    fetch("[https://swapi.dev/api/people/1](https://swapi.dev/api/people/1)")
        .then(res => res.json())
        .then(data => setStarWarsData(data))
    
    return (
        <div>
            <pre>{JSON.stringify(starWarsData, null, 2)}</pre>
        </div>
    )
}
```

### 為什麼會陷入無限迴圈？  
當這段程式碼執行時，React 幕後會引發嚴重的四步驟連鎖反應：  
1. 組件渲染 (Render)：React 首次載入 App 元件，從第 1 行執行到最後。  
1. 執行 fetch：執行到第 6 行，非同步向外部 API 發送網路請求。  
1. 更新狀態 (Set State)：API 回傳資料後，觸發第 8 行的 `setStarWarsData(data)`。  
1. 觸發重新渲染 (Re-render)：關鍵點！ React 的核心機制是「只要狀態（State）改變，該元件函式就會重新執行一次」，以確保畫面是最新的。

⚠️ 結果： 函式從頭執行後，又遇到了第 6 行的 `fetch` --> 再次發送請求 --> 再次更新狀態 --> 再次重新渲染。程式就此陷入無限迴圈，直到瀏覽器崩潰。

## :memo: 實作分享：利用 useEffect 進行隔離side effect  
在 FP 的哲學中，完全沒有side effect是不可能的（**網頁要載入資料、操作 DOM**）。React 的解決辦法是：允許side effect存在，但必須把它們「隔離」在特定的盒子裡。這個盒子就是 `useEffect`。

### 正確的解決方案

```jsx
import React from "react"

export default function App(props) {
    const [starWarsData, setStarWarsData] = React.useState(null)
    
    // 💡 正確做法：將side effect包裹在 useEffect 中
    React.useEffect(() => {
        fetch("[https://swapi.dev/api/people/1](https://swapi.dev/api/people/1)")
            .then(res => res.json())
            .then(data => setStarWarsData(data))
    }, []) // 👈 關鍵：傳入「空依賴陣列」
    
    return (
        <div>
            <pre>{JSON.stringify(starWarsData, null, 2)}</pre>
        </div>
    )
}
```

### 為什麼這樣就能解開無限迴圈？  
`useEffect` 的第二個參數是 依賴陣列（Dependency Array）：

當我們傳入一個 空陣列 `[]` 時，等於在告訴 React：「**這個side effect函式，只有在元件第一次被掛載（Mount）到畫面上時執行唯一一次。之後不管這個元件因為狀態改變而重新渲染了幾萬次，都請直接忽略、不要再執行它。**」

如此一來，`fetch` 成功拿到資料並更新狀態後，元件雖然會重新渲染，但 React 看到 `[]` 就會跳過 `useEffect`，成功終止了連鎖反應。

## :memo: 在前端開發中，常見的 Side Effects 有哪些？

在 React 的純函式世界裡，只要 **「做了跟渲染畫面（把 Props/State 轉換成 JSX）無關，且會影響到外部世界或依賴外部世界的行為」**，通通都算 Side Effect。

以下是前端開發中最常見的幾種副作用：

1. **發送網路請求（Data Fetching）**
   * 例如：使用 `fetch()` 或 `axios.get()` 向後端 API 索取資料。因為這需要時間等待，且依賴外部伺服器的回應，這是不純粹的行為。  
2. **手動操作 DOM（Manual DOM Manipulation）**
   * 例如：使用原生 JS 的 `document.getElementById()`、修改 `document.title`、或是手動加上事件監聽器（`window.addEventListener`）。  
3. **設定定時器（Timers）**
   * 例如：使用 `setTimeout()` 延遲執行，或是 `setInterval()` 每隔幾秒執行一次程式。  
4. **與瀏覽器儲存空間互動（Browser Storage API）**
   * 例如：讀取或寫入 `localStorage`、`sessionStorage` 或 `Cookies`。  
5. **記錄日誌（Logging）**
   * 例如：最常用的 `console.log()`，雖然看似無害，但它實質上改變了瀏覽器控制台（外部世界）的狀態，因此在嚴格的 FP 定義中也算副作用。

React 的核心觀念就是：**「在 Render 期間，請保持雙手乾淨（純粹），把以上這些Side Effects通通打包，丟給 `useEffect` 去處理！」**

## 小結  
React 的設計深受 Functional Programming 的啟發。保持元件函式的「純粹性（Pure）」可以讓 UI 變得極其好預測且不易出現 Bug。當我們需要處理不純粹的Side Effects時，謹記使用 `useEffect` 搭配正確的依賴陣列，將Side Effects安全地隔離起來，這就是 React 開發中維護資料流的核心關鍵！

## 參考資料
* [React 官方文件 - Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
* [Pure Functions in JavaScript](https://www.geeksforgeeks.org/javascript/pure-functions-in-javascript/)
* [A Visual Guide to React Mental Models](https://andyogo.github.io/custom-element-reactions-diagram/)

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>