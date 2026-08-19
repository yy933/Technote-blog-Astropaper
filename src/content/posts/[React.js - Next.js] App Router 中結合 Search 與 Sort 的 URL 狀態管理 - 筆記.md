---
title: "[React.js - Next.js] App Router 中結合 Search 與 Sort 的 URL 狀態管理 - 筆記"
pubDatetime: 2026-08-19T10:13:26.070Z
tags: ["Frontend","React.js","App Router","Next.js","Concepts","Issue","Client Component","URLSearchParams"]
description: "tags: React.js Next.js App Router Client Component URLSearc..."
hackmd_id: "BJf5bbMDfl"
---

##### tags: `React.js` `Next.js` `App Router` `Client Component` `URLSearchParams` `Frontend` `Concepts`

## Table of contents


## 情境  
在建構多條件篩選與排序頁面時，經常需要同時支援 **關鍵字搜尋（Search）** 與 **結果排序（Sort）**。

## 常見問題：搜尋狀態被強制清空  
當使用者先在網址輸入了搜尋條件（如 `?q=dragon`），接著點擊「依熱門度排序」按鈕時，如果直接重新組裝網址（例如 `${pathname}?sort=popular`），會導致原本的搜尋條件 `q=dragon` 被覆蓋抹除，使用者體驗極差。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡 **核心原則：**  
在 App Router 中，網址（URL）是狀態的最佳單一事實來源（Single Source of Truth）。更新特定 URL 參數時，必須完整保留既有的其他參數（Keep existing query params intact）。

</blockquote>

## 觀念解析  
要解決上述問題，必須理解 Next.js 提供的 Navigation Hooks 與瀏覽器原生 API 的分工：

### 1. `useSearchParams()` 的唯讀限制  
Next.js 提供的 `useSearchParams()` 是 Client Component 用來讀取 URL Query String 的 Hook。**但它回傳的是一個 唯讀（Read-only） 的 `ReadonlyURLSearchParams` 物件。**

- 可以：`searchParams.get('sort')`、`searchParams.has('q')`、`searchParams.toString()`
- 不能：直接呼叫 `.set()` 或修改內容（會無效或報錯）。

### 2. 透過原生 `URLSearchParams` 進行轉換與編輯  
為了打破唯讀限制，我們需要將 `searchParams` 轉為字串後，傳入 JavaScript 瀏覽器原生的 `URLSearchParams` 建構式，複製出一份可編輯（Mutable）的實體副本。

```typescript
// 1. 將唯讀的 searchParams 轉為純字串，建立可編輯的原生物件
const urlSearchParams = new URLSearchParams(searchParams.toString());

// 2. 使用原生 API 進行編輯（更新或新增）
urlSearchParams.set('sort', 'popular'); 

// 3. 轉回字串供 router.push 使用
const newQueryString = urlSearchParams.toString(); // "q=dragon&sort=popular"
```

## `set()` 的特性

原生 `URLSearchParams` 提供了 `.set(key, value)` 方法，它擁有非常優雅的行為特性：

* 若 Key 已存在：覆蓋該 Key 的舊值（例如將 `sort=recent` 改為 `sort=popular`）。
* 若 Key 不存在：自動追加該 Key/Value 條件。
* 其他既有 Key：完全不受影響，原封不動保留（例如原本的 `q=dragon` 會繼續留在網址中）。


## 實作案例：完整保留狀態的 SortButton 元件  
以下為封裝好的排序按鈕元件，能在使用者切換排序時，無縫保留網址列上的搜尋關鍵字或其他篩選條件：

```typescript
'use client';

import { usePathname, useRouter, useSearchParams } from 'next/navigation';

interface SortButtonProps {
  children: React.ReactNode;
  sort: string;
}

export default function SortButton({ children, sort }: SortButtonProps) {
  const pathname = usePathname();
  const router = useRouter();
  const searchParams = useSearchParams();

  // 判斷當前按鈕是否為選中狀態
  const isActive = searchParams.get("sort") === sort;

  function handleSort() {
    // 1. 複製一份可編輯的 URLSearchParams 實例
    const urlSearchParams = new URLSearchParams(searchParams.toString());

    // 2. 更新或設定 sort 參數（其他既有參數如 search, category 等均會保留）
    urlSearchParams.set('sort', sort);

    // 3. 重新組裝完整的 URL 並推入路由
    const url = `${pathname}?${urlSearchParams.toString()}`;
    router.push(url);
  }

  return (
    <button
      onClick={handleSort}
      className={`px-3 py-1.5 text-sm rounded-full border cursor-pointer ${
        isActive 
          ? "text-white bg-orange-400 border-orange-400" 
          : "border-gray-300 text-gray-700 hover:bg-gray-100"
      }`}
    >
      {children}
    </button>
  );
}
```

## Recap


| 評估維度 | `searchParams` (`useSearchParams`) | `new URLSearchParams(...)` |  
| :--- | :--- | :--- |  
| **來源與性質** | Next.js 專屬 Hook 回傳，唯讀（Read-only） | 瀏覽器原生 API，可變（Mutable） |  
| **主要功能** | 讀取當前 URL 的 Query 狀態（如 `.get()`） | 修改、新增或刪除 Query 狀態（如 `.set()`, `.delete()`） |  
| **使用情境** | UI 條件比對（如判斷按鈕是否高亮 `isActive`） | 事件觸發時（如按鈕 Click、搜尋輸入）動態組裝新 URL |  
| **搭配作法** | 傳入 `toString()` 作為建構參數 | 處理完畢後調用 `.toString()` 供 `router.push()` 使用 |