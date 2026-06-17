---
title: "Sass 編譯：指令列實作 - 筆記"
pubDatetime: 2025-04-16T21:32:14.000Z
modDatetime: 2026-05-25T10:04:23.750Z
tags: ["Node.js","cheatsheet","CSS","Sass"]
description: "Table of contents 1. 檢查 Node.js 版本 先確認是否已經安裝 v14 以上的 Node.j..."
hackmd_id: "HJKqXWACJl"
---

## Table of contents


 

## 1. 檢查 Node.js 版本  
先確認是否已經安裝 v14 以上的 Node.js 版本。  
在終端機輸入`node -v`檢查。

## 2. 安裝 Sass
* 透過 npm 安裝用於 Node.js 環境的 sass 工具包：  
`npm install sass@(version)`
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

某些作業系統可能會預裝了 Sass，但有可能是已經停止維護的實作版本 (implementation) ，在語法的支援上可能會有問題。可以使用 `--force` 參數來強制覆蓋，以避免此問題。  
`npm install sass@(version) --force`

</blockquote>

* 確認安裝成功
 `npx which sass`
 如果沒有`which`就按照終端機指令安裝。`which` 指令會回傳設定檔的本機位置:
 `C:\Users\[username]\node_modules\.bin\sass.CMD`  
副指令` --version` 會回傳版本號:  
`npx sass --version`

* 查看內建說明書  
內建說明書通常可用以下指令來開啟，以 sass 為例：  
`man sass`  
`npx sass -help` 或 `npx sass --help`  
Sass 剛好沒有提供 `man` 指令 (`man` 是 manual 的意思)。使用 `npx sass -help` 來開啟文件，在文件上方找到基本的語法結構，以及描述：  
![螢幕擷取畫面 2025-04-17 125644](/images/r1QYYbA0yl.png)


## 準備測試專案  
準備以下結構的測試專案:
```
test
├── src
│  └── main.scss
└── dst
```

在 `test/src/main.scss` 建立設定檔：

```scss
// test/src/main.scss
$bg_color : #b3d9ff;
$font_color : black;
body{
  background-color:$bg_color;
  color: $font_color;
}
```

## 常見編輯情境與指令實作

三種最常見的編譯情境：

1. 編譯單一 sass 檔案成為 css 檔案  
1. 追蹤單一 sass 檔案並持續編譯成 css  
1. 追蹤資料夾並持續編譯成 css

### 編譯單一 sass 檔案成為 css 檔案
```
cd test/src
npx sass main.scss output.css 
```
編譯後會出現結果如下:  
![螢幕擷取畫面 2025-04-17 130122](/images/Hk3oqbC0kl.png)


### 追蹤單一 sass 檔案並持續編譯成 css  
將 compile 過程自動化。此時可以使用` --watch `選項，一旦 `.scss` 檔案有變動，就會自動編譯成 `.css` 檔案。

`npx sass --watch main.scss:output.css `  
`sass --watch` 會啟動一個程序來監控檔案。

在輸入指令後，可以在 `.scss` 檔案上做任何修改，觀察 terminal 裡的變化。  
修改時，terminal 自動監測到變化，並自動同步了 `.scss` 到 `.css` 的編譯，從作業系統的角度來說，當我們啟動了` sass --watch` 時，它其實開啟了一個新的 node 程序 (process)。

按下 Ctrl-C 可以停止監控。



### 追蹤資料夾並持續編譯成 css

實務上，我們會有多個 `scss` 檔案，這時候，就可以**直接追蹤整個資料夾 (包含子資料夾)** 中的 `.scss `檔案並輸出 `.css `檔案至另一個資料夾。

先建立一個 `scss_dir` 資料夾，並且把 `main.scss` 移到 `scss_dir` 資料夾裡:

```bash
# 1. 進入 test 資料夾
cd test

# 2. 建立新資料夾 src/scss_dir
mkdir src/scss_dir

# 3. 移動 main.scss 到新資料夾
mv src/main.scss src/scss_dir/

```
或是一行完成：  
`mkdir -p src/scss_dir && mv src/main.scss src/scss_dir/`

目前專案結構如下:
```
test
├── src
│  └── scss_dir
│      └── main.scss
└── dst
```

接著輸入以下指令，啟動監控程序：

```bash
npx sass --watch scss_dir:css_dir
```
執行指令時，專案目錄自動生成了` css_dir` 資料夾，並且映射 (map) 了一套檔名相同的` .css `檔案：  
![螢幕擷取畫面 2025-04-17 132948](/images/BkNBZMCRJe.png)

資料夾結構:
```
test
    ├─dst
    └─src
        ├─css_dir
        └─scss_dir
```
接下來在 `scss_dir `資料夾裡的變動都會同步到 `css_dir` 資料夾裡。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

使用Webpack 建立本地開發環境，請參考[這篇文](https://hackmd.io/YEDcK4FoT-aWsLgJxRhBww)。

</blockquote>

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>