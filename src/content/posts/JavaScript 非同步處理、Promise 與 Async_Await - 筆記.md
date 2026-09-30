---
title: "JavaScript 非同步處理、Promise 與 Async/Await - 筆記"
pubDatetime: 2026-09-30T11:29:52.638Z
tags: ["Interview Preparation","JavaScript","Asynchronous","Promise","Async-Await","Frontend","asynchronous","API"]
description: "Tags: Interview Preparation JavaScript Asynchronous Promise..."
---

###### Tags: `Interview Preparation` `JavaScript` `Asynchronous` `Promise` `Async-Await` `Frontend` `API`


## Table of contents


## 核心觀念總覽

* **同步 (Synchronous) vs. 非同步 (Asynchronous)**：
  * **同步**：程式碼依序由上至下執行，前一個任務未完成前會**阻擋（Block）** 後續程式碼。
  * **非同步**：發起耗時任務（如 API 網路請求）後不會阻擋主線程，而是在背景處理，待結果回傳後再透過事件佇列（Task Queue / Microtask Queue）執行回調（Callback）。
* **Promise 的三大狀態（Three States of Promise）** （⚠️ 重要觀念！）：
  1. **Pending（進行中 / 待定）**：初始狀態，非同步操作尚未完成，也尚未失敗。
  2. **Fulfilled / Resolved（已實現 / 成功）**：非同步操作順利完成，並回傳結果值。
  3. **Rejected（已拒絕 / 失敗）**：非同步操作失敗，並拋出錯誤原因（Error Reason）。
  * ⚠️ **不可逆性**：Promise 狀態一旦從 `Pending` 轉變為 `Fulfilled` 或 `Rejected`，狀態便被**鎖定（Settled）**，無法再次改變。
* **語法演進：Callback ➔ Promise ➔ Async/Await**：
  * 早期使用 Callback 容易陷入**回調地獄（Callback Hell）**。
  * ES6 導入 **Promise**，透過 `.then()` 鏈結（Chaining）改善代碼結構。
  * ES2017 (ES8) 導入 **`async/await`**，本質上是 Promise 的語法糖（Syntactic Sugar），讓非同步程式碼讀起來就像同步程式碼一樣直覺。



## 1. 串接 API 兩種寫法對比與速查

| 比較維度 | Promise `.then()` / `.catch()` 寫法 | ES8 `async / await` 寫法 |  
| :--- | :--- | :--- |  
| **程式碼可讀性** | 透過鏈結（Chaining）處理，層級多時較冗長 | 結構平鋪直敘，極具可讀性（像同步程式碼） |  
| **錯誤處理機制** | 使用末端的 **`.catch(error => ...)`** | 必須包裹在 **`try ... catch`** 區塊中 |  
| **變數作用域** | 變數常被限制在各個 `.then()` 的回調函式作用域中 | 變數存在於同一層級的函式作用域，取用極為方便 |  
| **執行流程控管** | 預設為背景非同步執行（順序為 `1 ➔ 3 ➔ 2`） | 搭配 `await` 會暫停該 `async` 函式執行，等待 Promise 完成 |



## 2. 範例程式碼對比 (Code Examples)

### 範例情境：串接 `JSONPlaceholder` API 取得文章資料

#### 寫法 A：傳統 Promise 鏈結寫法 (`.then` & `.catch`)

```javascript
function getPostPromise() {
  console.log('1. 開始請求 (Start)');

  // Fetch 回傳一個 Promise
  fetch('[https://jsonplaceholder.typicode.com/posts/1](https://jsonplaceholder.typicode.com/posts/1)')
    .then((response) => {
      // fetch 的第一階段只回傳 Response Header，解析 JSON 也是非同步操作 (回傳 Promise)
      return response.json();
    })
    .then((data) => {
      console.log('2. 成功取得資料 (Async Data):', data.title);
    })
    .catch((error) => {
      // 捕捉網路失敗或 JSON 解析失敗等錯誤
      console.error('捕捉到錯誤 (Error):', error);
    });

  console.log('3. 請求發出完畢 (End)');
}

getPostPromise();
// 執行順序輸出：
// 1. 開始請求 (Start)
// 3. 請求發出完畢 (End)
// 2. 成功取得資料 (Async Data): sunt aut facere... (微任務最後執行)
```

