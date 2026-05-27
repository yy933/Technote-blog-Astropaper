---
title: "Server-Sent Events - 筆記"
pubDatetime: 2026-05-25T01:57:22.000Z
tags: ["Node.js","Backend","Express.js"]
description: "Table of contents :memo: 什麼是ServerSent Events ServerSent Ev..."
hackmd_id: "Hy6D7cblzg"
---

## Table of contents

## :memo: 什麼是Server-Sent Events
Server-Sent Events（簡稱 SSE） 是一種**讓伺服器（Server）可以主動、即時地將資料推送給瀏覽器（Client）的技術**。

在傳統的網頁運作模式中，都是「瀏覽器主動問，伺服器才回答」（呼叫 API、點擊連結）。**但有些場景需要即時更新（例如：實時股價、運動賽事比分、ChatGPT 一字一字蹦出來的文字），如果瀏覽器一直每秒鐘問一次，伺服器很快就會崩潰。**

SSE 就是為了解決這個問題而誕生的，**它讓伺服器和瀏覽器之間建立一條單向的永久通道，伺服器只要有新資料，就可以隨時「源源不絕地灌水」給瀏覽器**。


## :memo: SSE 的運作原理
我們可以把 SSE 想像成「訂閱電子報」或「看 YouTube 直播」：

* **瀏覽器發起訂閱**：瀏覽器向伺服器發送一個特殊的 HTTP 請求，說：「我想訂閱這個事件串流（Event Stream）。」
* **伺服器保持連線**：伺服器不急著關閉連線，而是回應一個特定的 Header`（Content-Type: text/event-stream）`，把通道維持打開狀態。
* **伺服器主動推送**：每當伺服器有新消息，就會順著這條通道把資料送過去，瀏覽器收到後立刻更新畫面。


## :memo: 實作分享 (前端+後端Express)
### 1. 後端（Express）
伺服器不需要關閉請求（不呼叫 `res.send` 或 `res.json`），而是用 `res.write` 持續寫入資料。
```javascript
import express from 'express';
const app = express();

app.get('/api/stream', (req, res) => {
    // 1. 關鍵：設定 Header，告訴瀏覽器「這是一個持續的事件串流」
    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');

    // 2. 每隔 2 秒鐘，伺服器主動推送一次時間給前端
    const intervalId = setInterval(() => {
        const time = new Date().toLocaleTimeString();
        
        // SSE 的標準格式必須以 "data: " 開頭，並以 "\n\n" 結尾
        res.write(`data: 目前伺服器時間是 ${time}\n\n`);
    }, 2000);

    // 3. 如果使用者關閉網頁，清空計時器，釋放記憶體
    req.on('close', () => {
        clearInterval(intervalId);
    });
});

app.listen(3000, () => console.log('SSE 伺服器已啟動：http://localhost:3000'));
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

### Tips:
```javascript
req.on('close', () => {
        clearInterval(intervalId);
        res.end();
    });
```
這一段就是Event Emitter的應用，這裡的 `req`（HTTP Request 請求物件）在 Node.js 底層，其實就是一個繼承了 EventEmitter 的實例。

#### 幕後運作流程：
1. 使用者關閉網頁：當使用者在瀏覽器按下了「分頁的 X」或是關閉瀏覽器。
1. 瀏覽器發出斷線訊號：瀏覽器會向你的伺服器發送一個 TCP 的斷線封包（FIN 包）。
1. Node.js 底層接招：伺服器的實體網路卡收到封包，交給作業系統，最後傳進 Node.js 的核心。
1. 底層自動呼叫 `.emit()`：Node.js 發現這個特定的 HTTP 連線被切斷了，它就會在內部找到對應的 `req` 物件，並在底層默默執行：
```
// 這行程式碼寫在 Node.js 的源碼（C++/JS）底層
req.emit('close');
```
5. 觸發監聽器：因為底層執行了 `.emit('close')`，寫在 Express 裡的 `.on('close', ...) `就會立刻被通知，開始執行清空計時器（`clearInterval`）的工作。

</blockquote>

### 2. 前端（瀏覽器 JavaScript）
前端非常簡單，瀏覽器內建了一個叫 `EventSource` 的物件，專門用來處理 SSE。
```javascript
// 連接到後端的串流網址
const eventSource = new EventSource('/api/stream');

// 監聽來自伺服器的訊息
eventSource.onmessage = function(event) {
    console.log("收到伺服器推送：", event.data);
    // 這裡可以寫 document.getElementById(...).innerText = event.data 來更新畫面
};

// 如果發生錯誤（例如斷線），EventSource 會自動嘗試重新連線
eventSource.onerror = function(error) {
    console.error("SSE 連線發生錯誤", error);
};
```

### 補充: SSE可以自訂事件名稱
SSE 允許伺服器自訂事件名稱，而它在前端的監聽方式，跟 `addEventListener` 結合。

#### 後端:
```javascript
// 加上 event: 自訂名稱
res.write(`event: score_update\n`);
res.write(`data: {"teamA": 2, "teamB": 1}\n\n`);
```

#### 前端:
```javascript
// SSE 的 EventSource 也能用 addEventListener！
eventSource.addEventListener('score_update', (event) => {
    console.log("比分更新囉：", event.data);
});
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

#### SSE 與 WebSocket 有什麼不同？
很多人會把 SSE 和 WebSocket 搞混，因為它們都能做到「即時推送」，但它們的設計邏輯完全不同：
| 特性 | Server-Sent Events (SSE) | WebSocket |
| :--- | :--- | :--- |
| **傳輸方向** | **單向** (只有伺服器傳給瀏覽器) | **雙向** (瀏覽器和伺服器可互相對傳) |
| **通訊協定** | 標準的 **HTTP** (好相容、不容易被防火牆擋) | 獨立的 **ws:// 協定** (需要伺服器額外支援) |
| **自動重連** | **內建** (瀏覽器斷線會自己嘗試重新連線) | **無** (工程師必須自己寫程式碼處理重連) |
| **資料格式** | 只能傳送 **純文字** (通常把 JSON 轉成字串) | 可以傳送 **文字與二進位資料** (如圖片、檔案) |
| **常見場景** | ChatGPT 輸出、股價即時看板、通知中心 | 即時多人聊天室、線上多人遊戲、共用白板 |

#### 補充
SSE 規範：每一行 `data:`  後面只能放單行文字。如果直接把帶有換行的 JSON 塞進去，瀏覽器會解析失敗。
```
// 傳送 JSON 的正確方式：轉成單行字串
const payload = JSON.stringify({ status: "success", msg: "hello" });
res.write(`data: ${payload}\n\n`);
```

</blockquote>

## SSE跟 Event Emitter 有關係嗎？
在後端專案中，SSE經常搭配Event Emitter一起使用。

可以用 Event Emitter 在伺服器內部傳遞消息，再用 SSE 把消息發送給瀏覽器。
例如：

* 使用者在 Express 買了東西，觸發了` orderEmitter.emit('newOrder')`。
* Express 收到這個事件後，立刻透過 SSE 管道 `res.write()` 推送給後台的管理員。
* 管理員的網頁就「叮咚」一聲跳出通知，完全不需要重新整理網頁！




---



<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>