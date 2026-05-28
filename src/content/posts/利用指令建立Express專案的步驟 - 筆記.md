---
title: "利用指令建立Express專案的步驟 - 筆記"
pubDatetime: 2025-04-06T22:57:14.000Z
modDatetime: 2026-05-25T10:04:23.171Z
tags: ["Express.js","Node.js","cheatsheet"]
description: "Table of contents 1. 新增專案資料夾 mkdir project cd project 2. 設定..."
hackmd_id: "HyOQDaC9xe"
---

## Table of contents


 

### 1. 新增專案資料夾
* `mkdir project`
* `cd project`
### 2. 設定 package.json
* `npm init -y`
* 設定程式入口為 app.js
 `touch app.js`
* 用`code .` 打開VS code
  設定 package.json 檔案: 把預設的main屬性改成`"main": "app.js"`
### 3. 安裝 Express
`npm install express`

### 4. 設定主程式 app.js
* 建構應用程式伺服器
```
// Include Express from modules
const express = require('express')

// app server
const app = express()
```
* 設定埠號 port 
```
// declare server related variables
const port = 3000
```
* 設定首頁路由
```
// routing: home page
app.get('/', (req, res)=>{
  res.send('hello world')
})
```
* 監聽伺服器並用callback印出一段 console 訊息，確認伺服器是否有成功啟動。
```
// listen the server and launch
app.listen(port, ()=>{
  console.log(`App is runnung on http://localhost:${port}`)
})
```
* 利用`node app.js`測試伺服器是否成功啟動
 ![](https://i.imgur.com/OV9arDa.jpg)
用瀏覽器打開網址 `localhost:3000/`，會看到在路由裡設定的內容：
![](https://i.imgur.com/bS3miVY.jpg)

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

直接用Node.js架設伺服器，參考這篇：[建立Node.js伺服器](https://hackmd.io/XAQmAmFwQEKtgctCEczopQ)

</blockquote>


### 5. 設定常用腳本
* 安裝nodemon: `  sudo npm install nodemon -g     // -g 會安裝在全域`
  `sudo` 是用管理員級的權限來安裝， `-g`會安裝在全域，所以安裝過一次之後的專案就不用重新安裝。
  安裝完後先用ctrl+C關閉伺服器，再用`nodemon app.js`重啟
<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

[nodemon ](https://www.npmjs.com/package/nodemon)這個工具最主要是去偵測開發者專案內的檔案，一旦在專案資料夾內的檔案有被修改變更，那麼 nodemon 就會自動重啟伺服器。於是重新整理瀏覽器後，就可以看到修改後的畫面，省去了開發者每次改完程式碼後，都需要按「Ctrl + C」停止伺服器，然後再重新啟動伺服器，瀏覽器畫面才會更新的惱人步驟。

</blockquote>
* 定義開發用腳本
「啟動伺服器」是一個很常見的情境，這種情境會直接寫成腳本 (script)。

:arrow_right: 修改 package.json 設定常見的腳本：

```
// 修改 package.json
"scripts": {
  "start": "node app.js",
  "dev": "nodemon app.js",
  "test": "echo \"Error: no test specified\" && exit 1"
  }
```
只要用 npm run 加上腳本，就可以執行內容，例如：

`npm run start` → 等於執行 `node app.js`
`npm run dev` → 等於執行 `nodemon app.js`

### 6. 設定版本控制

* 初始化 git: `git init`
* .gitignore 
除了 node_modules 以外，作業系統還可能有一些自動生成的檔案，以下是一份常見的檔案清單，可以整份直接貼上，不需要等遇到時才一個個處理：

```javascript
# OS X
.DS_Store*
Icon?
._*

# Windows
Thumbs.db
ehthumbs.db
Desktop.ini

# npm
node_modules
*.log
*.gz
```
* 建立第一筆 commit:
`git add .`  
`git commit -m "feat: project init"`
參考: [Conventional Commits](https://wadehuanglearning.blogspot.com/2019/05/commit-commit-commit-why-what-commit.html)

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>