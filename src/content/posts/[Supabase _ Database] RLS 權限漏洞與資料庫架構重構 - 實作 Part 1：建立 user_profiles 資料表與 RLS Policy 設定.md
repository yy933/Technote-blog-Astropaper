---
title: "[Supabase / Database] RLS 權限漏洞與資料庫架構重構 - 實作 Part 1：建立 user_profiles 資料表與 RLS Policy 設定"
pubDatetime: 2026-07-27T09:53:02.815Z
tags: ["Database","Supabase","PostgreSQL","web security","Database Design","Issue","Project"]
description: "Table of contents 1. 建立 user_profiles 資料表與外鍵關聯 首先進入 Supabas..."
---

## Table of contents

## 1. 建立 `user_profiles` 資料表與外鍵關聯

首先進入 Supabase Dashboard 的 **Table Editor**（schema 選擇 `public`），點擊 **New Table** 建立資料表：

### 欄位規格表 (Schema Definition)
* **Table Name**: `user_profiles`
* **Enable Row Level Security (RLS)**: 保持 **勾選 (ON)**

| 欄位名稱 (Column) | 資料型別 (Type) | 預設值 (Default) | 說明 / 備註 |  
| :--- | :--- | :--- | :--- |  
| `id` | `uuid` | *(自動產生)* | 主鍵，並作為 Foreign Key |  
| `created_at` | `timestamptz` | `now()` | 建立時間 |  
| `name` | `text` | *(無)* | 使用者姓名 |  
| `account_type` | `text` | `'rep'` | 權限角色（預設給予最小權限 `'rep'`） |

### 設定外鍵關聯 (Foreign Key Relationship)  
為了保證資料完整性（Data Integrity），必須確保每個 Profile 都對應到真實存在的使用者：  
1. 點擊 **Add foreign key relationship**。  
2. **Schema** 選擇 `auth` ➔ **Table** 選擇 `users`。  
3. 將 `user_profiles.id` 關聯（Reference）至 `auth.users.id`。  
4. 點擊 **Save** 儲存。



## 2. 觀念解析：為什麼新增 Profile 不需要 INSERT RLS？

在常見的 CRUD 操作中，通常需要撰寫 `FOR INSERT` 的 RLS Policy，但在此處**完全不需要**：

### 關鍵原因：Security Definer (高權限執行)
* 未來我們寫入 `user_profiles` 的方式不是透過前端 API 發送 `INSERT` 請求，而是透過資料庫的 **Trigger（觸發器）**。
* 撰寫 Trigger 函式時會宣告為 **`SECURITY DEFINER`**，這代表該函式是以**資料庫擁有者（Admin）高權限**的角度執行，而非觸發事件的使用者（Caller）。
* 由於擁有最高權限，Trigger 在執行寫入時會**自動繞過（Bypass）RLS 檢查**，因此無需額外設定 INSERT Policy。

```text
[ 前端 Signup ] ➔ 觸發 Trigger ➔ 以 SECURITY DEFINER 權限執行 ➔ 自動繞過 RLS 寫入 user_profiles
```

## 3. 設定 SELECT RLS Policy：允許已驗證使用者讀取 Profiles  
雖然寫入不需要 Policy，但前端應用程式需要 `SELECT` 所有 Profile 資料來呈現圖表與驗證角色，因此必須設定讀取權限。

### Dashboard 設定步驟
* 進入 Authentication ➔ Policies 頁面。
* 找到 `user_profiles` 資料表，點擊 Create policy。

```sql
-- 建立允許已驗證使用者檢視所有 Profiles 的 Policy
CREATE POLICY "Users can view all profiles"
ON "public"."user_profiles"
AS PERMISSIVE
FOR SELECT
TO authenticated
USING (
  true
);
```

### 語法重點解析：
* `TO authenticated`：直接在 Supabase Target Roles 下拉選單選擇 authenticated。這會在 SQL 中自動加上 `TO authenticated` 子句，過濾掉 `anon`（匿名）請求，無需再呼叫 `auth.role() = 'authenticated'` 函式。
* `USING (true)`：RLS 規範必須填寫 USING 條件。填寫 `true` 代表無額外限制，只要通過 `TO authenticated` 驗證的使用者，即可讀取該表的所有資料列（Rows）。

## 4. 下一步 (Next Steps)  
資料表與 RLS 基礎防護已建置完成，接下來的重構步驟：

[ ] 修改前端 Sign Up 註冊元件（新增 `name` 欄位與 `account_type` 單選按鈕）。

[ ] 撰寫 Supabase Database Trigger，將註冊時的 `user_metadata` 自動搬移至 `user_profiles`。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>