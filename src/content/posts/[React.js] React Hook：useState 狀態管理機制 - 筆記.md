---
title: "[React.js] React Hook：useState 狀態管理機制 - 筆記"
pubDatetime: 2026-06-22T09:23:02.215Z
tags: ["React.js","JavaScript","Frontend","React Hook"]
description: "Table of contents :memo: 簡介 在傳統的 JavaScript 開發中，我們可以直接修改變數（..."
hackmd_id: "rJai9uUzMx"
---

## Table of contents

## :memo: 簡介  
在傳統的 JavaScript 開發中，我們可以直接修改變數（如 `count = count + 1`），但這種作法在 React 的世界裡是無法驅動畫面更新的。**因為 React 無法得知變數在記憶體中的改變，進而錯失了重新繪製網頁的時機。**

為了讓網頁具有「動態互動性」，React 引入了 Hook 機制，而其中最核心、最常用的便是 **`useState`**。它代表的重大意義，就是**賦予 Functional Component（函式元件）擁有內部私人記憶與驅動畫面重新渲染（Re-render）的能力**。


## :memo: 核心概念

### 1. useState 的基本語法與功能  
`useState` 是一個函式，它接受一個參數作為狀態的**初始值**，並回傳一個包含兩個元素的**陣列**。我們通常會使用 JavaScript 的解構賦值（Destructuring）來接收它：

```javascript
const [state, setState] = useState(initialValue);
```

* `state`（當前的狀態值）： 用來在元件內部讀取目前的數據（在該次渲染中就像唯讀的常數快照）。
* `setState`（更新狀態的函式）： 修改這個狀態的唯一管道。呼叫此函式並傳入新值，會觸發 React 的重新渲染機制。
* `initialValue`（初始值）： 只會在元件第一次被建立（Mount）時作為預設值。

### useState 的特性與注意事項  
在使用 useState 時，有三個非常關鍵的底層特性需要特別注意，這也是初學者在開發 React 時最容易踩雷的地方：

#### 特性 1：狀態更新是「非同步」的（或稱 Batching 批次處理）  
呼叫狀態更新函式（如 `setCount`）後，無法在下一行程式碼立即拿到更新後的值。

```javascript
const [count, setCount] = useState(0);

const handleClick = () => {
  setCount(count + 1);
  console.log(count); // ❌ 這裡印出來的依然會是舊的 "0"！
};
```

* 原因： React 為了效能著想，會把同一個事件處理函式裡的所有狀態更新「收集起來」，等事件內的程式碼全部執行完畢後，才「一次性（批次）」統一更新狀態並重新渲染畫面。

* 解決方案： 如果下一個狀態需要依賴前一個狀態計算（例如希望連續加兩次），應該傳入一個Callback Function。這個Callback Function會保證接收到 React 記憶體中最即時、最新的舊狀態（通常命名為 prev）：

```javascript
setCount(prevCount => prevCount + 1); // prevCount 保證拿到最新當下的狀態
```

#### 特性 2：狀態是唯讀的（Immutable / 不可變性）  
如果 State 是物件（Object）或陣列（Array）等參考型別，不能直接去修改它的內部屬性。

```javascript
const [user, setUser] = useState({ name: 'Alex', age: 25 });

// ❌ 錯誤寫法：直接修改物件屬性。
// 雖然內容變了，但物件在記憶體中的指向（位址）沒變，React 偵測不到差異，因此不會重新渲染畫面！
user.age = 26; 
setUser(user); 

// ⭕ 正確寫法：用展開運算子（...）建立一個全新的物件，賦予新的記憶體指向
setUser({
  ...user,     // 複製舊有的屬性
  age: 26      // 覆蓋想要修改的屬性
});
```

#### 特性 3：每一次渲染都有它自己的 State 與事件處理函式  
每次 React 因為狀態改變而重新渲染元件時，**本質上都是重新呼叫一次該函式元件**。

