---
title: "[Supabase / Database] RLS 權限漏洞與資料庫架構重構 - 實作 Part 3： sales_deals 資料表重構與新增 INSERT RLS Policies"
pubDatetime: 2026-07-28T03:42:29.886Z
tags: ["Database","Supabase","PostgreSQL","web security","Database Design","Issue","Project"]
description: "Table of contents 1. 重構 sales_deals 資料表 (Add Foreign Key) 為..."
---

## Table of contents

## 1. 重構 sales_deals 資料表 (Add Foreign Key)

為了將交易紀錄與使用者帳號關聯，我們需要在 `sales_deals` 中建立 `user_id` 欄位並指向 `user_profiles(id)`：

```sql
-- 1. 新增 user_id 欄位並設定 Foreign Key 關聯 (當使用者被刪除時保留紀錄，欄位設為 NULL)
ALTER TABLE public.sales_deals
ADD COLUMN user_id uuid REFERENCES public.user_profiles(id) ON DELETE SET NULL;

-- 2. 為外鍵建立索引 (Index)，大幅優化後續 RLS 與 JOIN 查詢的效能
CREATE INDEX idx_sales_deals_user_id ON public.sales_deals(user_id);
```

## 2. INSERT RLS Policies 核心權限設計  
在重構 `sales_deals` 的新增權限時，業務邏輯包含兩種角色情境：

* Sales Rep（業務人員）：只能新增「屬名為自己 (`user_id = auth.uid()`)」的交易紀錄。
* Admin（管理者）：可以為「任何人」新增交易紀錄。

### 關鍵語法觀念
* `WITH CHECK`：適用於 `INSERT` 與 `UPDATE` 動作，用來驗證「即將寫入或變更的新資料」是否符合規則。
* 直接使用欄位名稱：在 `WITH CHECK` 內部，可以直接呼叫`user_id`，PostgreSQL 會自動將其識別為「使用者企圖寫入的那筆資料欄位」。
* `EXISTS (SELECT 1 ...)`：當需要「跨表」查詢權限（如檢查 `user_profiles` 中的 `account_type`）時使用。`SELECT 1` 比 `SELECT *` 更高效，因為資料庫只需要知道「是否存在這筆資料（`True`/`False`）」，而不需要提取資料內容。

## 3. 完整 SQL 程式碼範例  
進入 Supabase Dashboard ➔ SQL Editor 執行以下 SQL：

```sql
-- ========================================================
-- 1. Sales Rep 的 INSERT Policy (只能新增自己的 Deals)
-- ========================================================
CREATE POLICY "Allow authenticated reps to insert their own deals"
ON public.sales_deals
FOR INSERT
TO authenticated
WITH CHECK (
  -- 檢查 1：確保寫入的 user_id 就是發起請求的使用者本人
  user_id = auth.uid()
  AND
  -- 檢查 2：跨表確認該使用者在 user_profiles 中的身份為 'rep'
  EXISTS (
    SELECT 1 
    FROM public.user_profiles
    WHERE user_profiles.id = auth.uid()
      AND user_profiles.account_type = 'rep'
  )
);

-- ========================================================
-- 2. Admin 的 INSERT Policy (可以為任何人新增 Deals)
-- ========================================================
CREATE POLICY "Allow authenticated admins to insert any deal"
ON public.sales_deals
FOR INSERT
TO authenticated
WITH CHECK (
  -- 僅需跨表確認發起請求的使用者在 user_profiles 中的身份為 'admin'
  EXISTS (
    SELECT 1 
    FROM public.user_profiles
    WHERE user_profiles.id = auth.uid()
      AND user_profiles.account_type = 'admin'
  )
);
```

## 4. Reps 與 Admins RLS Policy 對照表

| 角色類別 | 政策名稱 | WITH CHECK 核心條件 | 權限效果 |  
| :--- | :--- | :--- | :--- |  
| **Sales Rep** | `Allow authenticated reps to insert their own deals` | `user_id = auth.uid() AND EXISTS (...)` | **綁定本人**：只能幫自己新增 Deal |  
| **Admin** | `Allow authenticated admins to insert any deal` | 僅 `EXISTS (...)` | **開放全權**：可填入任意使用者的 `user_id` |


> 💡 註：當 PostgreSQL 資料表有多個同動作（如 `FOR INSERT`）的 Policy 時，會以 `OR` 邏輯結合。只要發起的 Request 符合其中一種 Policy 的條件，就能成功執行該動作。

## 5. 驗證與測試步驟  
1. Rep 權限測試：
  - 以 Rep 帳號登入前端，新增一筆資料，並帶入自己的 `user_id` ➔ 成功。
  - 試圖在 payload 中將 `user_id` 修改為其他人的 UUID ➔ 被 RLS 擋下 (Error)。

2. Admin 權限測試：
  - 以 Admin 帳號登入前端，新增交易並選擇任意 Rep 的 `user_id` ➔ 成功。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>