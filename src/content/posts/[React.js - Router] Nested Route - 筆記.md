---
title: "[React.js - Router] Nested Route - 筆記"
pubDatetime: 2026-07-17T10:15:35.173Z
tags: ["React.js","React Router","Nested Route","Outlet","Advanced React","Frontend"]
description: "什麼是 Nested Route ? 在現代網頁設計中，我們常常遇到這種需求：當網..."
hackmd_id: "rkeeVpdv4Gl"
---

## Table of contents

## 什麼是 Nested Route ?   
在現代網頁設計中，我們常常遇到這種需求：**當網址改變時，網頁的某個大區塊（例如導覽列、側邊欄）保持不動，只有裡面的主內容區在切換。**

**Nested Route** 的核心概念就是「**大頁面包小頁面**」。它允許我們在父路由（外殼）內部宣告子路由（子頁面），並透過局部更新的方式，只替換變動的元件，而不需要重新渲染整個網頁。



## 1. 案例：後台管理系統

想像一個常見的後台介面，網址與畫面結構如下：

* `/dashboard/users` ➡️ 顯示「會員管理」
* `/dashboard/products` ➡️ 顯示「商品管理」

這兩個頁面都共享了同一個外殼——左側的「功能選單欄」與上方的「Header 狀態欄」。

```text
 網址：/dashboard
 +-----------------------------------+
 |             Header                |
 +------------+----------------------+
 |            |                      |
 |  功能選單  |   (這裡目前是一片空白) |
 |  (Sidebar) |                      |
 |            |                      |
 +------------+----------------------+

 當你點擊「會員管理」，網址變成 /dashboard/users：
 +-----------------------------------+
 |             Header                |
 +------------+----------------------+
 |            |  [Nested 子頁面]     |
 |  功能選單  |                      |
 |  (Sidebar) |   會員列表表格...    |
 |            |                      |
 +------------+----------------------+
 ```
 
## 傳統路由 vs. Nested Route 差異

### 傳統路由 (頁面獨立)
* 畫面切換行為：換網址時，整張網頁全部洗掉，外殼（Sidebar）跟著重新渲染。
* 使用者體驗影響：畫面閃爍、輸入框打到一半的字消失、側邊欄捲軸歸零。

### Nested Route 
* 畫面切換行為：換網址時，外殼保持原樣，僅有指定的空白區塊（插槽）進行切換。
* 使用者體驗影響：元件狀態完美保留，切換流暢，效能極佳。
 
 
 ## 2. React Router 實作步驟  
在 React Router 中實作 Nested Route 只需要兩個關鍵步驟：

* 路由配置：將 `<Route>` 元件進行上下階層的嵌套包裹。
* 插槽挖空：在父路由元件中使用 `<Outlet />` 當作子頁面的放置槽。

### 步驟 A：路由配置 (`App.jsx`)  
將子路由包裹在父路由的 `<Route>` 裡面。此時子路由的 `path` 會自動繼承並拼接到父路由後面。

```javascript
import { BrowserRouter, Routes, Route } from "react-router-dom";
import DashboardLayout from "./DashboardLayout";
import Users from "./Users";
import Products from "./Products";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* 1. 父路由：定義了共同的外殼組件 DashboardLayout */}
        <Route element={<DashboardLayout/>} path="dashboard">
          
          {/* 2. 子路由：網址會自動拼接成 /dashboard/users */}
          <Route element={<Users/>} path="users" />
          
          {/* 3. 子路由：網址會自動拼接成 /dashboard/products */}
          <Route element={<Products/>} path="products" />
          
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

### 步驟 B：在外殼組件中挖空 (DashboardLayout.jsx)  
這是最核心的部分！**父元件必須宣告 `<Outlet />`，告訴 React Router：「當子路由符合時，把子元件塞進這個位置！」**

```javascript
import { Outlet, Link } from "react-router-dom";

export default function DashboardLayout() {
  return (
    <div className="dashboard-container">
      {/* 固定不動的側邊選單欄 */}
      <aside className="sidebar">
        <nav>
          {/* 注意：這裡直接寫相對路徑即可 */}
          <Link to="users">會員管理</Link>
          <Link to="products">商品管理</Link>
        </nav>
      </aside>

      {/* 右側主內容區 */}
      <main className="main-content">
        {/* 關鍵點！符合的子路由組件 (Users 或 Products) 會被動態渲染在這裡 */}
        <Outlet/> 
      </main>
    </div>
  );
}
```

## 3. 使用 Nested Route 的三大優勢

### 1. 程式碼重複利用（DRY - Don't Repeat Yourself）  
不需要在 `Users.jsx` 寫一遍 `Sidebar`，又在 `Products.jsx` 寫一遍 `Sidebar`。所有共享的 UI（如 `Navigation`, `Footer`）都只需要在外殼寫一次，後續維護極度省力。

### 2. 狀態完美保留（State Preservation）  
因為切換子網址時，父組件（Layout）完全沒有經歷卸載與重繪（Unmount & Re-render），因此側邊欄的滾動條位置、輸入框裡打到一半的文字、甚至是展開的選單狀態，都會完美保留。

### 3. 語意化的網址結構  
網址的階層（如 `/dashboard/users`）與程式碼中組件嵌套的結構完全對應，能一目了然整個專案的頁面邏輯。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>