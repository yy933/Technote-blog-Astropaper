---
title: "[Supabase / Database] RLS 權限漏洞與資料庫架構重構"
pubDatetime: 2026-07-27T08:30:32.881Z
tags: ["Database","Supabase","PostgreSQL","Database Design","web security","Issue","Project"]
description: "Table of contents 1. 需求與問題 (Requirements & Problems) 我們希望在系..."
---

## Table of contents

## 1. 需求與問題 (Requirements & Problems)

我們希望在系統中新增兩種使用者帳號類型（Account Type）：
* **Sales Reps（業務代表）**：僅能新增與存取「屬於自己」的交易資料（Deals）。
* **Admins（管理者）**：可以為「任何業務代表」新增與管理交易資料。

然而，在現行的設計中，我們遇到了**兩個嚴重的安全性與架構瓶頸**：

```text
[ 嘗試方案 A ] 儲存於 user_metadata ➔ ❌ 安全漏洞：可被前端用 auth.updateUser() 惡意竄改提權
[ 嘗試方案 B ] 依靠 sales_deals.name 比對 ➔ ❌ 判斷瓶頸：純文字無拘束且 JWT 內無可靠姓名資料
```

### 瓶頸一：`user_metadata` 存在權限漏洞 (Self-Upgrade Risk)  
Supabase 允許將自訂資料存入 `auth.users.raw_user_metadata`。雖然可以在 RLS 中透過 `auth.jwt() -> 'user_metadata' ->> 'account_type'` 來讀取，但 Supabase SDK 提供前端 `supabase.auth.updateUser()` 函式，這代表使用者可以在瀏覽器端自由修改自己的 `user_metadata`（例如將 `rep` 改為 `admin`）。

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

⚠️ 安全法則：切勿將關鍵的權限/角色資訊（如 `account_type`）儲存在 `user_metadata` 中並據此撰寫 RLS Policy。

</blockquote>

### 瓶頸二：`sales_deals` 表的文字欄位與權限驗證問題  
現有的 `sales_deals` 表是透過純文字欄位 `name` 來辨識資料擁有者。

* 資料不嚴謹：文字欄位容易打錯字、出現重複，且非資料庫產生的唯一識別碼（UID）。
* 無法寫入 RLS：JWT 內只有 `Email`，沒有姓名，RLS Policy 無法拿 `sales_deals.name` 與 JWT 進行可靠的比對。

## 2. 解決方式：三表關聯架構 (The 3-Table Architecture)  
為了徹底解決上述問題，我們將架構重構成三張表：

```text
[ auth.users ]
       │ ID (PK / Supabase 內建管理)
       │
       ▼
 [ user_profiles ]  <─── (存放 Name & Account Type，RLS 權限判斷核心)
       │ ID (PK / FK to auth.users.id)
       │
       ▼
  [ sales_deals ]   <─── (移除 Name 欄位，改用 User ID)
         User_ID (FK to user_profiles.id)
```

* 改用 `user_id` (UUID) 替代純文字 `name`：`sales_deals.user_id` 作為外鍵連至 `user_profiles.id`。
* 建立獨立的 `user_profiles` 資料表：儲存 `name` 與 `account_type`。此表受 RLS 保護，前端無法直接任意修改，確保權限資料的絕對安全。
* 資料查詢：前端圖表若需要顯示業務姓名，透過 `JOIN` 將 `sales_deals` 與 `user_profiles` 進行關聯即可。

## 3. 安全傳輸機制：Database Trigger（觸發器）  
既然 `user_metadata` 容易被竄改，註冊時該如何安全地將 `name` 與 `account_type` 寫入 `user_profiles`？

### 核心策略：將 `user_metadata` 作為「一次性傳送包裹」

```text
[ 前端 Signup Form ] ──> 寫入 auth.users (帶入 metadata 包裹)
                                │
                        (自動觸發 DB Trigger)
                                │
                                ▼
                     [ 寫入 user_profiles 表 ]  <─── 🛡️ RLS 權限控管層
```

1. 註冊階段：前端註冊時，將 `name` 與 `account_type` 作為一次性欄位帶入 `user_metadata` 中。  
1. 觸發器自動搬移：資料庫設定 Database Trigger，當 `auth.users` 有新資料寫入時，自動將資料複製並轉寫至 `user_profiles` 表。  
1. 安全隔離：RLS 完全不依賴 `user_metadata` 作為權限比對。即使使用者後續在前端改動 `user_metadata`，也不會影響到 `user_profiles` 內部的真實權限設定。

## 4. 工作流 (Workflow)
### 階段 A：使用者註冊與觸發器運作  
1. 使用者在前端填寫 Email、密碼、`name` 並選擇 `account_type`（`Rep` 或 `Admin`）。  
1. Supabase 將使用者新增至 `auth.users`。  
1. Database Trigger 被觸發，自動將 `id`, `name`, `account_type` 寫入 `user_profiles`。  
1. Supabase 發放 JWT 並在 client 端建立 Session。

### 階段 B：新增交易資料 (New Deal) 的 UX 與 RLS 防護
- **若身份為 Sales Rep（業務代表）：**
  - UX 層級：新增 Deal 表單的「姓名」自動帶入該 Rep 姓名並設為唯讀（Read-only）。
  - RLS 防護：資料庫檢查插入的 `sales_deals.user_id` 是否等於當前請求者 JWT 的 `auth.uid()`。

- **若身份為 Admin（管理者）：**
  - UX 層級：表單可下拉選擇任意 Rep 的姓名。
  - RLS 防護：RLS 確認請求者在 `user_profiles` 中的 `account_type` 為 `admin` 後放行。

## 5. 重構執行待辦清單 (Refactoring To-Do List)  
[ ] 建立 `user_profiles` 資料表 並設定適當的 RLS Policy。

[ ] 撰寫 Supabase Trigger 函式，處理註冊後的資料自動搬移。

[ ] 修改 `Sign Up` 註冊元件，允許傳入 `name` 與 `account_type`。

[ ] 重構 `sales_deals` 資料表：移除 `name` 欄位，新增 `user_id` 欄位與對應 RLS。

[ ] 前端選單與統計邏輯更新：

  - 撈取 `user_profiles` 列表以供選單使用。
  - 更新 Deal 表單與 `fetchMetrics` 統計函式（改用 `JOIN` 撈取業務姓名以繪製圖表）。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>