---
title: "[React.js - Router] 如何決定是否使用 Nested Routes？- 筆記"
pubDatetime: 2026-07-18T16:09:51.000Z
tags: ["React.js","React Router","Nested Route","Frontend","Advanced React"]
description: "Table of contents 核心判斷標準：視覺結構與狀態留存 很多人會誤以為「網址有斜線（例如 /user/s..."
hackmd_id: "Sy-0MmYNGl"
---

## Table of contents

## 核心判斷標準：視覺結構與狀態留存  
很多人會誤以為「網址有斜線（例如 `/user/settings`）」就一定要寫成 Nested Routes。這是不對的。

決定是否使用Nested Routes的判斷標準：  
1. **視覺重複性（Visual Hierarchy）**：切換網址時，畫面上是否有「很大一部分的元件（如導覽列、側邊欄、分頁標籤）」是**完全不需要動、保持原樣**的？  
2. **狀態留存性（State Preservation）**：切換網址時，那些重複的元件內部狀態（如側邊欄的滾動位置、輸入框打到一半的字、選單展開狀態）是否**需要被完美保留**？

> 💡 **核心結論**：如果希望切換網址時，外殼「完全不動且不重新渲染」，只刷新裡面的小區塊，就必須使用 Nested Routes。



## 1. 什麼時候「要」使用？

### 情境 A：Layout 外殼架構  
不論是後台管理系統（Admin Dashboard）還是前台電商，只要擁有固定的頂部導覽列（Navbar）、側邊選單（Sidebar）或頁尾（Footer）。
* **範例網址**：`/admin/analytics` 與 `/admin/orders`
* **原因**：Sidebar 和 Navbar 在換頁時不需要卸載（Unmount）重繪，用 Nested Route 可以維持極致流暢的體驗。

### 情境 B：頁面內部的 Tab（分頁標籤）切換  
在某一個特定頁面中（例如使用者個人資料頁），內部有複數個分頁，且希望分頁切換時能同步更新網址（方便使用者複製網址分享）。
* **範例網址**：
  * `/user/profile/info`（個人基本資料）
  * `/user/profile/security`（帳號安全設定）
  * `/user/profile/notifications`（通知設定）
* **原因**：使用者的頭像、基本框架都在外層固定不變，只有下方分頁內容在變。

### 情境 C：主從式架構（Master-Detail Pattern）  
左側是列表（List），點擊列表項目後，右側直接顯示詳細內容（Detail）。
* **範例網址**：`/emails`（郵件箱）➡️ 點擊某封信 ➡️ `/emails/message-123`（右側展開信件內容）
* **原因**：左側的郵件列表捲軸位置、未讀紅點狀態，在點擊閱讀時絕對不能被重置，此時右側詳細頁面非常適合做成 Nested Route。



## 2. 什麼時候「不要」使用？（避免盲目使用）

### 情境 A：網址看似有階層，但畫面是完全獨立的全新頁面  
有些網址雖然看起來像父子關係，但一換頁，整個畫面（包括導覽列）全部都要換掉。
* **反面案例**：
  * `/products`（商品列表頁，充滿購物風格的電商排版）
  * `/products/checkout`（結帳購物車頁，為了讓使用者專注結帳，畫面必須完全乾淨，沒有導覽列、沒有雜訊）
* **為什麼不用**：這兩者在視覺上沒有任何共同的 Layout 外殼，硬寫成 Nested Route 反而要寫很多髒程式碼去隱藏外殼。請直接拆成兩個獨立的頂層路由（Root Routes）。

### 情境 B：單純的動態參數切換（Dynamic ID）  
只是從列表頁點進去單一獨立的詳細頁。
* **反面案例**：從 `/blog`（部落格文章列表）點進去 `/blog/how-to-learn-react`（文章內文）
* **為什麼不用**：如果閱讀文章時，列表頁的元件完全不需要留在畫面上，那就只是單純的路由切換，不需要 `<Outlet />`。



## 3. 快速決策速查表


| 評估問題 | 🟢 答案為「是」 ➡️ 使用 Nested | 🔴 答案為「否」 ➡️ 使用獨立路由 |  
| :--- | :--- | :--- |  
| **畫面是否有共享的外殼（Layout）？** | 是，換頁時外殼要留在畫面上。 | 否，換頁時整個畫面要全洗掉。 |  
| **換頁時，外殼的狀態需要保留嗎？** | 是，例如輸入框文字、捲軸位置不能不見。 | 否，全新頁面，狀態本來就該清空。 |  
| **子頁面的網址與父頁面有強烈依附關係嗎？** | 是，子頁面離開了父頁面就失去上下文意義。 | 否，雖然網址相似，但功能與架構各自獨立。 |

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>