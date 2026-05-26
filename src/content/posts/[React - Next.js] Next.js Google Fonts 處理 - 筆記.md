---
title: "[React - Next.js] Next.js Google Fonts 處理 - 筆記"
pubDatetime: 2025-06-03T21:00:36.000Z
tags: ["React.js","Node.js","Next.js"]
description: " Table of contents 為什麼要優化 Google Fonts？ 之前引入Google Fonts的方法，..."
---

## Table of contents

## 為什麼要優化 Google Fonts？
之前引入Google Fonts的方法，是在`<head>` 中以 `<link>` 方式載入字型，這會造成瀏覽器額外的request，影響頁面首次渲染時間（FCP）和最大內容繪製時間（LCP）。Next.js 13 提供內建的 Google Fonts 載入方案，可以減少請求次數、提升性能，甚至支援字型子集與自訂字型變數。

## 使用方法
```jsx
import { Inter, Roboto } from "next/font/google";

const inter = Inter({
  subsets: ["latin"],
  weight: ["400", "700"],  // 載入的字體粗細，減少不必要的值以加速載入
  variable: "--font-inter", // CSS變數名稱，可自訂
  display: "swap",         // 字型顯示方式，推薦用 swap。確保文字優先顯示系統字型，字型載入後再切換
});
```
在layout中使用(整個專案的主要字體設定):
```jsx
export default function RootLayout({ children }) {
  return (
    <html lang="en" className={inter.variable}>
      <body className={inter.className}>
        {children}
      </body>
    </html>
  );
}
```

如果只要設定局部字體(例如標題用不同字體):
先設定變數，再到CSS中使用
```css
// global.css
@tailwind base;
@tailwind components;
@tailwind utilities;

h1, h2, h3, h4, h5, h6 {
  font-family: var(--font-Roboto);
}
```

### inter.className
內容：一個類似 font-inter_abc123 的隨機 class 名稱。
用途：直接套用整體字體樣式。
用法：可以把這個 `className` 加到 `<body>` 或 `<main>`，整個區域會套用 Inter 字體。
```tsx
<body className={inter.className}>
```

### inter.variable
內容：在設定中指定的 CSS 變數名（這裡是 `--font-inter`）。
用途：在 Tailwind 或自定義 CSS 中用 CSS 變數控制字體。
搭配 Tailwind config 使用範例：
```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        inter: ['var(--font-inter)'],
      },
    },
  },
}
```

然後在元件中這樣使用：
```tsx
<div className="font-inter">這段字會使用 Inter 字體</div>
```


記得還是要把 `inter.variable` 加到 `<body>` 或全域容器上，讓 CSS 變數生效：
```tsx
<body className={inter.variable}>
```

兩者也可以同時用:
```tsx
<body className={`${inter.className} ${inter.variable}`}>
```
這樣既能立即看到字體效果，也能在 Tailwind 中用 `font-inter` 控制字體。


## 好處
* 減少外部請求
Google Fonts 會被 Next.js 伺服器側預先加載並內嵌到 CSS 中。
* 改善 SEO 與性能
字型渲染更快，減少 FOIT（Flash of Invisible Text）。
* 整合 Tailwind
利用 variable 屬性可直接設定 Tailwind CSS 的 font-family。
* 若需要使用客製字型（自訂字型檔），可使用 `next/font/local`