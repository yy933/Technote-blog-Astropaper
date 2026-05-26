---
title: "瀏覽器渲染流程與DOM - 筆記"
pubDatetime: 2025-02-13T03:26:56.000Z
tags: ["Interview Preparation","HTML","browser","DOM","CSS"]
description: " Table of contents :memo: 瀏覽器渲染流程 當我們在瀏覽器輸入網址並請求網頁時，瀏覽器會經歷以下..."
---

## Table of contents

## :memo: 瀏覽器渲染流程

當我們在瀏覽器輸入網址並請求網頁時，瀏覽器會經歷以下步驟來渲染畫面：

### 1. 解析 HTML / CSS 檔案，建立物件模型

* HTML → DOM (Document Object Model)
瀏覽器讀取 HTML 原始碼，並將其解析為DOM（Document Object Model）。
DOM 是一個樹狀結構，每個 HTML 標籤都是一個節點 (Node)。

例如:
```html
<html>
  <body>
    <h1>Hello World</h1>
  </body>
</html>
```
會解析成:
```css
Document
 ├── html
     ├── body
         ├── h1
```

* CSS → CSSOM (CSS Object Model)
瀏覽器讀取 CSS 樣式，並將其解析為CSSOM（CSS Object Model）。
例如:
```css
h1 { color: blue; font-size: 24px; }
```
會解析成:
```css
CSSOM:
h1 {
  color: blue;
  font-size: 24px;
}
```

在這個步驟，JavaScript可以介入操作，修改DOM中的節點。JavaScript 能夠處理的是「物件模型」，也就是 DOM 和 CSSOM，模型是瀏覽器為了要讓 JavaScript 去操作網頁元素而建立出來的。

### 2. 組合 DOM 和 CSSOM，建構 Render Tree，準備開始運算
* 瀏覽器將DOM 樹和CSSOM 樹結合，生成渲染樹 (Render Tree)。
* 這棵樹只包含需要顯示的元素（例如 `display: none` 的元素不會出現在渲染樹中）。


### 3. 計算每個元素的畫面位置，產生 Layout
計算每個元素的大小（width、height）和位置，確保它們正確排列在頁面上。

### 4. 繪製畫面細節 （Painting）
瀏覽器將渲染樹的內容轉換為實際像素，並將它們顯示在螢幕上。

### 5. 合成（Compositing）
如果有動畫、變換（如 transform）、z-index 等，瀏覽器會將元素拆分成不同層（Layers），再進行合成 (Compositing)。


## :memo: DOM是什麼

DOM（Document Object Model，文件物件模型）是**瀏覽器將 HTML 轉換成的 JavaScript 物件模型，可以讓我們透過 JavaScript 操作網頁**。

#### (1) DOM 的基本結構
DOM 是樹狀結構，每個 HTML 標籤都是一個節點，例如：
```htmlembedded
<!DOCTYPE html>
<html>
  <head>
    <title>My Page</title>
  </head>
  <body>
    <h1>Hello World</h1>
    <p>Welcome to my website</p>
  </body>
</html>
```
對應的 DOM 結構：
```html
Document
 ├── html
      ├── head
      │    ├── title
      ├── body
           ├── h1
           ├── p
```
瀏覽器載入 HTML 以後，會進一步把 HTML 的語法結構，解析為物件模型，所謂的「物件」擁有屬性與方法，object model 和 HTML 文件的結構、階層關係是一致的。任何在網頁裡出現的內容，無論看得到或看不到的，都會被解析成為 DOM 的一部分，包含註解。

<blockquote class="my-6 p-4 bg-red-50 dark:bg-red-950/30 border-l-4 border-red-500 rounded-r-md text-red-900 dark:text-red-200 blocknoted-fix">

📌 DOM 是瀏覽器開放給程式語言操作網頁元素的一種介面。

</blockquote>

#### (2) DOM 節點
DOM tree的樹狀裡每一個部分叫做「節點 (node)」，節點有四種類型：

* 元素節點 (element node)
代表 HTML 標籤，例如 `<html>、<body>、<h1>、<p>`。
* 文字節點 (text node)
代表標籤內的文字內容，例如 `<h1>Hello World</h1>` 的 "Hello World" 就是一個文字節點。
* 屬性節點 (attribute node)
-- 代表 HTML 標籤的屬性，例如 `class="title"`。
-- 屬性不屬於 DOM 樹的節點，而是附加在元素節點上。
-- 可以用 `.getAttribute()` 和 `.setAttribute()` 來讀取或修改。
* 註解節點 (comment node)
代表 HTML 裡的 `<!-- 註解 -->` 內容

#### (3) 根節點：document
在 DOM 的樹狀結構裡，根節點是 document，代表網頁的本身。
在 DevTool 的 Elements 面板裡看到的內容就是 DOM 的結構，他不是 HTML 文件，而是 DOM 操作的結果，如果使用 JavaScript 改變 DOM 的狀態，瀏覽器顯示在 Elements 會同步更新。


## :memo: 瀏覽器也是物件模型
在 document 物件模型的上層還有 `window`，也就是「瀏覽器物件模型 (Browser Object Model)」，簡稱為 BOM。

* BOM 的根節點是 window，也就是視窗：

window 之下還有其他的子物件包括 screen、navigator、location、history、frames、document。剛才提到的 document 是其中一部分。
* window 是一個全域物件 (global object)，它不會受到 local scope 的限制。全域物件是可以省略不寫的，例如

`window.alert()` 等於 `alert()`
`window.document` 等於 `document`

## 小結
1. DOM Tree 是 HTML 的樹狀結構，瀏覽器會解析 HTML 並建立 DOM 樹。

2. DOM 節點類型：
* 元素節點：HTML 標籤（如 `<div>`）
* 文字節點：標籤內的文字內容
* 屬性節點：標籤的屬性（如 `class="title"`）
* 文件節點：整個 document
* 註解節點：HTML 註解

關於操作DOM節點，詳見[另一篇筆記](https://hackmd.io/@yy933/S1O0CfjK1g)。


---

## 參考資料
* ALPHA CAMP Material


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>