---
title: "[React.js - Router] Index Route - 筆記"
pubDatetime: 2026-07-18T10:20:30.425Z
tags: ["React.js","React Router","Nested Route","Frontend","Advanced React"]
description: "Table of contents Index Route 定義 在Nested Route的架構下，當使用者僅輸入父..."
hackmd_id: "SkSKxAu4fg"
---

## Table of contents

## Index Route 定義  
在Nested Route的架構下，當使用者僅輸入父路由的網址（例如 `/dashboard`）時，因為網址尚未指定任何子路徑，父組件中的 `<Outlet />` 區塊預設會是一片空白。

* **`Index Route` 的唯一使命**：**「充當Nested Route中的『預設首頁』，用來填補父路由剛匹配時的空白區塊。」** 
* 它沒有自己的 `path` 屬性，而是完全依附在父路由的網址路徑上。



## 1. 為什麼需要 Index Route？

延續後台管理系統的案例，我們設定了以下路由：
* `/dashboard/users` ➡️ 顯示會員列表
* `/dashboard/products` ➡️ 顯示商品列表

如果使用者在瀏覽器直接輸入 **`/dashboard`**，React Router 只會渲染出 `DashboardLayout`（外殼）。此時因為網址後面空空的，外殼裡面的 `<Outlet />` 沒有任何子元件可以承接，導致主內容區變成**一片尷尬的空白**。

為了提供良好的使用者體驗，我們必須指定一個「預設頁面」（例如：總覽圖表頁、數據儀表板），這個角色就是 **Index Route**。



## 2. React Router 實作方式

設定 Index Route 的方式非常直覺：  
1. **不要**在 `<Route>` 寫 `path` 屬性。  
2. 直接加上一個 **`index`** 的布林屬性（或寫 `index={true}`）。

### 程式碼範例 (App.jsx)

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import DashboardLayout from "./DashboardLayout";
import DashboardHome from "./DashboardHome"; // 預設的總覽首頁
import Users from "./Users";
import Products from "./Products";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route element={<DashboardLayout path="dashboard"/>}>
          
          {/* 關鍵：Index Route */}
          {/* 當網址精確匹配 /dashboard 时，此元件會自動塞進父層的 <Outlet/> 中 */}
          <Route index element={<DashboardHome />} />
          
          {/* 其他一般子路由 */}
          <Route element={<Users path="users"/>} />
          <Route element={<Products path="products"/>} />
          
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

## 3. 網址與畫面渲染對照表  
底下的對照表清楚呈現了網址改變時，父元件與 `<Outlet />` 的動態搭配關係：

| 當前網址 | 渲染的父元件 (外殼) | `<Outlet />` 實際渲染的子元件 | 畫面呈現效果 |  
| :--- | :--- | :--- | :--- |  
| **`/dashboard`** | `DashboardLayout` | `DashboardHome` *(Index)* | **預設總覽首頁** |  
| `/dashboard/users` | `DashboardLayout` | `Users` | 會員管理頁面 |  
| `/dashboard/products` | `DashboardLayout` | `Products` | 商品管理頁面 |


## 4. Index route 特性
* **依附性**：Index route沒有 `path`，它代表的是當父路由 `path` 完全精確匹配（Exact Match） 且沒有後續子路徑時的狀態。
* **終點性（Leaf Route）**：Index route本身已經是結構的終點（葉子節點，Leaf Route），不可以在它內部再嵌套包裹其他的子路由。
* **體驗優化**：只要網頁有使用「巢狀外殼（Layout）」，都強烈建議配置一個 `index` 路由，避免使用者看到不完整的骨架網頁。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>