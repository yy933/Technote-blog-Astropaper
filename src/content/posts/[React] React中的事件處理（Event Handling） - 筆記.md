---
title: "[React] React中的事件處理（Event Handling） - 筆記"
pubDatetime: 2026-05-26T03:29:26.518Z
tags: ["JavaScript","React.js"]
description: " Table of contents 基本用法：加上事件處理函式 jsx export default function..."
---

## Table of contents


## 基本用法：加上事件處理函式
```jsx
export default function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```
* `handleClick` 是事件處理函式
* 透過 `onClick={handleClick}` 作為prop傳入 button
* 命名慣例：handleClick, handleMouseEnter
* 也可以這樣寫:
```jsx
<button onClick={() => alert('You clicked me!')}>Click me</button>
```
<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
### ⚠️ 常見錯誤：不小心「呼叫」了函式
```jsx
<button onClick={handleClick}> ✅ 傳入函式
<button onClick={handleClick()}> ❌ 呼叫函式（在渲染階段會直接執行）
    
<button onClick={() => alert('...')}> ✅ 傳入匿名函式
<button onClick={alert('...')}> ❌ 立即執行 alert
```
</div>

## 讀取事件處理器中的 props
```jsx
function AlertButton({ message, children }) {
  return <button onClick={() => alert(message)}>{children}</button>;
}
```