---
title: "[React.js, Supabase] React 19 + Supabase Auth 登入完整流程 - 筆記"
pubDatetime: 2026-07-25T05:22:37.855Z
tags: ["React.js","Supabase","JavaScript","Frontend","Authentication","JWT","cheatsheet","token","Backend","web security","React19"]
description: "Table of contents 架構說明：React 19 Form Action + Supabase Auth..."
hackmd_id: "B19Og6-Bfx"
---

## Table of contents

## 架構說明：React 19 Form Action + Supabase Auth  
本架構使用 React 19 的 **`useActionState`** 處理表單驗證與 Action 狀態，搭配 Supabase 的 **`signInWithPassword`** 與全域 **`onAuthStateChange`** 事件監聽，打造驗證流程。



## 1. 登入流程架構圖 (Architecture Flow)

```text
[階段 1：表單觸發 Signin.jsx]
  1. 使用者輸入 Email / Password 並點擊 Submit
  2. <form action={submitAction}> 觸發 useActionState
  3. isPending 自動設為 true（按鈕變更為 "Signing in" 並禁用）
          │
          ▼
[階段 2：發送請求 AuthContextProvider.jsx]
  4. 執行 Context 提供的 signInUser(email, password)
  5. 呼叫 supabase.auth.signInWithPassword(...)
  6. Supabase 後端比對 Hash 密碼：
     ├── ❌ 驗證失敗 ➔ 回傳 error 訊息
     └── 🟢 驗證成功 ➔ 回傳 Session (含 JWT) 並自動寫入 LocalStorage
          │
          ▼
[階段 3：全域狀態觸發 AuthContextProvider.jsx]
  7. 登入成功主動觸發 supabase.auth.onAuthStateChange 監聽器
  8. 執行 setSession(session) 與 setLoading(false)
  9. AuthContext 全域 Session 狀態即時更新
          │
          ▼
[階段 4：UI 與路由響應 Signin.jsx]
  10. 若失敗：Action return new Error(...)，頁面渲染 alert 錯誤訊息
  11. 若成功：isPending 恢復 false，受保護路由（Protected Route）自動導頁至 /dashboard
```

## 2. 四階段程式碼拆解

### 階段1：`useActionState` 處理表單與載入狀態 (`Signin.jsx`)  
React 19 提供了 `useActionState` 來簡化非同步表單的處理：

```javascript
// components/Signin.jsx
const [error, submitAction, isPending] = useActionState(
  async (prevState, formData) => {
    // 1. 從原生 FormData 獲取輸入欄位資料
    const email = formData.get("email");
    const password = formData.get("password");

    // 2. 呼叫 AuthContext 的登入函式 (signInUser在AuthContextProvider中定義，階段2會提到)
    const { success, error: signInError } = await signInUser(email, password);

    // 3. 處理錯誤：回傳 Error 物件給 useActionState 的 error 狀態
    if (signInError) return new Error(signInError);

    // 4. 成功後可回傳 null，全域 Session 更新會觸發 Route 導頁
    return null;
  },
  null // 初始 error 狀態
);
```
* `isPending`：當非同步 Action 正在執行時自動為 `true`，可用於原生 UI 的禁用處理 `(disabled={isPending})` 與 Accessibility 標示 `(aria-busy={isPending})`。
* `error`：當 Action 函式回傳 `Error` 物件時，`error` 變數會捕捉該物件，方便渲染錯誤訊息。

### 階段 2：與 Supabase Auth 通訊 (`AuthContextProvider.jsx`)  
`signInUser` 負責封裝 Supabase 的認證邏輯：

```javascript
// context/AuthContextProvider.jsx
const signInUser = async (email, password) => {
  try {
    // 1. 轉小寫防呆並發送驗證請求
    const { data, error } = await supabase.auth.signInWithPassword({
      email: email.toLowerCase(),
      password,
    });

    // 2. 顯式處理 Supabase 回傳的 API 錯誤
    if (error) {
      return { success: false, error: error.message };
    }
    // 如果成功則回傳data
    return { success: true, data };
  } catch (error) {
    // 3. 捕捉網路中斷等未預期例外
    return {
      success: false,
      error: "An unexpected error occurred. Please try again.",
    };
  }
};
```
* Supabase 會在雲端比對 Salted Hash 密碼。
* 驗證通過後，Supabase SDK 會自動將回傳的 JWT (access_token) 儲存至瀏覽器的 LocalStorage。

