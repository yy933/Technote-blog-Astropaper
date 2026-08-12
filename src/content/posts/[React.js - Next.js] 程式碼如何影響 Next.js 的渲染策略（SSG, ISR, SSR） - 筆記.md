---
title: "[React.js - Next.js] 程式碼如何影響 Next.js 的渲染策略（SSG, ISR, SSR） - 筆記"
pubDatetime: 2026-08-12T04:59:29.262Z
tags: ["React.js","Next.js","App Router","Rendering Strategies","Frontend","Concepts"]
description: "Table of contents 核心概念：Code drives Rendering Strategy 在 Nex..."
hackmd_id: "HyQb_OtIzg"
---

## Table of contents

## 核心概念：Code drives Rendering Strategy

在 Next.js (App Router) 中，我們不需要撰寫繁瑣的配置文件來指定某個頁面是 SSG、ISR 還是 SSR。Next.js 在執行建置（`npm run build`）時，會**自動掃描頁面中的程式碼**，根據是否匯出了特定變數、是否使用了動態 API（如 Cookies）、以及 `fetch` 參數來決定最終的渲染策略。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡 **結論**：可以透過「**頁面級宣告（Route Segment Config）**」或「**Data Fetching 參數**」兩種管道，靈活主導每一個 Route 的渲染與快取行為。

</blockquote>




## 1. 觸發 ISR（漸進式靜態再生成）

當頁面資料可以被快取一段時間，但希望在設定的時間視窗過期後**定期在背景更新**時使用。

### 語法與範例

#### **頁面層級設定（Route Segment Config）**：  
在頁面檔案頂部導出 `revalidate` 變數（單位為秒）。
```typescript
   // page.tsx
   export const revalidate = 60; // 此頁面每 60 秒重新驗證快取一次

   export default async function NewArrivalsPage() {
     const data = await fetch('https://api.example.com/models');
     // ...
   }
```

#### Fetch 層級設定：  
在特定的 `fetch` 請求中帶入 `next.revalidate` 屬性。

```typescript
export default async function NewArrivalsPage() {
  const res = await fetch('https://api.example.com/models', {
    next: { revalidate: 60 }, // 設定此 API 請求的快取生命週期為 60 秒
  });
  const data = await res.json();
  // ...
}
```

## 2. 觸發 SSR / Dynamic Rendering（動態伺服器端渲染）  
當頁面必須在每次請求當下（Request Time）都取得最新的資料、不可以使用任何快取時使用。

### 語法與範例
#### Fetch 層級設定（禁用快取）：  
傳入` { cache: 'no-store' }` 強制每次 Request 重新獲取資料。

```typescript
export default async function RealTimeDashboard() {
  const res = await fetch('https://api.example.com/live-data', {
    cache: 'no-store', // 不進行快取，每次請求重新 Fetch
  });
  const data = await res.json();
  // ...
}
```

#### 使用伺服器端 Dynamic APIs  
只要頁面中呼叫了獲取請求細節的 API（如 `cookies()` 或 `headers()`），Next.js 會自動將該頁面升級為動態 SSR。

```typescript
import { cookies } from 'next/headers';

export default async function ProfilePage() {
  const cookieStore = await cookies();
  const token = cookieStore.get('token');
  // 只要使用了 cookies()，此頁面自動成為動態 SSR 頁面
}
```

#### 頁面層級強制宣告

```typescript
export const dynamic = 'force-dynamic'; // 強制整個 Route 採用動態 SSR 渲染
```

## 觸發 SSG / Static Rendering（靜態頁面生成）  
當頁面資料在打包建置時（Build Time）即可確定，後續不再變動時使用。

### 語法與範例
#### 預設行為：  
若頁面中完全沒有使用任何動態 API（如 `cookies()`），也沒有寫 `cache: 'no-store'` 或 `revalidate`，Next.js 預設即為 SSG。

#### Fetch 層級設定（強制快取）

```typescript
export default async function AboutPage() {
  const res = await fetch('https://api.example.com/company-info', {
    cache: 'force-cache', // 建置時抓取一次並凍結快取
  });
  const data = await res.json();
  // ...
}
```

#### 頁面層級強制宣告

```typescript
export const dynamic = 'force-static'; // 強制整個 Route 靜態化
```

## 4. 渲染策略語法對照與速查表

| 渲染策略 | 頁面級設定 (Route Config) | Fetch 參數 / API 使用 | 適用情境 |  
| :--- | :--- | :--- | :--- |  
| **ISR** | `export const revalidate = 60;` | `{ next: { revalidate: 60 } }` | 定期更新的靜態資料（如商品列表、新聞目錄） |  
| **SSR (Dynamic)** | `export const dynamic = 'force-dynamic';` | `{ cache: 'no-store' }`<br>或使用 `cookies()` / `headers()` | 實時性高或使用者個人化頁面（如 Dashboard、購物車） |  
| **SSG (Static)** | `export const dynamic = 'force-static';` | `{ cache: 'force-cache' }`<br>或預設不安插任何動態設定 | 完全固定的靜態頁面（如 About、FAQ） |