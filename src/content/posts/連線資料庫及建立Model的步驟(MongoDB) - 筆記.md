---
title: "連線資料庫及建立Model的步驟(MongoDB) - 筆記"
pubDatetime: 2023-09-25T01:32:26.000Z
tags: ["MongoDB","Mongoose","Express.js","Node.js","cheatsheet","database"]
description: " Table of contents 新增資料庫 先到[MongoDB Atlas](https://www.mongo..."
---

## Table of contents


 

## 新增資料庫
* 先到[MongoDB Atlas](https://www.mongodb.com/atlas/database)建一個雲端資料庫，或是建立一個本地資料庫([這裡](https://www.mongodb.com/try/download/community)下載)

## 連線專案伺服器與資料庫 
* 在專案中安裝MongoDB的ODM-Mongoose:`npm i mongoose`
* 設定專案與資料庫的連線: 
 ### 在`app.js`中載入`mongoose`並設定連線
```javascript
  const mongoose = require('mongoose') 
  mongoose.connect(process.env.MONGODB_URI) 
  ```
### 設定環境變數
  連線字串裡包含了「帳號」與「密碼」等敏感資訊，因此，藉由設定環境變數的方式，來將這些敏感資訊傳入程式碼，避免敏感資訊直接暴露在程式碼中。
  <div class="alert info">

  :bulb: MongoDB的連線字串(Connection Strings) for Node.js: 
  `mongodb+srv://[使用者名稱:密碼]@[資料庫伺服器位址]/[資料庫名稱]?[其他選項]`
  詳細可參考文件: [Connection Strings](https://www.mongodb.com/docs/manual/reference/connection-string/#std-label-connections-connection-examples) 

  </div>
  
  `process.env`是 Node.js 提供的一個介面，讓我們可以調用宣告在系統層的環境變數。環境變數採用的命名方式通常為 **全大寫＋底線**。
  接下來，為了方便管理環境變數，將透過套件 `dotenv` 來設定環境變數。
  + **安裝dotenv**:　`npm i dotenv -D`
  加上 -D 的意思：將套件安裝到 `package.json` 的 devDependencies 上面，表示 production 環境不使用，僅 development 環境使用的套件。
  + **建立.env文件**: 在專案中建立 `.env` 文件，並在上面定義環境變數，把資料庫的連線字串複製上去。
` MONGODB_URI=mongodb+srv://<username>:<password>@<host>/<database>?retryWrites=true&w=majority`
 + **把`.env`文件加入`.gitignore`中**: 
  ```javascript=1
    # npm
    node_modules
    *.log
    *.gz

    # dotenv
    .env
  ```
  + **引入dotenv**: 在`app.js` 上方引入` dotenv`，讓 Node.js 能讀到寫在 `.env` 的環境變數。
```javascript
  // app.js
const express = require('express')
const mongoose = require('mongoose') // 載入 mongoose

// 僅在非production環境時, 使用 dotenv
if (process.env.NODE_ENV !== 'production') {
  require('dotenv').config()
}

const app = express()
mongoose.connect(process.env.MONGODB_URI) // 連線到 mongoDB
```
###  取得連線狀態：設定db
  執行了 `mongoose.connect` 之後會得到一個**連線狀態**，我們需要設定一個參數，把連線狀態暫存下來，才能繼續使用。
 ```javascript=1
     // 取得資料庫連線狀態
    const db = mongoose.connection
    // 連線異常
    db.on('error', () => {
      console.log('mongodb error!')
    })
    // 連線成功
    db.once('open', () => {
      console.log('mongodb connected!')
    })
```
   + **db.on()**: 在這裡用 `on` 註冊一個事件監聽器，用來監聽 `error` 事件有沒有發生，語法的意思是「只要有觸發 `error` 就印出 `error` 訊息」。
   + **db.once()**: 針對「連線成功」的` open `情況，我們也註冊了一個事件監聽器，相對於「錯誤」，連線成功只會發生一次，所以這裡特地使用` once`，由於` once `設定的監聽器是一次性的，一旦連線成功，在執行 callback 以後就會解除監聽器。
:arrow_right: 完成連線設定，終端機執行`npm run dev`，出現`mongodb connected!`代表連線成功。
####  處理 DeprecationWarning 警告
  MongoDB 正準備棄用 (deprecate) 某些舊版作法，需要做一些調整。
  例如：
  ```javascript
mongoose.connect(process.env.MONGODB_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  
```
回到終端機看到 `mongodb connected！`並且警告訊息消失代表調整完成。
## 建立Model
  * 在專案中建立models資料夾，並且新增存放model結構的js檔案(如:data.js)
  * 定義資料結構：設定data.js中的schema(資料庫綱要)。
例如：
```javascript
const mongoose = require('mongoose')
const Schema = mongoose.Schema //mongoose提供的模組
const dataSchema = new Schema({
  name: {
    type: String, // 資料型別是字串
    required: true // 這是個必填欄位
  },
  isDone: {
    type: Boolean,
    default: false //設定預設值，也就是在資料生成時，自動帶入的屬性值
  }
})
module.exports = mongoose.model('Data', dataSchema)
```
這裡 Schema 大寫表示用 `new Schema()` 的方式來建構一個新的 Schema 物件實例，把我們設計的資料結構當成參數傳給 `new Schema()`。然後透過 `module.exports` 輸出，`mongoose.model` 會複製我們定義的 Schema ，並編譯成一個可供操作的 model 物件，匯出的時候我們把這份 model 命名為 `Data`，以後在其他的檔案直接使用 `Data` 就可以操作相關的資料了！

---

<span style="font-size: 20px; font-weight: 700" > 參考資料</span>
* AlphaCamp material