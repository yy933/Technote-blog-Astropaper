---
title: "在Express專案中使用樣板引擎Handlebars - 筆記"
pubDatetime: 2025-04-23T00:15:06.000Z
tags: ["Express.js","Node.js","cheatsheet","browser"]
description: " Table of contents [建立Express專案](https://hackmd.io/7L4TYmU9R..."
---

## Table of contents


[建立Express專案](https://hackmd.io/7L4TYmU9R_2k20BaxounTA)之後，使用樣板引擎把帶有 HTML 內容的「樣板檔案」 (template files) 轉換成真正的 HTML 檔案再回應到瀏覽器上。這裡使用的是Handlebars這個樣板引擎。

### 1. 下載與安裝 express handlebars
* `npm i express-handlebars`

### 2. 設定在 Express 中使用的樣版引擎
* 在程式入口 app.js 中加入：
```javascript
// require express-handlebars here
const exphbs = require('express-handlebars')
```
* 載入之後，要告訴 Express：把樣板引擎交給 express-handlebars
```javascript
// setting template engine
app.engine('handlebars', exphbs.engine({ defaultLayout: 'main' }))
app.set('view engine', 'handlebars')
```
<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
`app.engine`：透過這個方法來定義要使用的樣板引擎，其中
* 第一個參數是這個樣板引擎的名稱，也就是handlebars
* 第二個參數是放入和此樣板引擎相關的設定。這裡設定了預設的佈局（default layout）需使用名為 main 的檔案。稍後再來建立這支 main 檔案。
* `app.set`：透過這個方法告訴 Express 要設定的 view engine 是 handlebars。
</div>
### 3. 定義佈局（layout）和局部樣板（partial template）
在同一個網站內，幾乎每一個頁面都會套用的版型，就稱作佈局（layout）；
在不同頁面會有不同內容的地方，就稱作「局部樣板 (partial template)」

#### 建立 views 和 layouts 資料夾
結構如下圖:
![ExportedContentImage_09 (1)](https://hackmd.io/_uploads/S1iCnJrkll.png)
![ExportedContentImage_10 (1)](https://hackmd.io/_uploads/HJR16yBkll.png)
注意：這些資料夾名稱都要遵守 Express 框架的慣例(`views`, `layouts`)，如果名稱不一樣，Express 就會找不到檔案，而產生類似` no such file or directory` 的錯誤訊息。
* 把「佈局」的部分放在 layouts 資料夾中
* 把「局部樣板」的部分放在 views 這個資料夾中

#### 建立 layout
在 layouts 這個資料夾中，先建立一支名為 main.handlebars 的檔案

<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
之前在`app.js`中設定：
`app.engine('handlebars', exphbs({ defaultLayout: 'main' }))`
宣告預設用名為 `main.handlebars` 這支檔案當作佈局。
</div>

```html
<!-- ./views/layouts/main.handlebars -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <!-- partial templates will replace the part of "body" here -->
  {{{ body }}}
</body>
</html>
```

:pencil: 這裡用` {{{ body }}}` 的意思，就是會**把局部樣板的內容都放在這裡顯示**。
可以把共通的部分，例如navbar，放在這裡：
```html
<!-- … -->
<body>

  <!-- navigation -->
  <nav class="navbar navbar-dark bg-dark">
    <div class="container">
      <a class="navbar-brand" href="#">Movie List</a>
    </div>
  </nav>

  <!-- partial templates will replace the part of "body" here -->
  {{{ body }}}
</body>
<!-- … -->
```
#### 建立 partial templates
接著來建立第一支局部樣板的檔案。它會放在 `views` 這個資料夾中，取名為 `index.handlebars`。這支局部樣板內所撰寫的內容，之後就會出現在 layouts 裡 `main.handlebars` 檔案的` {{{ body }}}` 中。
```html
<!-- ./views/index.handlebars -->
<h1>Create your own server with Node.js</h1>
```

:bulb: **在`main.handlebars`中引入partials:**
把一些可以重用的元件封裝成partials，再放到`main.handlebars`中，例如navbar：
1. 建立 partial 檔案
```
views/
├── layouts/
│   └── main.handlebars
├── partials/
│   └── navbar.handlebars ← 👈 這裡放 partial
├── index.handlebars
```
2. `navbar.handlebars `的內容：
```html
<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <div class="container-fluid">
    <a class="navbar-brand" href="/">Home</a>
    <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
      <div class="navbar-nav">
        <a class="btn btn-sm btn-outline-secondary {{#if (eq active 'about')}}active{{/if}}" href="/about">About</a>
        <a class="btn btn-sm btn-outline-secondary {{#if (eq active 'portfolio')}}active{{/if}}" href="/portfolio">Portfolio</a>
        <a class="btn btn-sm btn-outline-secondary {{#if (eq active 'contact')}}active{{/if}}" href="/contact">Contact</a>
      </div>
    </div>
  </div>
</nav>
```
3. 在 `main.handlebars` 中引入 partial
注意語法，`{{> navbar}}`才能正確引入partial
```html
<body>
  {{> navbar}} 
  {{{body}}}
</body>
```
4. 記得到`app.js`指定partials資料夾
`app.js`
```javascript
app.engine('handlebars', exphbs.engine({
  defaultLayout: 'main',
  partialsDir: __dirname + '/views/partials' // 👈 加這行
}))
```
### Express 中路由設定
```javascript
// app.js
// ...
// routes setting
app.get('/', (req, res ) => {
  res.render('index')
})
// ...
```

### Express 的靜態檔案 
如果需要載入靜態檔案，參考這篇文的設定：[在Express專案中載入靜態檔案](https://hackmd.io/77SsyLdwSN2uKoDMoY9EBA)

### 其他常用的工具/語法
#### helpers
在 express-handlebars 中，helpers（輔助函式）是用來擴充 Handlebars 模板功能的小函式，我們可以在模板中呼叫這些函式，讓 HTML 的渲染更靈活、更動態。
簡單來說，就是：
> Helpers 是自己定義的小工具，可以在模板裡使用，像是格式化日期、條件判斷、字串處理等等。

以上述的navbar中的`eq`為例子：
```html
<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <div class="container-fluid">
    <a class="navbar-brand" href="/">Home</a>
    <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
      <div class="navbar-nav">
        <a class="btn btn-sm btn-outline-secondary {{#if (eq active 'about')}}active{{/if}}" href="/about">About</a>
        <a class="btn btn-sm btn-outline-secondary {{#if (eq active 'portfolio')}}active{{/if}}" href="/portfolio">Portfolio</a>
        <a class="btn btn-sm btn-outline-secondary {{#if (eq active 'contact')}}active{{/if}}" href="/contact">Contact</a>
      </div>
    </div>
  </div>
</nav>
```
這裡`eq`用來判斷兩者是否相等，在`app.js`中加入helper:
```javascript
const exphbs = require('express-handlebars')

app.engine('handlebars', exphbs.engine({
  defaultLayout: 'main',
  partialsDir: __dirname + '/views/partials',
  helpers: {
    eq: (a, b) => a === b  // 自定義 helper 判斷相等
  }
}))
```

但是，如果需要使用多個helpers，全部都寫在`app.js`中的話，`app.js`會變得很雜亂。
因此，最好還是將helpers封裝成一個獨立的檔案，再引入`app.js`中：
* 在專案根目錄下創建一個 `helpers.js` 檔案，然後把所有的 helpers 都寫進去。
```javascript
module.exports = {
  eq: (a, b) => a === b,
  uppercase: (str) => str.toUpperCase(),
  formatDate: (date) => new Date(date).toLocaleDateString(),
}
```
* 在 `app.js` 引入並註冊這些 helpers
```javascript
const express = require('express')
const exphbs = require('express-handlebars')
const helpers = require('./helpers')  // 引入 helpers.js

const app = express()

app.engine('handlebars', exphbs.engine({
  defaultLayout: 'main',
  partialsDir: __dirname + '/views/partials',
  helpers: helpers  // 註冊所有 helpers
}))

app.set('view engine', 'handlebars')
app.use(express.static('public'))

app.listen(3000, () => {
  console.log('Server is running on port 3000')
})
```
這樣子更方便管理與擴充，`app.js`看起來也更乾淨！
<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
* 在npm上面搜尋"express handlebars helpers"也有很多現成的package可以使用，例如這個[@budibase/handlebars-helpers](https://www.npmjs.com/package/@budibase/handlebars-helpers)有超過130種helpers，可再依需求安裝。
* express-handlebars 內建的 helpers : [Built-in Helpers](https://handlebarsjs.com/guide/builtin-helpers.html)
</div>
<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>