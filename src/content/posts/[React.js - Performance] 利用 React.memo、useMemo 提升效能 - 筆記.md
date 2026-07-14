---
title: "[React.js - Performance] 利用 React.memo、useMemo 提升效能 - 筆記"
pubDatetime: 2026-07-14T08:38:46.462Z
tags: ["React.js","useMemo","Performance","Referential Equality","Advanced React","React Hook","cache","Frontend"]
description: "Table of contents 核心觀念：避免重複且無意義的重繪與計算 在 React 的渲染流水線中，效能優化主..."
hackmd_id: "B1W4qPXVfg"
---

## Table of contents

## 核心觀念：避免重複且無意義的重繪與計算  
在 React 的渲染流水線中，效能優化主要圍繞在「如何減少不必要的運算與 Virtual DOM 樹的比對」。這兩個 API 正是實現 **Memoization（快取機制）** 的兩大利器，但它們守備的範圍完全不同：

* **`React.memo`**：**針對元件（Component）**。防止子元件在 Props 沒有改變的情況下，因為父元件的 Re-render 而跟著被迫重新渲染。
* **`useMemo`**：**針對數值與計算結果（Value / Operation）**。防止元件在每次 Re-render 時，重複執行耗時、昂貴的 JavaScript 運算，並用來**鎖定參照型態的記憶體地址**。


## 1. `React.memo`：組件重繪的防護罩

### 運作機制  
預設情況下，**React 的元件是「一人得道，雞犬升天」——父元件更新，底下所有的子元件一律無條件重繪。**

使用 `React.memo` 包裹子組件後，React 會在父組件更新時進行攔截，將新舊 Props 進行 **淺比較（Shallow Comparison）**：
* 若 Props **完全相同** ➡️ 直接沿用上一次渲染好的 Virtual DOM，跳過本次執行。
* 若 Props **有所變更** ➡️ 正常執行組件重繪。

### 範例程式碼

```javascript
import React from 'react';

const ChildComponent = React.memo(({ title }) => {
  console.log("ChildComponent 渲染");
  return <h2>{title}</h2>;
});

export default ChildComponent;
```

## 2. useMemo：昂貴計算的快取記憶卡
### 運作機制  
當元件內部有需要消耗大量 CPU 運算資源的邏輯（例如：大量資料過濾、大陣列排序）時，我們不希望每次 Render（可能只是因為無關的 State 改變）都重新跑一次該程式。useMemo 會將計算結果保存於記憶體中，只有當指定的依賴陣列（Dependency Array）內容發生變化時，才會重新執行計算函式；否則，一律以 $O(1)$ 的速度直接回傳暫存的值。

### 範例程式碼

```javascript
import React, { useMemo } from 'react';

function ProductList({ products, filterText }) {
  // 只有在 products 或 filterText 改變時，才重新跑大數據過濾，避免畫面卡頓
  const filteredProducts = useMemo(() => {
    console.log("執行昂貴的大數據過濾...");
    return products.filter(p => p.name.includes(filterText));
  }, [products, filterText]);

  return (
    <ul>
      {filteredProducts.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

## 3. `React.memo()`經典地雷：Referential Equality  
在 JavaScript 中，比較變數相等性（===）時分為兩種機制：

* Primitive Types（原始型態，如數字、字串、布林值）：比對 「值（Value）」 是否相等。
* Reference Types（參照型態，如物件、陣列、函式）：比對 「記憶體地址（Reference）」 是否相等。

🚨 這是實務上最常讓 `React.memo` 破功的兇手：  
每次只要用大括號 `{}` 宣告一個新物件，JavaScript 就會在記憶體中開闢一塊全新、獨一無二的地址。內容長得再像，`地址 A === 地址 B` 永遠都是 `false`。

### 範例程式碼

```javascript
import React from "react"
import GrandParent from "./GrandParent" // 假設內部使用了 React.memo 包裹

export default function App() {
    const [count, setCount] = React.useState(0)
    const [darkMode, setDarkMode] = React.useState(false)

    // 🔴 每次 App 因為 count 改變而 Re-render 時，此處都會在記憶體重新產生一個全新的物件地址！
    const style = {
        backgroundColor: darkMode ? "#2b283a" : "#e9e3ff",
        color: darkMode ? "#e9e3ff" : "#2b283a",
    }
    
    // ❌ 雖然操作 count 時 style 內容完全沒變，但因為地址變了，此 Effect 每次都會被觸發
    React.useEffect(() => {
        console.log("style changed")
    }, [style])

    return (
        <div className="container">
            <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
            <button onClick={() => setDarkMode(prev => !prev)}>Toggle DarkMode</button>
            
            {/* ❌ 即使 GrandParent 被 React.memo 保護，也會因為 style 地址每次都不同而強迫 Re-render！ */}
            <GrandParent style="{style}"/>
            
            {/* ✓ 沒有傳 style 物件的 GrandParent 則能安全保住，不會受 count 影響 */}
            <GrandParent/> 
        </div>
    )
}
```

#### 說明  
效能優化失敗的連鎖反應：
* 點擊 `count` ➡️ App 元件 Re-render ➡️ `App()` 函式從頭重新執行。
* 執行到 `const style = {...}`，瀏覽器在記憶體中開闢全新地址 B。
* `useEffect` 比對dependency：`新地址 B === 舊地址 A` 結果為 `false` ➡️ 觸發 Side Effect 噴出 `"style changed"`。
* 子元件 `<GrandParent style={style} />` 進行 Props 淺比較：發現 `style` 的引用地址變了 ➡️ `React.memo` 判定 Props 改變，子元件無奈跟著集體重繪。

## 4. 解決方法：使用 `useMemo` 鎖定記憶體參照  
要中斷這個無意義的連鎖更新，必須使用 `useMemo` 將物件的記憶體地址「固定」起來：

```javascript
// ✓ 只有當真正與外觀相關的 darkMode 改變時，才允許重新配置記憶體地址
const style = React.useMemo(() => {
    return {
        backgroundColor: darkMode ? "#2b283a" : "#e9e3ff",
        color: darkMode ? "#e9e3ff" : "#2b283a",
    }
}, [darkMode])
```
#### 說明
* 當 `count` 改變時 ➡️ `App` Re-render ➡️ `useMemo` 發現dependency `[darkMode]` 沒變 ➡️ 直接回傳上次的舊地址 A。
* `useEffect` 與 `GrandParent` 的 `React.memo` 淺比較皆判定：`地址 A === 地址 A` 為 `true` ➡️ 成功攔截阻斷，跳過無意義更新！

## 5. 效能優化的黃金法則：不要過度優化 (Premature Optimization)
> "Memoization is not free."

**任何優化都是有代價的。比對新舊 Props、宣告快取記憶體空間、監聽依賴陣列，本身都需要消耗額外的 CPU 與記憶體開銷。**

❌ 什麼時候「不應該」使用？
- **元件結構極其簡單：**  
如果子元件只是渲染幾行 HTML，重繪的代價極低，這時強行加上 `React.memo` 去做 Props 比較，反而是在幫瀏覽器增加額外的工作量。

- **簡單的日常運算：**  
`const total = useMemo(() => a + b, [a, b]) `是經典的過度優化反面教材。簡單的加減乘除直接計算即可，比對依賴陣列的成本甚至高於直接相加。

- **Props 天生就高頻率變動：**  
如果子元件的 Props 本身就是每秒都在變的數據（例如計時器、滑鼠座標），`React.memo` 每次比較都會得出「不相同」的結論，這會導致每次都必須重繪，Props 比對直接變成了雙重耗能。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>