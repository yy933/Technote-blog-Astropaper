---
title: "[React - Next.js] 在本地環境建立 Next.js 開發環境 - 筆記"
pubDatetime: 2026-05-25T11:17:36.443Z
tags: ["React.js","Node.js","cheatsheet","Next.js"]
description: " Table of contents 從頭開始建立 Next.js 專案（使用 App Router） 1. 新增一個專..."
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
:::warning
📌 注意：使用的是 Next.js 13+ 的 App Router 架構，資料夾名稱必須是 `app/`，不能用 `pages/`。而且要有 `layout.jsx` 才能跑起來。
:::

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

:::success
:moon: TypeScript + Tailwind CSS + App Router專案建立cheatsheet看[這篇](https://hackmd.io/5RO8eGeHT0qCT54qgdH5CQ)
:::