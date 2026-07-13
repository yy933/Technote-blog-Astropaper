---
title: "[React.js - Concept] React Strict Mode - 筆記"
pubDatetime: 2026-07-13T07:49:14.723Z
modDatetime: 2026-07-13T07:51:32.919Z
tags: ["React.js","Strict Mode","Pure Function","Side Effects","useEffect","Advanced React","Frontend","Concepts","Performance"]
description: "Table of contents React Strict Mode 核心觀念 React Strict Mode（..."
hackmd_id: "HkTrSGMEGx"
---

## Table of contents

## React Strict Mode 核心觀念
**React Strict Mode（嚴格模式）** 是 React 提供的一種開發檢測工具。它就像一個**「程式碼品質糾察隊」**，只在**開發環境（Development Mode）**下起作用，正式上線（Production）時會自動關閉，完全不影響線上效能。

它的核心目的非常單純：**強迫你的元件多渲染一次（雙重渲染，Double-render），並故意模擬「掛載 -> 卸載 -> 掛載」的生命週期，用來暴力抓出程式碼裡面的潛在 Bug（例如副作用、不純的函式、記憶體洩漏）。**



## 1. 為什麼 Strict Mode 會讓資料「無性生殖」？

在開發時，我們常常會不小心寫出「不純的元件（Impure Component）」。以下是一段經典的錯誤範例：

### 錯誤範例：在 Render 流程中修改外部資料
```javascript
import React from "react"
import productsData from "./data" // 外部陣列

export default function App() {
    const [count, setCount] = React.useState(0)

    // 嚴重錯誤：直接修改了元件外部的變數（副作用 Side Effect）
    productsData.push({ id: "123", name: "+ Create new item" })

    return (
        <button onClick={() => setCount(c => c + 1)}>觸發 Re-render</button>
    )
}
```

### Strict Mode的「抓 Bug」原理：  
在 React 的規範中，元件的 Render 函式必須是一個「純函式（Pure Function）」。
**意即：給定相同的 Props/State，元件每次執行都必須回傳一模一樣的 JSX，且絕對不能改變任何外部世界的變數。**

* 正常環境下：按一次按鈕，`App()` 執行一次，外部陣列被 push 1 個項目，你可能覺得畫面看起來還正常。
* Strict Mode 介入：為了檢查你的元件是不是純函式，React 會故意連續執行兩次 `App()`。因為 React 偷偷執行了第二次，`productsData.push()` 也被強制多執行了一次！這會導致畫面上突然蹦出雙倍的重複資料。

💡 結論：  
React 故意透過這種「刻意搞破壞」的雙重渲染，讓不純的程式碼在開發階段直接原形畢露，避免 Bug 被帶到正式線上環境。

## Side effects的正確處理與計時器清理機制  
為了避免上述慘劇，任何非同步操作、計時器、訂閱事件等「副作用」，都必須嚴格封裝在 `useEffect`中，並落實 Cleanup（清理） 機制。

* 經典計時器範例：`Timer.jsx`

```javascript
import React from "react"

export default function Timer() {
    const [seconds, setSeconds] = React.useState(0)
    
    React.useEffect(() => {
        // 1. 儲存計時器識別碼 (身分證字號)
        const id = setInterval(() => {
            setSeconds(prevSeconds => prevSeconds + 1)
            console.log("Timer is running")
        }, 1000)
        
        // 2. 正確：回傳一個「尚未執行」的箭頭函式作為錦囊
        return () => clearInterval(id)
        
        // ❌ 錯誤寫法：return clearInterval(id) (這樣寫會馬上執行清理!)
    }, [])

    return <h2>{seconds} seconds</h2>
}
```

### 觀念釐清
**Q1: 為什麼要寫 `const id = setInterval(...)`？**  
`setInterval` 啟動時，瀏覽器會在背後開啟一個計時器執行緒，並回傳一個獨一無二的整數識別碼（id）。我們必須用變數記住這個 `id`，未來才能拿著這張「號碼牌」去叫瀏覽器把特定的計時器停掉。

**Q2: 為什麼要 `return` 一個箭頭函式，而不是直接執行 `clearInterval(id)`？**  
這涉及 JavaScript 的函式執行時機：
* 錯誤寫法 `return clearInterval(id)`：因為加上了括號 `()`，JavaScript 會在 `useEffect` 剛載入的當下立刻執行清除。計時器在剛出生的微秒內就被自己親手捏碎，一秒鐘都不會跑。
* 正確寫法 `return () => clearInterval(id)`：這是在回傳一個「尚未執行的工具（Callback）」。React 會把這個指令默默收起來，直到元件即將從畫面上消失（Unmount）的那一刻，React 才會掏出這個錦囊並執行 `clearInterval(id)`，精準關閉計時器。

#### 生活化比喻：消防隊與滅火器  
錯誤寫法（直接呼叫）：安全顧問（useEffect）一在廚房升好火作菜（setInterval），立刻拿起滅火器把火噴熄（立刻執行 clearInterval）。結果廚房根本沒火，什麼菜都煮不出來。

正確寫法（回傳函式）：安全顧問把火升好後，拿著一個滅火器交給老闆（React）並交代：「如果哪天你要關店了，再用這個滅火器把火熄滅。」這就是 Cleanup 函式的本質。

## 3. Strict Mode 如何幫忙抓出記憶體洩漏 (Memory Leak)？  
在 React 18 之後，**Strict Mode 在開發環境下會故意對每個元件執行一次 Mount -> Unmount -> Mount 的三部曲演習。**

如果在 `useEffect` 裡忘記寫 `return () => clearInterval(id)`：

1. 元件首度掛載，計時器 A 開始跑。  
1. Strict Mode 故意將元件瞬間卸載（Unmount），但因為你沒寫 Cleanup，計時器 A 依然在背景偷跑。  
1. Strict Mode 再次重新掛載元件，此時又開啟了一個全新的計時器 B。  
1. 最終你只會看到畫面上的秒數瘋狂亂跳（因為 A 和 B 同時在更新同一個 state），主控台不斷噴出重複的 Log。

透過這種嚴苛的演習，Strict Mode 能確保你的元件在被頻繁切換、銷毀時，不會留下任何殘留的計時器、事件監聽器（`addEventListener`）或未取消的 WebSocket 連線，從而完美根除記憶體洩漏（Memory Leak）。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>