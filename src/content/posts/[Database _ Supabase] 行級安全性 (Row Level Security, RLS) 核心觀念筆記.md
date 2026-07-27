---
title: "[Database / Supabase] 行級安全性 (Row Level Security, RLS) 核心觀念筆記"
pubDatetime: 2026-07-27T03:04:17.423Z
tags: ["Database","Supabase","Backend","Authentication","token","JWT","Concepts","web security"]
description: "Table of contents 1. 為什麼需要 RLS？ 在前端雖然可以透過路由鎖住 /dashboard 頁面..."
---

## Table of contents

## 1. 為什麼需要 RLS？

在前端雖然可以透過路由鎖住 `/dashboard` 頁面，但使用者仍可能**直接在網址列輸入 URL** 存取敏感頁面。

* **問題**：當使用者未登入（未認證）時，直接輸入 URL，`fetchMetrics` 函式依然會執行並成功抓取圖表資料（如 `sales_deals` 資料表）。
* **原因分析**：未登入時，請求攜帶的是匿名金鑰（`anon` key，具備 `anon` 角色）；登入後攜帶的才是具備 `authenticated` 角色的 JWT。
* **安全漏洞**：如果資料庫未開啟 RLS，即便只是 `anon` 角色的匿名請求，也能直接穿越資料庫大門存取資料。



## 2. 白話比喻：城堡與守衛 (The Castle Metaphor)

為了更好地理解 RLS，可以將整個驗證與授權機制比喻為一個 **「城堡」**：

* 🔑 **JWT (JSON Web Token)**：寄給城堡的「信件」中所附上的**簽名證書**。
  * `anon` key ➔ 臨時**訪客證**（Visitor's pass）
  * `authenticated` role ➔ 正式**員工證**（Full employee badge）
* 💂 **RLS 政策 (Policy)**：駐守在資料庫每個「房間／資料表（Table）」門口的**守衛／保鑣（Bouncers）**。
  * 門衛會檢查信件寄件者的證書（JWT）與信件內容（Request）。
  * 根據特定指令決定是否放行（例如：「只允許持有員工證的人查看自己的資料」）。



## 3. RLS 的核心特性

```text
[ 請求到達資料庫 ] ──> [ RLS 門衛攔截 ] ──> 預設全擋 (Deny All)
                                               │
                                       檢查 Policy 指令
                                               │
                         ┌─────────────────────┴─────────────────────┐
                         ▼                                           ▼
                 符合條件 (e.g. auth.role = 'authenticated')    不符合條件
                         │                                           │
                         ▼                                           ▼
                     🟢 放行存取                                 ❌ 拒絕存取
```

### 資料庫層級的安全性 (Database-level security)

安全機制直接在 PostgreSQL 資料庫內部強制執行，無法在應用程式層（Application layer）被旁路規避。

### 自動化執行 (Automated enforcement)

每次存取資料表時自動觸發，開發者不需要在前端/後端程式碼中手動寫重複的安全檢查。

### 預設拒絕存取 (Default to Deny)

一旦開啟 RLS，預設為全面封鎖（Lockdown）。如果沒有設定明確的 Allow Policy，任何請求都無法通過。

### 細粒度控制 (Individual Row Control)

可精細控制「特定使用者只能看見/修改資料庫中的特定幾筆資料（Rows）」。

### 情境感知與靈活性 (Context-aware & Flexible)

可讀取當前請求者的 Context（如 `User ID`）動態做決策。

### 操作權限區分 (Operation-specific)

可針對不同的 CRUD 操作（`SELECT` / `INSERT` / `UPDATE` / `DELETE`）撰寫獨立的政策。

## 4. Supabase 輔助函式 (Helper Functions)  
在撰寫 RLS Policy 時，Supabase 提供了以下原生 SQL 輔助函式來解析 JWT：

| 輔助函式 | 說明 | 實用範例 |  
| :--- | :--- | :--- |  
| `auth.uid()` | 取得當前請求使用者的 UUID | `auth.uid() = user_id`（僅能存取自己的資料） |  
| `auth.jwt()` | 取得完整的 JWT JSON 物件 | 用於解析自訂的 Metadata 或 Claims |  
| `auth.role()` | 取得當前角色（anon 或 authenticated） | `auth.role() = 'authenticated'`（僅限登入會員） |  
| `auth.email()` | 取得當前使用者的 Email | 用於特定公務 Email 的權限判斷 |

>💡 組合技：這些輔助函式可以結合 SQL 查詢條件（如檢查寫入的資料內容是否符合規定），提供非常強大且靈活的防護網。


### Supabase RLS設定範例

以前面提到的`sales_deals` 資料表為例，開啟RLS（進入 Supabase Dashboard 點擊 Authentication ➔ Policies ➔ Enable RLS，並點擊 Create policy）：

```sql
-- 建立僅限已驗證使用者存取的 Policy
CREATE policy "Authenticated users only"
ON "public"."sales_deals"
as PERMISSIVE
FOR ALL  -- 這裡也可視需要修改成 for SELECT，使用者只能讀取無法修改資料表
TO public
using (
  (auth.role() = 'authenticated')
);
```

#### 細節解析：

* `FOR ALL`：代表同時套用至 `SELECT`, `INSERT`, `UPDATE`, `DELETE` 操作（若僅用於 Dashboard 讀取，亦可改為 `FOR SELECT`）。
* `TO public`：Supabase會對所有進來的請求進行檢查。也可改成`TO authenticated`直接指定此 Policy 僅套用至具備 JWT 已驗證角色的請求，直接在引擎層級過濾掉 `anon`（匿名）請求，效能更好。

如此一來，只有通過驗證、攜帶 `authenticated` 角色 JWT 的使用者，才能對 `sales_deals` 資料表進行資料操作，徹底防止未登入者透過 URL 或 API 直接撈取資料！

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>