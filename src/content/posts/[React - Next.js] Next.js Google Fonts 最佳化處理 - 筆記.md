---
title: "[React - Next.js] Next.js Google Fonts 最佳化處理 - 筆記"
pubDatetime: 2026-08-10T08:35:00.435Z
tags: ["React.js","Node.js","Next.js","TypeScript"]
description: "Table of contents 為什麼要用 next/font/google？ 傳統在 HTML <head 引入..."
hackmd_id: "SylIMO1feg"
---

## Table of contents

## 為什麼要用 `next/font/google`？

傳統在 HTML `<head>` 引入 Google Fonts（`<link href="https://fonts.googleapis me...">`）會在瀏覽器渲染時產生額外的 DNS 尋找與 HTTP 請求，導致：
* **FOIT / FOUT**（字型未載入完成時的閃爍或跳動）。
* 影響 **FCP (First Contentful Paint)** 與 **LCP (Largest Contentful Paint)** 效能分數。

### Next.js 的優勢與原理：  
1. **Build-time 靜態託管 (Self-hosting)**：Next.js 在專案打包時自動下載 Google 字型並託管在自己的 Server / CDN 上，**完全不對外向 Google 發送 Request**（零外部請求、符合 GDPR 隱私規範）。  
2. **零頁面偏移 (Zero CLS)**：自動生成匹配的 CSS 備用字型（Fallback font），消除字型切換時的佈局跳動。  
3. **自動優化**：預設開啟 `subsets` 限制字型集大小，減少下載體積。

## 基本使用方法 (TypeScript)

```typescript
import { Inter, Roboto } from "next/font/google";

// 1. 可變字型 (Variable Font)：不需指定 weight
const inter = Inter({
  subsets: ["latin"],
  variable: "--font-inter", // 定義 CSS 變數名稱
  display: "swap",         // 確保載入期間優先顯示系統字型
});

// 2. 非可變字型：必須指定 weight 陣列
const roboto = Roboto({
  subsets: ["latin"],
  weight: ["400", "700"],
  variable: "--font-roboto",
  display: "swap",
});
```

## 兩大核心屬性：`className` vs `variable`

| 屬性 | 內容與用途 | 使用場景 |  
| :--- | :--- | :--- |  
| **`.className`** | 自動產生的雜湊 Class 名稱（例如 `__className_abc123`） | **全域預設字體**：直接套用到 `<body>` 或容器上 |  
| **`.variable`** | 宣告 CSS 變數（例如 `--font-inter`） | **局部字體控制**：結合 CSS 或 Tailwind 在特定元件/標題使用 |


## 最佳實踐：在 `app/layout.tsx` 註冊全域字體  
建議將全域內文字型使用 `.className`，並同時注入所有字型變數 `.variable`：

```typescript
// app/layout.tsx
import "./globals.css";
import { Inter, Montserrat_Alternates } from "next/font/google";

const inter = Inter({
  subsets: ["latin"],
  variable: "--font-inter",
  display: "swap",
});

const montserrat = Montserrat_Alternates({
  subsets: ["latin"],
  weight: ["600", "700"],
  variable: "--font-montserrat",
  display: "swap",
});

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="zh-TW">
      {/* 1. 注入 CSS 變數  2. 套用預設字體 className */}
      <body
        className={`${inter.variable} ${montserrat.variable} ${inter.className} antialiased`}
      >
        {children}
      </body>
    </html>
  );
}
```

## 搭配 Tailwind CSS 劃分字體  
Tailwind CSS v4 設定（最新）  
直接在 `app/globals.css` 的 `@theme` 區塊中連結變數：

```css
@import "tailwindcss";

@theme inline {
  --font-inter: var(--font-inter);
  --font-montserrat: var(--font-montserrat);
}

/* 針對 HTML 標題統一套用標題字型 */
h1, h2, h3, h4, h5, h6 {
  font-family: var(--font-montserrat), sans-serif;
}
```

在 JSX 中使用 Tailwind class：

```jsx
<h1 className="font-montserrat text-3xl font-bold">這個標題使用 Montserrat</h1>
<p className="font-inter">這段內文使用 Inter</p>
```

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        inter: ['var(--font-inter)', 'sans-serif'],
        montserrat: ['var(--font-montserrat)', 'sans-serif'],
      },
    },
  },
}
```

## 補充：載入本地字型 (`next/font/local`)  
若需使用自行下載的 `.woff2` 或 `.ttf` 字型檔：

```javascript
import localFont from "next/font/local";

const myCustomFont = localFont({
  src: "../public/fonts/CustomFont.woff2",
  variable: "--font-custom",
  display: "swap",
});
```