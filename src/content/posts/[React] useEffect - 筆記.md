---
title: "[React] useEffect - 筆記"
pubDatetime: 2026-05-26T03:01:46.978Z
tags: ["JavaScript","React.js","React Hook"]
description: " Table of contents 為什麼需要useEffect? 先回顧一下React的主要功能： 負責將 UI 渲..."
---

## Table of contents



## 為什麼需要useEffect?

先回顧一下React的主要功能：
* 負責將 UI 渲染到瀏覽器的 DOM 上
* 管理元件的狀態（state）與屬性（props）
* 在每次 render 之間「保留」state，讓 UI 能根據資料變化自動更新
* 當state或props改變時，自動重新 render 對應的 UI 區塊(只會更新必要的部分，稱為 virtual DOM diffing)

React無法直接控制的(Out)side effects:
* `localStorage`的讀寫
* 與外部 API 或資料庫的互動（如 `fetch`、`axios`）
* 訂閱事件（如 WebSocket、Firebase 連線）
* 手動操作真實 DOM（例如使用 `document.querySelector` 或 `document.addEventListener`）
* 任何非「只根據 `state` 或 `props` 的值，來決定畫面長相與內容」的外部行為

:arrow_right: 為了處理這些外部行為(side effects)，我們需要使用`useEffect`。

先看一個錯誤範例，在React元件中fetch API：
```jsx
import React from "react"

export default function App(props) {
    const [starWarsData, setStarWarsData] = React.useState(null)
         
    fetch("https://swapi.dev/api/people/1")
        .then(res => res.json())
        .then(data => setStarWarsData(data))
 
    return (
        <div>
            <pre>{JSON.stringify(starWarsData, null, 2)}</pre>
        </div>
    )
}
```
這樣寫，會**造成無窮迴圈(infinite loop)**！為什麼？
每次 React 執行 `App()` 函式時：
1. App 函式被呼叫 → `fetch()` 執行 → `setStarWarsData(data)` 觸發狀態改變。
1. 狀態改變後，React 重新 render → `App()` 被再次呼叫 → 又執行一次 `fetch()` → 又呼叫 `setStarWarsData()`。
1. 如此重複無數次，造成 infinite re-render loop。

## 什麼是useEffect

`useEffect(setup, dependencies?)` 是 React 的 Hook：
* 和外部系統同步（例如 API、WebSocket、定時器）
* 在 component render 之後執行Side Effect
* Cleanup函式

### useEffect 的兩個參數
1. setup 函式（必填）
* 通常是callback function
* **寫下Side Effect邏輯**，例如訂閱資料、DOM 操作等。
* 可**選擇性 return 一個 cleanup 函式**。
* React 會在元件加入 DOM 時執行 setup。
* 每次dependencies變動時，先執行前一次的 cleanup，再執行新的 setup。
* 元件卸載時也會執行 cleanup。

2. dependencies 陣列（非必須）
* 列出在 setup 內有用到的 reactive 值（如 props、state、或元件內宣告的變數與函式）。
* 必須寫成固定長度、inline 的陣列，例如 `[count, user.name]`。
* React 會使用 `Object.is` 判斷dependencies是否改變(i.e. dependencies沒變就不會re-render)。
* 如果不寫這個陣列，Effect 每次 re-render 都會執行。

| dependencies | 說明                                 |
| ------------ | ---------------------------------- |
| `[a, b]`     | a 或 b 改變時執行（推薦）                    |
| `[]`         | 只在初次渲染時執行一次（相當於 `componentDidMount`） |
| 無            | 每次 render 都會執行（不建議，效能差）            |

### 基本範例
```jsx
useEffect(() => {
  // setup (effect logic, eg. API call, addEventListener, etc)
  console.log("Effect started");

  return () => {
    // cleanup
    console.log("Effect cleaned up");
  };
}, [dependency1, dependency2]);
```

