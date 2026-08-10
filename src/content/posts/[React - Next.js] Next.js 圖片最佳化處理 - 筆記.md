---
title: "[React - Next.js] Next.js 圖片最佳化處理 - 筆記"
pubDatetime: 2026-08-10T09:31:59.085Z
tags: ["React.js","Node.js","Next.js","TypeScript"]
description: "Table of contents 為什麼要用 Next.js 的 <Image / 元件？ 1. 圖片延遲載入（la..."
hackmd_id: "r19Q9D1Mex"
---

## Table of contents

## 為什麼要用 Next.js 的 `<Image />` 元件？
### 1. 圖片延遲載入（lazy loading）  
當圖片還沒出現在瀏覽器可視範圍內（viewport），它就不會馬上下載，而是等使用者瀏覽頁面到那一區才載入。這樣做的好處是**可以加快初始載入速度、減少不必要的流量，對手機用戶尤其友善（節省數據）**。

### 2. 響應式(RWD)尺寸、自動格式最佳化（WebP / AVIF）  
Next.js 自動為不同裝置提供不同尺寸的圖片，也會轉換為效能更好的格式：
* WebP：現代瀏覽器廣泛支援，容量比 JPG 小 20~30%
* AVIF：更先進，容量更小，但支援度稍低

這樣做的好處是**畫質幾乎一樣，但檔案更小，能減少延遲時間、節省流量，更可依照使用者的裝置，自動挑選最佳尺寸。**

### 3. 預留空間避免 Layout Shift（CLS）
**CLS = Cumulative Layout Shift**  
當網頁載入時，如果圖片還沒出現，其他內容會跳來跳去 → 體驗差、SEO 減分。  
`<Image />` 要求指定寬高比例，編譯時會自動為圖片預留佔位區塊，載入時不會造成畫面跳動**（良好 UX & SEO）。**



## 基本用法

### 1. 靜態圖片匯入 (Static Import)  
將圖片放在專案資料夾（如 `@/public/`），透過相對路徑匯入。**優勢：自動取得寬高，且支援模糊載入佔位圖。**  
圖片統一放在根目錄`/public`資料夾中

```jsx
import Image from 'next/image'
import MyImage from '@/public/example.jpg' // 注意：需使用相對路徑或別名

export default function Page() {
  return (
   <Image
  src={myImage}
  alt="範例圖片"
  placeholder="blur" // 自動產生 blurDataURL 並提供模糊載入過渡效果
/>
  );
}
```
### 2. 本地靜態路徑 (Public Path)  
直接寫字串路徑引用 `/public` 資料夾底下的圖片，此方式必須手動設定 `width` 與 `height`：

```jsx
import Image from 'next/image'
 
export default function Page() {
  return (
    <Image
      src="/example.jpg"  // 對應 public/example.jpg
      alt="Picture of the author"
      width={500}
      height={500}
    />
  )
}
```
## 首頁圖片效能優化 (priority)  
位於首頁 (Above the fold) 或 Hero 區塊的關鍵圖片，必須加上 `priority` 屬性。這會停用該圖片的 Lazy Loading 並提前預載，**能大幅提升 LCP (Largest Contentful Paint) 分數。**

```typescript
 <Image
      src="/hero-banner.jpg"  // 優先載入，防止LCP分數下降
      alt="Hero banner"
      width={1200}
      height={600}
	  priority
    />
```


## 遠端圖片設定 (Remote Images)  
若引用外部 URL（例如 S3、Unsplash、Cloudinary 等），必須在 `next.config.ts` (或 `.js`) 中加入白名單：

```typescript
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 's3.amazonaws.com',
        port: '',
        pathname: '/my-bucket/**',
      },
    ],
  },
};

export default nextConfig;
```

## 版面排版技巧
### 1. 搭配 Tailwind CSS 固定尺寸

```typescript
<Image alt="Example" className="w-[200px] h-auto rounded-lg shadow-md" height="{300}" src="/example.jpg" width="{400}"/>
```

### 2. 自適應父容器 (`fill`)  
當圖片尺寸不固定（如響應式卡片圖），可使用 `fill` 讓圖片填滿父容器。
* 父容器條件：必須加上 `relative` 與明確的寬高。
* 效能建議：務必加上 `sizes`，防止手機下載過大圖片。

```typescript
<div className="relative w-full h-[300px] overflow-hidden rounded-xl">
  <Image
    src="/example.jpg"
    alt="Example"
    fill
    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
    className="object-cover" {/* 若要完整顯示圖片不裁切，改為 object-contain */}
  />
</div>
```

## SVG 處理建議  
不建議將 SVG 傳入 `<Image />`。因為 SVG 本質是向量圖形，本身就不存在壓縮與解析度問題，經過 `<Image />` 反而會產生多餘的包裝。

### 建議作法：

* 使用原生 `<img />`：

```typescript
<img src="/logo.svg" alt="Logo" className="w-10 h-auto" />
```

* 或直接將 SVG 作為 React 元件引入（如使用 `@svgr/webpack`）。








:::

* width 和 height 是必填！
* 若用 Tailwind 的 `w-[]` 也要提供實際尺寸（因為 Next 要預估空間）
* 圖片建議放在 `/public` 資料夾下，使用 `/` 開頭引用

###  Tailwind 和 `<Image />` 的尺寸搭配使用
```jsx
<Image
  src="/image.jpg"
  alt="..."
  width={100}
  height={100}
  className="w-[100px] h-auto"
/>
```
這樣瀏覽器會知道寬高比例，Tailwind 負責 CSS 呈現。

### 圖片自適應父容器
```jsx
<div className="relative w-[300px] h-[200px]">
  <Image
    src="/example.jpg"
    alt="..."
    fill
    className="object-cover"
  />
</div>
```
* 外層容器一定要設定 `relative` 和固定寬高
* `object-cover`, `object-contain` 自由控制比例:
      - `object-cover` : 圖片會保持原比例（不變形），若比例不合，多出來的會被裁掉；容器會被塞滿，不會留白。
      - `object-contain` : 圖片會保持原比例，會完整呈現在容器中，不會被裁切；若比例不同，容器邊緣會留空白。

## SVG 要用 `<Image />`嗎？  
不建議，因為SVG本身是向量圖，沒尺寸模糊問題，`<Image />`不會優化 SVG。  
用原生 `<img />` 會更輕量快速。
```jsx
<img src="/logo.svg" alt="Logo" className="w-10 h-auto" />
```