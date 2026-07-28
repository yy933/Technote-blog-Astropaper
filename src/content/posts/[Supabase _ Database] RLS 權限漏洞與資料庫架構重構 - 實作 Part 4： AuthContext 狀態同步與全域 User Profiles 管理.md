---
title: "[Supabase / Database] RLS 權限漏洞與資料庫架構重構 - 實作 Part 4： AuthContext 狀態同步與全域 User Profiles 管理"
pubDatetime: 2026-07-28T08:11:20.925Z
tags: ["Database","Supabase","PostgreSQL","web security","Database Design","Issue","Project"]
description: "Table of contents 1. 在 AuthContext 中抓取 User Profiles 為什麼需要抓..."
---

## Table of contents

## 1. 在 AuthContext 中抓取 User Profiles
### 為什麼需要抓取所有 `user_profiles` 資料？   
這筆資料會在前端的多個元件（Components）中被頻繁使用，因此適合放在全域的 AuthContext 中統一管理：

* 身分驗證與權限控制：登入時從 Session (`auth.users`) 取得使用者 ID，再比對 `user_profiles` 資料表來判斷該使用者是 Admin 還是 Rep（`account_type` 欄位存放於此）。
* 表單選單 (Form Options)：在新增 Deals 的表單中，用來渲染下拉選單（Dropdown Options），讓 Admin 或 Rep 可以選擇對應的使用者名稱 (`name`)。
* 頁面頂端 Header 顯示：除了顯示登入者的 Email，還需要顯示其名稱與帳號類型（Account Type）。

### 權限前置條件 (RLS Policy Check)  
能在前端一次拉取所有 Profile，前提是 Supabase 已經設定了允許選擇（`SELECT`）的 RLS Policy：

* Policy 名稱：`users can view all profiles`
* 條件：只要是通過驗證的使用者 (`TO authenticated`) 即可執行 `SELECT`。

## React 狀態與實作邏輯 (Implementation)  
在 AuthContext 內新增一個狀態，並在 useEffect 中定義並執行非同步函式 fetchUsers。

### 關鍵實作細節
* 定義 State：使用 `const [users, setUsers] = useState([])` 儲存所有使用者資料。
* 精確欄位選擇 (Select Fields)：不需要 `SELECT *`，只抓取需要的欄位：`id, name, account_type`。
* 錯誤處理 (Error Handling)：使用 `try...catch...throw` 模式處理非同步異常。

### 程式碼範例 (AuthContext.jsx)

```javascript
import { useState, useEffect } from 'react';
import { supabase } from './supabaseClient';

export function AuthProvider({ children }) {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    async function fetchUsers() {
      try {
        // 從 user_profiles 資料表抓取特定欄位
        const { data, error } = await supabase
          .from('user_profiles')
          .select('id, name, account_type');

        if (error) throw error;

        // 印出結果並更新 State
        console.log('Fetched users data:', data);
        setUsers(data);
      } catch (error) {
        console.error('Error fetching user profiles:', error.message);
      }
    }

    // 呼叫函式
    fetchUsers();
  }, []);

  // ...
}
```

## 問題：剛寫完上述程式碼並存檔測試時，console.log 為何可能會顯示空陣列 []？
### 原因解析  
在頁面剛載入或元件 Initial Mount 時，Supabase 的登入狀態（Session）可能尚未完成非同步初始化（`auth.uid()` 還不存在或未包含在 JWT 中）。

因為我們的 RLS 要求必須是 `authenticated`（已認證用戶），當狀態尚未載入完成時，Supabase 就會因為權限拒絕而回傳空的結果集。


## useEffect Dependencies Array與 Auth Session 同步機制
### 1. 問題原因分析 (Root Cause)
**為何一開始在 `useEffect` 裡抓資料會拿到空陣列，登入後也沒有自動更新？**

* 初次渲染（Initial Render）：頁面剛載入時，Supabase 尚未完成登入驗證，此時 `session` 為 `null`（未認證）。
* 被 RLS 擋下：因為 `user_profiles` 有設定 RLS Policy（要求必須是 authenticated），所以在未登入狀態下只能抓到空陣列`[]`。
* Dependencies Array 空陣列 `[]` 的限制：舊的 `useEffect` 設定為 `[]`，代表只會在元件初次 Mount 時執行一次。當使用者點擊登入、`session` 改變引發 Re-render 時，舊的 `useEffect` 不會重新觸發，因此永遠抓不到登入後的 `Profile` 資料。

### 2. 解決方案：Dependencies Array + Guard Clause  
要讓資料隨登入狀態變化而更新，必須建立一個依賴於 `session` 狀態的 `useEffect`，並加上防護條件（Guard Clause）：

* Dependencies Array設為 `[session]`：告知 React 當 `session` 狀態發生改變（如：從 `null` 變成取得 JWT 物件）時，重新執行內部的非同步抓取邏輯 `fetchUsers()`。
* 早期回傳防護 (Guard Clause: `if (!session) return`)：在初次 Mount 或尚未登入時，直接終止函式執行，避免發出無效且一定會被 RLS 擋下的 API 請求。

## 完整實作程式碼 (AuthContext.jsx)  
將邏輯重構並將狀態傳遞至 `AuthContext.Provider`：

```javascript
import { useState, useEffect, createContext } from 'react';
import { supabase } from './supabaseClient';

export const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [session, setSession] = useState(null);
  const [users, setUsers] = useState([]);

  // ... 處理 session 監聽 (onAuthStateChange) 的邏輯 ...

  // 專門處理抓取 user_profiles 的 Effect
  useEffect(() => {
    // 1. Guard Clause：如果還沒登入 (session 為 null)，直接結束不發送請求
    if (!session) return;

    async function fetchUsers() {
      try {
        const { data, error } = await supabase
          .from('user_profiles')
          .select('id, name, account_type');

        if (error) throw error;

        console.log('Fetched users data:', data);
        setUsers(data); // 更新 State
      } catch (error) {
        console.error('Error fetching user profiles:', error.message);
      }
    }

    fetchUsers();
  }, [session]); // 2. 關鍵：將 session 放入Dependencies Array

  return (
    // 3. 將 users 狀態放入 Provider value，讓全域元件皆可取用
    <AuthContext.Provider value={{ session, users }}>
      {children}
    </AuthContext.Provider>
  );
}
```


## 運作流程與測試比對 (Execution Flow)

| 階段 | session 狀態 | useEffect 動作 | 主控台 (Console) 結果 |  
| :--- | :--- | :--- | :--- |  
| **1. 頁面初次載入** | `null` | 觸發，但被 `if (!session) return` 攔截 | 乾淨，不發送無效請求 |  
| **2. 使用者點擊登入** | 更新為 Session Object | `session` 改變 ➔ 觸發 `fetchUsers()` | 順利通過 RLS 審查 |  
| **3. API 回傳資料** | 保持不變 | 執行 `setUsers(data)` 存入全域 Context | 印出 `Fetched users: [{id: ..., name: 'Pam', ...}]` |

## 總結
- 只要非同步 API 請求需要依賴使用者認證權限 (RLS)，就應該將 `session`（或 `Auth` 狀態）放入 `useEffect` 的Dependencies Array中。
- 搭配 `if (!session) return` 可以避免在未登入時發出多餘的無效請求，提升前端效能與 Log 乾淨度。