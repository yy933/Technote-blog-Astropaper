---
title: "[React - Next.js] Next.js 圖片處理 - 筆記"
pubDatetime: 2025-05-25T19:29:28.000Z
modDatetime: 2026-05-25T10:04:23.410Z
tags: ["React.js","Node.js","Next.js"]
description: "Table of contents Next.js 中的圖片處理 <Image /有什麼好處？ 1. 圖片延遲載入（l..."
hackmd_id: "r19Q9D1Mex"
---

## Table of contents

## Next.js 中的圖片處理 `<Image />`有什麼好處？
### 1. 圖片延遲載入（lazy loading）
當圖片還沒出現在瀏覽器可視範圍內（viewport），它就不會馬上下載，而是等使用者瀏覽頁面到那一區才載入。這樣做的好處是可以加快初始載入速度、減少不必要的流量，對手機用戶尤其友善（節省數據）。

### 2. 響應式(RWD)尺寸、自動格式最佳化（WebP / AVIF）
Next.js 自動為不同裝置提供不同尺寸的圖片，也會轉換為效能更好的格式：
* WebP：現代瀏覽器廣泛支援，容量比 JPG 小 20~30%
* AVIF：更先進，容量更小，但支援度稍低

這樣做的好處是畫質幾乎一樣，但檔案更小，能減少延遲時間、節省流量，更可依照使用者的裝置，自動挑選最佳尺寸。

### 3. 預留空間避免 Layout Shift（CLS）
CLS = Cumulative Layout Shift
當網頁載入時，如果圖片還沒出現，其他內容會跳來跳去 → 體驗差、SEO 減分。
Next.js 預留出圖片空間，等圖片載入後就不會整個畫面跳動（良好 UX & SEO）。

給 `<Image />` 明確設定 width 和 height，它會：
* 預先計算好圖片的比例（例如 4:3）
* 用 padding 技巧讓區塊在載入前就佔好位子

## 用法
圖片統一放在根目錄`/public`資料夾中
```jsx
import Image from 'next/image'
import MyImage from '/public/example.jpg' 

<Image
  src={MyImage}        // 或直接寫 "/example.jpg"
  alt="描述文字"
  width={500}
  height={300}
/>
```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

如果要使用遠端圖片(Remote image):
例如：
```jsx
import Image from 'next/image'
 
export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/my-bucket/profile.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  )
}
```
Next.js 的 Image 元件（`next/image`） 在處理遠端圖片時，預設不允許直接使用外部 URL，需要在 `next.config.js` 中設定明確允許遠端網域：
```jsx
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https', // 通訊協定，只能是 'http' 或 'https'
        hostname: 's3.amazonaws.com', // 網域名稱
        port: '', // 連接埠，通常留空 (Optional)
        pathname: '/my-bucket/**', // 圖片在該主機下的路徑，可使用 * 或 ** 萬用字元，例如'/**', 就是允許Imgur的所有圖片
        search: '', // URL 的查詢參數（如 ?id=123），通常不用填
      },
    ],
  },
}
```

</blockquote>

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