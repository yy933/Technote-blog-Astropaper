---
title: "應用Redis在Node.js中實作快取(Cache) - 筆記"
pubDatetime: 2026-05-25T11:17:37.307Z
tags: ["cache","database","Node.js","Redis","Express.js"]
description: " Table of contents :memo: 簡介 當網站有些常常需要存取的資料、或是要依賴第三方API回傳資料時..."
---

## Table of contents

## :memo: 簡介
當網站有些常常需要存取的資料、或是要依賴第三方API回傳資料時，經常需要花許多時間等待對方的伺服器回應，造成頁面載入時間過長；回想一下，當我們在瀏覽購物網站時，對於那些需要等待很久的網站頁面，會有什麼反應呢？耐心等它跑完、或是直接關掉？大部分的人應該是選擇後者吧！

因此，提升頁面載入速度可以**有效改善使用者體驗**，同時也可以**減少伺服器的負擔**，如果是使用第三方API，也不需要不斷對它發送請求(畢竟第三方API時常有存取次數限制)，可以先將這些資料快取存起來，之後發送請求的時候就可以使用快取中的資料，直到快取過期。 


## :memo: 情境
做一個食譜的網站，透過串接[spoonacular API](https://spoonacular.com/food-api)取得食譜的資料，但是存取資料有次數限制(~~當然也是可以課金解決~~)，另外就是等待回傳的時間稍慢，頁面載入時間長。因此，透過快取先把資料存起來，之後的請求可以先去快取中找資料，找不到的話再向第三方API發送請求獲得資料，可以有效減少發送重複請求的次數，提升載入的速度。

這裡使用[Redis](https://redis.io/)資料庫來儲存快取資料，Redis是一種非關聯式資料庫(NoSQL)，主要以key-value pair的方式儲存資料在記憶體內(in-memory)，效能極高，更多關於Redis的功能介紹可參考以下文章：
- [資料庫的好夥伴：Redis](https://blog.techbridge.cc/2016/06/18/redis-introduction/)
- [Redis(AWS)](https://aws.amazon.com/tw/redis/)
- [Redis 學習筆記(2)-Redis特點](https://ithelp.ithome.com.tw/articles/10285358)
- [Redis了解與應用](https://hackmd.io/@cynote/BkobMykLw)


## :memo: 實作分享
1. **前置作業(環境設置)**
   * Node.js
   * npm
   * Redis server：先安裝Redis server，安裝方法依照不同作業系統選擇、或是可以用Docker安裝，參考以下文章：
    [MacOS](https://medium.com/dean-lin/%E6%89%8B%E6%8A%8A%E6%89%8B%E5%B8%B6%E4%BD%A0%E5%9C%A8-macos-%E5%AE%89%E8%A3%9D-redis-another-redis-desktop-manager-8d0b45062f9) 
    [Windows](https://marcus116.blogspot.com/2019/02/how-to-install-redis-in-windows-os.html)
    [Linux](https://redis.io/docs/getting-started/installation/install-redis-on-linux/)
    [Docker](https://medium.com/idomongodb/installing-redis-server-using-docker-container-453c3cfffbdf)
    通常會同時安裝redis-cli，當然也可以順便安裝GUI(如[Redis Desktop Manager](https://github.com/uglide/RedisDesktopManager)、[RedisInsight](https://redis.com/redis-enterprise/redis-insight/))，安裝 Redis Desktop Manager 可以參考[這篇文章](https://marcus116.blogspot.com/2019/02/redis-redis-redis-desktop-manager.html)
    
2. **初始化專案並安裝相關套件**
```
mkdir node-redis-cache
cd node-redis-cache
npm init -y
npm i axios express ioredis dotenv
```
* `axios`: HTTP client，用來向API發送請求
* `express`: Node.js的輕量級框架
* `ioredis`: Node.js的Redis client，用來和Redis資料庫溝通的工具 
* `dotenv`: 幫助載入`.env`中的環境變數的工具

專案架構如下：
```
node-redis-cache
├─ .env
├─ app.js
├─ package-lock.json
├─ package.json
└─ redis.js

```
3. **建立Express伺服器**
建立一個基本的Express伺服器：
```javascript
//app.js
const express = require('express')
const app = express()
require('dotenv').config()
const port = process.env.PORT || 3000

app.listen(port, () => {
  console.log(`Express server is running on port ${port}`)
})
```
4. **連線Redis資料庫**
建立一個檔案來管理Redis連線資訊。這裡`const redis = new Redis()`建立的Redis實例會連線到host:127.0.0.1和port:6379，也可以加入密碼或使用者名稱等設定，詳情參考[ioredis的文件](https://github.com/redis/ioredis#connect-to-redis)
```javascript
// redis.js
const Redis = require('ioredis')
const redis = new Redis()

redis.on('error', (error) => {
  console.log('Redis client error: ', error)
})

redis.on('connect', () => {
  console.log('Connection Successful!!')
})

module.exports = redis
```

5. **加入第三方API，使用`axios`請求資料**
使用spoonacular API需要先取得一個API key，先[註冊一個帳號](https://spoonacular.com/food-api/console#Dashboard)，這個API內容十分豐富，文件也寫得非常清楚好懂，取得key之後可以先玩玩看:wink:這裡我選擇[Get Random Recipes](https://spoonacular.com/food-api/docs#Get-Random-Recipes)來實作，在`app.js`中加入以下程式碼：（記得在`.env`中加入`API_KEY`）
```javascript
//app.js
...
const axios = require('axios')
app.get('/recipes', async (req, res, next) => {
    try{
    const recipes = await axios({
      method: 'get',
      url: 'https://api.spoonacular.com/recipes/random',
      headers: { 'Content-Type': 'application/json' },
      params: {
        limitLicense: true,
        number: 60,
        tags: 'vegetarian',
        apiKey: process.env.API_KEY
      }
    })
     // 處理一下回傳的data
     const recipesRawData = recipes.data.recipes.map(recipe => ({
       id: recipe.id,
       dishName: recipe.title,
       vegetarian: recipe.vegetarian,
       glutenFree: recipe.glutenFree,
       dairyFree: recipe.dairyFree,
       cookingTime: recipe.readyInMinutes,
       servings: recipe.servings,
       image: recipe.image,
       instruction: Object.assign({}, recipe.analyzedInstructions[0]?.steps.map(s => s.step)),
       ingredient: Object.assign({}, recipe.extendedIngredients?.map(i => i.original) ),
       fullDetailsUrl: recipe.spoonacularSourceUrl
       }))
    } catch(error){
       console.log(error)
    }
})

```
6. **寫入快取邏輯**
:bell: 思路：如果快取中有資料，就從快取回傳資料；如果快取中沒有資料，則對API發送請求，並將回傳的資料存入快取。<br>
因此，在`GET /recipes` 寫入以下邏輯：
```javascript
//app.js
...
// 引入Redis client
const redis = require('./redis')

app.get('/recipes', async (req, res, next) => {
  try {
    // 先到Redis中找資料，尋找key是recipes的資料
    await redis.get('recipes', async (err, data) => {
      if (err) {
        console.log(err)
      }
      // 如果有此筆資料，則從Redis取得資料回傳
      if (data) {
        const recipesData = JSON.parse(data)
        return res.json(recipesData)
      } else {
        // 如果Redis中找不到，則呼叫第三方API請求資料
        const recipes = await axios({
          ...
        })
        
        const recipesRawData = recipes.data.recipes.map(recipe => ({
          ...
        }))
    // 得到的資料存入Redis方便下次使用，設定key是recipes，並設定快取過期時間為86400秒(1天)
    await redis.set('recipes', JSON.stringify(recipesRawData), 'EX', 86400, err => {
      if (err) console.log(err)
      })
    return res.json(recipesRawData)
      }
    })
  } catch(error) { console.log(error) }
})
```
:pushpin: Points：
- [`redis.set`](https://redis.io/commands/set/)和[`redis.get`](https://redis.io/commands/get/)是Redis設置/取出key-value pair的方法，這裡使用`JSON.parse()`將字串轉換為物件和`JSON.stringfy()`將物件轉換成字串存入Redis。
- 幫快取資料設置過期時間，過期後該筆資料會自動被清除，更多關於Redis過期資料參考[這裡](https://redis.io/commands/expire/#how-redis-expires-keys)。

7. **使用Postman測試**
接著來測試加入快取後回傳的速度，先讓伺服器運作：
`node app.js`
接著到Postman設置環境並發送請求，第一次請求由於快取中還沒有資料，因此必須向第三方API請求資料，回傳時間比較長，花了1288ms：
![](https://hackmd.io/_uploads/ByhYniNa2.png) <br>
接著到Redis中查看資料是否成功存入：
![](https://hackmd.io/_uploads/B1iM6oEp2.png)
成功寫入了一筆key為recipes、value是回傳的60筆食譜資料轉成的字串。
<br>
接著再發送第二次請求，這次是從Redis中取得資料：
![](https://hackmd.io/_uploads/HyZ10oV6h.png)
這次只花了17ms就回傳，速度大幅提升!

完整的`app.js`程式碼如下：
```javascript
const express = require('express')
const app = express()
const redis = require('./redis')
require('dotenv').config()
const port = process.env.PORT || 3000
const axios = require('axios')

app.get('/recipes', async (req, res, next) => {
  try {
    // Looking for data in redis store first
    await redis.get('recipes', async (err, data) => {
      if (err) {
        console.log(err)
      }
      if (data) {
        const recipesData = JSON.parse(data)
        return res.json(recipesData)
      } else {
        // 如果快取沒有，則從 API 獲取資料
        const recipes = await axios.get('https://api.spoonacular.com/recipes/random', {
      headers: { 'Content-Type': 'application/json' },
      params: {
        limitLicense: true,
        number: 60,
        tags: 'vegetarian',
        apiKey: process.env.API_KEY
      }
    })
        // 處理一下API回應的data
        const recipesRawData = recipes.data.recipes.map(recipe => ({
          id: recipe.id,
          dishName: recipe.title,
          vegetarian: recipe.vegetarian,
          glutenFree: recipe.glutenFree,
          dairyFree: recipe.dairyFree,
          cookingTime: recipe.readyInMinutes,
          servings: recipe.servings,
          image: recipe.image,
          instruction: Object.assign({}, recipe.analyzedInstructions[0]?.steps.map(s => s.step)),
          ingredient: Object.assign({}, recipe.extendedIngredients?.map(i => i.original)),
          fullDetailsUrl: recipe.spoonacularSourceUrl
        }))
        // save recipe data in cache（expiry time: 86400 sec = 1 day）
        await redis.set('recipes', JSON.stringify(recipesRawData), 'EX', 86400)
        return res.json(recipesRawData)
      }
    })
  } catch (error) {
    console.error('Error fetching recipes:', error);
    return res.status(500).json({ error: 'Failed to fetch recipes' }); }
})
app.listen(port, () => {
  console.log(`Express server is running on port ${port}`)
})

```

## 小結
以上簡單分享應用Redis在Node.js中實作快取，實際上快取的設計還有很多細節需要考慮，例如當快取滿了清除資料的順序、如何將快取資料和關聯式資料庫的資料同步、快取資料失效或過期的處理......等，往後再更進一步探討！


---

## 參考資料
* [在 Node.js 使用 Redis 來 Cache API 的回傳資料，以減少查詢資料的回傳時間](https://medium.com/dean-lin/%E5%9C%A8-node-js-%E4%BD%BF%E7%94%A8-redis-%E4%BE%86-cache-api-%E7%9A%84%E5%9B%9E%E5%82%B3%E8%B3%87%E6%96%99-%E4%BB%A5%E6%B8%9B%E5%B0%91%E6%9F%A5%E8%A9%A2%E8%B3%87%E6%96%99%E7%9A%84%E5%9B%9E%E5%82%B3%E6%99%82%E9%96%93-4eeeb677b81e)
* [Caching HTTP Requests Between Services Using Redis, Axios, and Node.js](https://blog.mailpace.com/blog/caching-http-requests-between-services/)
* [Connecting to a Redis database using Node.js](https://northflank.com/guides/connecting-to-a-redis-database-using-node-js)
* [Redis 做資料快取的基本使用 (搭配Node.js)](https://huskylin.github.io/2020/07/10/Redis-%E5%81%9A%E8%B3%87%E6%96%99%E5%BF%AB%E5%8F%96%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8-%E6%90%AD%E9%85%8DNode-js/index.html)
* [Caching data in Node.js application with Redis](https://blog.tericcabrel.com/caching-data-nodejs-application-redis/)
* [Optimizing Node.js Performance with Redis Caching](https://blog.bitsrc.io/optimizing-node-js-performance-with-redis-caching-f509edf33e04)
* [Caching like a boss in NodeJS](https://medium.com/@danielsternlicht/caching-like-a-boss-in-nodejs-9bccbbc71b9b)


::: success
:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！
:::