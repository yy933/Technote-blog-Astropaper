---
title: "在 JSX 中使用 JavaScript - 筆記"
pubDatetime: 2025-04-28T22:19:14.000Z
modDatetime: 2026-05-25T10:04:23.530Z
tags: ["JavaScript","React.js"]
description: "Table of contents JSX 讓我們可以在 JavaScript 中寫 HTML 樣式的標記語法，並透過..."
hackmd_id: "Skblsk0Jxe"
---

## Table of contents

JSX 讓我們可以在 JavaScript 中寫 HTML 樣式的標記語法，並透過 `{} `將變數、邏輯、函式或物件嵌入標記中。

## 如何在 JSX 中撰寫表達式。

### 傳遞字串：用引號`" "` 包起來
`<img src="url" alt="文字說明" />`
使用引號：會被當成「字串」

### 傳遞變數，要用 `{}` 包起來：
```jsx
const avatar = 'https://img.url';
<img src={avatar} alt="說明" />
```

### JSX是撰寫JavaScript的一種特殊寫法
* JavaScript 變數
```jsx
export default function TodoList() {
  const name = 'Gregorio Y. Zara';
  return (
    <h1>{name}'s To Do List</h1>
  );
}
```
* 函式呼叫
```jsx
const today = new Date();

function formatDate(date) {
  return new Intl.DateTimeFormat(
    'en-US',
    { weekday: 'long' }
  ).format(date);
}

export default function TodoList() {
  return (
    <h1>To Do List for {formatDate(today)}</h1>
  );
}
```

* 物件屬性存取
```jsx
const person = { name: 'Gregorio' };
<h1>{person.name}'s List</h1>
```

## 如何在 JSX 使用 `{}`
只能在 兩個地方 使用 {}：
1. 標籤中的文字內容：
```jsx
<h1>{name}'s To Do List</h1> // OK!
<{tag}>Text</{tag}>          // ❌ 標籤名稱不能用變數
```
2. 屬性值中（等號後面）：
```jsx
src={avatar}       // OK，插入變數
src="{avatar}"     // ❌ 變成字串 "{avatar}" 而不是變數
```

## 如何在 JSX 中傳入物件 / 使用 inline CSS styles。
使用物件：雙層大括號` {{ }}`
因為 JSX 中的 {} 是 JS 表達式，若要插入 JS 物件，就會是：
```jsx
<div style={{ backgroundColor: 'black', color: 'pink' }}></div>
```
* 第一層(外層) `{}`：JSX 表達式
* 第二層(內層) `{}`：JavaScript 物件本體
```jsx
const person = {
  name: 'Gregorio',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};
<div style={person.theme}>
  <h1>{person.name}'s Todos</h1>
</div>
```

inline CSS styles:
```jsx
export default function TodoList() {
  return (
    <ul style={{
      backgroundColor: 'black',
      color: 'pink'
    }}>
      <li>Improve the videophone</li>
      <li>Prepare aeronautics lectures</li>
      <li>Work on the alcohol-fuelled engine</li>
    </ul>
  );
}
```
### 注意事項

* 不能用 HTML 的 class 屬性 → 要寫成 className
* style 屬性要用 camelCase，例如 backgroundColor
* 不能這樣寫：`src="{avatar}"`（這樣是傳字串，而不是變數）

## Recap
| 類型                     | 正確範例                                       | 錯誤範例                     |
|--------------------------|------------------------------------------------|------------------------------|
| 插入變數到內容           | `<h1>{name}'s List</h1>`                       | `<h1>"{name}"</h1>`          |
| 插入變數到屬性值         | `src={avatar}`                                | `src="{avatar}"`             |
| 插入 JS 物件（如 style） | `style={{ color: 'red' }}`                    | `style="{ color: 'red' }"`   |
| 標籤名稱                 | ✘ 不可寫成 `<{tag}>`                          | —                            |