---
title: "[Next.js] App Router 中 searchParams 的完整解析與實務應用 - 筆記"
pubDatetime: 2026-08-12T15:34:38.876Z
modDatetime: 2026-08-12T15:35:45.015Z
tags: ["Next.js","App Router","React.js","TypeScript","Frontend"]
description: "Table of contents 1. Server Component 中的 searchParams（Page..."
hackmd_id: "H1fBAb5UGx"
---

## Table of contents

## 1. Server Component 中的 searchParams（Page Props）

在Server component（`page.tsx`）中，`searchParams` 會作為頁面元件的 Props 自動傳入。

### Next.js 15+ 重大更新：`searchParams` 變更為 Async (Promise)

在 Next.js 15 之前，`searchParams` 是同步物件；但在 **Next.js 15+** 中，為了非同步渲染與效能優化，`searchParams` 被改為 **`Promise`**，使用時**必須加上 `await`**。

```typescript
// app/products/page.tsx
import type { Metadata } from 'next';

type SearchParams = Promise<{
  query?: string;
  page?: string;
  sort?: string;
  tags?: string | string[]; // 當同名參數出現多次時為陣列
}>;

export default async function ProductsPage({
  searchParams,
}: {
  searchParams: SearchParams;
}) {
  // 1. 必須使用 await 解析 Promise (建議搭配 ?? {} 防護)
  const { query, page = '1', sort = 'desc', tags } = (await searchParams) ?? {};

  return (
    <div>
      <h1>搜尋關鍵字: {query || '無'}</h1>
      <p>當前頁碼: {page}</p>
      <p>排序方式: {sort}</p>
    </div>
  );
}
```

## 2. Client Component 中的 useSearchParams (Hook)  
在 Client Component（加上 `'use client'` 的元件）中無法透過 Props 讀取，必須呼叫 `next/navigation` 提供的 `useSearchParams()`。

```typescript
'use client';

import { useSearchParams, useRouter, usePathname } from 'next/navigation';

export default function SearchInput() {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const { replace } = useRouter();

  // 取得目前 query 的值
  const currentQuery = searchParams.get('query') || '';

  // 當使用者輸入並更新 URL 參數
  const handleSearch = (term: string) => {
    const params = new URLSearchParams(searchParams);
    
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }

    // 更新 URL，但不觸發全頁刷新 (不保留上一頁歷史可使用 replace)
    replace(`${pathname}?${params.toString()}`);
  };

  return (
    <input
      type="text"
      defaultValue={currentQuery}
      onChange={(e) => handleSearch(e.target.value)}
      placeholder="即時搜尋..."
    />
  );
}
```

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

💡 Client 端注意事項：在 Client Component 中使用 `useSearchParams()` 時，建議將該元件包覆在 `<Suspense>` 內，否則 Next.js 在靜態打包（Build Time）時可能會觸發警告或無法正確進行靜態優化。

</blockquote>

## 3. `searchParams` 對渲染策略（Rendering Strategy）的影響  
存取 `searchParams` 會直接決定頁面的渲染行為：

### 3-1. 自動切換為動態渲染（SSR / Dynamic Rendering）

- URL 的查詢參數（`?query=xxx`）是在使用者發起請求（Request Time）當下隨機決定的。
- 只要頁面內取用了 `searchParams`（或呼叫了 `useSearchParams()`），Next.js 便無法在建置時確定畫面，因此該頁面會自動被判定為 `Dynamic (ƒ)` 頁面。

### 3-2. 觀念釐清：`generateStaticParams` 的適用範圍

- `generateStaticParams` 是用來處理動態路由路徑（例如 `/[id]`、`/[category]`）。
- `searchParams` 無法透過 `generateStaticParams` 進行預先靜態建置（SSG）。

## 4. 進階技巧
### 技巧 A：處理陣列型態的 Query Parameters  
當網址出現同名的多個參數時（例如 `?tag=react&tag=nextjs`）：

```typescript
// 網址：/blog?tag=react&tag=nextjs

// Server Component (Next.js 15)
const { tag } = await searchParams; 
// tag 的型別會是 string[] -> ['react', 'nextjs']

// Client Component
const searchParams = useSearchParams();
const tags = searchParams.getAll('tag'); // -> ['react', 'nextjs']
```

### 技巧 B：更新單一參數，同時「保留」其他既有參數  
在做分頁（Pagination）或篩選器（Filter）時，變更 `page` 參數時不能清空使用者原本選好的其他參數（如 `category` 或 `sort`）：

```typescript
'use client';

import { useSearchParams, useRouter, usePathname } from 'next/navigation';

export function PaginationButton({ pageNumber }: { pageNumber: number }) {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const { push } = useRouter();

  const handlePageChange = () => {
    // 複製現有的所有 URL 參數
    const params = new URLSearchParams(searchParams.toString());
    
    // 只更新 page 參數，保留其餘舊參數
    params.set('page', pageNumber.toString());

    // 產生更新後的完整 URL
    push(`${pathname}?${params.toString()}`);
  };

  return <button onClick={handlePageChange}>第 {pageNumber} 頁</button>;
}
```

## Recap: Server Component vs Client Component 比較

| 比較項目 | Server Component (`page.tsx`) | Client Component (`'use client'`) |  
| :--- | :--- | :--- |  
| **讀取管道** | 頁面 Props (`{ searchParams }`) | Hook (`useSearchParams()`) |  
| **Next.js 15+** | 變更為 **`Promise`** (需 `await`) | 維持 Hook 調用方式 |  
| **修改 URL** | 透過原生 `<form method="GET">` / `<Form>` / `<Link>` | 搭配 `useRouter().push/replace` |  
| **渲染影響** | 讀取後頁面自動變為 **Dynamic SSR (`ƒ`)** | 建議包覆在 `<Suspense>` 內以利打包優化 |  
| **適用情境** | 初始資料撈取與伺服器端過濾（Server-side Fetching） | 即時輸入、點擊篩選、防抖（Debounce）搜尋 |