---
title: "Cross-Origin Resource Sharing (CORS) - 筆記"
pubDatetime: 2026-06-05T09:27:43.866Z
tags: ["Node.js","Express.js","Backend"]
description: "Table of contents :memo: 什麼是 CORS？ CORS 是 跨來源資源共用（CrossOrig..."
hackmd_id: "B1WnfflbMl"
---

## Table of contents

## :memo: 什麼是 CORS？
CORS 是 **跨來源資源共用（Cross-Origin Resource Sharing） 的縮寫**。

簡單來說，它是一套瀏覽器的安全機制。當你的前端網頁（例如：`http://localhost:3000`）想要發送 API 請求給另一個不同網址的後端伺服器（例如：`http://localhost:8000`）時，瀏覽器為了保護使用者，預設會阻止這種跨網域的行為。

**必須透過後端伺服器在回應中加上特定的 HTTP 標頭（Headers），告訴瀏覽器「我允許這個前端網站存取我的資料」，瀏覽器才會安全地放行。**


## :memo: 核心觀念：什麼叫「跨來源 (Cross-Origin)」？
瀏覽器在判斷兩個網址是不是「同一個來源」時，會嚴格檢查以下三個元素（簡稱 SPA 檢查）：

1. 通訊協定 (Protocol)：如 `http` 與 `https`
1. 網域 (Domain)：如 `example.com` 與 `localhost`
1. 連接埠 (Port)：如 `:3000` 與 `:8000`

只要這三個當中有任何一個不一樣，就是「跨來源」！ 
### 來源判斷對照表：
假設目前前端網頁所在的網址為：`http://localhost:3000`

| 要請求的 API 網址 | 是否跨來源？ | 原因 |
| :--- | :--- | :--- |
| `http://localhost:3000/api/data` | **同來源 (Same-Origin)** | 協定、網域、Port 完全相同。 |
| `https://localhost:3000/api/data` | ❌ **跨來源** | 協定不同（`http` vs `https`） |
| `http://127.0.0.1:3000/api/data` | ❌ **跨來源** | 網域不同（`localhost` vs `127.0.0.1`） |
| `http://localhost:8000/api/data` | ❌ **跨來源** | **Port 不同（`:3000` vs `:8000`）** *(前後端分離開發最常見的情況)* |


## :memo: 為什麼需要 CORS？（同源政策）

瀏覽器之所以管得這麼寬，是因為一個非常重要的網路安全防禦機制：**同源政策（Same-Origin Policy）**。

**想像一個情境：**
你登入了某家網路銀行（`bank.com`），瀏覽器本地端記住了你的登入憑證 Cookie。
接著你不小心點進一個惡意網站（`evil.com`），該網站偷偷寫了一段 JavaScript 程式碼，自動去呼叫 `bank.com/api/transfer`（轉帳 API）。

如果沒有「同源政策」，瀏覽器就會乖乖幫惡意網站發送請求（還附帶你的網銀 Cookie），你的錢可能就會在不知情的情況下被轉走！

因此，瀏覽器預設不允許 `evil.com` 讀取 `bank.com` 的回應資料。**但是在現代前後端分離的開發模式下，前端接 API 往往是安全且合法的需求，這時候我們就需要透過 CORS 機制在後端幫合法的前端「白名單」。**

## :memo: 實作分享 (後端 Express 解決 CORS)
在 Node.js / Express 專案中，最標準且優雅的作法是使用官方提供的 `cors` 中介軟體（Middleware），而不需要自己手動去處理複雜的 `res.setHeader`。

### 1. 安裝套件
在專案根目錄的終端機輸入：

```bash
npm install cors
```

### 在 `server.js` 中引入並掛載
打開進入點 `server.js`，在掛載路由之前引入並使用它：

```javascript
import express from 'express';
import cors from 'cors'; // 1. 引入 cors 套件
import { apiRouter } from './routes/apiRoutes.js';

const PORT = 8000;
const app = express();

// 2. 全域掛載 CORS 中介軟體
// 預設允許「所有來源（*）」跨域請求，本地開發（Development）時最方便
app.use(cors()); 

app.use(express.json());

// 路由模組化
app.use('/api', apiRouter);

app.listen(PORT, () => console.log(`server connected on port ${PORT}`));
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

Tips: 生產環境（Production）的安全設定
在專案準備部署上線時，不建議使用 `cors()` 預設的 `*`，這會讓任何外部網站都能輕易撈取你的 API 資料。

我們應該將其限制為只有我們的前端網域可以存取：
```javascript
// 正式環境建議寫法：
const corsOptions = {
    origin: 'https://your-frontend-website.com', // 只允許這個網域
    methods: 'GET,POST,PUT,DELETE',              // 限制允許的 HTTP 方法
    optionsSuccessStatus: 200 
};

app.use(cors(corsOptions));
```


</blockquote>

## :memo: 補充觀念：簡單請求 vs 預檢請求
當瀏覽器在處理 CORS 時，會根據請求的複雜度分為兩種行為：

1. **簡單請求 (Simple Requests)**：
**滿足特定條件**（例如：使用 `GET`、`POST`、`HEAD` 方法，且 Headers 只有基本欄位如 `Content-Type: text/plain` 或 `application/x-www-form-urlencoded`）。**瀏覽器會直接發送真正的請求，並檢查回應中的 `Access-Control-Allow-Origin`。**

1. **預檢請求 (Preflight Requests)**：
**當你使用 `PUT`、`DELETE`，或者 `Content-Type` 設定為 `application/json`（當代前端發送 API 最常見的格式）時，瀏覽器會覺得這個請求比較危險。**

在發送真正的請求之前，瀏覽器會自動先發送一個 HTTP 方法為 `OPTIONS` 的預檢請求，去詢問後端伺服器：「請問我可以發送這個帶有 JSON 的請求嗎？」得到後端點頭允許（回傳 200 OK 與對應 Header）後，瀏覽器才會真正把資料送出去。

💡 這也就是為什麼有時候在瀏覽器的 DevTools Network 中，會看到同一個 API 觸發了兩次連線（一次為 OPTIONS，一次為真正的 Method）。
使用 Express 的 `cors` 套件會自動幫我們完美處理好 `OPTIONS` 的回應！


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>