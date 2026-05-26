---
title: "在Express專案中載入靜態檔案 - 筆記"
pubDatetime: 2026-05-26T03:29:26.586Z
tags: ["Express.js","Node.js","cheatsheet","browser"]
description: " Table of contents 什麼是靜態檔案 靜態檔案（static files）指的是不需要再經過伺服器額外處..."
---

## Table of contents

## 什麼是靜態檔案
靜態檔案（static files）指的是**不需要再經過伺服器額外處理的檔案**。常見的靜態檔案例如：在網頁中載入的 CSS、JavaScript 或圖片檔，這些檔案通常不需要再經過伺服器額外處理，**伺服器只需要提供一個連結，讓瀏覽器直接抓取這些檔案即可**。

## 建立靜態檔案資料夾

把所有網頁上需要使用到的 CSS、JavaScript 或者圖檔都放在一個名為 public 的資料夾以方便管理，所以先在根目錄下開一個 public 資料夾。

:bulb:（延續[這篇文](https://hackmd.io/44K-PkTHQ6SJmNZPt4psGQ)的專案結構）
建立`public`資料夾與其中`stylesheets` 和 `javascripts` 的資料夾:
![ExportedContentImage_01 (4)](https://hackmd.io/_uploads/SkvIgxr1ex.png)
將需要使用到的js和css檔案放進資料夾，例如Bootstrap：(到[Bootstrap官網](https://getbootstrap.com/docs/5.1/getting-started/download/)下載，在 Bootstrap v5 中有額外使用 Popper.js 這個套件，因此需要一併下載：[Popper.js官網](https://floating-ui.com/?utm_source=popper.js.org)))
![ExportedContentImage_07 (2)](https://hackmd.io/_uploads/HJKg-xBJxe.png)

## 載入靜態檔案
因為在 Bootstrap 中會用到 popper.js，因此**載入時需要留意順序， popper.js 一定要放在 Bootstrap 之前** :
```html
<!-- ./views/layouts/main.handlebars -->
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- ... -->
  <!-- link css files here -->
  <link rel="stylesheet" href="/stylesheets/bootstrap.css">
</head>
<body>
   <!-- ... -->

  <!-- include javascript files here -->
  <script src="/javascripts/popper.js"></script>
  <script src="/javascripts/bootstrap.js"></script>

</body>
</html>
```

## 設定 Express 路由以提供靜態檔案
在`app.js`中告訴Express靜態檔案的位置:
```javascript
// app.js

// require packages used in the project
// ...

// setting template engine
// ...

// setting static files
app.use(express.static('public'))

// routes setting
// ...

// start and listen on the Express server
// ...
```
`app.use(express.static('public'))`就是在告訴 Express 靜態檔案是放在名為 public 的資料夾中，它不必針對這個資料夾內的檔案做什麼，只要產生對應的路由讓我們用就好。





<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>