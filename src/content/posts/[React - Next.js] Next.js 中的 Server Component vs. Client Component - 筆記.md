---
title: "[React - Next.js] Next.js 中的 Server Component vs. Client Component - 筆記"
pubDatetime: 2026-05-25T11:17:36.388Z
tags: ["React.js","Node.js","Next.js"]
description: " Table of contents Server Component vs. Client Component 類型 ..."
---

## Table of contents

## Server Component vs. Client Component
| 類型              | Server Component                  | Client Component |
| --------------- | --------------------------------- | ----------------------------------- |
| **執行位置**        | 伺服器端 (Node.js 環境)                 | 使用者的瀏覽器端（Browser）                   |
| **預設行為**        | 預設為 Server Component              | 需要加 `use client` 宣告                 |
| **能否使用 hook**   | ❌ 不可以用 React Hooks（如 `useState`）  | ✅ 可以使用 Hooks                        |
| **能否使用瀏覽器 API** | ❌ 不可以（如 `localStorage`, `window`） | ✅ 可以使用                              |
| **效能表現**        | 更快載入，減少 bundle size               | 較慢，會額外產生 JS bundle                  |
| **用途**          | 資料庫查詢、資料處理、SEO 頁面產生               | 表單互動、動畫、狀態管理等需要 JS 的部分              |

## Server Component 的特點
* 適合用於：

  - 擷取資料（例如：從 DB、API 拿資料）
  - 渲染靜態或 SSR 頁面
  - 初始渲染不需要使用者互動的部分

* 無法做的事：

  - 使用 `useState`, `useEffect`, `useContext`
  - 存取瀏覽器 API，如 window, document

📍預設情況下，Next.js所有 component 都是 Server Component！
```jsx
// 預設就是 Server Component
export default function ProductList({ products }) {
  return (
    <ul>
      {products.map((p) => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

## Client Component 的特點
使用方式：要在最上面加 `use client`
```jsx
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}
```

* 適合用於：
  - 使用者互動（按鈕、表單）
  - 動畫
  - 瀏覽器端狀態儲存（如 localStorage）

## 如何搭配使用？
1. 在 **Server Component 中呼叫 DB 或 API 拿資料**
1. 再把資料傳給 **Client Component 負責顯示或互動**

```jsx
// Server Component
import Counter from './Counter';

export default async function Page() {
  const data = await fetchData(); // Server-side 資料存取
  return (
    <div>
      <h1>{data.title}</h1>
      <Counter /> {/* Client Component */}
    </div>
  );
}
```
Next.js 的混合渲染機制：
* Server Component 負責資料載入、初始渲染
* Client Component 處理互動與狀態更新

畫面怎麼組合？
* Server Component：先在 server 渲染 HTML。
* Client Component：在 client 上「hydrate」（接管互動邏輯）。

## 何時需要使用 Client Component？
要用到瀏覽器端功能或side effect，就必須使用 Client Component。具體來說：

### 1. 使用 React hooks
`useState`、`useEffect`、`useContext`、`useRef` 等，這些都只能在瀏覽器執行。
```tsx
'use client';
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### 2. 需要用戶互動
按鈕點擊、表單輸入、展開/收合、切換 tab

### 3. 使用瀏覽器 API
localStorage, window, document, navigator 等

### 4. 整合第三方互動式 UI 套件
像是使用 Chart.js、Swiper.js、Framer Motion 等，它們依賴 DOM 和瀏覽器。

## 不需要使用 Client Component 的情況
 Server Component 的使用情境非常多，像是：
 | 使用情境                      | Server Component |
| ------------------------- | ---------------- |
| 顯示從資料庫取得的內容               | ✅                |
| 靜態頁面         | ✅                |
| SSR 或 SSG 載入的內容           | ✅                |
| 僅組合其他 Client Component    | ✅                |
| 使用 `<Image />`、`<Link />` | ✅                |

### 判斷原則
| 判斷問題                               | 是否使用 Client Component |
| ---------------------------------- | --------------------- |
| 有用 `useState`, `useEffect` 等 hook？ | ✅                     |
| 有使用者互動？                            | ✅                     |
| 有用到瀏覽器功能？                          | ✅                     |
| 沒有互動，只是顯示資料？                       | ❌（用 Server 就好）        |
| 是資料列表、動態頁面、SSR 結果？                 | ❌                     |



## 注意事項
* `use client` 必須放在檔案第一行（不能前面有註解或 import）。
* 如果一個 component 是 Client Component，它裡面引用的子 component 也必須是 client。
* 盡量讓頁面中大部分邏輯用 Server Component，這樣 bundle size 更小，效能更好。