### Cleanup Function
先看一個例子：
```jsx
// App.jsx
import React from "react"
import WindowTracker from "./WindowTracker"

export default function App() {

    const [show, setShow] = React.useState(true)
    
    function toggle() {
        setShow(prevShow => !prevShow)
    }

    return (
        <main className="container">
            <button onClick={toggle}>
                Toggle WindowTracker
            </button>
            {show && <WindowTracker />}
        </main>
    )
}
// WindowTracker.jsx
import React from "react"

export default function WindowTracker() {
    const [windowWidth, setWindowWidth] = React.useState(window.innerWidth)
    
    React.useEffect(() => {
        window.addEventListener("resize", function() {
            setWindowWidth(window.innerWidth)
        })
    }, [])
    
    return (
        <h1>Window width: {windowWidth}</h1>
    )
}
```
* `App.jsx`：按下按鈕可切換` WindowTracker` 元件的顯示與隱藏。
* `WindowTracker.jsx`：會監聽瀏覽器視窗寬度變化，並顯示當前寬度。

以下這段程式碼有個bug:
```jsx
window.addEventListener("resize", function() {
    setWindowWidth(window.innerWidth)
})
```
每次`WindowTracker` 被掛載時，都會新增一個 `resize` 事件監聽器，但當元件被卸載時，它不會自動移除監聽器！

:warning: 會發生什麼問題?
* 每次點「Toggle」切換 `WindowTracker`，就會重複加一個 resize 監聽器。
* 即使 `WindowTracker`已經不在畫面上(從DOM中移除)，監聽器仍然存在！
* 記憶體洩漏：監聽器無限制地累積。
* 有可能會出現錯誤或效能問題（例如：無限更新 state）。

✅ 正確做法：加上 Cleanup Function！
```jsx
React.useEffect(() => {
    function handleResize() {
        setWindowWidth(window.innerWidth)
    }

    window.addEventListener("resize", handleResize)

    // 加上 cleanup function：元件卸載時移除監聽器
    return () => {
        window.removeEventListener("resize", handleResize)
    }
}, [])
```
Cleanup的功能:
* 元件卸載時移除，乾淨又有效率
* 不再更新已卸載的元件，避免錯誤或警告
* 有效釋放資源

### useEffect return cleanup
```jsx
useEffect(() => {
  // 設定一些side effects（例如：事件監聽）
  window.addEventListener("resize", handleResize)

  // return 的函式就是 Cleanup Function！
  return () => {
    window.removeEventListener("resize", handleResize)
  }
}, []) // dependencies可依需要設定
```
#### Cleanup Function 何時執行？
* 元件卸載（unmount）時 → 清理還沒停止的操作，例如計時器、訂閱等。

#### 為什麼重要？
* 避免記憶體洩漏（如還在等待的請求、未移除的監聽器）
* 防止重複綁定事件或計時器
* 維持效能與正確的行為

#### 常見使用情境

| 情境                  | 清理方式                       |
| ------------------- | -------------------------- |
| `setInterval`       | `clearInterval(timerId)`   |
| `setTimeout`        | `clearTimeout(timerId)`    |
| 事件監聽器            | `removeEventListener(...)` |
| WebSocket 資源       | `socket.close()`           |
| 外部訂閱（e.g. Firebase） | `unsubscribe()`            |

#### 範例：自動更新時間 + 清理 interval

```jsx
import { useEffect, useState } from "react"

function Clock() {
  const [time, setTime] = useState(new Date())

  useEffect(() => {
    const timerId = setInterval(() => {
      setTime(new Date())
    }, 1000)

    return () => clearInterval(timerId) // 清理 interval
  }, [])

  return <p>{time.toLocaleTimeString()}</p>
}
```

## 常見陷阱
### 1. 無窮迴圈
```jsx
useEffect(() => {
  setCount(count + 1)  // 錯誤：改變 state → 觸發 render → 又進入 useEffect
}, [])
```
正確寫法：加入正確dependencies、或使用 `setCount(prev => prev + 1)`

