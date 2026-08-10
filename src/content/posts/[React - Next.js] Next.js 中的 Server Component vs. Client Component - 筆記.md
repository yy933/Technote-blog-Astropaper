---
title: "[React - Next.js] Next.js 中的 Server Component vs. Client Component - 筆記"
pubDatetime: 2026-08-10T09:58:01.248Z
tags: ["React.js","Node.js","Next.js","TypeScript"]
description: "Table of contents Server Component vs. Client Component 特性..."
hackmd_id: "rJB-icWfll"
---

## Table of contents

## Server Component vs. Client Component

| 特性 / 類型 | Server Component (RSC) | Client Component (RCC) |  
| :--- | :--- | :--- |  
| **預設狀態** | ✅ **App Router 預設** | 需要在檔案最頂端加上 `'use client'` |  
| **執行位置** | **僅在 Server 端**執行 | **Server 端預渲染 (SSR)** 產生 HTML + **Client 端 Hydrate** |  
| **Client JS Bundle** | 零（不佔用使用者瀏覽器 JS 體積） | 會打包進 Client JS Bundle |  
| **React Hooks** | ❌ 不支援 (`useState`, `useEffect` 等) | ✅ 完整支援 |  
| **瀏覽器 API** | ❌ 無法存取 (`window`, `localStorage` 等) | ✅ 完整支援 |  
| **資料存取與安全性** | ✅ 可直接讀取 DB、ORM、存取敏感 API Key | ❌ 需透過 API Route / Server Action 存取 |  
| **主要用途** | 資料擷取、頁面骨架、SEO 內容、靜態渲染 | 狀態管理、事件監聽 (`onClick`)、動畫、瀏覽器 API |

## Server Component (RSC) 的特點與優勢  
1. **零 Bundle 體積 (Zero Bundle-Size)**：RSC 的程式碼與相依套件（如 `lodash`、`marked`）只在伺服器端執行，不會下載到使用者瀏覽器。  
2. **資料存取極簡化**：元件可直接宣告為 `async`，並以 `await` 讀取資料庫或 API：

* 適合用於：
  - 擷取資料（例如：從 DB、API 拿資料）
  - 渲染靜態或 SSR 頁面
  - 初始渲染不需要使用者互動的部分

* 無法做的事：

  - 使用 `useState`, `useEffect`, `useContext`
  - 存取瀏覽器 API，如 `window`, `document`

📍預設情況下，Next.js所有 component 都是 Server Component！

```typescript
// app/products/page.tsx (預設為 Server Component)
import { db } from "@/lib/db";

export default async function ProductsPage() {
  // 直接呼叫 DB，敏感資訊不會洩漏到前端
  const products = await db.product.findMany();

  return (
    <main className="p-6">
      <h1 className="text-2xl font-bold">產品列表</h1>
      <ul>
        {products.map((p) => (
          <li key={p.id}>{p.name} - ${p.price}</li>
        ))}
      </ul>
    </main>
  );
}
```

## Client Component (RCC) 的特點與宣告方式  
當需要處理使用者互動或瀏覽器狀態時，在檔案最上方（所有 `import` 之前）宣告 `'use client'`：

```typescript
// components/Counter.tsx
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button
      onClick={() => setCount(count + 1)}
      className="px-4 py-2 bg-blue-600 text-white rounded-lg"
    >
      Clicked {count} times
    </button>
  );
}
```

* 適合用於：
  - 使用者互動（按鈕、表單）
  - 動畫
  - 瀏覽器端狀態儲存（如 `localStorage`）

## 如何搭配使用？
### 1. 預設向下渲染法則 (Contagion)  
在 Client Component 中直接 `import` 的元件，都會自動被視為 Client Component 納入前端 Bundle：

```typescript
'use client';
import HeavyComponent from './HeavyComponent'; // ❌ HeavyComponent 也會被迫變成 Client Component
```

### 2. 進階解法：Children Prop Pattern (解耦 Server/Client)  
若想讓 Client Component 包裹 Server Component（例如 Modal、Sidebar、Context Provider），請將 Server Component 作為 `children` 傳入：

```typescript
// components/ClientWrapper.tsx (Client Component)
'use client';

import { useState } from 'react';

export default function ClientWrapper({ children }: { children: React.ReactNode }) {
  const [isOpen, setIsOpen] = useState(true);

  return (
    <div className="border p-4">
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && children} {/* children 依然保持為 Server Component */}
    </div>
  );
}
```

```typescript
// app/page.tsx (Server Component)
import ClientWrapper from '@/components/ClientWrapper';
import ServerDataList from '@/components/ServerDataList'; // 伺服器元件

export default function Page() {
  return (
    <ClientWrapper>
      {/* 成功將 RSC 嵌入 RCC 中，且 ServerDataList 仍享有一切 RSC 優勢 */}
      <ServerDataList/>
    </ClientWrapper>
  );
}
```

- Next.js 的混合渲染機制：
  - Server Component：先在 Server 算好 HTML 與 RSC Payload 送至前端。
  - Client Component：先在 Server 產生初始 HTML，送到前端後進行「Hydrate」綁定事件邏輯。


## 何時需要使用 Client Component？  
要用到瀏覽器端功能或side effect，就必須使用 Client Component。具體來說：

### 1. 使用 React hooks  
`useState`、`useEffect`、`useContext`、`useRef` 等，這些都只能在瀏覽器執行。

```typescript
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
`localStorage`, `window`, `document`, `navigator` 等

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
| 沒有互動，只是顯示資料？                       | ❌（用 Server Component 就好）        |  
| 是資料列表、動態頁面、SSR 結果？                 | ❌                     |

## 快速決策樹：我該用哪種 Component？

```ruby
需要處理用戶點擊 (onClick/onChange) 或狀態 (useState/useEffect)？
 ├── 是 ──> 使用 Client Component ('use client')
 └── 否 ──> 需要存取 瀏覽器 API (window/localStorage/Custom Hooks)？
            ├── 是 ──> 使用 Client Component ('use client')
            └── 否 ──> 保持預設 Server Component (RSC)
```

## 注意事項
* `'use client'` 指令必須放在所有 `import` 敘述與程式碼之前（僅允許在上方放置註解）。
* 若想讓 Client Component 包裹 Server Component（例如 Modal、Sidebar、Context Provider），請將 Server Component 作為 `children` 傳入。
* 盡量讓頁面樹狀結構的根部與絕大部分邏輯保留為 Server Component，僅將需要互動的小元件拆出來加上 `'use client'`，能保持最小的 Bundle Size 並獲得最佳效能。