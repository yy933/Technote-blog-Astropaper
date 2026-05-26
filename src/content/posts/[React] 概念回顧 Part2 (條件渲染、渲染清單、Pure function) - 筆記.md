---
title: "[React] 概念回顧 Part2 (條件渲染、渲染清單、Pure function) - 筆記"
pubDatetime: 2026-05-26T03:01:46.990Z
tags: ["JavaScript","React.js","Interview Preparation"]
description: " Table of contents 本篇回顧涵蓋的筆記範圍 [[React] 條件渲染 (Conditional Re..."
---

## Table of contents

## 本篇回顧涵蓋的筆記範圍
* [[React] 條件渲染 (Conditional Rendering) - 筆記](https://hackmd.io/tx8UqNHNQ0aw27hXc4FMLw)
* [[React] 管理渲染清單 (Rendering Lists) - 筆記](https://hackmd.io/6F21PDAPTUaoJdYMilaxGw)
* [[React] Pure Function - 筆記](https://hackmd.io/qIh7AQhiSoSdnFeuOxdC6w)



## Conditional Rendering 是很常見的情境問題，如果 render 的結果只有兩種變化，可以使用什麼方式來撰寫邏輯？

1. 三元運算子（Ternary Operator）
最常見、可讀性佳，適合 A / B 二選一的情境：
```jsx
{isLoggedIn ? <LogoutButton /> : <LoginButton />}
```

2. 邏輯 AND 運算子（&&）
只在條件成立時顯示某個元素，否則什麼都不顯示：
```jsx
{isLoggedIn && <LogoutButton />}
```

3. Early Return
適合在整個元件內容中，某種條件下不渲染或只渲染特定內容：
```jsx
if (!isLoggedIn) {
  return <LoginPrompt />;
}
return <Dashboard />;
```

:bulb: 「一切都是元件」= React 視 UI 為函式式元件的組合，這讓開發者能更簡潔、可維護、模組化地建構 UI。

:arrow_right: 參考 [[React] 條件渲染 (Conditional Rendering) - 筆記](https://hackmd.io/tx8UqNHNQ0aw27hXc4FMLw)



## 如何選定適合的 key？

選定適合的 key 是 React 中管理 list 元素更新效能的關鍵，**好用的 key 能幫助 React 更快辨識哪些項目被新增、刪除或重新排序。**

### 最佳做法：使用「資料中獨一無二且穩定的值」
例如：**資料庫的 id、唯一 username、UUID 等**
```jsx
{items.map(item => (
  <li key={item.id}>{item.name}</li>
))}
```

### 不建議：使用陣列的 index 當作 key（除非資料永遠不變動）
```jsx
// 這樣寫可能導致重新排序時產生 bug
{items.map((item, index) => (
  <li key={index}>{item.name}</li>
))}
```
--> 為什麼不好？
* 順序一變，React 會誤判哪些項目需要更新，導致錯誤渲染或效能下降。
* 對表單欄位特別危險（可能輸入錯位）。

### 避免：使用不穩定的 key（如隨機產生的值）
不要動態生成 key，例如 `key={Math.random()}`。這會導致每次渲染時 key 都不同，進而讓所有的元件和 DOM 元素重新渲染，不僅會影響性能，還會讓user input消失。
```jsx
<li key={Math.random()}>{item.name}</li> // 每次 render 都是新 key，會整個重繪
```

:bulb: 選擇key的原則：
* 唯一：同一 list 裡不能有重複的 key
* 穩定：重新 render 時，key 不會變動
* 可預測：來自資料本身的特徵欄位最合適

:arrow_right: 參考 [[React] 管理渲染清單 (Rendering Lists)  - 筆記](https://hackmd.io/6F21PDAPTUaoJdYMilaxGw#%E9%87%8D%E9%BB%9E%E6%A6%82%E5%BF%B5%EF%BC%9AKey-%E7%9A%84%E4%BD%BF%E7%94%A8)


## 什麼是 Pure Function？為什麼 Pure function 在 React 裡面是重要的？

### 什麼是 Pure Function？
Pure Function 有兩個核心特徵：
1. 相同輸入，一定會產生相同輸出
* 不依賴函式外部的變數或狀態
* 沒有隨機性、時間性或外部side effects影響
2. 不產生side effects
* **不會改變外部狀態（如全域變數、DOM、資料庫等）**
* 僅根據輸入產生回傳值
```jsx
// 純函式
function add(a, b) {
  return a + b;
}
// 非純函式（有side effects）
let count = 0;
function addWithSideEffect(a) {
  count += a; // 改變外部變數
  return count;
}
```
### 為什麼 Pure function 在 React 裡面是重要的？
React 的核心哲學就是：「UI = f(state)」——介面是狀態的純函式映射。因此：

* 更容易預測與測試
相同的 props 傳入 component，render 出來的結果應該要一樣。
* 更容易追蹤 bug
不會意外改到其他狀態或資料，debug 較單純。
* React 效能優化（例如 memoization）
React 依賴**props 沒變就不重 render**，而純函式保證這件事是安全的。

#### 需要side effects怎麼辦？
如果需要side effects（例如資料請求、訂閱、DOM 操作等），React 建議使用` useEffect()` ，這樣可以清楚區分哪些地方是pure function、哪些是有side effects的。

:bulb: 使用Pure Function 更容易預測與測試、更容易追蹤 bug、也更方便效能優化。

:arrow_right: 參考 [[React] Pure Function - 筆記](https://hackmd.io/qIh7AQhiSoSdnFeuOxdC6w)