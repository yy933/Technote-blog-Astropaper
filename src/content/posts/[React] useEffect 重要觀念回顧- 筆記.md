---
title: "[React] useEffect 重要觀念回顧- 筆記"
pubDatetime: 2025-05-14T21:44:42.000Z
modDatetime: 2026-05-25T10:04:23.584Z
tags: ["JavaScript","React.js"]
description: "Table of contents React 元件為何被視為「純函式（pure function）」？ 給定相同的..."
hackmd_id: "ryjVTe7Wgl"
---

## Table of contents

## React 元件為何被視為「純函式（pure function）」？
* 給定相同的 props 或 state，元件會產出一樣的 UI。
* 渲染本身不應產生side effect，例如不應直接修改外部系統或狀態。

## 什麼是side effect？常見例子？  
side effect = 影響或連動到 React 控制範圍以外的行為 (外部行為)

常見例子：
* 存取或修改 localStorage
* 呼叫 API
* 操作真實 DOM（非 React 處理的）
* 使用 WebSocket、setTimeout 等

## 哪些不是side effect？  
React 自己會處理的，不算side effect。

非side effect範例：

* 更新 state
* 將資料渲染成 DOM
* 讓 UI 隨資料變化而更新

## useEffect 什麼時候會執行？什麼時候不會？  
🟢 會執行：

* 元件第一次渲染時
* dependencies array有變化時的每次 re-render

🔴 不會執行：

dependencies array沒變，React 就不會重新執行 useEffect

## 什麼是dependencies array？  
是 `useEffect`的第二個參數  
告訴 React：只有在這些變數變動時才需要重新執行副作用

例如：

```jsx
useEffect(() => {
  // side effects logic
}, [count, user])
```
➡️ 只有 count 或 user 變了，這段 effect 才會重跑。