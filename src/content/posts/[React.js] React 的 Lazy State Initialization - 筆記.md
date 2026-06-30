---
title: "[React.js] React 的 Lazy State Initialization - 筆記"
pubDatetime: 2026-06-30T08:04:14.999Z
tags: ["React Hook","Frontend","React.js","JavaScript","Performance"]
description: "Table of contents :memo: 簡介 在 React 的開發過程中，我們經常使用 useState..."
hackmd_id: "rkrqVeZmzx"
---

## Table of contents

---

## :memo: 簡介  
在 React 的開發過程中，我們經常使用 `useState` 來管理元件的狀態。隨著專案規模擴大，有些狀態的「初始值」可能需要經過複雜的計算（例如：生成大量初始資料、或是讀取 `localStorage`）。

如果發現 **每次點擊按鈕或更新畫面時，網頁都隱約有卡頓感**，這很有可能是因為 `useState` 寫法正在悄悄地浪費 CPU 的運算資源。

要優雅地解決這個問題，我們可以運用 React 內建的效能優化利器：**Lazy State Initialization（狀態延遲初始化）**。



## :memo: 核心觀念：React 元件的重新渲染機制  
在探討 Lazy Initialization 之前，我們先複習 React 的核心渲染公式：

**UI = f(state)**

當一個元件的 State 改變時，React 為了確保畫面是最新的，會**重新執行整個元件函式（Re-render）**。這意味著函式內部的每一行程式碼都會從頭到尾再跑一遍。

如果我們在 `useState` 的括號內直接放入一個需要消耗效能的函式呼叫，就會引發隱形的效能危機。



## :memo: 錯誤範例：每次渲染都在做白工  
假設我們正在開發一個骰子遊戲，首次載入網頁時，需要初始化 10 個帶有隨機點數與獨一無二 `id` 的骰子物件：

```jsx
import React from "react"
import { nanoid } from "nanoid"

export default function App() {
    // ❌ 錯誤示範：將「函式執行結果」直接傳入 useState
    const [dice, setDice] = React.useState(generateAllNewDice())

    function generateAllNewDice() {
        console.log("generateAllNewDice 正在被狂call！")
        return Array.from({ length: 10 }, () => ({
            value: Math.floor(Math.random() * 6 + 1),
            isHeld: false,
            id: nanoid()
        }))
    }

    function hold(id) {
        setDice(oldDice => oldDice.map(die =>
            die.id === id ? { ...die, isHeld: !die.isHeld } : die
        ))
    }

    return (
        <main>
            {/* 渲染骰子畫面與點擊事件 */}
            <button onClick={() => hold(dice[0].id)}>點擊第一個骰子</button>
        </main>
    )
}
```

### 為什麼這段程式碼有效能危機？

1. **首次渲染 (Mount)**：React 載入 App，執行第 6 行的 `generateAllNewDice()`，成功將回傳的陣列作為 dice 的初始狀態。  
1. **點擊骰子更新狀態**：當使用者點擊按鈕觸發 `hold` 時，`setDice` 被呼叫 --> 狀態改變 --> 整個 App 函式重新執行 (Re-render)。  
1. **效能危機點**：函式從頭執行後，再度來到了第 6 行。雖然 React 知道 `dice` 早就有值了，但因為是寫 `generateAllNewDice()`（帶有括號，直接執行），**JavaScript 的機制依然會強迫跑完這個建立 10 個物件、跑迴圈、生成 `nanoid` 的昂貴計算。**   
打開瀏覽器的 Console，會發現每點擊一次，`generateAllNewDice` 正在被狂call！ 紀錄就會瘋狂跳出來。這些多餘的計算隨後通通被 React 丟棄，造成嚴重的資源浪費。

## :memo: 實作分享：利用 Lazy State Initialization 進行優化  
要阻止這個資源浪費，解法非常簡單：**不要給它執行結果，而是給它一個「任務（函式本人）」。**  
這種**讓初始值「等到真正有需要時才計算，且只計算一次」的技巧，就叫做 Lazy State Initialization。**

正確的解決方案

```jsx
import React from "react"
import { nanoid } from "nanoid"

export default function App() {
    // 💡 正確做法：傳入一個「匿名箭頭函式」，在裡面呼叫目標函式
    const [dice, setDice] = React.useState(() => generateAllNewDice())

    function generateAllNewDice() {
        console.log("generateAllNewDice 只有在第一次會被印出來！")
        return Array.from({ length: 10 }, () => ({
            value: Math.floor(Math.random() * 6 + 1),
            isHeld: false,
            id: nanoid()
        }))
    }

    // 後續邏輯不變...
}
```

## :memo: 什麼時候該使用 Lazy State Initialization？  
在 React 開發中，並非所有的 `useState` 都要寫成箭頭函式。如果初始值很單純（例如：`useState(0)`、`useState(false)`），直接傳入即可，因為基礎型別的傳遞幾乎不耗費效能。

只有當初始值牽涉到以下 「昂貴計算 (Expensive Operations)」 時，才強烈建議使用 Lazy 初始化：

* 陣列與物件的密集操作  
例如：需要用 `map()`、`filter()`、`reduce()` 處理長度高達數百數千的陣列，或像本例一樣需要同時調用外部庫（如 `nanoid()`）多次來初始化資料。

* 與瀏覽器儲存空間或外部 I/O 互動  
最常見的情境就是從 `localStorage` 讀取並還原快取資料：

```jsx
// 💡 讀取硬碟與 JSON 解析都是同步且耗時的，非常適合 Lazy 初始化
const [user, setUser] = useState(() => {
    const localData = localStorage.getItem("user_profile")
    return localData ? JSON.parse(localData) : defaultUser
})
```

## 小結  
保持元件在 Render 期間的純粹與高效是前端優化的核心課題。透過 Lazy State Initialization，我們成功的將耗時的「狀態初始計算」與「組件重新渲染」完美分開。

## 參考資料
* [React 官方文件 - Lazy initial state (Avoiding recreating the initial state)](https://react.dev/reference/react/useState#avoiding-recreating-the-initial-state)
* [Prevent State From Being Reset Each Render](https://www.epicreact.dev/tutorials/build-react-hooks/state-is-reset-to-initialstate-on-each-render/solution)
* [React 官方文件 - You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>