## 寫法 B：現代 Async / Await 寫法 (try ... catch)

```javascript
// 在函式前加上 async 關鍵字，使其回傳一個 Promise
async function getPostAsync() {
  console.log('1. 開始請求 (Start)');

  try {
    // await 會暫停 async 函式內部的執行，直到 Promise 被 resolve
    const response = await fetch('[https://jsonplaceholder.typicode.com/posts/1](https://jsonplaceholder.typicode.com/posts/1)');
    const data = await response.json();
    
    console.log('2. 成功取得資料 (Async Data):', data.title);
  } catch (error) {
    // 必須使用 try...catch 來捕獲 await 拋出的錯誤
    console.error('捕捉到錯誤 (Error):', error);
  }

  console.log('3. 請求發出完畢 (End)');
}

getPostAsync();
// 執行順序輸出：
// 1. 開始請求 (Start)
// 2. 成功取得資料 (Async Data): sunt aut facere... (await 暫停了該函式後續執行)
// 3. 請求發出完畢 (End)
```

## 回答策略 
### 「請說明你對 Promise 與 Async/Await 的理解，以及在前端如何處理非同步請求與錯誤？」

推薦答題架構：
- 定義與狀態（What & States）：  
非同步處理是前端串接 RESTful API 的核心。JavaScript 透過 Promise 物件來管理非同步操作，它有三種狀態：Pending（進行中）、Fulfilled（成功）與 Rejected（失敗），且狀態改變後不可逆。

- 說明寫法演進（Promise vs. Async/Await）：  
早期的 `.then()` 鏈結寫法解決了 Callback Hell，但當流程複雜時變數作用域較難傳遞。因此現代開發我更偏好使用 `async`/`await`，它是 Promise 的語法糖，能讓我們以看似『同步』的語法寫出非同步邏輯，大幅提升程式碼可讀性與維護性。

- 強調錯誤處理與防禦性編程（Error Handling）：  
在實務中，處理非同步請求時絕對不能忽視錯誤處理。
  - 使用 Promise 鏈結時必須搭配 .catch()。
  - 使用 async/await 時，必須將邏輯包裹在 try ... catch 區塊中。
    
另外要留意的是，原生的 `fetch` 在遇到 `HTTP 404` 或 `500` 錯誤時並不會主動觸發 `reject`（除非網路斷線），因此在實務上我們需要主動檢查 `response.ok` 屬性，或統一透過 Axios 等 HTTP 套件進行 Interceptor（攔截器）與統一錯誤處理。

## FAQ
### Q1: 原生 fetch() 在遇到 HTTP 404 或 500 錯誤時，會觸發 catch 嗎？  
不會！   
`fetch()` 只有在網路中斷（Network Error）或請求被阻擋（Blocked）導致 Request 無法順利送出時，才會將 Promise 狀態變更為 Rejected。

如果伺服器有回傳回應，即使 status code 是 `404 Not Found` 或 `500 Internal Server Error`，`fetch()` 的 Promise 依然會判定為 Resolved。實務上必須檢查 `response.ok`（當 status 為 `200`-`299` 時為 `true`）：

```javascript
const response = await fetch('/api/data');
if (!response.ok) {
  throw new Error(`HTTP 錯誤！狀態碼：${response.status}`); // 手動拋出錯誤以進入 catch 區塊
}
```

### Q2: 如果有多個獨立的 API 請求要同時發起（非同步並行），應該怎麼寫才不會造成效能瓶頸？  
不應該使用多個連續的 `await`（這會變成串行，增加等待總時間），而是應該使用 `Promise.all()` 或 `Promise.allSettled()` 進行並行發送：

```javascript
async function fetchJson(url) {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`HTTP Error ${res.status} on ${url}`);
  return res.json();
}

const results = await Promise.allSettled([
  fetchJson('/api/user'),
  fetchJson('/api/posts')
]);

// 分別檢查各個 Promise 的結果
const user = results[0].status === 'fulfilled' ? results[0].value : null;
const posts = results[1].status === 'fulfilled' ? results[1].value : [];

if (results[0].status === 'rejected') {
  console.warn('User API 載入失敗：', results[0].reason);
}

if (results[1].status === 'rejected') {
  console.warn('Posts API 載入失敗：', results[1].reason);
}
```