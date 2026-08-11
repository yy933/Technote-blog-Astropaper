---
title: "[React.js - Next.js] 理解 Next.js 中的 Client Boundary與設計選擇 - 筆記"
pubDatetime: 2026-08-11T08:13:54.210Z
tags: ["React.js","Next.js","App Router","Server Component","Client Component","Frontend","Concepts","Issue"]
description: "Table of contents 什麼是 Client Boundary？ 在 Next.js (App Route..."
hackmd_id: "Bk6zBIOLfx"
---

## Table of contents


## 什麼是 Client Boundary？

在 Next.js (App Router) 中，所有元件**預設都是 Server Component (RSC)**。當我們在檔案最頂端加入 `'use client'` 指令（Directive）時，不僅該元件本身會變成 Client Component，它**內部所 `import` 並渲染的所有子元件（Children），也都會被納入 Client Boundary**，一併打包送到瀏覽器端執行。

> 💡 **核心觀念**：`'use client'` 劃定的是一條「網路邊界（Network Boundary）」。一旦這條邊界建立起來，邊界底下的所有程式碼都會失去 Server Component 的優勢（如 0 Bundle-Size、直接讀取 DB 等）。因此，**Client Boundary 應該劃定得越小越好**。
> 
## 為何需要 Client Boundary？

* **Server Component 的局限**：伺服器元件（RSC）只在 Server 端渲染，無法在瀏覽器端因為 State（狀態）改變而進行二次重新渲染（Re-render）。
* **狀態引發的連鎖反應**：若一個組件擁有了 State（例如點擊次數 `hits`），它就必須變成 Client Component。但當它重新渲染時，其內部的子組件若為 Server Component，就會面臨「無法跟著重新渲染」的衝突。
* **React 的硬性規則**：為了防止上述矛盾，React 規定 Client Component 只能 `import` 其他 Client Component。



## Client Boundary 的運作機制
- 邊界建立與隱式轉換：在檔案頂端加入 `'use client'` 即建立了「用戶端邊界」。在該檔案內部直接 `import` 進來的所有模組/組件，都會隱式被視為 Client Component 並在 Client 端 Hydrate，無須在每個子組件檔案都加上 `'use client'`。

- **決定因素在於「模組 Import」而非「JSX 嵌套」**：Client Boundary 的限制作用於檔案/模組層級（Module-level）。打包工具（Bundler）是沿著 `import` 連結來追蹤與打包程式碼的。

## 指令（Directive）的核心本質

* **`'use client'` 是 React 的官方標準**：它並非 Next.js 獨創，而是 React 19 規範中的官方指令（類似 Vanilla JS 的 `'use strict'`），用來指示編譯器如何處理該檔案及其子樹。
* **避免過度使用（Overkill）**：最常見的錯誤作法是為了在 Navbar 中使用 `usePathname()` 來做 Active Link，而直接在 `Root Layout` 加了 `'use client'`。這樣會直接打破整套 App 的伺服器渲染優勢，將全站都變成 Client Component。


## 實作案例：Active Link 樣式的兩種架構與權衡

在導覽列（Navbar）中，我們經常需要透過 `usePathname()` 取得當前網址來決定導覽連結的Active Style。此時對於 Client Boundary 的範圍有兩種設計選擇：

### 方案 A：將整支 `Navbar` 設為 Client Component（推薦！）

將 `'use client'` 放在 `Navbar` 元件頂部，在內部呼叫 `usePathname()` 算出高亮狀態後，將布林值（`isActive`）作為 Prop 傳遞給子元件 `NavLink`。

```typescript
// Navbar.tsx (Client Component)
'use client';

import { usePathname } from 'next/navigation';
import NavLink from './NavLink';

export default function Navbar() {
  const pathname = usePathname();

  return (
    <nav>
      <NavLink
          href="/3d-models"
          className={pathname === "/3d-models" ? "is-active" : undefined}>
      3D Models
      </NavLink>      
    </nav>
  );
}
```

* 優點：開發阻力最小（Less Friction），狀態完全由外層統一控制（Control from outside），資料流清晰且易於維護。
* 權衡：整支 Navbar 及其內建的所有連結都會被劃入 Client Boundary。

### 方案 B：僅將單一 NavLink 設為 Client Component  
Navbar 保持為預設的 Server Component，僅將最微小單位的 `NavLink` 加上` 'use client'`，讓各個按鈕自己去比對 `usePathname()` 與內部的 `href`。

```typescript
// NavLink.tsx (Client Component)
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';

export default function NavLink({ href, children }: { href: string; children: React.ReactNode }) {
  const pathname = usePathname();
  const isActive = pathname === href;

  return (
      <Link
          href={href}
          className={isActive ? "is-active" : undefined}>
      {children}
      </Link>      
  );
}
```

* 優點：把 Client Boundary 的範疇壓縮到極致最小（僅單一 `<Link>` 按鈕）。
* 權衡：每個 `NavLink` 都要獨立訂閱 `usePathname()`，當選單複雜時程式碼可能較為繁瑣。

## 快速決策表

| 評估維度 | 方案 A：Navbar 設為 Client Component | 方案 B：NavLink 設為 Client Component |  
| :--- | :--- | :--- |  
| **Client Boundary 範圍** | 中等（包含整個導覽列區塊） | 極小（僅包含單一連結按鈕） |  
| **開發經驗與阻力** | 🟢 較順暢，由外層傳遞 `isActive` 掌控全局 | 🟡 每個連結需自行處理 Hooks 比對 |  
| **程式碼可讀性** | 🟢 高，單一邏輯集中管理 | 🟡 邏輯分散在各個小元件內部 |  
| **適合情境** | 選單結構明確、項目固定的導覽列 | 連結組件需獨立封裝、高度複用於不同外殼 |