---
title: "[Supabase] anon key 與 RLS (Row Level Security) - 筆記"
pubDatetime: 2026-07-24T08:21:19.531Z
tags: ["Supabase","JWT","Database","Backend","token","web security","Authentication","React.js"]
description: "Table of contents 什麼是 anon key ? anon key（Anonymously Key /..."
hackmd_id: "HJe25cgSGx"
---

## Table of contents

## 什麼是 anon key ?
**anon key（Anonymously Key / 匿名金鑰）** 是 Supabase 提供給前端（瀏覽器、手機 App）使用的公開金鑰。

* **功能**：作為前端存取 Supabase API Gateway 的「入門憑證」，用來識別專案，並代表 **「未登入 / 訪客 (anon)」** 的身份。
* **安全性**：可安全暴露在前端環境變數（如 `.env`）中。它的安全防線並非靠隱藏金鑰，而是配合資料庫內部的 **RLS（Row Level Security）** 規則。



## 1. anon key 的本質就是一個 JWT

`anon key` 完全遵循標準 JWT（JSON Web Token）架構，由三段字串組成：`Header.Payload.Signature`。

如果把 Supabase 後台的 `anon key` 複製下來，貼到 [jwt.io](https://www.jwt.io/) 解密，會看到完整的結構：

* Header (標頭)：紀錄演算法，例如 `{"alg": "HS256", "typ": "JWT"}`。
* Payload (內容)：存放身份與權限資訊，最重要的是包含了 `"role": "anon"`。
* Signature (簽名)：Supabase 用伺服器內部的 JWT Secret 加密產生的防偽簽名。只要有人試圖改動 Payload 裡的 `"role": "anon"`（例如想把自己改成最高權限），簽名比對就會失敗，請求會立刻被 Supabase 阻擋。

如果將 `anon key` 解碼，會看到如下內容：

```json
{
  "iss": "supabase",
  "ref": "your-project-ref",
  "role": "anon", // 👈 核心：標記角色為匿名訪客
  "iat": 1600000000,
  "exp": 1900000000
}
```

### 身份轉化機制 (Auth Roll Shift)  
當前端對 Supabase 發起請求時，會透過 HTTP Header (`Authorization: Bearer <token>`) 攜帶 JWT：

```
                    ┌─── 1. 未登入 ───> 帶 anon key ────────────> 資料庫角色：anon
前端發起 API 請求 ───┤
                    └─── 2. 已登入 ───> 帶 user access_token ───> 資料庫角色：authenticated
```

* 未登入（訪客）：帶有 `anon key`（JWT 內的 `role: "anon"`），資料庫以 `anon` 角色執行查詢。
* 已登入使用者：登入成功後，Supabase Client 會自動用使用者的 `access_token`（JWT 內的 `role: "authenticated"`）覆蓋 `anon key`，資料庫改以 `authenticated` 角色執行查詢。


## 2. 什麼是 RLS (Row Level Security) ?  
RLS（行級安全性 / 列級安全性原則） 是建置於 PostgreSQL 資料庫層級的安全防禦機制。

### 白話比喻：傳統權限 vs. RLS
* 傳統權限 (Table-level)：像「門禁卡」。拿到卡片的人可以進入 `sales_deals` 房間，並看光房間內所有的資料。
* RLS 權限 (Row-level)：像「銀行保險箱」。所有人都可以進入 `sales_deals` 房間，但每個人只能打開印有自己名字的那一排保險箱 (Row)。

### 為什麼 Supabase 架構必須開啟 RLS？  
因為 Supabase 讓前端直接與資料庫溝通，不經過傳統的中間層 API 伺服器（Node.js/Spring Boot）。如果不開啟 RLS，任何人打開瀏覽器 DevTools 就能拿 `anon key` 刪光或看光整張資料表！

在傳統開發中，前端不會直接連資料庫，我們會寫一段 Express/Spring Boot 後端程式：

```
前端 React  ───>  後端 Node.js API (寫安全檢查)  ───>  資料庫
```

後端程式會幫我們寫 `WHERE user_id = current_user.id` 來防範 A 使用者抓到 B 使用者的資料。

但在 Supabase 架構中，沒有傳統後端 API：

```
前端 React  ───(帶著 anon key)───>  Supabase (直接存取資料庫)
```

因為前端可以直接呼叫 `supabase.from('sales_deals').select()`，如果沒有安全機制，任何人開啟瀏覽器 DevTools 就能抓出全公司的資料！

所以 Supabase 將安全檢查直接搬進資料庫內部，這就是 RLS。

## 3. RLS 實作與語法範例  
在 Supabase SQL 控制台中，可以針對資料表寫入 RLS Policy：

### A. 限制使用者只能存取「自己的」資料

```sql
CREATE POLICY "使用者只能看自己的資料" 
ON sales_deals 
FOR SELECT 
USING ( auth.uid() = user_id );
```

* 運作原理：`auth.uid(`) 會自動解析當前請求 JWT 內的使用者 ID。當前端呼叫 `.select()` 時，PostgreSQL 會在背景自動加上 `WHERE user_id = auth.uid()`。


### B. 允許訪客 (anon key) 讀取產品列表，但禁止寫入

```sql
CREATE POLICY "所有人皆可讀取產品" 
ON products 
FOR SELECT 
TO anon 
USING ( true );
```

* 運作原理：持有 `anon key` 的訪客可以執行 `SELECT`，但如果嘗試執行 `INSERT` 或 `DELETE`，資料庫會直接拒絕請求。


## 4. 兩大金鑰對比：anon key vs. service_role key  
Supabase 後台會提供兩把金鑰，開發時切勿混淆：

| 金鑰類型 | 本質 | 權限範圍 | 存放與使用位置 |  
| :--- | :--- | :--- | :--- |  
| **anon key** | JWT (`role: anon`) | **受限於 RLS**，通常僅有公開讀取或受限權限 | **前端** (React, Vue, Mobile App) |  
| **service_role key** | JWT (`role: service_role`) | **完全繞過 RLS**，擁有資料庫最高管理權限 (Admin) | **後端伺服器** (Node.js, Next.js API Routes)，**嚴禁暴露給前端** |



## 總結
* `anon key` = 帶著 `"role": "anon"` 標籤的 JWT，**用來告訴資料庫：「現在來存取的是前端訪客」。**
* RLS = 資料庫內部的規則守門員，它會根據 `anon key` 或使用者登入後的 JWT 裡面的身份資訊，決定 「**這筆資料列 (Row) 能不能讓你讀取或修改**」。


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>