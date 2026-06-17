---
title: "[Express.js] Express Middleware - 筆記"
pubDatetime: 2026-06-09T11:00:50.463Z
tags: ["Node.js","Express.js","Backend"]
description: "Table of contents :memo: 什麼是 Middleware？ Middleware（中介軟體） 是..."
hackmd_id: "ByaP1dSbMg"
---

## Table of contents

## :memo: 什麼是 Middleware？  
Middleware（中介軟體） 是 Express 的核心靈魂。

簡單來說，它就像是從「請求發出（Request）」到「回應回傳（Response）」之間的一道道關卡或加工站。當一個請求進來時，它會像流水線上的產品一樣，依序經過你設定的各個中介軟體。

每個中介軟體在本質上都是一個 函式 (Function)，它可以選擇：

* 執行任何程式碼（如：記錄 Log、計算時間）。
* 修改請求（`req`）與回應（`res`）物件的內容。
* 攔截並終結這個請求（如：權限不足直接回傳錯誤，不讓它過關）。
* 呼叫 `next()`，把請求傳遞給下一個加工站。

## :memo: Middleware 的三大要素  
一個標準的 Express 中介軟體函式會接收三個參數：

1. `req` (Request)：請求物件，包含了前端傳來的資料（Headers, Query, Body 等）。  
1. `res` (Response)：回應物件，用來回傳資料給前端。  
1. `nex`t：這是一個回呼函式（Callback function），也是中介軟體的靈魂。**如果不呼叫 `next()`，請求就會像卡在傳送帶上一樣，前端會一直處於「轉圈圈」等不到回應的狀態。**

## :memo: 實際範例  
以下面這段簡單的 Express 程式碼為例：

```javascript
import express from 'express'

const app = express() 

// 🎯 這行就是掛載一個內建的 Middleware！
app.use(express.static('public'))

// 🎯 這是最後的路由處理器（Route Handler），本質上也是一種中介軟體
app.get('/', (req, res) => {
    res.send('<!doctype html><html><body>Hello Express!</body></html>')
})

app.listen(8000, () => console.log('listening 8000'))
```

### 幕後運作流程  
當使用者在瀏覽器輸入網址並發送請求時，Express 會由上而下依序檢查經過的傳送帶：

1. 第一站：`express.static('public')`
    - 這個中介軟體會先去專案中的 `public` 資料夾尋找是否有對應的靜態檔案。
    - 情境 A（找到了）：如果網址是 `/logo.png` 且資料夾內確實有這張圖，它會直接回傳圖片（終結週期），請求到此結束，不會再往下走。
   - 情境 B（沒找到）：如果網址是 `/`，它在 `public` 找不到對應檔案，就會在底層默默呼叫 `next()`，把請求放行到下一站。

1. 第二站：`app.get('/', ...)`  
請求來到這裡，Express 比對路徑符合 `/`，於是執行裡面的邏輯，呼叫 `res.send()` 回傳 HTML 碼（終結週期）。

## :memo: 什麼時候使用 Middleware？（常見場景）  
中介軟體依據來源主要分為三種：內建（Built-in）、第三方（Third-party） 以及 自定義（Custom）。

### 1. 解析前端資料（內建）  
當前端發送 JSON 資料過來時，Node.js 預設只看得懂字串。我們需要加工站幫忙解析：

```javascript
// 解析成功後，才能在後續的路由中使用 req.body 取得物件
app.use(express.json()) 
```

### 2. 處理跨域問題與安全性（第三方）  
像是我們常用的 cors 套件，本質上就是一個第三方中介軟體，它會在請求進來時，自動在 Header 補上允許跨域的標籤：


```javascript
import cors from 'cors'
app.use(cors()) // 幫所有請求自動加工 Header
```

### 3. 權限驗證與保全（自定義）  
假設網站有些秘密頁面（如後台 /dashboard）必須登入才能看，可以自己寫一個「保全中介軟體」：


```javascript
// 自定義一個檢查登入狀態的保全
const checkLogin = (req, res, next) => {
    const isLoggedIn = req.headers.authorization; // 模擬檢查憑證
    
    if (isLoggedIn) {
        next(); // 🔑 通過檢查！放行去下一站
    } else {
        res.status(401).send('錯誤：您尚未登入，拒絕存取！'); // ❌ 攔截！在此終結請求
    }
}

// 只有進入 /dashboard 之前，需要經過 checkLogin 這道關卡
app.get('/dashboard', checkLogin, (req, res) => {
    res.send('歡迎來到最高機密後台中心');
});
```

## 總結
* Express 專案就像一間工廠，`app.use()` 就是在組裝流水線上的加工設備。
* 順序非常重要！ 寫在上面的中介軟體會先被執行。
* 每個中介軟體要嘛用 `res.send()` 等方法終結請求，要嘛呼叫 `next()` 交棒給下一位，二選一。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>