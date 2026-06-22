---
title: "[React.js] 應用 useState 時的常見錯誤 - 筆記"
pubDatetime: 2026-06-22T09:49:43.743Z
tags: ["React.js","React Hook","Issue","Frontend","JavaScript"]
description: "Table of contents :memo: 操作 useState 的常見錯誤 常見錯誤 1：將 useStat..."
hackmd_id: "S1yIlFLzGg"
---

## Table of contents

## :memo: 操作 useState 的常見錯誤
### 常見錯誤 1：將 `useState` 寫在事件處理函式（Event Handler）內部

```javascript
   function changeMind() {
     // ❌ 錯誤：在點擊事件觸發時才宣告狀態
     const [isGoingOut, setIsGoingOut] = React.useState(false)
     setIsGoingOut(prev => !prev)
   }
```

### 常見錯誤 2：使用傳統陣列方法直接修改陣列狀態
```javascript
// ❌ 錯誤：嘗試直接 push 元素進去
setMyFavoriteThings(prevFavThings => prevFavThings.push("Item"))
```

這樣寫到底會引發什麼問題？違反了 React 的哪些核心底層設計？

## :memo: 核心原因與觀念拆解
### 1. Hook 只能在元件最頂層呼叫 (Hooks Rule)  
React 官方對所有以 `use` 開頭的 Hook 有一條不容違背的規定：**只能在 React 的「函式元件」或「自訂 Hook」的最頂層（Top Level）呼叫。 絕對不能寫在一般 JS 函式、事件處理器、`if` 判斷式或 `for` 迴圈內。**

**為什麼 React 要有這個限制？**
* **底層追蹤機制**： React 底層在紀錄元件的狀態時，是以「呼叫順序」作為依據，將 Hook 依序存放在一個內部陣列/鏈結串列中。
* **順序錯亂的後果**： 如果將 Hook 放進點擊才觸發的事件函式中，每次渲染時 Hook 的呼叫次數和順序就會變得動態、不可預測。React 就會徹底失去對應狀態的能力，導致狀態管理機制全面崩潰。
* **生命週期悖論**： 在畫面渲染時，若狀態被鎖在事件函式裡，元件根本讀取不到該變數（引發 ReferenceError）；且每次點擊都會現場「重新蓋一座預設值記憶庫」，導致舊的修改在重新渲染時瞬間灰飛煙滅。

### 2.遵守陣列與物件的「不可變性 (Immutability)」  
在 React 中，陣列（Array）和物件（Object）屬於參考型別（Reference Type）。**React 的狀態變更偵測是比對前後兩次狀態的「記憶體指向（位址）」，而非逐一檢查內部的資料。**

**為什麼不能寫 `prevFavThings.push("Item")`？**
* **致命傷一：push() 的回傳值是長度！**  
JS 的 `Array.prototype.push()` 執行後回傳的是「陣列的新長度（數字）」，而不是新陣列。如果你這樣寫，狀態會從原本的 Array 突變成 Number（例如數字 3），直接破壞資料結構。
* **致命傷二：改動了原陣列（Mutation）！**  
`push()`、`pop()`、`splice()` 等方法會直接修改原始記憶體裡的陣列。**對於 React 而言，雖然陣列內容變了，但它的記憶體指標（位址）完全沒變。React 檢查時會認為「資料沒有變動」，進而跳過重新渲染，導致網頁畫面完全卡死。**



## :memo: 正確的實作最佳實踐 (Best Practice)
### 1. 狀態切換元件的正確結構  
必須「先擁有記憶，才能修改記憶」。狀態必須宣告在元件一出生（最頂層）的地方：

```javascript
import React from "react"

export default function App() {
    // ⭕ 正確：在元件最頂層宣告狀態，確保生命週期內一直被記住，且任何地方都讀得到
    const [isGoingOut, setIsGoingOut] = React.useState(false)
    
    function changeMind() {
        // 函式內部只負責「發出修改記憶的指令」
        setIsGoingOut(prev => !prev)
    }

    return (
        <main>
            <h1 className="title">Do I feel like going out tonight?</h1>
            <button onClick={changeMind} className="value">
                {isGoingOut ? "Yes" : "No"}
            </button>
        </main>
    )
}
```

### 2. 陣列狀態更新的正確做法  
要更新陣列或物件狀態，正確的思維是：「**在記憶體中建立一個全新的複本，修改複本後再投遞給更新函式。**」 我們應該**改用展開運算子（Spread Operator）**：

```javascript
setMyFavoriteThings(prevFavThings => {
    // ⭕ 正確：[...arr] 會建立一個擁有全新記憶體位址的陣列，並複製舊有內容
    return [
        ...prevFavThings,
        "Item" // 在新陣列最後方補上新資料
    ]
})
```

這樣新舊陣列的記憶體指向不同，React 就能瞬間偵測到差異，高高興興地幫你將新資料渲染到畫面上！

## Recap：陣列方法防踩雷清單  
在 React 中操作陣列狀態時，請務必牢記以下黑白名單：

| ❌ 禁用（會直接改動原陣列，導致畫面不更新） | ⭕ 建議改用（會回傳全新陣列，安全觸發渲染） |  
| :--- | :--- |  
| `push()` (尾部新增) / `unshift()` (頭部新增) | 展開運算子 `[...arr, item]` / `[item, ...arr]` |  
| `pop()` (尾部刪除) / `shift()` (頭部刪除) | `filter()` (過濾特定元素) / `slice()` (切片複製) |  
| `splice()` (拼接/刪除特定位址元素) | `filter()` 或組合展開運算子 |  
| `reverse()` (反轉) / `sort()` (排序) | 先用展開運算子複製 `[...arr].sort()` 或 `[...arr].reverse()` 再操作 |

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>