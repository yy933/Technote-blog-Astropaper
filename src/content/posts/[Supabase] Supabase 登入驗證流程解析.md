---
title: "[Supabase] Supabase 登入驗證流程解析"
pubDatetime: 2026-07-30T07:14:52.859Z
tags: ["Database","Supabase","PostgreSQL","web security","Authentication","JWT","token","session","cookie","Concepts","cheatsheet"]
description: "Table of contents 1. 登入驗證流程 (Authentication Phase) 當我們在前端呼叫..."
hackmd_id: "BJts7ddrGl"
---

## Table of contents

## 1. 登入驗證流程 (Authentication Phase)  
當我們在前端呼叫 `supabase.auth.signInWithPassword({ email, password })` 時，發生的事情如下：

```text
[前端 React] ──(1) Email + Password ──> [Supabase Auth (GoTrue)]
                                              │
                                       (2) 比對雜湊密碼
                                              │
[前端 React] <──(3) 返回 JWT (Session) ───────┘
```

* 發送憑證：前端將帳號密碼透過 HTTPS 發送到 Supabase 的 Auth 伺服器（GoTrue API）。
* 密碼比對：GoTrue 會去內部隱藏的 `auth.users` 資料表尋找該 Email，並使用加鹽雜湊（Bcrypt）比對密碼。
* 核發 JWT (JSON Web Token)：驗證成功後，Supabase 會簽發一對關鍵 Token 並組合成 Session 物件傳回前端：
  - `access_token`：短效期的 JWT（預設 1 小時失效），裡面包含了使用者的 `id` (UUID)、`email` 與 `role` (`authenticated`)。
  - `refresh_token`：長效期的隨機字串（預設無限制或數天），專門用來在 `access_token` 過期時換取新的 Token。

## 2. Session、LocalStorage 與 Cookie 存放機制  
在純前端（React + Vite SPA） 的預設環境下，Supabase SDK 的處理方式如下：

```text
[Supabase Client SDK]
   ├── 自動寫入 ──> [Browser LocalStorage]
   │                 └── key: `sb-<project-id>-auth-token`
   └── 保持在記憶體 ──> [React State / Memory]
```

* 存放於 LocalStorage：Supabase SDK 收到response後，會自動將 JSON 格式的 Session（包含 `access_token` 與 `refresh_token`）序列化並存入 `localStorage`。

* 為什麼不用 Cookie？
  - 在 SPA (Single Page Application) 架構中，前端與 Supabase 是透過 API 通訊，`localStorage` 在客戶端讀取與管理最為方便。
  - 注意：只有在 SSR 架構（如 Next.js、Remix）中，為了讓伺服器端渲染時能讀取 Session，才會使用 `@supabase/ssr` 將 Token 轉存進 HttpOnly Cookie 中。

## 3. 與資料庫溝通：JWT 如何驅動 RLS  
當使用者登入後要向 PostgreSQL 查詢資料（如 `supabase.from('repairs').select('*')`）：

```
[React Client] ──(1) API Req + Header: "Authorization: Bearer <access_token>" ──> [PostgreSQL]
                                                                                        │
                                                                                 (2) 解析 JWT
                                                                                 (3) 執行 RLS Policy
                                                                                        │
[React Client] <──(4) 僅返回該使用者有權限看的資料 ──────────────────────────────────────────┘
```

1. 帶入 Header：Supabase Client 會自動將存放在記憶體/LocalStorage 中的 `access_token` 放到 HTTP Header 裡的 `Authorization: Bearer <access_token>`。

2. PostgreSQL 解密與解析：Postgres 的 PostgREST API 收到請求後，利用設定好的 `JWT_SECRET` 解密此 Token。

3. 觸發 RLS (Row Level Security)：  
資料庫會自動將 JWT 內的 `sub` (使用者 UUID) 轉化為內建函數 `auth.uid()`。  
如果資料表設定了這條 RLS 規則：

