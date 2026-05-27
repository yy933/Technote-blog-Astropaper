---
title: "[React - Next.js] Next.js 中的動態路由（Dynamic Routes） - 筆記"
pubDatetime: 2025-05-26T21:51:46.000Z
tags: ["React.js","Node.js","Next.js"]
description: "Table of contents 什麼是動態路由（Dynamic Routes）？ Dynamic Route 指的..."
hackmd_id: "Bkg-ZoZflx"
---

## Table of contents

## 什麼是動態路由（Dynamic Routes）？
Dynamic Route 指的是 URL 中會根據參數變動的頁面。例如：
* `/products/123`
* `/blog/my-first-post`
* `/users/john-doe`

上面的路徑都有一段「不固定的變數值」，我們就可以用動態路由來處理。

## 為什麼要用動態路由？
如果每一頁都手動建立成靜態頁面，非常不切實際。使用動態路由的好處有：
* 節省手動建立每一個頁面的工夫
* 搭配資料庫/後端 API，自動對應的資料頁面
* 可支援無限多個「個別詳細頁」

## 用法
### 用法一：基本寫法（單一層級）
資料夾結構：
```bash
/app
  /products
    /[id]         ← 動態路由
      page.jsx
```
> `/products/123`、`/products/abc` 都會對應到 `/products/[id]/page.jsx`

`/products/[id]/page.jsx` 範例：

```tsx
import { getProductById } from '@/lib/products';

export default async function ProductPage({ params }) {
  const product = await getProductById(params.id);
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}
```
params 是哪裡來的？
--> 在 Next.js App Router 中，Next.js 會把網址的變數傳進 params 物件。

### 用法二：巢狀動態路由（Nested）
例如：`/users/123/posts/456`
```bash
/app
  /users
    /[userId]
      /posts
        /[postId]
          page.jsx
```
使用方式：
```tsx
export default function PostPage({ params }) {
  return (
    <div>
      User ID: {params.userId}, Post ID: {params.postId}
    </div>
  );
}
```

### 用法三：Catch-All Routes（萬用路由）
`/docs/anything/goes/here`
```bash
/app
  /docs
    /[...slug]
      page.jsx
```
params.slug 是陣列：`["anything", "goes", "here"]`

### 用法四：Optional Catch-All Routes（可選萬用）
```jsx
/app
  /docs
    /[[...slug]]
      page.jsx
```
* /docs 也會對應
* /docs/a/b/c 也會對應
* params.slug 是 undefined 或陣列

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

詳細說明，參考[Next.js文件](https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes)

</blockquote>

## 注意事項 & 限制
| 項目                                    | 說明                                    |
| ------------------------------------- | ------------------------------------- |
| `params` 只在 App Router 有              | 在 App Router (`app/`) 中會自動注入 `params` |
| 可搭配 `generateStaticParams`            | 做靜態生成（SSG）時使用                         |
| Route 只能在 `page.jsx` 定義               | 動態資料夾只能搭配 `page.jsx`，不能單獨用在 component |
| `params.id` 一定是 string                | 即使是數字 id，params 傳進來的一律是字串             |
| 可與 `metadata`, `loading`, `error` 等共用 | 各頁專屬錯誤/loading 處理也支援動態路由              |

## 實用進階補充
`generateStaticParams`（靜態生成路由）
```jsx
export async function generateStaticParams() {
  const products = await getAllProducts();
  return products.map((p) => ({ id: p.id }));
}
```
配合上面 `page.jsx`，Next.js 就能在 build 時預先建立所有頁面！