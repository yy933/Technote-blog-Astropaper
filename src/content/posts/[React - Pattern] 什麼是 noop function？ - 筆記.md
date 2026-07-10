---
title: "[React - Pattern] 什麼是 noop function？ - 筆記"
pubDatetime: 2026-07-10T08:15:59.408Z
tags: ["React.js","JavaScript","Design Pattern","Defensive Programming","Frontend"]
description: "Table of contents 核心觀念 noop (讀作noop) 是 \"No Operation\" 的縮寫，在..."
hackmd_id: "H117vmAmzx"
---

## Table of contents

## 核心觀念
**`noop`** (讀作no-op) 是 **"No Operation"** 的縮寫，在程式設計中指的是**「什麼都不做的函式」**或**「空函式」**。

在 JavaScript 與 React 開發中，它最常以箭頭函式 `() => {}` 或匿名函式 `function() {}` 的形式出現。它被呼叫時會正常執行、正常結束，但裡面沒有任何程式碼，因此不會產生任何行為，並預設回傳 `undefined`。


## 1. 為什麼我們需要 noop function？

在元件開發中，我們經常會設計Callback Functions作為屬性（Props）傳遞。如果外部在使用該元件時忘記傳入對應的 Callback，元件內部直接呼叫就會引發嚴重的 JavaScript 執行期錯誤（Runtime Error）。

因此，使用 noop function 作為**預設值（Default Props）**，是一種極為經典的 **Defensive Programming** 技巧，用來確保程式的強健性與安全性。

### 情境一：外部未傳入 Props 導致的程式崩潰 (Crash)  
假設我們寫了一個彈出視窗（Modal）或切換開關（Toggle）元件，並預期點擊時觸發外部的 `onToggle` 函式：

```javascript
// ❌ 沒寫預設值
export default function Toggle({ children, onToggle }) {
  // ...
  function handleClick() {
    onToggle(); // 如果外部沒傳入此 Prop，這裡會是 undefined
  }
}
```

當使用者點擊按鈕時，瀏覽器會直接噴出大錯：`TypeError: onToggle is not a function`，導致整個 React 應用程式瞬間白畫面崩潰。

### 情境二：使用 noop function 安全降級  
當我們為 `onToggle` 加上解構賦值的預設值 `() => {}`：

```javascript
// 🟢 加上預設值 (noop function)
export default function Toggle({ children, onToggle = () => {} }) {
  // ...
}
```

即便外部忘記傳入 `onToggle`，程式執行到 `onToggle()` 時實際上是在執行那個空函式。雖然什麼事都沒發生，但因為它是一個合法的函式，瀏覽器就不會噴錯，程式得已安全、平穩地繼續運行。

## 2. 實際應用場景
### 場景一：與 `useEffect` 搭配控制複合元件狀態  
在開發狀態與行為分離的複合元件（Compound Components）時，我們常利用 `useRef` 跳過第一次渲染，並在後續狀態改變時觸發 `onToggle`：

```javascript
import React from "react"

const ToggleContext = React.createContext()

export default function Toggle({ children, onToggle = () => {} }) {
    const [on, setOn] = React.useState(false)
    const firstRender = React.useRef(true) // 用來記錄是否為初次渲染
    
    function toggle() {
        setOn(prevOn => !prevOn)
    }

    React.useEffect(() => {
        // 利用 useRef 確保初次掛載不觸發回呼
        if (firstRender.current) {
            firstRender.current = false
        } else {
            onToggle() // 🟢 即使外部沒傳入，這裡觸發 noop 也不會崩潰
        }
    }, [on])

    return (
        <ToggleContext.Provider value={{ on, toggle }}>
            {children}
        </ToggleContext.Provider>
    )
}

export { ToggleContext }
```

### 場景二：單元測試（Unit Testing）中的 Dummy 參數  
在針對 React 元件進行單元測試（如使用 Jest 或 React Testing Library）時，若某個元件必須傳入特定的 Function Prop 才能正常運作，但該次測試的重點不在於該 Function 的行為，我們就能直接塞入一個 noop function。

```javascript
// 測試元件是否能正常渲染，直接傳入 noop 作為必填 prop
test("應該正常渲染 Toggle 元件", () => {
  render(<Toggle onToggle={() => {}}>點我</Toggle>);
});
```

## 3. 進階優化：獨立提取 noop 節省記憶體？  
在追求極致效能或撰寫大型基礎套件（如 Axios, Lodash）時，重複寫 `() => {}` 會在每次元件渲染時重新在記憶體中建立一個新的空函式實體。

雖然這個開銷微乎其微，但更優雅且老練的作法是在全域定義一個專用的 noop 變數：

```javascript
// 在元件外部定義一次，重複利用
const noop = () => {};

export default function Toggle({ children, onToggle = noop }) {
    // ...
}
```

這樣一來，無論元件重新渲染幾次，指向的都是記憶體中同一個 noop 實體，這也是許多開源框架底層常見的封裝手法。

## 4. 常見觀念問答
**Q: 既然只要確保它不噴錯，用 `onToggle?.()`（Optional Chaining）是不是也能達到一樣效果？**  
A: 可以，這也是現代 JavaScript 非常常見的替代方案。  
如果寫成 `onToggle?.()`，當 `onToggle` 是 `undefined` 時就會自動跳過不執行。  
不過，設定預設值 `onToggle = () => {}` 的優點在於：在元件內部的任何地方，你都可以非常有自信地直接呼叫 `onToggle()`，而不需要在每個呼叫點都小心翼翼地加上 `?.`，程式碼的可讀性與一致性會更高。

**Q: 寫了 noop 預設值後，外部傳入的函式還能拿到參數嗎？**  
A: 可以。預設值只有在外部「完全沒傳（`undefined`）」的時候才會啟動。如果外部有傳入標準的 callback 函式（如 `(status) => console.log(status)`），React 就會完全使用外部傳入的函式與參數，兩者互不干涉。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>