:::warning
* dependencies是空陣列`[]`表示這個 effect 只在初次 render 後執行一次。

❓那為什麼`setCount(...)` 還會導致無限渲染？
> :bulb: 關鍵點: 在 Effect 中執行了會觸發 re-render 的東西（像 `setCount`），而不是 `useEffect` 自己重跑。

* 執行了 `setCount(...)`，這個行為會導致 React 發起一個新的 render。
* 雖然 `useEffect(..., []) `不會再次執行，但 React 會**重新執行整個 component function。**
* 問題就在這裡：每次渲染，都從 `useState(0)` 開始，這樣 `count` 永遠是 0 → `setCount(1)` → 又觸發 render → 又是 0 → 無限循環。

🧠 也就是說，無限重複的其實是component function，不是useEffect
```jsx
function App() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    setCount(count + 1)  // 這會導致 state 改變 → component 重新 render
  }, [])

  return <p>{count}</p>
}
```
**正確的做法是：讓初始狀態設定後不會再觸發更新**
```jsx
useEffect(() => {
  // 只有在 count 是 0 的情況下才更新，確保只更新一次
  if (count === 0) {
    setCount(1)
  }
}, [count])

// 或者
const [count, setCount] = useState(() => 1) // 初始就給值，不用 effect
```
:::

### 2. 非同步函式不能直接作為 `useEffect` callback
```jsx
// 錯誤 ❌
useEffect(async () => {
  const res = await fetch("...")
}, [])

// 正確寫法✅
useEffect(() => {
  async function fetchData() {
    const res = await fetch("...")
    ...
  }
  fetchData()
}, [])
```

### 3. 舊閉包（stale closure）
```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log(count)      // ❗這裡的 count 永遠是初始值
    setCount(count + 1)     // ❗這裡也是
  }, 1000)
  return () => clearInterval(id)
}, [])  // ⚠️ 重點：這裡dependencies是空的
```
#### 問題點
* 使用 `useEffect(..., [])` 時，它只會在初次 render 時執行一次。
* 所以當 `setInterval(...)` 裡的 callback 被建立時，它只捕捉了當下 count 的值，比如說 0。
* 即使 `count` 後來改變了，這段 callback 中的 `count` 仍然是舊的值，不會更新！
* 這就是 **舊閉包（stale closure）** 問題。

#### 正確寫法
```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(prev => prev + 1)  // ✔️ 用函式更新法，不依賴舊值
  }, 1000)
  return () => clearInterval(id)
}, [])  // 保持空dependencies沒問題
```
這樣寫的話，React 會幫我們自動取得「最新」的 `count` 值，不會依賴當初 `useEffect` 進去時的 `count` 值。

另一種解法:加入dependencies
```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1)
  }, 1000)
  return () => clearInterval(id)
}, [count])  // ✅ 加上 count dependencies
```
但是!
* 每次 `count` 改變，就會清除再重設一次 `setInterval`
* 等於計時器會不斷重啟 → **不建議用在 interval**

## 其實你不一定需要useEffect！
React 的核心精神: 處理「資料 ➜ 畫面」，只有在要做的事是 **「跟畫面渲染以外的東西溝通」時，才需要 `useEffect`。**
詳細請見[[React] 到底什麼時候需要 useEffect ? - 筆記](https://hackmd.io/G7rtvRhhQ2iVPqOu4hIm5A)

## Recap

* `useEffect(() => {...}, [deps])`
* 沒 `deps`：每次 render 執行`useEffect`
* 空陣列 `[]`：只在初次 render 執行一次
* `deps` 有值時：只有`deps`變化才觸發
* 清除side effects：return cleanup function
* 非同步邏輯不能直接寫 `async () => {}`，要包成函式
* 注意舊閉包（stale closure）陷阱