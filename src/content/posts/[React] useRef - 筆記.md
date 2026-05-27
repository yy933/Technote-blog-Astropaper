---
title: "[React] useRef - 筆記"
pubDatetime: 2025-05-15T19:09:24.000Z
tags: ["JavaScript","React.js","React Hook"]
description: "Table of contents useRef 是什麼？ useRef 是一個可以保存資料，但不會觸發重新渲染的 H..."
hackmd_id: "ryfmizX-gl"
---

## Table of contents

## useRef 是什麼？
`useRef` 是一個**可以保存資料，但不會觸發重新渲染**的 Hook。
```jsx
const myRef = React.useRef(initialValue)
```
* 回傳一個物件：`{ current: initialValue }`
* 可以修改 current 的值，但不會讓元件重新 render

## useRef 和 useState的差別
`useRef` 和 `useState` 很像，都是用來保存資料，但差異在於：
* Ref 的 `.current` 可以直接修改，不需要透過 `setter`（不像 `useState` 要用 `setState`）。
* 修改 `.current` 不會觸發元件重新渲染，而 `useState` 會。
* `Ref`常用來存取DOM元素，而不需像過去那樣用 `id` 搭配 `document.getElementById()`。

| 特性            | `useRef`                  | `useState`           |
| ------------- | ------------------------- | -------------------- |
| 直接修改資料？       | ✅ 可以（`ref.current = xxx`） | ❌ 不行（需 `setState()`） |
| 是否會 re-render | ❌ 不會                      | ✅ 會                  |
| 常見用途          | 存 DOM、記錄值、避免 re-render    | 控制 UI、畫面需更新的資料       |

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

* 雖然可以直接修改 `ref.current`，但不應該用來取代 `state` 控制畫面。
* ref 適合拿來**保存「不影響畫面但需要記住的值」或「DOM 元素」**。

</blockquote>

## 範例一：點擊按鈕，自動聚焦到輸入框
```jsx
import React, { useRef } from "react"

export default function FocusInput() {
  const inputRef = useRef(null)

  function handleClick() {
    // 透過 ref 存取 DOM 元素並聚焦
    inputRef.current.focus()
  }

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="Focused Input" />
      <button onClick={handleClick}>Focus Input</button>
    </div>
  )
}
```
### 說明
| 用法                         | 說明                               |
| -------------------------- | -------------------------------- |
| `useRef(null)`             | 建立一個可以綁定 DOM 元素的 ref，初始值為 `null` |
| `ref={inputRef}`           | 把這個 ref 綁定到 `input` 元素上          |
| `inputRef.current.focus()` | 透過 JavaScript 使 input 聚焦       |

## 範例二：統計點擊次數，但畫面不更新
### 情境:想要記錄使用者點了幾次按鈕，但不想讓畫面每次都重新 render
```jsx
import React, { useRef, useState } from "react"

export default function ClickTracker() {
  const clickCount = useRef(0)
  const [_, forceUpdate] = useState(false) // 只用來強制更新畫面一次看結果

  function handleClick() {
    clickCount.current += 1
    console.log("目前點擊次數：", clickCount.current)
    // 畫面不會重新 render！
  }

  return (
    <div>
      <button onClick={handleClick}>點我統計次數</button>
      <p>點擊次數只會顯示在 console 中。</p>
      <p>（目前畫面並不會因為點擊而更新）</p>
    </div>
  )
}
```
### 說明
| 概念                      | 說明                            |
| ----------------------- | ----------------------------- |
| `useRef` 不會造成 re-render | 改變 `.current` 的值時，不會觸發畫面更新    |
| 用來存「不需要觸發 UI 更新」的資料     | 如：點擊次數、計時器、前一個 props/state 值等 |
| `.current` 是 ref 的真正值   | 它就像是一個可變物件屬性，React 不會追蹤它變了沒   |

## Recap
若資料會影響畫面顯示 ➜ 用 `useState` (會重新渲染畫面)；
若資料只需記錄或操作 DOM ➜ 用 `useRef`