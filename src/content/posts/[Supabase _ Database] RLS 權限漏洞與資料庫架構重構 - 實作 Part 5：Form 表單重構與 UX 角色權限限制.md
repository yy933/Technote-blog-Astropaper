---
title: "[Supabase / Database] RLS 權限漏洞與資料庫架構重構 - 實作 Part 5：Form 表單重構與 UX 角色權限限制"
pubDatetime: 2026-07-28T10:59:07.388Z
tags: ["Database","Supabase","PostgreSQL","web security","Database Design","Issue","Project"]
description: "Table of contents 資料庫 PK/FK 架構釐清 (Primary Key vs. Foreign K..."
---

## Table of contents

## 資料庫 PK/FK 架構釐清 (Primary Key vs. Foreign Key)  
在重構表單之前，必須清楚掌握 `sales_deals` 資料表關鍵欄位之間的對應關係：

* `id` (Primary Key)：建立資料表時 Supabase 自動產生的流水號主鍵，代表「這一筆 Deal 自身的唯一識別碼」。
* `user_id` (Foreign Key)：指向 `user_profiles.id`（進一步對應 `auth.users.id`），代表「擁有/建立這筆 Deal 的使用者 ID」。

### 核心轉變  
舊版架構中表單直接將使用者姓名（`name`）寫入資料表；新版架構則改為寫入 `user_id` (UUID)，以維護資料關聯性與安全性。

## Form 表單邏輯重構 (Server Action & Form Submission)  
在 `Form.jsx` 中，當使用者送出表單時，我們透過表單選擇的 `name` 找出對應使用者的 `id`，再寫入 Supabase。

### 實作 (`Form.jsx`)

```javascript
import { useActionState } from "react";
import supabase from "../supabase/supabase-client";
import { useAuth } from "../Hooks/useAuth";

function Form() {
  const { users, session } = useAuth();

  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      // 1. 取得表單選擇的姓名
      const submittedName = formData.get("name");

      // 2. 從全域 users 陣列中比對找出該使用者的 Profile 物件
      const user = users.find((u) => u.name === submittedName);

      if (!user) {
        return new Error("User not found");
      }

      // 3. 組成符合新資料庫架構的 Deal 物件 (改傳 user_id)
      const newDeal = {
        user_id: user.id,
        value: Number(formData.get("value")),
      };

      // 4. 新增至 sales_deals 資料表
      const { error } = await supabase.from("sales_deals").insert([newDeal]);

      if (error) {
        console.error("Error adding deal:", error.message);
        return new Error("Failed to add deal");
      }

      return null;
    },
    null
  );

  // ...
}
```

## UX 階層的角色權限限制 (UX Limitation for Reps)  
雖然已經有 Supabase RLS 防禦底層資料，但理想的系統應該在 UX 介面層 也做好相應的權限引導：

- 一般專員 (Rep)：
  - 只能幫自己新增 Deal。
  - UI：將姓名欄位改為 **唯讀文字框**（`<input readOnly />`），直接顯示登入者的名字，無法切換。

- 管理員 (Admin)：
  - 可以幫任何 Rep 新增 Deal。
  - UI：保持 下拉選單（`<select>`），但 **自動過濾掉 Admin 帳號，只呈現所有 Rep 供選擇**。

### 完整實作程式碼 (Form.jsx)

```javascript
import { useActionState } from "react";
import supabase from "../supabase/supabase-client";
import { useAuth } from "../Hooks/useAuth";

function Form() {
  const { users, session } = useAuth();

  // 1. 找出當前登入的使用者 (搭配 Optional Chaining 防止初始載入報錯)
  const currentUser = users.find((u) => u.id === session?.user?.id);

  // 2. 產生選單選項：過濾掉 Admin，只保留 Rep 帳號
  const generateOptions = () => {
    return users
      .filter((user) => user.account_type === "rep")
      .map((user) => (
        <option key={user.id} value={user.name}>
          {user.name}
        </option>
      ));
  };

  return (
    <div className="add-form-container">
      <form action={submitAction} aria-label="Add new sales deal">
        
        {/* 3. 根據角色進行條件渲染 */}
        {currentUser?.account_type === "rep" ? (
          /* Rep 分支：唯讀 Input 鎖定為登入者姓名 */
          <label htmlFor="deal-name">
            Name:
            <input
              type="text"
              id="deal-name"
              name="name"
              value={currentUser?.name || ""}
              readOnly
              className="rep-name-input"
              aria-label="Sales representative name"
              aria-readonly="true"
            />
          </label>
        ) : (
          /* Admin 分支：下拉選單選取指定的 Rep */
          <label htmlFor="deal-name">
            Name:
            <select
              id="deal-name"
              name="name"
              defaultValue={users.find((u) => u.account_type === "rep")?.name || ""}
              aria-required="true"
              disabled={isPending}
            >
              {generateOptions()}
            </select>
          </label>
        )}

        {/* 金額輸入框與送出按鈕 */}
        <label htmlFor="deal-value">
          Amount: $
          <input
            id="deal-value"
            type="number"
            name="value"
            defaultValue={0}
            min="0"
            step="10"
            disabled={isPending}
          />
        </label>

        <button type="submit" disabled={isPending}>
          {isPending ? "Adding..." : "Add Deal"}
        </button>
      </form>
    </div>
  );
}

export default Form;
```

## 廢棄屬性與 Code Cleanup  
由於已將使用者與驗證資料移至全域 `AuthContext`統一管理：

* 移除 `Form` 的冗餘 Props：`Form` 元件不再需要接收舊有的 `metrics` 或 `data` Props。
* 清理 `Dashboard.jsx`：將原本傳給 `Form` 的 `<Form metrics="{metrics}"/>` 簡化為 `<Form/>`。


## RLS 與 UX 雙層防禦測試與驗證

| 驗證情境 | 登入角色 | UI 介面表現 | 後端 RLS 行為 |  
| :--- | :--- | :--- | :--- |  
| **情境 A** | **Rep** (例如 Dwight) | 姓名欄位為 `<input readOnly>`，顯示 Dwight，無法選其他人。 | 僅可新增 `user_id` 為自己的 Deal。 |  
| **情境 B** | **Admin** (例如 Pam) | 姓名欄位為 `<select>` 下拉選單，包含所有 Rep 帳號（不含 Admin）。 | 可任意選取並幫指定 Rep 新增 Deal。 |  
| **突破邊界測試** | **Rep 繞過前端 UI** | （若故意竄改發包封包嘗試幫他人新增） | 遭到 Supabase RLS 政策封鎖，拋出 `Failed to add deal` 錯誤。 |

## 總結心法
- 多層防護（Defense in Depth）：RLS 負責「安全性與資料防禦底線」，UX 條件渲染負責「操作流暢度與使用者引導」，兩者缺一不可。

- 受控與唯讀（Controlled & ReadOnly）：對於不需要使用者修改但需要隨表單送出的數值，使用 `<input readOnly value={...} />` 能確保值順利寫入 FormData，同時避免手動干預。