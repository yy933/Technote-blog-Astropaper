---
title: "[React, Next.js, TypeScript] 在本地環境建立 Next.js 開發環境 (Cheatsheet)"
pubDatetime: 2026-08-10T04:23:48.273Z
tags: ["cheatsheet","React.js","TypeScript","Node.js","Next.js"]
description: "Table of contents <blockquote class=\"my6 p4 bgorange50 dark..."
hackmd_id: "SJMGjf4Glx"
---

## Table of contents

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

(2026/08更新！）  
Next.js 14/15 到 2026 年（Next.js 15+ / React 19 生態系），在 CLI 工具名稱、Node.js 版本要求、Tailwind CSS 設定以及 shadcn/ui 的 CLI 指令上都有不少關鍵更新。

</blockquote>

## 系統環境需求
* **Node.js**：建議 `20.x` 或 `22.x` 以上 (LTS 版本)。
```bash
node -v
```

## 建立專案（使用 create-next-app）  
建議直接使用官方 CLI 快速建立（預設搭載 App Router 與 TypeScript）：

```bash
npx create-next-app@latest my-app --typescript
cd my-app
```
說明：
* `my-app`：專案名稱
* `--typescript`：自動加入 TypeScript 支援

CLI 互動選單建議選項：

* Would you like to use TypeScript? Yes
* Would you like to use ESLint? Yes
* Would you like to use Tailwind CSS? Yes
* Would you like your code inside a src/ directory? Yes / No (依個人習慣)


## 啟動開發伺服器
```bash
cd my-app
npm run dev
# 或 pnpm dev / bun dev
```

打開瀏覽器存取`localhost:3000`。

## 專案結構範例 (App Router)
```ruby
my-app/
├── app/
│   ├── layout.tsx       # Root Layout (必要)
│   ├── page.tsx         # / 首頁
│   ├── globals.css      # 全局樣式 (Tailwind CSS v4)
│   └── about/
│       └── page.tsx     # /about 頁面
├── components/          # 共用元件
│   └── ui/              # shadcn/ui 元件放這裡
├── public/              # 靜態資源 (圖片、字型)
├── tsconfig.json
└── next.config.ts
```

## 確認 App Router 是否開啟  
`app/` 資料夾存在，代表已使用 App Router。  
如還是 `pages/` 架構，可手動切換（建議用最新版 Next.js）。

### 建立第一個 Route（使用 App Router）
*  `app/page.tsx` → 對應 / 首頁
*  `app/about/page.tsx` → 對應 /about 頁面

## 路由與 Layout 基本寫法

### 1. Root Layout (`app/layout.tsx`)

```typescript
// app/layout.tsx
import './globals.css';
import type { Metadata } from 'next';
import type { ReactNode } from 'react';

export const metadata: Metadata = {
  title: 'My App',
  description: 'Next.js + TypeScript + Tailwind CSS',
};

export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="zh-TW">
      <body className="bg-background text-foreground antialiased">
        {children}
      </body>
    </html>
  );
}
```

### 2. 新增 Route 與動態路由 (`app/about/page.tsx`)

```typescript
// app/about/page.tsx
// 一般頁面
export default function AboutPage() {
  return <h1>About Us</h1>;
}

// 注意：Next.js 15+ 動態路由 params 為 Promise 類型
export default async function PostPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  return <div>Post ID: {id}</div>;
}
```

進入 `/about` 時會自動對應該檔案。


## Tailwind CSS (v4) 整合  
Tailwind CSS 已簡化設定，`app/globals.css` 中僅需：

```css
@import "tailwindcss";
```

於元件中使用：

```typescript
export function Card() {
  return (
    <div className="p-6 bg-white dark:bg-zinc-900 rounded-xl shadow-md border border-zinc-200 dark:border-zinc-800">
      <h2 className="text-xl font-semibold">Hello Next.js</h2>
    </div>
  );
}
```

## 安裝與設定 shadcn/ui
```bash
# 初始化 shadcn
npx shadcn@latest init

# 新增元件（例如 Button、Card）
npx shadcn@latest add button card
```

於程式碼中使用:
```typescript
import { Button } from "@/components/ui/button";

export default function Page() {
  return <Button variant="default">Click me</Button>;
}
```

## 部署與常用指令（Vercel）


### 常用指令表

| 指令                         | 說明                      |  
| -------------------------- | -------------------------- |  
| `npm run dev`              | 啟動本地開發伺服器 (`localhost:3000`) |  
| `npm run build`            | 編譯並打包專案                 |  
| `npm run start`            | 啟動編譯後的 Production 伺服器  |  
| `npm run lint`             | 執行 ESLint 程式碼檢查          |  
| `npx shadcn-ui@latest add` | 快速新增 shadcn/ui UI 元件      |  
| `npx vercel`               | 部署專案至 Vercel              |

> 進階設定：Prisma、API Route、環境變數　（待補）