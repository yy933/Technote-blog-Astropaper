---
title: "[Supabase / Database] RLS 權限漏洞與資料庫架構重構 - 實作 Part 2：撰寫 Database Trigger 自動同步 User Profiles"
pubDatetime: 2026-07-27T10:34:42.216Z
tags: ["Database","Supabase","PostgreSQL","web security","Database Design","Issue","Project"]
description: "Table of contents 1. Trigger 架構與運作原理 在 PostgreSQL 中，建立一個完整的..."
---

## Table of contents

## 1. Trigger 架構與運作原理

在 PostgreSQL 中，建立一個完整的觸發器（Trigger）包含兩個部分：  
1. **Trigger Function（觸發器函式）**：定義當事件發生時，資料庫「具體要執行什麼動作」（使用 `PL/pgSQL` 撰寫）。  
2. **Trigger Object（觸發器物件）**：定義「何時觸發」（例如：在 `auth.users` 發生 `INSERT` 之後）以及套用範圍。

---

## 2. 完整 SQL 程式碼範例

進入 Supabase Dashboard ➔ **SQL Editor**（左側選單第 3 個圖示），貼入以下 SQL 並按下 **Run** 執行：

```sql
-- ========================================================
-- 1. 建立 Trigger Function (觸發器函式)
-- ========================================================
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS trigger 
LANGUAGE plpgsql 
SECURITY DEFINER
SET search_path = ''
AS $$  
BEGIN   
  -- 將新註冊的使用者資料插入 public.user_profiles   
  INSERT INTO public.user_profiles (id, name, account_type)   
  VALUES (     
    new.id,     
    new.raw_user_meta_data ->> 'name',     
    new.raw_user_meta_data ->> 'account_type'   );    
  -- 回傳代表剛加入的整筆使用者資料列 (new)   
  RETURN new;  
END;
$$;

-- ========================================================
-- 2. 建立 Trigger Object (觸發器物件)
-- ========================================================
CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW 
  EXECUTE PROCEDURE public.handle_new_user();
```

## 3. SQL 語法解析 

### A. 觸發器函式語法解析 (`public.handle_new_user`)

* `CREATE OR REPLACE FUNCTION public.handle_new_user()`：在 `public` 核心 Schema 建立（或覆蓋）名為 `handle_new_user` 的函式。
* `RETURNS trigger`：宣告這是專屬於觸發器使用的特殊函式，執行完畢會回傳觸發器訊號。
* `LANGUAGE plpgsql`：指定使用 PostgreSQL 的過程語言（PL/pgSQL）編寫。
* **`SECURITY DEFINER`（關鍵權限設定）**：
  * **功用**：使該函式以 **建立者（Database Owner / Admin）最高權限** 執行，而非觸發事件的一般使用者。
  * **好處**：寫入 `user_profiles` 時會自動繞過（Bypass）RLS 檢查，因此不需另外設定 `INSERT` RLS Policy。
* **`SET search_path = ''`（安全性防衛）**：
  * **安全性防護**：清空搜尋路徑，強制要求語法必須明確寫出 Schema（如 `public.user_profiles`），防止搜尋路徑劫持攻擊（Search Path Injection Attacks）。
* `AS $$ ... $$;`：Dollar-quoting 語法界定符，內部包裹函式核心執行的程式碼區塊（`BEGIN ... END;`）。
* **`new` 變數與 `->>` JSON 操作符**：
  * `new`：PostgreSQL 觸發器的特殊變數，代表**剛被寫入 `auth.users` 的那一筆新紀錄（Row）**。
  * `new.id`：直接讀取新使用者的 UUID。
  * `new.raw_user_meta_data`：`auth.users` 中存放註冊 JSON 資料的欄位。
  * `->>` 雙箭頭：從 JSON 物件中**提取數值並轉為純文字 (Text)** 形態。例如 `new.raw_user_meta_data ->> 'account_type'`。
* `RETURN new`：回傳處理完成的新資料列，讓 Supabase 正常結束註冊流程。



### B. 觸發器物件語法解析 (`on_auth_user_created`)

* `CREATE TRIGGER on_auth_user_created`：建立名為 `on_auth_user_created` 的觸發器。
* `AFTER INSERT ON auth.users`：指定**觸發時機**為資料成功寫入 `auth.users` 表「之後」。
* `FOR EACH ROW`：指定針對「每一個新增資料列」都會觸發一次。
* `EXECUTE PROCEDURE public.handle_new_user()`：條件滿足時自動呼叫執行上述寫好的 Trigger Function。



## 4. 驗證與測試步驟

語法執行完畢後，需進行舊資料清理與新功能測試：

1. **清理舊資料**：
   * 進入 Supabase Dashboard ➔ **Authentication ➔ Users**。
   * 將舊有的測試帳號全部刪除（確保 `auth.users` 與 `user_profiles` 資料一致，沒有無對應 Profile 的孤立帳號）。

2. **前端測試**：
   * 在前端註冊一個 **Admin** 帳號（如 Pam）。
   * 在前端註冊一個 **Sales Rep** 帳號（如 Dwight）。

3. **後端資料核對**：
   * **檢查 `auth.users`**：確認使用者已建立，且點擊查看 Raw JSON 確認 `raw_user_meta_data` 包含 `name` 與 `account_type`。
   * **檢查 `public.user_profiles`**：進入 **Table Editor**，確認 Trigger 是否已自動將 `id`、`name` 與 `account_type`（如 `'admin'` / `'rep'`）寫入對應的資料列。

## 5. 下一步 (Next Steps)  
Trigger 設定完成後，使用者註冊與 Profile 資料搬移已經可以自動且安全地執行。接下來的重構步驟：

[ ] 重構 sales_deals 資料表：移除 name 純文字欄位，改用 user_id 作為外鍵。

[ ] 更新 sales_deals 的 RLS Policy：針對 Reps 與 Admins 設定動態權限。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>