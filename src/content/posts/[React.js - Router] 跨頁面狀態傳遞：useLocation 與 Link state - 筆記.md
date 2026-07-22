---
title: "[React.js - Router] 跨頁面狀態傳遞：useLocation 與 Link state - 筆記"
pubDatetime: 2026-07-22T04:22:51.344Z
tags: ["others"]
description: "Table of contents 保留使用者操作狀態（State Persistence） 在開發列表頁（List..."
hackmd_id: "H1szzTpNzl"
---

## Table of contents

## 保留使用者操作狀態（State Persistence）  
在開發列表頁（List View）與詳細頁（Detail View）時，常見的問題是：

* 使用者在列表頁進行了條件篩選（如：`/vans?type=luxury`）或滾動位置搜尋。
* 點擊某個項目進入詳細頁（`/vans/123`）後，點擊「返回按鈕」回到列表頁。
* 若未傳遞原先的 URL 參數，使用者會回到未篩選的初始列表頁，造成體驗斷層。

React Router 提供了 `<Link state>` 與 `useLocation` 組合，讓開發者可以在頁面切換時攜帶「隱形資料」，完美實現返回時狀態保留的功能。

## 步驟拆解
### Step 1. 在傳送方（發出頁面）：`<Link state={...}>`  
`<Link>` 元件除了 `to` 屬性（指定目標網址）外，還支援 `state` 屬性。

💡 state 屬性的特點：

* 可以傳遞任何 JavaScript 物件。
* 資料不會顯示在網址列（URL）上，適合傳遞不希望暴露在網址上的情境資料。
* 資料會儲存在瀏覽器的 `window.history.state` 中。

```javascript
// Vans.jsx (列表頁)
import { Link, useSearchParams } from "react-router-dom"

export default function Vans() {
    const [searchParams] = useSearchParams()

    return (
        <Link 
            to={van.id} 
            // 💡 將當前的 URL Search Params (例如 "?type=luxury") 打包進 state 傳出
            state={{ search: `?${searchParams.toString()}` }}
        >
            <img src={van.imageUrl} alt={van.name} />
            {/* ... */}
        </Link>
    )
}
```

### Step 2. 在接收方（目標頁面）：`useLocation()`  
`useLocation` 是 React Router 用來取得當前路由 `location` 物件的 Hook。

`location` 物件包含以下資訊：
* `pathname`：當前路徑名稱（例如 `/vans/123`）
* `search`：當前 URL 查詢字串（例如 `?type=luxury`）
* `hash`：URL 的 Anchor 錨點
* `state`：上一頁傳過來的 `state` 資料物件（若無則為 `null`）

```javascript
// VanDetail.jsx (詳細頁)
import { Link, useParams, useLocation } from "react-router-dom"

export default function VanDetail() {
    const location = useLocation()

    // 💡 取得上一頁帶過來的 search 參數，若無（例如直接貼網址進來）則退回空字串
    const search = location.state?.search || ""

    return (
        <div className="van-detail-container">
            {/* 💡 拼接返回目標：回到 "../?type=luxury"，帶使用者返回篩選後的列表 */}
            <Link
                to={`..${search}`}
                relative="path"
                className="back-button"
            >
                &larr; <span>Back to all vans</span>
            </Link>

            {/* ...詳細頁內容 */}
        </div>
    )
}
```

## 2. 運作流程圖解

```
[Vans 列表頁] (網址: /vans?type=luxury)
  │
  ├─ 使用者點擊露營車卡片
  ├─ <Link to="1" state={{ search: "?type=luxury" }}> 攜帶 state 切換頁面
  │
  ▼
[VanDetail 詳細頁] (網址: /vans/1)
  │
  ├─ useLocation() 接收 location.state ➡️ { search: "?type=luxury" }
  ├─ 動態組裝返回按鈕 <Link to={`..${search}`} relative="path">
  ├─ 實際生成的 HTML 連結目標："/vans?type=luxury"
  │
  ▼
[點擊 Back 按鈕] ➡️ 成功返回並恢復使用者的篩選狀態！
```

## 3. 防護機制：Optional Chaining (?.) 的重要性  
在讀取 `location.state` 時，務必使用 Optional Chaining (`?.`) 並提供預設回退值：

```javascript
// 🟢 正確且安全的寫法
const search = location.state?.search || ""
```

### 為什麼這很關鍵？
* 直接貼上網址 / 書籤開啟：使用者並非從列表頁點擊 `<Link>` 進來，`location.state` 會是 `null`。
* 重新整理頁面（Refresh）：在某些瀏覽器環境中 `history.state` 可能被重置。

若直接寫 `location.state.search`，在上述情況下會引發 `TypeError: Cannot read properties of null` 導致頁面崩潰！使用 `?.` 可確保即使沒有 `state`，也能優雅地降級退回原來的連結目標（例如 `".."`）。


## 4. 比較 searchParams vs. location.state

| 特性 | URL Search Params (`?type=luxury`) | Link State (`state={{ ... }}`) |  
| :--- | :--- | :--- |  
| **可見性** | **公開**（顯示於瀏覽器網址列） | **隱蔽**（不影響 URL 結構） |  
| **可分享性** | **可分享**（複製網址給他人可保留狀態） | **不可分享**（僅限當次瀏覽器 Session/歷史記錄） |  
| **資料類型** | 僅限 **字串 (String)** | 任意 **JavaScript 物件 / 陣列** |  
| **主要用途** | 列表篩選、分頁、搜尋關鍵字 | 跨頁面歷史回溯、帶入暫存 Context（如前頁按鈕名稱） |


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>