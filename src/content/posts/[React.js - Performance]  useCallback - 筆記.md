---
title: "[React.js - Performance]  useCallback - 筆記"
pubDatetime: 2026-07-16T11:17:43.559Z
tags: ["React.js","useCallback","useMemo","Performance","Advanced React","React Hook"]
description: "Table of contents useCallback 核心定義 在 React 的世界中，函式（Function..."
hackmd_id: "By9r5EL4fg"
---

## Table of contents

## useCallback 核心定義  
在 React 的世界中，**函式（Functions）也是一種物件**。這意味著：**每次元件 Re-render 時，在元件內部宣告的函式都會被重新建立，並分配到一個全新、不同的記憶體地址。**

* **`useCallback` 的唯一使命**：**「把函式的記憶體地址（Reference）鎖定（Cache）起來。」** 
* 只有當 **指定的Dependency Array** 發生變化時，React 才會允許重新建立該函式並更新其地址；否則，每次 Re-render 都會回傳一模一樣的函式地址。



## 1. 為什麼需要 useCallback？

如果我們只是在一般按鈕上綁定一個點擊事件，**完全不需要** 使用 `useCallback`。因為重新建立一個函式地址的開銷極低。

`useCallback` 的真正戰場，是為了解救 **「被 `React.memo` 保護，卻因為函式位址改變而優化失效的子元件」**。

### 經典案例分析

```javascript
import React from "react"
import ChildComponent from "./ChildComponent" // 假設內部使用了 React.memo

export default function App() {
    const [count, setCount] = React.useState(0)
    const [text, setText] = React.useState("")

    // 🔴 破功點：每次 App Re-render（例如輸入文字時），此處都會在記憶體重新分配一個全新的函式位址！
    const handleIncrement = () => {
        setCount(prev => prev + 1)
    }

    return (
        <div>
            <input value={text} onChange={(e) => setText(e.target.value)} />
            
            {/* ❌ 即使 ChildComponent 用了 React.memo，也會因為 handleIncrement 位址每次都不同而跟著無意義重繪！ */}
            <ChildComponent onIncrement="{handleIncrement}"/>
        </div>
    )
}
```

### 解法：使用 useCallback 鎖定函式地址

```javascript

// ✓ 使用 useCallback 包裹，並綁定空dependency array（代表這個函式的地址一輩子不變）
const handleIncrement = React.useCallback(() => {
    setCount(prev => prev + 1)
}, []) 
```
* 當 `text` 改變時 ➡️ App Re-render ➡️ `useCallback` 發現依賴項 `[]` 沒變 ➡️ 直接回傳上次的舊函式地址。
* `ChildComponent` 的 `React.memo` 進行 Props 淺比較：發現 `onIncrement` 的引用地址與上一次完全相同 ➡️ 成功攔截阻斷，跳過無意義重繪！

## useMemo vs useCallback 傻傻分不清楚？  
這兩個 API 都是在做 Memoization（快取），但它們快取的「東西」不一樣：

| 特性 | `useMemo` | `useCallback` |  
| :--- | :--- | :--- |  
| **快取的目標** | 函式執行的 **「結果（值）」** | **「函式（Callback）」** 本身 |  
| **運作概念** | 「幫我執行這段計算，並**記住算出來的答案**。」 | 「先不要執行，**記住這個函式本身和它的記憶體地址**。」 |  
| **回傳內容** | 任何值（字串、數字、過濾後的陣列、物件...） | 一個可被呼叫的函式 `() => { ... }` |


### 程式碼等價關係：  
事實上，`useCallback` 只是 `useMemo` 的語法糖。以下兩段程式碼在 React 底層的運作完全一模一樣：

```javascript
// 1. 使用 useCallback (最直覺、推薦的做法)
const handleIncrement = React.useCallback(() => {
    setCount(prev => prev + 1)
}, [])

// 2. 使用 useMemo 達到一模一樣的效果 (必須回傳一個函式)
const handleIncrement = React.useMemo(() => {
    return () => {
        setCount(prev => prev + 1)
    }
}, [])
```

## 什麼時候「不要」使用？（避免盲目包裹）  
就像 `useMemo` 一樣，`useCallback` 本身也有快取和比對dependency array的效能開銷。

**❌ 這些時候請不要用：**
- 子元件根本沒有被 `React.memo` 包裹：  
如果子元件每次都會無條件重繪，辛辛苦苦在父元件幫函式加 `useCallback` 鎖定地址，子元件拿到一模一樣的地址依然會被重新渲染。這時的 `useCallback` 完全是在做白工，甚至倒貼了效能。

- 只是傳給一般的 HTML 元素（如 `<button onClick={...}>`）：

```javascript
// 🔴 這是完全沒必要的過度優化！
const handleClick = React.useCallback(() => {
    console.log("Clicked")
}, [])

return <button onClick={handleClick}>Click me</button>
```

原生 HTML 標籤（如 `button`）並不會因為傳入的 `onClick` 地址不同而產生 React 元件層級的效能問題。直接寫箭頭函式即可。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>