---
title: "Promise 封裝 - 筆記"
pubDatetime: 2025-04-28T00:50:46.000Z
modDatetime: 2026-05-25T10:04:23.687Z
tags: ["JavaScript","Interview Preparation","asynchronous"]
description: "Table of contents 什麼是 Promise Promise 是一個物件建構子 (constructor..."
hackmd_id: "SyQHrn31ee"
---

## Table of contents


## 什麼是 Promise  
`Promise` 是一個物件建構子 (constructor)，使用時需要先從 `Promise` 物件產生物件實例 (instance)，再使用繼承特性的 instance 去包裝程式碼的 callback 流程。`Promise` 使得處理非同步操作變得更加直觀，避免了callback地獄（callback hell）的問題。

![ExportedContentImage_02 (4)](/images/S1brUhhkll.png)


## Promise基本用法
```javascript
// 建立一個新的 Promise 物件
const myPromise = new Promise((resolve, reject) => {
  const success = true;  // 假設這是某個非同步操作的結果
  
  if (success) {
    resolve("Operation was successful!");  // 非同步操作成功時調用 resolve
  } else {
    reject("Operation failed.");  // 非同步操作失敗時調用 reject
  }
});

// 使用 .then() 處理成功的結果，.catch() 處理錯誤
myPromise
  .then((result) => {
    console.log(result);  // "Operation was successful!"
  })
  .catch((error) => {
    console.log(error);  // "Operation failed."
  });

```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

### 解析：
* `new Promise()` 創建了一個 `Promise` 物件，並接受一個函式，該函式有兩個參數：
  - `resolve(value)`：當操作成功時，調用此方法來傳遞結果。
  - `reject(error)`：當操作失敗時，調用此方法來傳遞錯誤訊息。
* `then()`：當 `Promise` 被 `resolve`，即操作成功時，執行 `then()` 中的callback。
* `catch()`：當 `Promise` 被 `reject`，即操作失敗時，執行 `catch()` 中的callback。

</blockquote>


`Promise` 是用於進行流程控制的物件 (容器)，它具備了 callback 的優點，但透過 `.then()` 來標明流程，而 `.then() `之間可以互相鏈結 (chaining)，把之前「一層包一層的 callback」，轉換成 `.then()` 的串接。

### 範例1  
![ExportedContentImage_03 (2)](/images/ByMOP331ge.png)

### 範例2
```javascript
const myPromise = new Promise((resolve, reject) => {
  resolve("First step completed");
});

myPromise
  .then((result) => {
    console.log(result);  // "First step completed"
    return "Second step completed";
  })
  .then((result) => {
    console.log(result);  // "Second step completed"
  })
  .catch((error) => {
    console.log("Error:", error);
  });
```

## 練習：用Promise封裝程式碼

```javascript
let http = require('http')
let https = require('https')

let requestData = () => {
  // return promise instance here
}

http.createServer((req, res) => {
  let imgPath = requestData()
  // add this to somewhere
  res.end(`<h1>DOG PAGE</h1><img src='${imgPath}' >`)
}).listen(3000)

console.log('Server start: http://127.0.0.1:3000')
```
### **Promise封裝後：**  
修改 `requestData `函式，讓它返回一個 `Promise` 物件，並在資料請求完成後解析該資料。  
在 `http.createServer` 裡，我們也需要處理 `Promise` 的結果，並確保圖片路徑 `imgPath `在資料獲取後再返回給客戶端。

```javascript
let http = require('http')
let https = require('https')

let requestData = () => {
  return new Promise((resolve, reject) => {
    // 模擬從外部API獲取資料
    https.get('https://dog.ceo/api/breeds/image/random', (res) => {
      let data = ''

      // 監聽資料
      res.on('data', (chunk) => {
        data += chunk
      })

      // 監聽響應結束
      res.on('end', () => {
        try {
          let imgData = JSON.parse(data)
          resolve(imgData.message)  // resolve 當成功解析
        } catch (error) {
          reject('Error parsing data')  // reject 當解析錯誤
        }
      })
    }).on('error', (err) => {
      reject('Request failed: ' + err.message)  // request錯誤時reject
    })
  })
}

http.createServer((req, res) => {
  requestData()
    .then((imgPath) => {
      res.end(`<h1>DOG PAGE</h1><img src='${imgPath}' >`)
    })
    .catch((err) => {
      res.end(`<h1>Error: ${err}</h1>`)
    })
}).listen(3000)

console.log('Server start: http://127.0.0.1:3000')
```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

改寫重點：
* **`requestData` 函式包裝成 `Promise`：**
**將 `requestData` 包裝成一個回傳 `Promise` 的函式**，並在 `https.get` 的回調函式中處理數據。當資料獲取並成功解析後，使用 `resolve(imgData.message)` 返回圖片路徑；若發生錯誤則使用 `reject` 返回錯誤訊息。
 
* 使用 `.then()` 和 `.catch()` 處理 `Promise`：  
在 `http.createServer` 中，當接收到請求時，使用 `requestData().then().catch()` 來處理非同步操作的結果，**確保只有在資料請求成功後才會回應圖片路徑。如果請求過程中有錯誤，則會進入 `catch`，並顯示錯誤訊息**。

</blockquote>
 #### 簡單流程說明：  
1. `requestData()` 被呼叫時，它會回傳一個 `Promise`。  
1. 當 `https.get()` 成功獲取並解析 JSON 資料後，`Promise` 被` resolve`，並將 `imgData.message`（圖片 URL）傳遞給 `then()` 方法。  
1. 如果請求失敗或解析失敗，`Promise` 會被 `reject`，並將錯誤傳遞給 `catch()` 方法。

#### 更簡單說明:  
`requestData() `函式會發出一個 HTTP 請求➡️從一個公開的 API 獲取隨機圖片 URL
  - 請求成功：API回傳資料➡️解析資料，將圖片的 URL 傳遞給 `Promise` 的 `resolve` 方法`resolve(imgData.message)`。
  - 請求失敗或 JSON 解析錯誤：進入 `catch()`➡️錯誤回傳。

**✨`requestData()` 並不直接回傳解析過的 JSON 物件，而是回傳一個 `Promise`**，並且這個 `Promise` 會在資料請求完成、並且 JSON 解析成功後，解析成包含圖片 URL 的值。

## `Promise.all`  
關於`Promise.all`，參考[這篇筆記](https://hackmd.io/9l_LMhZcQC66OtC3S8HYtg)。