```sql
-- 範例：使用者只能看自己的維修紀錄
CREATE POLICY "User can view own repairs" 
ON repairs FOR SELECT 
USING (auth.uid() = user_id);
```

Postgres 就會在資料庫底層自動過濾，只回傳符合條件的資料，前端完全無法越權翻看別人的資料。

## 4. 狀態改變時的機制  
Supabase SDK 最強大的地方在於內部維護了一個狀態機，透過 `supabase.auth.onAuthStateChange()` 可以監聽以下所有情境：

### 1. Token 自動刷新 (Auto Refresh)
* 情境：`access_token` 只有 1 小時效期，快過期時怎麼辦？
* 機制：Supabase SDK 在背景設定了 Timer。當檢測到 `access_token` 即將到期（或發起請求發現過期時），SDK 會自動拿 `refresh_token` 向 Supabase Auth 發送請求，更換一組全新的 `access_token` 與 `refresh_token` 並更新 `localStorage`。
* 觸發事件：`TOKEN_REFRESHED`

### 2. 切換分頁 / 開啟多個 Tab
* 情境：使用者在 Tab A 點擊登出，Tab B 會發生什麼事？
* 機制：Supabase SDK 監聽了瀏覽器的 `window.addEventListener('storage')` 事件。
* 變化：當 Tab A 變更了 `localStorage`（例如寫入新 Session 或清空），Tab B 會即時捕捉到變更並同步更新記憶體中的 `Auth` 狀態。這表示我們在 Tab A 登出，Tab B 頁面也會立刻知道並自動觸發頁面跳轉。

### 3. 關閉瀏覽器 / 重新開啟
* 情境：使用者關閉瀏覽器，隔天再打開網站。
* 機制：因為 Session 存在 `localStorage`（除非手動清空，否則不會因關閉瀏覽器而消失）。
* 變化：
  - 當網頁重新載入，Supabase SDK 初始化時會優先讀取 `localStorage`。
  - 如果發現有舊的 Session，會先檢查 `access_token` 是否過期。
  - 如果過期，自動用 `refresh_token` 嘗試刷出一組新的 `access_token`。
  - 刷新成功 ➔ 保持登入狀態（`SIGNED_IN`）。
  - 刷新失敗（例如 `refresh_token` 在後台被撤銷或失效）➔ 清空 `localStorage`，轉為未登入狀態（`SIGNED_OUT`）。

### 4. 登出 (Sign Out)
* 情境：使用者點擊「登出」按鈕，執行 `supabase.auth.signOut()`。
* 機制：
  - 伺服器端：Supabase 發送請求讓該使用者的 `refresh_token` 立即失效（Revoke）。
  - 客戶端：Supabase SDK 清空 `localStorage` 中的 `sb-<project-id>-auth-token` 鍵值，並將記憶體中的 `user` 與 `session` 設為 `null`。
* 觸發事件：`SIGNED_OUT`，React 的 `AuthContext` 收到變更，重新渲染並觸發重定向（導回 `/signin`）。


## 5. 總結對照表

| 操作 / 情境 | LocalStorage 狀態 | Supabase 觸發事件 | 對應的 React 行為 |  
| :--- | :--- | :--- | :--- |  
| **成功登入** | 寫入 `access` & `refresh` token | `SIGNED_IN` | AuthContext 取得 `user`，導向 `/dashboard` |  
| **Token 到期** | 更新為新 Token | `TOKEN_REFRESHED` | 背景自動完成，使用者無感知 |  
| **分頁同步** | 被其他分頁修改時同步更新 | `STORAGE` 變更事件 | 多分頁狀態即時保持一致 |  
| **重開瀏覽器** | 讀取歷史 Token 並自動校驗 | `INITIAL_SESSION` | 若 Token 有效，自動保持登入狀態 |  
| **點擊登出** | 完全抹除 (Removed) | `SIGNED_OUT` | AuthContext 的 `user` 變為 `null`，導向 `/signin` |


## 參考資料
* [Self-Hosting Auth(Supabase doc)](https://supabase.com/docs/reference/self-hosting-auth/introduction)