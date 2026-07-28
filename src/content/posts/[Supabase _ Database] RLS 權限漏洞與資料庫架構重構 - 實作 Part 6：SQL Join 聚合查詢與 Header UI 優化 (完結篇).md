---
title: "[Supabase / Database] RLS 權限漏洞與資料庫架構重構 - 實作 Part 6：SQL Join 聚合查詢與 Header UI 優化 (完結篇)"
pubDatetime: 2026-07-28T11:13:21.966Z
tags: ["Database","Supabase","PostgreSQL","web security","Database Design","Issue","Project"]
description: "Table of contents 刪除舊欄位與 ERD 關聯釐清 在完成 user_id 外鍵遷移後，舊的 sale..."
---

## Table of contents

## 刪除舊欄位與 ERD 關聯釐清  
在完成 `user_id` 外鍵遷移後，舊的 `sales_deals.name` 欄位已不再使用，需至 Supabase Table Editor 中將其刪除。

- 一對多關聯（1 : Many）：一位使用者（`user_profiles`）可以擁有許多筆不同的交易（`sales_deals`）。
- 外鍵約束 (Foreign Key Constraint)：`sales_deals.user_id` 指向 `user_profiles.id`，這確保了資料完整性 (Data Integrity)——系統無法新增一個指指向不存在使用者的 Deal。

## 跨表聚合查詢 (SQL Join + GROUP BY)  
要繪製統計圖表，我們需要從 `sales_deals` 計算金額總和（`SUM(value)`），並從 `user_profiles` 跨表取得業務員姓名（`name`）。

### 1. 原生 SQL 寫法

```sql
SELECT 
  SUM(sales_deals.value),
  user_profiles.name
FROM sales_deals
INNER JOIN user_profiles 
  ON sales_deals.user_id = user_profiles.id
GROUP BY user_profiles.name;
```

### 2. 轉譯為 Supabase JS Client 寫法  
PostgREST 能夠根據資料庫預先設定好的外鍵 (FK) 自動推導 Join 關聯，因此無需在 JS 中手動指定 `ON` 條條件：(使用官方提供的Translator一鍵轉譯:[SQL to REST API Translator](https://supabase.com/docs/guides/api/sql-to-rest))

```javascript
// fetchMetrics.js
import supabase from "../supabase/supabase-client";

export async function fetchMetrics() {
  const { data, error } = await supabase.from("sales_deals").select(
    `
    value.sum(),
    ...user_profiles!inner(
      name
    )
    `
  );

  if (error) {
    console.error("Error fetching metrics:", error);
    throw error;
  }

  // 將 Supabase 回傳格式轉為圖表專用格式 [{ name: "Dwight", sum_value: 3000 }]
  return data.map((item) => ({
    name: item.name || "Unknown",
    sum_value: item.sum,
  }));
}
```

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

💡 語法細節解析：
* `value.sum()`：對主表 `sales_deals` 的`value` 欄位做總和計算。
* `...user_profiles`：使用展平語法（Spread），將子表欄位直接拉到最外層。
* `!inner`：指定為 Inner Join（若使用者 Profile 已被刪除，則不顯示該筆 Deal）。

</blockquote>


## Header UI 優化與異步保護處理  
將原本冷冰冰的 Email 改為顯示 使用者姓名（Name） 與 角色權限（Account Type）。

### 1. 異步防護與安全性
* `session?.user?.id`：剛開頁面時 `session` 可能為 `null`，加上 Optional Chaining ?. 可防止 Runtime Error 導致頁面空白。
* `users.find()` 呼叫安全性：在 `AuthContext` 初始化時，`users` 預設值為空陣列 `[]`。在空陣列上執行 `.find()` 是安全的，會回傳 `undefined` 而不會崩潰。

### 2. 角色對映表與動態渲染 (Header.jsx)

```javascript
import { useAuth } from "../Hooks/useAuth";

function Header() {
  const { session, users, signOut } = useAuth();

  // 1. 找出當前登入者的完整 Profile 資料
  const currentUser = users.find((u) => u.id === session?.user?.id);

  // 2. 建立角色顯示對映字典 (Dictionary Map)
  const accountTypeMap = {
    rep: "Sales Rep",
    admin: "Admin",
  };

  // 3. 安全計算要顯示的角色名稱
  const displayAccountType = currentUser?.account_type
    ? accountTypeMap[currentUser.account_type]
    : "";

  return (
    <header className="header-container">
      <h1>Sales Dashboard</h1>
      
      {session && (
        <div className="user-info">
          {/* 4. 顯示：Name (Account Type) */}
          <span className="user-name">
            {currentUser?.name || "Loading..."}
            {displayAccountType && ` (${displayAccountType})`}
          </span>
          <button onClick={signOut}>Sign Out</button>
        </div>
      )}
    </header>
  );
}

export default Header;
```

## 專案重構總結 (Refactoring Architecture Summary)  
透過這一系列的重構，達成了以下優化目標：

```text
[使用者驗證 (Auth)] 
       │
       ▼
[AuthContext (全域 State)] ───► 取得 `session` & `user_profiles` 資料
       │
       ├──► [Header] ───► 顯示使用者姓名與權限標籤 (e.g. Dwight (Sales Rep))
       │
       ├──► [Form] ─────► 權限隔離 (Rep 鎖定姓名 / Admin 可選取不同 Rep)
       │
       └─► [Supabase RLS] ──► 最終安全防線 (後端阻擋越權寫入與讀取)
```

* 資料結構擴展性 (Scalability)：將使用者 Profile 與 Auth 分離，未來新增頭像、部門等欄位無需重構 Auth 機制。
* 安全性 (Security)：結合 前端 UX 限制 與 後端 RLS 政策，達成嚴密的雙層安全防禦。
* 效能與開發體驗 (DX)：善用 PostgREST 的 FK 自動推理機制，大幅簡化複雜的 SQL Join 寫法。