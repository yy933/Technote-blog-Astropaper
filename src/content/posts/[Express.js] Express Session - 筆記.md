---
title: "[Express.js] Express Session - 筆記"
pubDatetime: 2026-06-10T04:26:09.937Z
modDatetime: 2026-06-10T04:29:21.573Z
tags: ["Node.js","web security","Express.js","Backend","HTTP","cookie","session","authentication"]
description: "Table of contents :memo: 什麼是 Express Session？ 在 Web 開發中，exp..."
hackmd_id: "r1hT7P8-fg"
---

## Table of contents

## :memo: 什麼是 Express Session？  
在 Web 開發中，express-session 是 Express 官方維護的一個非常核心的中介軟體（Middleware）。它主要的功能是：**在「無狀態」的 HTTP 協定之上，為每個使用者建立一個「有狀態」的連線機制，用來跨頁面記錄使用者的狀態和資料。**

簡單來說，它就是伺服器端幫使用者準備的「專屬暫存置物櫃」，用來識別目前發送請求的到底是哪一個使用者。

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

關於使用者認證流程，可參考這篇[Cookie & Session - 筆記](https://hackmd.io/q_BI0kxESk2fztSxRmhwIA)

</blockquote>


## :memo: 為什麼需要 Session？（解決 HTTP 無狀態問題）
**HTTP 協定預設是沒有記憶的（Stateless）。**  
當使用者登入 A 頁面後，跳轉到 B 頁面時，伺服器根本不記得他是剛剛登入過的人。

* 如果不用 Session：使用者每換一個頁面、點一個按鈕，前端都必須重新傳送一次帳號密碼給後端驗證，這顯然不切實際。
* **使用 Express Session：當使用者第一次拜訪時，後端會自動產生一個獨一無二的 Session ID，並把這個 ID 透過 Cookie 塞進使用者的瀏覽器。之後瀏覽器每次發送請求，都會自動帶上這個 ID，伺服器就能藉此認出該使用者。**

## :memo: Express Session 的三大核心功能
### 1. 身分驗證與登入狀態保持 (Authentication)  
這是最常見的用途。當使用者輸入正確的帳密登入成功後，你可以把使用者的資訊（例如：使用者 ID、權限）存進 Session 中：

```javascript
// 登入成功時，把資料寫入 session 置物櫃
req.session.userId = user.id;
req.session.username = user.username;
req.session.role = 'admin';
```

在其他任何路由（例如：後台管理頁面 `/admin`），你只需要檢查這個置物櫃有沒有東西，就能判斷他有沒有登入：

```javascript
app.get('/admin', (req, res) => {
    // 檢查 session 置物櫃裡有沒有 userId
    if (req.session.userId) {
        res.send(`歡迎回來，管理員 ${req.session.username}`);
    } else {
        res.status(401).send('請先登入！');
    }
});
```

### 2. 跨頁面狀態共享（例如：購物車）  
在電商網站中，使用者在商品 A 頁面點擊「加入購物車」，接著跳轉到商品 B 頁面。  
利用 Session，後端可以輕鬆跨頁面記錄使用者的購物車內容，直到他最後結帳，不需要每次都回傳整張購物車清單。

```javascript
// 點擊加入購物車，將商品塞入該使用者的 session
req.session.cart = req.session.cart || [];
req.session.cart.push({ productId: 'p123', quantity: 1 });
```

### 3. 快閃訊息 (Flash Messages)
**常用於表單提交後的提示。** 例如使用者修改密碼成功後，網頁會重導向（Redirect）回首頁，首頁需要顯示一個「修改成功！」的提示泡泡。這種「只顯示一次、重整後就消失」的暫存訊息，也非常適合存放在 Session 中。

## :memo: 運作原理機制  
Express Session 的運作主要依賴 後端的 Session 紀錄 與 前端的 Cookie 互相配合：

1. 瀏覽器發送請求：使用者輸入帳密發送登入請求。  
1. 伺服器建立 Session：後端驗證成功，`express-session`在伺服器端開闢一塊空間存放該使用者的資料，並生成一個加密的 `connect.sid`（Session ID）。  
1. 核發 Cookie：伺服器在 HTTP 回應中，透過 `Set-Cookie: connect.sid=xxx` 把這個 ID 送回瀏覽器。  
1. 自動帶上 Cookie：接下來使用者拜訪該網站的任何一個頁面，瀏覽器都會在 HTTP Header 自動帶上 `Cookie: connect.sid=xxx`。  
1. 後端辨識認領：`express-session` 自動攔截這個 ID，去伺服器後台比對，把對應的資料還原到 `req.session` 物件中，供後續程式碼使用。

## :memo: 實作分享 (基本設定)  
在 Express 專案中引入並掛載 express-session 的基礎基本範例：

```javascript
import express from 'express';
import session from 'express-session';

const app = express();

app.use(session({
  secret: 'keyboard cat',     // 用來加密 Session ID Cookie 的字串（建議用環境變數隱藏）
  resave: false,               // 是否每次請求都重新儲存 Session（設為 false 可提升效能）
  saveUninitialized: false,    // 是否強制將未初始化的 Session 儲存（設為 false 節省空間）
  cookie: { 
    maxAge: 60000 * 30,        // 設定 Cookie 有效時間（微秒），此處為 30 分鐘
    secure: false              // 如果是 https 環境，必須設為 true
  }
}));
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

⚠️ 重要提醒：記憶體流失（Memory Leak）的潛在風險  
`express-session` 預設是把所有使用者的資料存在 Node.js 的記憶體（MemoryStore） 中。

* 開發環境（Development）：完全沒問題，速度極快。
* 生產環境（Production）：千萬不要用預設設定！ 當網站有大量使用者同時在線，Node.js 的記憶體很快就會被撐爆（Memory Leak），導致伺服器當機。而且只要伺服器一重啟，所有使用者的登入狀態都會瞬间消失。

💡 專業解決方案：搭配 Session Store  
在正式上線時，我們會搭配「持久化儲存套件」，將 Session 資料改存到外部資料庫。**最推薦 Redis（記憶體型資料庫，速度極快），或者是 MongoDB、MySQL 等。**

正式環境搭配 Redis 儲存範例：

```javascript
import RedisStore from 'connect-redis';
import { createClient } from 'redis';

const redisClient = createClient();
redisClient.connect().catch(console.error);

app.use(session({
  store: new RedisStore({ client: redisClient }), // 改存到 Redis，安全又不佔 Node.js 記憶體
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: { secure: true } // 線上環境開啟 https 安全傳輸
}));
```


</blockquote>

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>