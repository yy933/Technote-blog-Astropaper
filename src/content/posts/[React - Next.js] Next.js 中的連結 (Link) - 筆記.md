---
title: "[React - Next.js] Next.js 中的連結 (Link) - 筆記"
pubDatetime: 2025-05-25T20:30:30.000Z
tags: ["React.js","Node.js","Next.js"]
description: "Table of contents 什麼是 <Link？為什麼不用 <a？ 在 Next.js 中，<Link 是用來..."
hackmd_id: "BkP5Ew-zxx"
---

## Table of contents


## 什麼是 `<Link>`？為什麼不用 `<a>`？
在 Next.js 中，`<Link>` 是用來處理「前端路由跳轉」的元件。它包裝了原生的`<a>`，但提供了更優化的行為。
    
### `<Link>` 三大優勢
1. 客戶端跳轉（Single Page Application, SPA）
* 使用 `<Link>` 元件，頁面之間跳轉不會重新載入整個頁面，而是直接在前端載入新頁面元件。
* 優點：更快、流暢、無閃爍，保持SPA體驗。(關於SPA，參考[文末說明](##什麼是SPA？))
* 比較：
 `<Link href="/about">About</Link>` → 只載入 About 頁面
 `<a href="/about">About</a>` → 整頁重新載入
 
2. 在連結進入畫面時，自動預先載入目標頁面 (prefetch)
* `<Link>` 預設會在畫面中看得到該連結時，自動把該頁面的資料下載好。
* 這樣使用者點下去時，能幾乎「瞬間載入」。
也可以關閉這個行為：
```jsx
<Link href="/terms" prefetch={false}>Terms of Use</Link>
```

3. 頁面跳轉時保留 React 狀態
* 因為 `<Link>` 使用的是 client-side routing，整個 React app 是連續存在的。
* 跳頁時如同在一個 SPA 中切換頁面，不會重置整個 React app 狀態（例如已登入狀態、全域 context、不會重新載入 JS），提升使用者體驗。
* 對比原生 `<a>` 會導致整個應用程式重新啟動。


## 基本用法
```jsx
import Link from 'next/link';

export default function Navbar() {
  return (
    <nav>
      <Link href="/about">About</Link>
    </nav>
  );
}
```
<blockquote class="my-6 p-4 bg-red-50 dark:bg-red-950/30 border-l-4 border-red-500 rounded-r-md text-red-900 dark:text-red-200 blocknoted-fix">

不需要在 `<Link>` 裡包 `<a>`，在 Next 13+ 的 App Router 中已自動處理。

</blockquote>

## `<Link>`支援的props

| Prop       | 說明                                                  |
| ---------- | --------------------------------------------------- |
| `href`     | 必填。跳轉的路由（可使用相對路徑、絕對路徑、動態路由）                         |
| `prefetch` | 預設為 `true`，在畫面進入 viewport 前預載目標頁面                   |
| `replace`  | 若為 `true`，使用 `replaceState` 取代 `pushState`，不會留下瀏覽紀錄 |
| `scroll`   | 預設為 `true`，跳轉後會滾到頁面頂部。設為 `false` 可保持滾動位置            |
| `as`       | 用來自訂網址顯示（特別在動態路由中使用）                                |

## 動態路由範例

例如有這個檔案結構：
```bash
/app/product/[id]/page.jsx
```

跳轉時應這樣寫：
```jsx
<Link href={`/product/${product.id}`}>
  {product.name}
</Link>
```

或進一步用 as 自訂顯示網址：
```jsx
<Link href="/product?id=123" as="/product/123">Product</Link>
```
> ⚠️ as 通常在使用 query 形式的網址時使用，App Router 時代較少見。

## 其他注意事項
* 只限於跳轉至站內連結
若要連到外部網站，直接用 `<a href="https://...">` 並加 `target="_blank"、rel="noopener noreferrer"`
* Prefetch 功能會消耗頻寬
若目標頁面不太可能被使用者點擊，可手動關閉：
```jsx
<Link href="/terms" prefetch={false}>Terms</Link>
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

## 什麼是SPA？
SPA 是 Single Page Application（單頁式應用程式）的縮寫
SPA 是一種網頁開發方式，整個web app只有一個 HTML 頁面，使用 JavaScript（通常是 React、Vue、Angular 等）在瀏覽器上動態更新內容，而不是每次都向伺服器請求一個新的頁面。
這樣的好處是使用者跳頁更快、更順、更像原生 app 的體驗，狀態（登入、購物車）不會因換頁而重置。

### SPA vs MPA
| 比較項目  | 傳統多頁app（MPA）         | SPA（單頁式app）                     |
| ----- | ------------------- | ------------------------------ |
| 每次點連結 | 向伺服器請求一個新 HTML 頁面   | 不重新載入頁面，只換內容                   |
| 整頁刷新  | 會重新載入整頁             | 不會，保持畫面與狀態連續                   |
| 使用者體驗 | 有跳轉延遲，可能白畫面閃爍       | 頁面切換快速流暢                       |
| 範例框架  | PHP、Ruby on Rails 等 | React、Vue、Angular、Next.js（可混合） |

Next.js 是 SSR（伺服器端渲染）與 SPA 的混合架構
👉 使用 `<Link>` 元件實現 SPA 式的頁面切換體驗，但又能保留 SEO 好處（SSR）

### SPA 對 SEO 有影響嗎？
純 SPA 有 SEO 限制，但 Next.js 解決了這個問題。

👉為什麼純 SPA 對 SEO 不友善？
* SPA 的特性是：頁面內容是靠 JavaScript 在瀏覽器端渲染出來的。
* Google 雖然會嘗試執行 JavaScript，但不是每次都完整渲染，對一些內容抓取得不完全。
* 社群平台（像是 Line/Facebook）在分享時不一定會正確讀取 JS 渲染後的標題與圖片。所以關鍵內容（如產品名稱、文章內文）如果是透過 JS 動態載入，搜尋引擎有可能看不到。

👉Next.js 怎麼解決這問題？

Next.js 是「混合式框架」：
頁面在第一次載入時，會透過 SSR（Server Side Rendering） 預先渲染出 HTML，直接回傳給搜尋引擎（這就是 SEO 友善）。
```jsx
// pages/index.jsx
export default function Home() {
  return <h1>這裡是 SSR 預渲染的內容，搜尋引擎能看到！</h1>
}
```

* 使用者點擊 <Link> 時，仍然享有 SPA 的快速跳頁體驗
* 搜尋引擎抓取的則是伺服器預先渲染好的 HTML（完整的內容！）

</blockquote>