* **在每次獨立的呼叫（渲染）中，count 變數都只是該次渲染中一個固定的「常數（Snapshot 快照）」。**
* 當前的事件處理函式（如 `handleClick`）所讀取到的，也只會是該次渲染鎖定的快照值，直到下一次狀態成功更新、元件重新被呼叫，才會得到全新渲染下的常數值。

## :memo: 函式型狀態更新 (Functional Updates)  
結合上述的「非同步批次處理」特性，當我們需要依賴舊狀態（Old State）來決定或計算出新狀態（New State）時，為了避免拿到過期的舊資料（Stale State），我們應該傳遞一個callback function給 Setter。

### 錯誤示範（直接依賴舊變數變更）  
預期點擊按鈕後一次加 3：

```javascript
const handleClick = () => {
  setCount(count + 1); // 當前 count 為 0，React 收到指令：「下次請幫我改成 0 + 1」
  setCount(count + 1); // 當前 count 仍為 0，React 收到指令：「下次請幫我改成 0 + 1」
  setCount(count + 1); // 當前 count 仍為 0，React 收到指令：「下次請幫我改成 0 + 1」
  // 最終結果：畫面數字只加了 1 
};
```

### 正確示範（傳入callback function給 Setter）

```javascript
const handleClick = () => {
  setCount(prevCount => prevCount + 1); // 0 => 1
  setCount(prevCount => prevCount + 1); // 1 => 2
  setCount(prevCount => prevCount + 1); // 2 => 3
  // 最終結果：畫面數字正確加了 3
};
```

## :memo: 實作分享  
這裡實作一個簡單的「計數器」元件，展示 `useState` 狀態宣告、唯讀更新與函式型更新的綜合運用。

```javascript
// Counter.jsx
import React, { useState } from 'react';

function Counter() {
  // 1. 宣告一個 count 狀態，初始值為 0
  const [count, setCount] = useState(0);

  // 2. 點擊加 1 的邏輯（標準更新）
  const handleIncrement = () => {
    setCount(count + 1); // ⭕ 正確：不修改 count 本身，而是傳入計算後的新值
    
    // ❌ 絕對不可寫成 count++ 或 ++count！這會直接改動 const 常數且違反不可變性
  };

  // 3. 點擊加 3 的邏輯（函式型更新，避免非同步過期資料）
  const handleIncrementThree = () => {
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
  };

  return (
    <div style={{ padding: '20px', textAlign: 'center', fontFamily: 'Arial' }}>
      <h2>useState 實作示範</h2>
      <p style={{ fontSize: '24px' }}>目前計數：<strong>{count}</strong></p>
      
      {/* 呼叫標準更新 */}
      <button onClick={handleIncrement} style={{ marginRight: '10px', padding: '10px' }}>
        安全加 1
      </button>
      
      {/* 呼叫連續更新 */}
      <button onClick={handleIncrementThree} style={{ padding: '10px' }}>
        連續加 3 (Functional Update)
      </button>
    </div>
  );
}

export default Counter;
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

:pushpin: Points：

* 為什麼不能寫 `setCount(count++)`？  
後置遞增（`count++`）會先將當前的值（例如 0）傳給 setCount，事後才在記憶體試圖修改 count 的值。這會導致 React 接收到「請把狀態設定為 0」的指令，畫面完全不會更新，且引發 JavaScript 的 const 賦值錯誤（Assignment to constant variable）。

* 為什麼不能寫 `setCount(++count)`？  
前置遞增（`++count`）雖傳入了加 1 後的值，但本質上它仍然先執行了 `count = count + 1` 這一行直接改動狀態常數的越權行為，觸犯了 React 的不可變性鐵律。

</blockquote>

## 小結  
簡單總結 useState 的核心運作邏輯：
**觸發事件處理 ➡️ 呼叫設定函式傳入新值 ➡️ React 重新呼叫元件（Re-render） ➡️ 網頁 UI 畫面自動更新。**

在處理複雜資料結構（如物件或陣列）的 State 時，也務必記得要用展開運算子（`...`）複製出一份全新記憶體位址的物件再傳入，隨時銘記「狀態不可變性」與「渲染快照」的概念，就能輕鬆駕馭 React 的狀態管理！

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>