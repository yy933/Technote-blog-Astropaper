---
title: "利用JavaScript操作DOM Tree - 筆記"
pubDatetime: 2025-02-13T03:26:18.000Z
modDatetime: 2026-05-25T10:04:23.857Z
tags: ["browser","DOM","Interview Preparation"]
description: "Table of contents :memo: 選取節點 querySelector & querySelector..."
hackmd_id: "S1O0CfjK1g"
---

## Table of contents

## :memo: 選取節點


### querySelector & querySelectorAll
1. querySelector：只回傳第一個匹配的元素

```javascript
document.querySelector(CSS-selector)
```
例如:
```html
<h1>Title</h1>
<p>Paragraph 1</p>
<p>Paragraph 2</p>
```
選取方式:
```javascript
// 選擇第一段
let firstParagraph = document.querySelector("p");
```
2. querySelectorAll：回傳所有匹配的元素
```
document.querySelectorAll(CSS-selector)
```
選取方式:
```javascript
// 選擇所有段落
let allParagraphs = document.querySelectorAll("p");
```
會回傳一個類似陣列的 NodeList，是「類似陣列」，和 JavaScript 裡的陣列不一樣，不能新增刪除元素，只有幾個簡單的唯讀操作：

* 查看長度 length
* 遍歷內容 forEach
* 使用 index 來存取特定項目

### 早期語法：getElementBy* 家族
目前比較常使用的是`document.getElementById`：
```
document.getElementById('my-recipe')
// 類似 document.querySelector('#my-recipe')
```

## :memo: 查詢節點裡面的內容
### 選出節點的內容：innerText、textContent、innerHTML

```
document.querySelectorAll(CSS-selector).innerText
document.querySelectorAll(CSS-selector).textContent
document.querySelectorAll(CSS-selector).innerHTML
```

`innerText` 和 `textContent` 的功能一模一樣，只能處理文字，而 `innerHTML` 可以存取到 HTML 結構。

### 選出 HTML 屬性的值
```
document.querySelector(CSS-selector).attribute
// 例如:選取圖片屬性
document.querySelector('.my-card img').src
```
### 選出重覆結構裡的某項目
```html
<h1>Title</h1>
<ul class="contents">
  <li>item1</li>
  <li>item2</li>  
</ul>

```
選取方式:
```javascript
// 選擇列表中所有項目
document.querySelectorAll(".contents li");
// 選擇第一項 (從類似陣列的NodeLists中選擇)
document.querySelectorAll(".contents li")[0];
```



## :memo: 新增、刪除節點與操作CSS屬性

### 新增節點
```javascript
// 建立新的 <div> 元素
let newDiv = document.createElement("div");
```
### 抽換節點內容
```javascript
// 會解析 HTML 標籤
NODE.innerHTML = "htmlContent" 
// 不會解析 HTML 標籤（只處理文字）
NODE.innerText = "textContent" 
// 上面的範例，在newDiv插入內容
newDiv.textContent = "這是一個新的區塊";
```
### 將節點插入 DOM Tree：appendChild、insertBefore、replaceChild
![pasted_image_0__11_](https://hackmd.io/_uploads/Sycs-IjF1x.png)
例如以下HTML：
```html
<div class="container">
<!-- display data here -->
</div>
<script src="main.js"></script>
```
插入元素：
```javascript
const container = document.querySelector('.container')
const h1 = document.createElement('h1')
h1.innerHTML = 'This sentence is created by JavaScript'
container.appendChild(h1)
```


<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

📌 Modern Style：
現代的 JavaScript 有推出一套新的語法，試圖簡化選取節點的流程，如以下所示：
![image_1](https://hackmd.io/_uploads/BJbKfLsKJg.png)
這套語法的撰寫風格比較簡潔，而且同時可以插入節點或文字。可惜**在 IE 瀏覽器上尚未支援，目前仍不普及。**

</blockquote>

### 刪除節點
```javascript
parentElement.removeChild(NODE)
NODE.remove()
```
### 操作CSS屬性
#### Class
* `NODE.classList` - 查看目前所有 class 名稱，會回傳類似陣列的清單
* `NODE.classList.add(className1, className2)` - 加入一個或多個樣式
* `NODE.classList.remove(className1, className2)` - 刪除樣式
* `NODE.className = className` - 如果樣式只有一個，也可以直接寫入
#### 讀取與修改屬性
```javascript
let img = document.querySelector("img");
// 讀取屬性
console.log(img.getAttribute("src"));
// 修改屬性
img.setAttribute("alt", "新的描述");
```
#### Style 
* `NODE.style.backgroundColor`
* `NODE.style.borderStyle`
* ....
JS也可以修改DOM節點的style，但為了避免code太亂，還是先寫好CSS再透過`classList`操作是比較好的方式。
## 小結
JavaScript 可以操作 DOM：
* `document.querySelector()` 選取元素
* `.textContent` 修改內容
* `.appendChild()` 新增節點
* `.removeChild()` 刪除節點

關於DOM節點介紹，詳見[另一篇筆記](/posts/瀏覽器渲染流程與dom---筆記/)。


---

## 參考資料
* ALPHA CAMP Material


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>