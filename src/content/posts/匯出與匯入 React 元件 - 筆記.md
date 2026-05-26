---
title: "匯出與匯入 React 元件 - 筆記"
pubDatetime: 2026-05-25T11:17:37.235Z
tags: ["JavaScript","React.js"]
description: " Table of contents React 元件的最大價值之一是 「可重用性」 (reusability)。隨著元..."
---

## Table of contents

React 元件的最大價值之一是 **「可重用性」** (reusability)。隨著元件越來越多，建議將它們拆分成不同檔案，讓程式碼更清晰、更易維護。

## 什麼是 Root Component File？
* 一開始可能會將所有元件寫在同一個檔案中（如 `App.js`）。
* 隨著專案擴大，將元件獨立出來是常見做法。
* 在框架如 Next.js 中，**每個頁面可能就是一個 root component**。

## 如何匯出與匯入元件？
### 匯出元件（Export）
* Default Export（預設匯出）：
`export default function Gallery() { ... }`

* Named Export（具名匯出）：
`export function Profile() { ... }`


### 匯入元件（Import）
* Default Import：
`import Gallery from './Gallery.js';`

* Named Import：
`import { Profile } from './Gallery.js';`


## 為什麼要拆成多個檔案？
* 讓元件更模組化與可重用
* 更容易維護與擴充
* 避免單一檔案過於龐大

## 多個元件如何從同一檔案匯出？
* 一個檔案可以：
  - 有 **一個 `default export`**
  - 有 **多個 `named exports`**


* 範例：
`Gallery.js`
```jsx
export function Profile() {
  // ...
}

export default function Gallery() {
  // ...
}
```

`App.js`
```jsx
import Gallery from './Gallery.js';         // default import
import { Profile } from './Gallery.js';     // named import

export default function App() {
  return <Profile />; // 或 <Gallery />
}
```

## 其他注意事項
* `.js` 副檔名可以省略 (`./Gallery` vs `./Gallery.js`)
* 有些團隊會統一使用一種匯出方式，減少混用造成的混淆



## Recap
✅ 什麼是 root component file
✅ 如何匯出 / 匯入元件
✅ Default 與 Named 的用法與差異
✅ 同檔案多元件的匯出方式
✅ 模組化結構讓程式碼更清晰、更好用