---
title: "React 元件 - 筆記"
pubDatetime: 2026-05-25T11:17:36.187Z
tags: ["JavaScript","React.js"]
description: " Table of contents 元件是什麼？ 元件（Component）：把 HTML、CSS 和 JavaScr..."
---

## Table of contents


## 元件是什麼？
* 元件（Component）：把 HTML、CSS 和 JavaScript 合在一起，做成「可以重複使用」的 UI 元素。
* 例如：可以把目錄區塊做成 `<TableOfContents /> `元件，每一頁都用。

## 組合元件（Component Composition）
元件可以像積木一樣「組合」、「排序」、「巢狀」成整個頁面。
範例：
```jsx
<PageLayout>
  <NavigationHeader>
    <SearchBar />
    <Link to="/docs">Docs</Link>
  </NavigationHeader>
  <Sidebar />
  <PageContent>
    <TableOfContents />
    <DocumentationText />
  </PageContent>
</PageLayout>

```

## 如何定義元件？

### 步驟一：匯出元件


`export default function Profile() { ... }`

用 `export default` 讓這個元件可以被其他檔案匯入。

### 步驟二：定義函式

使用 **大寫開頭** 的函式名稱，例如 `Profile`。

小寫會被當作 `HTML` 標籤處理，無法正確渲染。

### 步驟三：加上 Markup（JSX）

JSX 語法：在 JavaScript 裡寫類似 HTML 的語法。

* 簡單寫法（單行）：
`return <img src="..." alt="..." />;`

* 多行寫法 **（要加括號）**：
```jsx
return (
  <div>
    <img src="..." alt="..." />
  </div>
);
```
:::warning
✅ 注意：`return` 後沒加括號，下面的 JSX 會被忽略！
:::

## 使用元件
* 直接像 HTML 標籤一樣使用自訂元件。
* 範例:
```jsx
function Profile() { ... }

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}

```
* React 如何分辨？
  - `<section>` → 小寫，原生 HTML。
  - `<Profile />` → 大寫，自訂元件。
  
* 最後瀏覽器看到的是純 HTML：
```html
<section>
  <h1>Amazing scientists</h1>
  <img src="..." alt="..." />
  <img src="..." alt="..." />
  <img src="..." alt="..." />
</section>
```

## 組織元件（Nesting and Organizing Components）
* 小元件可以跟主元件寫在同一個檔案（方便管理）。
* 如果太多元件，建議「分開檔案」整理。
:::warning
:bulb: 重要原則：**不要在元件內再定義新的元件！**

錯誤寫法（會變慢又容易出錯）：
```jsx
export default function Gallery() {
  function Profile() { ... } // 不要這樣寫!!
  ...
}
```

正確寫法：
```jsx
function Profile() { ... }
export default function Gallery() { ... }
```
:exclamation: 如果要把資料傳給子元件，用「props」，而不是把元件寫在一起。
:::

## Recap
### 核心觀念
1. 元件 = 函式 + JSX
* 一個元件就是一個 JavaScript 函式，回傳一段 JSX 標記。

2. 小寫是 HTML，大寫是元件
* `<div>` 是 HTML。
* `<MyComponent />` 是自定義的元件。

3. 組裝頁面 = 組裝元件
* 元件可以巢狀、重複使用，像堆積木一樣組成整個畫面。

### 常見錯誤

#### 1. 忘記大寫元件名稱
```jsx
function profile() { ... } // 🔴 錯誤：小寫 profile 不會被當成元件
function Profile() { ... } // ✅ 正確：大寫 Profile
```
:bulb: 元件名稱一定要「大寫開頭」。

#### 2. `return` 後面沒加括號，結果什麼都沒渲染

```jsx
return
  <div>Hello</div>; // 🔴 錯誤：<div> 這行直接被無視！
```

正確寫法：
```jsx
return (
  <div>Hello</div>
);
```
:bulb: 只要 return 後不是同一行，要用小括號包起來。

#### 3. 在元件內部「定義」子元件

```jsx
function Gallery() {
  function Profile() { ... } // 🔴 不要這樣，會拖慢效能
  return <Profile />;
}
```

:bulb: 所有元件都在檔案「最外層」定義，不要巢狀定義。

####  4. JSX 語法小失誤
* 所有標籤要正確關閉，例如 `<img />`。
* 在 JSX 裡，屬性用 駝峰式命名（小駝峰）：
  - `class` → `className`
  - `for` → `htmlFor`

### React元件範例
```jsx
function MyComponent() {
  return (
    <div>
      <h1>Hello, React!</h1>
    </div>
  );
}

export default MyComponent;
```