---
title: "建立 Node.js 伺服器 - 筆記"
pubDatetime: 2025-04-01T01:15:04.000Z
tags: ["Node.js","cheatsheet"]
description: "Table of contents 建立 Node.js 本機伺服器 1. 載入 Node.js HTTP 模組 1...."
hackmd_id: "SkMyc7Fp1l"
---

## Table of contents


## 建立 Node.js 本機伺服器
1. 載入 Node.js HTTP 模組
1. 定義和伺服器有關的變數
1. 設定回應的 HTTP 狀態碼 (status code)
1. 設定回應的內容類型 (Content-Type)
1. 把回應的訊息傳送給瀏覽器
1. 啟動並監聽伺服器

### 1. 載入 Node.js HTTP 模組

```javascript
// index.js
// Include http module from Node.js
const http = require('http')
```


### 2. 定義和伺服器有關的變數

```javascript
// Define server related variables
const hostname = 'localhost'
const port = 3000
```
使用 HTTP 模組中的` createServer` 方法 (method) 來建立伺服器：
```javascript
// Handle request and response here
const server = http.createServer((req, res) => {
  // Do something to handle request and response here
})
```

### 3. 設定回應的 HTTP 狀態碼 (status code)

```javascript
// Handle request and response here
const server = http.createServer((req, res) => {
  res.statusCode = 200
})
```

### 4. 設定回應的內容類型 (Content-Type)
除了 HTTP 狀態碼之外，在回應時還要告訴瀏覽器回應的「內容類型 (Content-Type)」為何。

一般來說，因為是用瀏覽器看網頁，所以內容類型大多是**HTML 文件 (text/html)** 。但有些時候伺服器回傳的可能是**純文字 (text/plain)** 、**PDF 檔 (application/pdf)** 、**影片檔 (video/mpeg4)** 、或者是經常用來傳送資料的 **JSON 檔 (application/json)** 。

```javascript
// Handle request and response here
const server = http.createServer((req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain')
})
```

### 5. 把回應傳送給瀏覽器
透過 `res.end()`可以把伺服器想要給瀏覽器的回應寫在這裡。

```javascript
// Handle request and response here
const server = http.createServer((req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain')
  res.end('This is my first server created in Node.js')
})
```

### 6. 啟動並監聽伺服器
啟動伺服器的方法，就是請它去 **「監聽」(listen) 傳來的請求**。

在 Node.js 中， 我們可以使用 `server.listen()` 這個方法來啟動伺服器：

```javascript
// Start and listen the server
server.listen(port, hostname, () => {
  console.log(`The server is listening on http://${hostname}:${port}`)
})
```

## 執行程式並啟動伺服器 (完整程式碼)
```javascript
// index.js
// Include http module from Node.js
const http = require('http')

// Define server related variables
const hostname = 'localhost'
const port = 3000

// Handle request and response here
const server = http.createServer((req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/plain')
  res.end('This is my first server created in Node.js')
})

// Start and listen the server
server.listen(port, hostname, () => {
  console.log(`The server is listening on http://${hostname}:${port}`)
})
```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

補充一下，如果在 `res.end()` 裡的字串有換行的話，就要以樣板字面值（template literals）來處理，不能用一般的單引號，而是要用「反引號」(back-tick)，當字串中想要**帶入變數或 HTML 時也是一樣**。例如，伺服器回傳HTML檔案：
```javascript
// Handle request and response here
const server = http.createServer((req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/html')
  res.end(`<h1>This is my first server created in Node.js</h1>`)
})
```

</blockquote>

## 執行 JavaScript 檔案

移動到檔案路徑:

```bash
$ cd [Your path]
```

切換到該資料夾下後，使用 `node` 指令去執行 `index.js` 這支檔案：
```bash
[Your Path] $ node index.js
```
順利的話，就會看到「The server is listening on http://localhost:3000」，這時候就可以打開瀏覽器，輸入 localhost:3000 ，就可以看到剛剛在 Node.js 的伺服器中透過 `res.end()` 所設定的回應文字。
如果想要中斷伺服器，可以回到終端機上，按下「Ctrl + C」即可。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

注意：如果在這裡發現程式碼有問題，需要修改，每次在修改完檔案後，都必須要

1）存檔
2）在終端機透過「Ctrl + C」先把伺服器停掉
3）再透過 node 這個指令，重新啟動

</blockquote>