### 階段 3：全域狀態監聽器發揮作用 (`AuthContextProvider.jsx`)  
Supabase SDK 具備事件驅動（Event-driven）特性，不需要在登入成功後手動 `setSession`：

```javascript
// context/AuthContextProvider.jsx
useEffect(() => {
  let isMounted = true;

  // 1. 網頁開啟/重新整理時，檢查初始 Session
  async function getInitialSession() {
    try {
      const { data, error } = await supabase.auth.getSession();
      if (error) throw error;
      if (isMounted) setSession(data.session);
    } catch (error) {
      if (isMounted) setSession(null);
    } finally {
      if (isMounted) setLoading(false);
    }
  }
  getInitialSession();

  // 2. 註冊全域認證狀態監聽器 (登入/登出時自動觸發)
  const { data: { subscription } } = supabase.auth.onAuthStateChange(
    (_event, session) => {
      setSession(session);
      setLoading(false);
    }
  );

  return () => {
    isMounted = false;
    // 元件卸載時取消訂閱，防止記憶體洩漏
    subscription?.unsubscribe();
  };
}, []);
```

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

可以將`useContext`封裝成一個custom hook `useAuth`，讓`AuthContextProvider`專心處理auth相關邏輯：

```javascript
// hooks/useAuth.js
import { createContext, useContext } from "react";

export const AuthContext = createContext(null);

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth has to be used within AuthContextProvider");
  }
  return context;
};
```

在`Signin.jsx`中可直接使用`useAuth`：

```javascript
// components/Signin.jsx
import { useAuth } from "../Hooks/useAuth";
const Signin = () => {
  const { signInUser } = useAuth();
      {/*Signin Logic*/}
    },
    null,
  );
  return (
    <>
        {/*Signin UI*/}
    </>
  );
};

export default Signin;
```


</blockquote>

### 階段 4：UI 錯誤處理與提示設計  
在前端 `Form` 裡面，透過 `useActionState` 傳回的 `error` 物件進行無障礙（A11y）渲染：

```javascript
<form action={submitAction}>
  <input
    type="email"
    name="email"
    disabled={isPending}
    aria-invalid={error ? "true" : "false"}
    aria-describedby={error ? "signin-error" : undefined}
  />
  
  <button type="submit" disabled={isPending} aria-busy={isPending}>
    {isPending ? "Signing in..." : "Sign In"}
  </button>

  {/* 顯示錯誤訊息 */}
  {error && (
    <div id="signin-error" role="alert" className="sign-form-error-message">
      {error.message}
    </div>
  )}
</form>
```

## 3. 其他注意事項


* 錯誤處理的回傳格式一致性  
在 `AuthContextProvider` 中回傳的是 `{ success: false, error: error.message }`（`error` 欄位已是 String），在 `Signin.jsx` 中應直接寫 `return new Error(signInError)`，而非 `new Error(signInError.message)`，避免抓到 `undefined`。

* `email.toLowerCase()`：Supabase 預設 Email 區分大小寫，登入與註冊前先轉小寫可大幅降低使用者輸入錯誤率。

* 自動連動 Routing：登入成功後，`onAuthStateChange` 會更新Context 的 `session`，建議配合 react-router-dom 的 `<Navigate />` 或保護路由，讓頁面在 `session` 有值時自動重導向至首頁/Dashboard。


## 參考資料
* [Supabase signInWithPassword (官方文件)](https://supabase.com/docs/reference/javascript/auth-signinwithpassword)
* [Supabase onAuthStateChange (官方文件)](https://supabase.com/docs/reference/swift/auth-onauthstatechange)


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>