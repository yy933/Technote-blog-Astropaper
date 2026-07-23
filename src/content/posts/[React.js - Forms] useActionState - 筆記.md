---
title: "[React.js - Forms] useActionState - 筆記"
pubDatetime: 2026-07-23T08:22:36.197Z
tags: ["React.js","React Hook","React 19","Advanced React","useActionState"]
description: "Table of contents useActionState 定義 在 React 19 之前，處理非同步表單（F..."
hackmd_id: "ryhLoBkSMg"
---

## Table of contents


## useActionState 定義  
在 React 19 之前，處理非同步表單（Form Submission）通常需要手動管理 `loading`、`error` 以及 `data` 等多個狀態。

* **`useActionState` 的使命**：**「自動化管理非同步 Action 的執行結果（State）與載入狀態（Pending State）。」**
* 它專為 **Form Actions** 與 **非同步狀態更新** 設計，讓開發者不用再寫繁瑣的 `try...catch...finally` 樣板程式碼（Boilerplate）。



## 1. 基本語法與回傳值解析

`useActionState` 接受一個非同步處理函式（Action）與初始狀態，並回傳包含目前狀態、綁定用 Action 以及載入狀態的陣列：

```javascript
const [state, formAction, isPending] = useActionState(fn, initialState, permalink?);
```


| 回傳 / 傳入值 | 說明 |  
| :--- | :--- |  
| **`fn`** | 執行的非同步 Action 函式。簽名為 `(previousState, formData) => nextState` |  
| **`initialState`** | 初始 State（如 `null` 或預設值） |  
| **`state`** |  Action 函式最後 `return` 的值（例如：錯誤訊息、伺服器回傳物件） |  
| **`formAction`** | 傳給 `<form action={formAction}>` 或 `startTransition` 用的包裝函式 |  
| **`isPending`** | 布林值（Boolean），代表該 Action 是否正在非同步執行中（用於 Loading UI） |
                                                     
## 為什麼需要 useActionState？
### 傳統做法：手動維護多個 State 與非同步流程  
在傳統 React 中，處理一個送出表單的非同步請求（如寫入 Supabase）需要大量的樣板程式碼：

```javascript
import React from "react"
import supabase from "./supabase-client"

export default function Form() {
  const [isPending, setIsPending] = React.useState(false)
  const [error, setError] = React.useState(null)

  const handleSubmit = async (e) => {
    e.preventDefault()
    setIsPending(true)
    setError(null)

    try {
      const formData = new FormData(e.currentTarget)
      const { error } = await supabase.from("sales_deals").insert([{
        name: formData.get("name"),
        value: Number(formData.get("value"))
      }])
      
      if (error) throw error
    } catch (err) {
      setError(err)
    } finally {
      setIsPending(false)
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" disabled={isPending} />
      <button disabled={isPending}>{isPending ? "Adding..." : "Add Deal"}</button>
      {error && <p>{error.message}</p>}
    </form>
  )
}
```

### 解法：使用 useActionState 精簡程式碼  
React 19 將非同步狀態與表單原生 action 綁定，不僅移除了 `onSubmit` 與 `preventDefault()`，還自動追蹤 `isPending`：

```javascript
import { useActionState } from "react"
import supabase from "./supabase-client"

export default function Form() {
  // ✓ state: Action 回傳的結果 | submitAction: 綁給 form 的處理函式 | isPending: 自動推導的載入狀態
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const newDeal = {
        name: formData.get("name"),
        value: Number(formData.get("value")),
      }

      const { error } = await supabase.from("sales_deals").insert([newDeal])

      if (error) {
        return new Error("新增失敗：" + error.message)
      }

      return null // 寫入成功，重置 error 為 null
    },
    null // 初始 error 狀態
  )

  return (
    <form action={submitAction}>
      <input name="name" disabled={isPending} />
      <button type="submit" disabled={isPending}>
        {isPending ? "Adding..." : "Add Deal"}
      </button>
      {error && <p className="error">{error.message}</p>}
    </form>
  )
}
```

## 3. 注意事項
### 1) Action 函式的第 1 個參數是 previousState  
這是最容易踩坑的地方！`useActionState` 傳入的 `Action` 函式第一個參數是上一次的 `State` 值，第二個參數才是 `formData`（或觸發時傳入的資料）：

```javascript
// ❌ 錯誤：以為第一個參數就是 formData
async (formData) => { ... }

// ✅ 正確：第一個參數是 previousState
async (previousState, formData) => { ... }
```

### 2) 自動整合 Transition 語意  
`useActionState` 產生的 `isPending` 在底層會自動包裹在 `React.startTransition` 之中，這意味著表單提交過程不會阻塞 UI 渲染與其他互動，並能與 `useFormStatus` 或其他 Concurrent React 特性無縫整合。


## 4. 比較useActionState vs useState

| 特性 | `useState` | `useActionState` |  
| :--- | :--- | :--- |  
| **主要情境** | 一般 UI 狀態變更（如 Toggle, Modal） | 非同步 Action / 表單提交 / 伺服器交互 |  
| **Pending 狀態** | 需手動建立另一個 `useState(false)` 管理 | 原生提供 `isPending`，自動切換布林值 |  
| **表單綁定** | 需寫 `onSubmit` 與 `e.preventDefault()` | 直接傳給 `<form action={...}>` |  
| **狀態更新依賴** | 傳入值直接覆蓋 | 依賴 Action 的 `return` 值作為新 State |

## 什麼時候不要使用？  
雖然 `useActionState` 很強大，但並非所有情況都適合：

❌ 這些時候請不要用：
* 純單純的 Client-side 同步輸入：如僅需要即時檢驗文字長度、即時搜尋過濾（用一般 `useState` 即可）。
* 完全沒有非同步 / Server Action 需求的邏輯：如果 `action` 內部完全沒有 Promise/API 呼叫，使用 `useActionState` 只是徒增程式碼複雜度。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>