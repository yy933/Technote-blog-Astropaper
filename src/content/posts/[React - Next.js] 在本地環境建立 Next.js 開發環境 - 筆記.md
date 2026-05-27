---
title: "[React - Next.js] 在本地環境建立 Next.js 開發環境 - 筆記"
pubDatetime: 2025-05-27T21:38:58.000Z
tags: ["React.js","Node.js","cheatsheet","Next.js"]
description: "Table of contents 從頭開始建立 Next.js 專案（使用 App Router） 1. 新增一個專..."
hackmd_id: "BkEhea6blg"
---

## Table of contents


## 從頭開始建立 Next.js 專案（使用 App Router）
1. 新增一個專案資料夾，檢查Node版本，並且安裝必要的套件

檢查Node版本:
```bash
node -v // 建議 Node.js 18 以上版本
```
建立專案:
```bash
mkdir nextjs-project
cd nextjs-project
npm init -y
npm i react@latest react-dom@latest next@latest
```

接著在 `package.json` 裡新增以下：
```
// package.json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```


2. 設定 `app/` 資料夾（Next.js App Router）
建立 `app/layout.jsx` 和 `app/page.jsx`（注意命名與結構）：

```jsx
// app/layout.jsx
export default function Layout({ children }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}

```

```jsx
// app/page.jsx
export default function Page() {
  return <h1>Hey Next.js!</h1>;
}
```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

📌 注意：使用的是 Next.js 13+ 的 App Router 架構，資料夾名稱必須是 `app/`，不能用 `pages/`。而且要有 `layout.jsx` 才能跑起來。

</blockquote>

3. 啟動開發伺服器
```
npm run dev
```
稍待片刻就建立完成了，可以打開瀏覽器看看: http://localhost:3000/


## 更方便的方法：用`create-next-app` 建立Next.js專案
```bash
npx create-next-app@latest my-next-app
```
依照提示選擇：
* TypeScript？ 目前使用JS，選 "No"
* 使用 App Router？選 "Yes"
* Tailwind？依需求
* ESLint？可保留
* src/ folder？可選擇

這樣會自動產生 `app/` 架構並配置好。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:moon: TypeScript + Tailwind CSS + App Router專案建立cheatsheet看[這篇](https://hackmd.io/5RO8eGeHT0qCT54qgdH5CQ)

</blockquote>