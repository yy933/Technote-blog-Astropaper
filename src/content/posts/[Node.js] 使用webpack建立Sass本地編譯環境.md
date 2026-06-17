---
title: "[Node.js] 使用webpack建立Sass本地編譯環境"
pubDatetime: 2025-04-17T21:57:10.000Z
modDatetime: 2026-05-25T10:04:23.696Z
tags: ["Node.js","Sass","CSS","cheatsheet","Webpack","npm"]
description: "Table of contents 前言 在實務上，多半不會從零開始建立編譯環境。 實務上由於需要整合各式各樣的開發流..."
hackmd_id: "Sks-N_IZa"
---

## Table of contents


## 前言
在實務上，多半不會從零開始建立編譯環境。

實務上由於需要整合各式各樣的開發流程，通常會**使用 webpack 或 gulp 等「任務管理工具」**，這裡介紹用 webpack 來做本地開發方案。

webpack 強調**把所有的任務區分成不同的模組，最後再統一打包、編輯，輸出成瀏覽器需要的靜態檔案** (包含 JavaScript、CSS、圖片⋯⋯)。這麼做的好處除了**精簡操作指令外，還可以將設定檔納入版本控制系統之中**。

除了 webpack，某些 IDE 也內建了類似的功能與設定介面 (有些團隊甚至會使用 bash 檔) ，可以達成類似的目的。

## 步驟

### 檢查 Node.js 版本
打開終端機輸入` node -v`，，有回傳版本就是安裝完成。

### 專案初始化
```bash
# 建立專案資料夾
mkdir webpack_test
cd webpack_test
# Node.js 環境初始化
npm init -y
```

### 安裝相關 npm 套件
```bash
npm install webpack@4.43.0 webpack-cli@3.3.11 mini-css-extract-plugin@0.9.0 css-loader@3.5.3 sass-loader@8.0.2
```

`webpack` 及 `webpack-cli` 是主要的打包工具，而 `css-loader`、 `sass-loader` 則是幫我們編譯 `sass` 檔案。最後再透過 `mini-css-extract-plugin` 將 `css` 檔案抽離出來。
 
### 建立設定檔
#### 專案架構規劃
![ExportedContentImage_03](/images/Sk6PPMRCkg.png)
```
webpack_test/
├── node_modules/         ← 安裝 npm 後自動產生
├── src/
│   ├── scss/
│   │   └── main.scss
│   └── main.js
├── index.html
├── package.json          ← npm init 產生
├── package-lock.json     ← 安裝套件後產生
├── webpack.config.js
```


指令快速建立: (目前位在`webpack_test`資料夾中)
```bash
mkdir -p src/scss && \
touch src/scss/main.scss src/main.js index.html webpack.config.js 
```


在 root 存放 index.html 及 webpack.config.js，其他資源都放在 src 資料夾中，這個資料夾代表編譯前的原始碼 (source)。

在 src 資料夾中，main.js 是 webpack 的進入點，webpack 會以這個檔案作為出發點依序去找相關聯的檔案。

接著在以下檔案寫點內容：
**Index.html**
網頁的 HTML 檔需要放在專案第一層:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <h1>Hello, Webpack Sass</h1>
</body>
</html>
```

**src/scss/main.scss**
Sass 設定檔放在 `src/scss/main.scss`：

```sass
$bg_color : #b3d9ff;
$font_color : black;
body{
  background-color:$bg_color;
  h1 {
    color: $font_color
  }
}
```

**src/main.js**
在 `src/main.js` 設定 `webpack` 的進入點，`webpack` 會以這個檔案作為出發點依序去找相關聯的檔案。

```javascript
import './scss/main.scss'
console.log('JS loaded!')
```

### Webpack 設定

設定 `webpack.config.js` :
```javascript
const path = require('path')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          'sass-loader'
        ]
      }
    ]
  },
  plugins: [new MiniCssExtractPlugin()]
}
```
#### 說明：
* `entry`：進入點指向 `'./src/main.js'`
* `output`：檔案編譯後的輸出位置，這裡會把 `main.js` 打包成 `dist `資料夾裡面的 `bundle.js`
* `rules` 裡面針對 `*.scss` 副檔名的檔案，會用 `mini-css-extract-plugin`、`css-loader` 及 `sass-loader` 三個 loader 來編譯
* 編譯後的 `scss` 只會改變副檔名，檔名的部分不變 (`main.css`)

### HTML 載入編譯後檔案

回到 `index.html `來載入預計會編譯好的檔案位置，分別為 `dist/bundle.js` 及 `dist/main.css`：(這兩個檔案目前還不存在，編譯後才會出現)

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <link rel="stylesheet" href="dist/main.css">
</head>

<body>
  <h1>Hello, Webpack Sass</h1>
  <script src="dist/bundle.js"></script>
</body>

</html>
```

### 設定編譯腳本
到 `package.json` 中新增一個 `script`
```json
"scripts": {
   "build": "webpack --mode production"
}
```

### 執行編譯
在終端機執行：`npm run build`

成功編譯的話，就可以在專案中看到 dist 資料夾，裡面包含了 js 及被編譯後的 css：
![螢幕擷取畫面 2025-04-17 144544](/images/HyCmm70AJg.png)
打開 index.html :
![螢幕擷取畫面 2025-04-17 144708](/images/Hkyv7X001l.png)

### :spiral_note_pad: Error

這次是使用 Node.js v16.16.0版本才成功編譯，如果Node.js版本太高可能會出現error，可以退回v16試試看。詳細情形如下：

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

### `Error: error:0308010C:digital envelope routines::unsupported`
在 Node.js 17 或以上的版本執行 Webpack 4 或 5，但這些版本的 Webpack **在 Node.js 17 開始，預設的 OpenSSL 加密方式不再相容**。**Node.js 17 開始，OpenSSL 3.0 被納入，而某些加密演算法（如 Webpack 使用的 MD4）不再被支援**，因此需要額外加 `--openssl-legacy-provider` 來讓它啟用舊版的加密方式。
### 除錯：
✅ 方法一：設置環境變數 `NODE_OPTIONS`
Windows（PowerShell）:
`$env:NODE_OPTIONS="--openssl-legacy-provider"`
Windows（CMD）:
`set NODE_OPTIONS=--openssl-legacy-provider`
Linux / macOS（終端機）:
`export NODE_OPTIONS=--openssl-legacy-provider`
最後再`npm run build`
:arrow_forward: 每次開一個新的終端機視窗或重開機後，都得再輸入一次，因為這只是暫時的環境變數設定，離開該終端機 session 就會失效。

✅ 方法二：在 `package.json` 裡修改 `script`
直接在 `package.json` 裡修改 `scripts`:
```json
"scripts": {
  "build": "NODE_OPTIONS=--openssl-legacy-provider webpack --mode production"
}
```
Windows 系統:
`"build": "set NODE_OPTIONS=--openssl-legacy-provider && webpack --mode production"`

✅ 方法三：降級 Node.js 到 16
如果不希望加參數，可以選擇安裝 Node.js 16.x（長期支援版 LTS），因為這版本不會有這個錯誤。

</blockquote>