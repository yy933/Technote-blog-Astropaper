---
title: "JavaScript 的同步與非同步觀念 - 筆記"
pubDatetime: 2026-05-25T11:17:35.986Z
tags: ["JavaScript","asynchronous"]
description: " Table of contents 同步請求（Synchronous Request） 📌 特點： 用戶端 (cli..."
---

## Table of contents


# 同步請求（Synchronous Request）
## 📌 特點：

* 用戶端 (client) 對伺服器端 (server) 送出 request後，程式會停下來等待回應，直到收到伺服器端的 response 之後，才繼續執行後續的程式碼，等待的期間無法處理其他事情。
* 會阻塞（Block）整個程式，在等待回應的期間，使用者無法進行其他操作。
## 📌 範例（使用 XMLHttpRequest 的同步模式）：


```javascript
let request = new XMLHttpRequest();
request.open("GET", "https://randomuser.me/api/", false); // `false` 代表同步請求
request.send();

console.log(request.responseText); // 這行會等 request 完成後才執行
console.log("這行只有在 request 完成後才會跑");
```
## 優缺點
### ✅ 優點：

* 直覺、流程簡單，執行順序符合一般邏輯（先等資料回來再處理）。
### ❌ 缺點：
* 這個作法並不理想，因為通常伺服器端的運算速度比本地電腦慢上好幾倍，會凍結 UI，造成畫面卡住，影響使用者體驗。
* 效率低，因為程式執行被擋住了，沒辦法處理其他事情。

🚨 瀏覽器已不建議使用同步請求，尤其在前端 UI 操作時，因為它會讓整個頁面無回應（卡住）。

# 非同步請求（Asynchronous Request）
## 📌 特點：

* 請求發送後，程式不會等待，而是繼續執行後面的程式碼，可以持續處理其他事情，甚至繼續送出其他 request。
* 當請求完成時，會透過回呼函式（callback）、`Promise` 或 `async/await` 來處理回應。
## 📌 範例（使用 fetch 發送非同步請求）：

```
console.log("發送請求...");

fetch("https://randomuser.me/api/")
  .then(response => response.json())
  .then(data => {
    console.log(data); // 這行會等 fetch 完成後執行
  })
  .catch(error => console.log(error));

console.log("請求發送後，程式繼續執行，不會卡住！");
```
📌 範例（使用 async/await 發送非同步請求）：


```
async function getUser() {
  console.log("發送請求...");
  let response = await fetch("https://randomuser.me/api/");
  let data = await response.json();
  console.log(data);
}

getUser();
console.log("這行會在請求完成前就執行！");
```
## 優缺點
### ✅ 優點：

* 不會阻塞 UI，網頁可以繼續運行，提升使用者體驗。
* 效能更好，因為請求發送後，程式可以繼續執行其他操作，不浪費時間。
### ❌ 缺點：

* 需要 回調函式（callback）、`Promise` 或 `async/await` 來處理非同步邏輯，比同步請求稍微複雜。
* 如果沒有處理好 錯誤處理（如 `.catch()` 或 `try...catch`），可能會導致程式無法正確執行。

# 實現非同步請求 (Asynchronous Request)
Ajax 技術的出現，讓瀏覽器可以向 Server 請求資料而不需費時等待。當瀏覽器接收到 response 之後，新的內容就會即時地添入原本網頁。
早期的Ajax:
```
function reqListener () {
  console.log(this.responseText);
}

var oReq = new XMLHttpRequest();
oReq.addEventListener('load', reqListener);
oReq.open('GET', 'http://www.example.org/example.txt');
oReq.send();
```
在上面的例子裡，我們首先建立了一個 XMLHttpRequest 物件，並使用` .open()` 開啟一個 URL，最後使用` .send()` 發出 request。

因為使用上很麻煩，所以實務上很少直接使用原生的 XMLHttpRequest。

## 替代方案
*  jQuery 的 [$.ajax()](http://api.jquery.com/jquery.ajax/)
*  HTML5 提供的 [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
*  [axios](https://github.com/axios/axios)
--廣泛的瀏覽器支持
--可支援 Node.js 從後端發送的 Http request，這意味著 axios 可以兼用於前端與後端專案。
--直接將回應的 JSON 資料轉換成 JavaScript 的 Object，這十分方便！

## 用 axios 發送 GET Requests
### 1. Promise 方式
```javascript
// Promise 方式
axios.get('www.example.com/api/')
  .then(function (response) {
    // 1.handle success
    console.log(response)
  })
  .catch(function (error) {
    // 2.handle error
    console.log(error)
  })
  .then(function () {
    // 3.always executed
  })
```

1. 第一個 `then()` 函式負責處理成功接收到的 response。其中 `response` 參數就是接收到的回應，而你所需的資訊則放置在 `response.data` 裡面。
1. `catch()` 函式負責處理發生錯誤的狀況，也就是 error。
1. 第二個 `then()` 則是一個選擇性的元件，無論 request 成功與否，它都將被執行。

### 2. 使用 `async/await`
```javascript
async function fetchData() {
  try {
    // 發送請求，等待 API 回應
    let response = await axios.get("https://randomuser.me/api/");
    // 請求成功 ➝ response.data 取得回應內容
    console.log(response.data); // 取得 API 回應的資料
  } catch (error) {
    // 請求失敗 ➝ catch(error) 處理錯誤
    console.error("發生錯誤：", error);
  }
}

fetchData();
```

---

## 參考資料
* ALPHA CAMP Material