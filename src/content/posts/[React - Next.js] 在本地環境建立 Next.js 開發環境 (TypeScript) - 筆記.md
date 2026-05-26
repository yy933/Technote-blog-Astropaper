---
title: "[React - Next.js] 在本地環境建立 Next.js 開發環境 (TypeScript) - 筆記"
pubDatetime: 2026-05-26T03:01:46.970Z
tags: ["cheatsheet","TypeScript","Next.js","Node.js","React.js"]
description: " Table of contents 建立專案 bash npx createnextapp@latest myapp ..."
---

## Table of contents



## 建立專案
```bash
npx create-next-app@latest my-app --typescript
cd my-app
```
Flags 說明：
* `my-app`：專案名稱
* `--typescript`：自動加入 TypeScript 支援


## 啟動開發伺服器
```bash
cd my-app
npm run dev
# 或 yarn dev / pnpm dev
```
打開瀏覽器`localhost:3000`查看。

## 專案結構
```ruby
my-app/
├── app/
│   ├── layout.tsx
│   ├── page.tsx         # Home
│   └── about/
│       └── page.tsx     # /about
├── components/
├── public/
├── styles/
│   └── globals.css
├── tailwind.config.ts
├── tsconfig.json
├── next.config.js
```

## 確認 App Router 是否開啟
`app/` 資料夾存在，代表已使用 App Router。
如還是 `pages/` 架構，可手動切換（建議用最新版 Next.js）。

### 建立第一個 Route（使用 App Router）
*  `app/page.tsx` → 對應 / 首頁
*  `app/about/page.tsx` → 對應 /about 頁面

## 建立 Layout
`app/layout.tsx`（必要）
範例:
```tsx
// app/layout.tsx
import './globals.css'
import { ReactNode } from 'react'

export const metadata = {
  title: 'My App',
  description: 'Next.js + TS + Tailwind',
}

export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="en">
      <body className="bg-white text-gray-900">{children}</body>
    </html>
  )
}
```

## 新增頁面元件 
```tsx
// app/about/page.tsx
export default function About() {
  return <h1>About Page</h1>
}
```

進入 `/about` 時會自動對應該檔案。

## 使用 TypeScript 建立元件

```tsx
// components/Button.tsx
type ButtonProps = {
  text: string
}

export function Button({ text }: ButtonProps) {
  return <button>{text}</button>
}
```

## Tailwind 快速確認
檢查以下三個檔案：
* `tailwind.config.ts`
* `postcss.config.js`
* `globals.css` 裡應該有：
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### 使用 Tailwind 元件
```tsx
<button className="px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white rounded-lg">
  點我
</button>
```

## 安裝 Shadcn/UI（Optional）
```bash
npx shadcn-ui@latest init
```
再依指示選擇：
```bash
Tailwind Config
App Router
TypeScript
要安裝的元件（例如 button、card）
```

### 使用Shadcn/UI元件（例如：Button）
```bash
npx shadcn-ui@latest add button
```
使用:
```tsx
import { Button } from "@/components/ui/button"

<Button>Click me</Button>
```

## 部署（Vercel）
```bash
npx vercel
```

## 常用指令
| 指令                         | 說明                         |
| -------------------------- | -------------------------- |
| `npm run dev`              | 啟動本地伺服器 (`localhost:3000`) |
| `npm run build`            | 打包產出                       |
| `npm run lint`             | 程式碼檢查                      |
| `npx shadcn-ui@latest add` | 加元件                        |
| `npx vercel`               | 快速部署                       |

> 進階設定：Prisma、API Route、環境變數　（待補）