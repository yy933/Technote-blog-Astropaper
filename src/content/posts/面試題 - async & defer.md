---
title: "面試題 - async & defer"
pubDatetime: 2026-05-25T11:17:37.387Z
tags: ["Interview Preparation","browser","DOM","HTML"]
description: " Q1. <script 在HTML中的位置放在<body最底部，請問：在瀏覽器渲染流程中，會在什麼時機點載入JavaS..."
---

## Q1.` <script>` 在HTML中的位置放在`<body>`最底部，請問：在瀏覽器渲染流程中，會在什麼時機點載入JavaScript 程式碼？為什麼需要這樣做？
A1. 瀏覽器解析HTML的順序是由上而下，當遇到一個`<script>` 需載入外部的JavaScript檔案時，此時瀏覽器會暫停解析下方的HTML內容，直到JS檔案解析完成，這種情況會導致頁面停滯，造成不好的使用者體驗。因此，將`<script>` 放在`</body>`之前可以讓所有靜態HTML內容解析完畢後，再載入JS檔案，避免頁面阻塞，帶來良好的使用者體驗。

## Q2. 若要在 DOM 生成的同時，一併載入 JavaScript，針對`<script>`還有什麼處理方式？

A2. 將 `<script>` 放在`<body>`最底部也有缺點，特別是當script和stylesheet檔案很大時，需要花很久的時間等待腳本載入，導致使用者一開始看到的畫面缺乏動態功能，同樣也會造成不好的使用者體驗。如果想要「在 DOM 生成的同時，一併載入 JavaScript」，現代的瀏覽器支援在`<script>`中加入**async**或**defer**的屬性，讓瀏覽器知道可以在生成DOM的同時，載入腳本。
### async

```htmlembedded
<script src="script1.js" async></script>
<script src="script2.js" async></script>
```

async也就是非同步，這告訴瀏覽器可以在解析HTML內容時，不用等腳本載入。
* 載入帶有async屬性的腳本和載入HTML是彼此獨立的，腳本下載完成後就會立即執行。
* 其他也帶有async屬性的腳本的載入也是彼此獨立，上面的例子中，script1或script2哪個先下載完就先執行哪個
* 在async的情況下，雖然腳本下載時不影響HTML的解析，但執行腳本時會暫停HTML解析。

note: 目前[98.09%的瀏覽器](https://caniuse.com/script-async)都支援async屬性

### defer

```htmlembedded
<script src="script1.js" defer></script>
<script src="script2.js" defer></script>
```

defer屬性同樣會告訴瀏覽器解析HTML內容，不用等腳本載入。
與async不同處：
* 帶有defer屬性的腳本，會等待HTML內容解析完成才會執行。
* 若有其他也帶有defer屬性的腳本，則會依序執行，上述的例子中，會先執行script1再執行script2，因此若有相依的腳本，則可以使用defer。

note: 目前[98.23%的瀏覽器](https://caniuse.com/?search=Defer)支援defer屬性

同場加映：[超級清楚的defer & async圖解](https://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html)，保證馬上學會!

## 參考資料
* [<script> 的 async 與 defer 有什麼不同？](https://www.explainthis.io/zh-hant/interview-guides/frontend/fe-script-async-defer-difference)
* [<script> 標籤應該放在 HTML 的什麼位置？<link> 呢？](https://www.explainthis.io/zh-hant/interview-guides/frontend/script-link-in-html)
* [async vs defer attributes](https://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html)
* [Efficiently load JavaScript with defer and async](https://flaviocopes.com/javascript-async-defer/)
* [Where should I put <script> tags in HTML markup?](https://stackoverflow.com/questions/436411/where-should-i-put-script-tags-in-html-markup)
                                                     
::: success
:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！
:::