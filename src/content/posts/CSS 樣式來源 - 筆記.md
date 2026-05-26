---
title: "CSS 樣式來源 - 筆記"
pubDatetime: 2026-05-26T03:29:26.375Z
tags: ["CSS","Interview Preparation","browser"]
description: " Table of contents CSS ：Cascading Style Sheets CSS 是 Cascadi..."
---

## Table of contents


 

## CSS ：Cascading Style Sheets
CSS 是 Cascading Style Sheets 的縮寫，style sheet 直譯是「樣式表」，而「Cascading」是 CSS 中的核心演算法，中文會翻成「層疊、級聯、串連」。
想像每一層都是一張樣式表，當有**多個來源都對同一個元素指定樣式的時候，就需要靠 Cascading 演算法，來判斷要套用哪個屬性到 HTML 中的目標元素上**，呈現最終畫面。

在 [W3C 的規範下](https://www.w3.org/TR/CSS22/cascade.html#cascade)，style sheets 的來源可以分為三種（優先順序從最低到最高）：

* User agent origin：由「瀏覽器廠商」制定 ⬅️ 最弱
* User origin：由「瀏覽器使用者」制定。
* Author origin：由「網頁開發者」制定 ⬅️ 最優先，我們寫的 CSS 


## User Agent Origin
此層級是「瀏覽器廠商」定義的樣式，而重置樣式的工具如 Reset CSS、Normalize CSS 等，就是用來解決這一層級中，不同廠牌瀏覽器下造成的預設樣式偏差問題，讓開發者得以在一致的基礎上，建構網頁 CSS 樣式設計。

## User Origin
此層級是由「瀏覽器使用者」定義的樣式，可以覆蓋掉 user agent origin 的樣式。

## Author Origin
此層級由「網頁開發者」制定，**優先級最高，可以覆蓋掉上述兩個來源的定義**。而載入 CSS 樣式到 HTML 的方法有很多種，**優先順序的大原則是「越靠近 HTML 標籤的，優先級越高」**。

例如此圖中，div 的背景色會是紅色，優先順序如下：
![ExportedContentImage_06](https://hackmd.io/_uploads/Hkd0DXrRJg.png)

<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
📌 當使用者自行設定的樣式加上 `!important`，**優先度最高**，開發者加多少 `!important` 都擋不住！因為 CSS 本質上設計就是「尊重使用者需求」。

| 優先順序（從低到高）         | 是否加 `!important` | 備註說明                                |
|------------------------------|---------------------|-----------------------------------------|
| 1. User Agent（瀏覽器）      | 無                  | ex: 預設 `<h1>` 是粗體                   |
| 2. Author（開發者寫的）      | 無                  | 我們平常寫的樣式                         |
| 3. User（使用者自定義）      | 無                  | 使用者在瀏覽器或 reader mode 設定         |
| 4. Author（開發者）          | `!important`        | ex: `color: red !important;`           |
| 5. User（使用者）            | `!important`        | ex: 使用者寫了 `!important` 也擋不住！   |
</div>


## CSS 選擇器與優先級
在談論 CSS 優先級問題時，我們會把選擇器分成以下三個等級：

* 等級一：以 ID 選擇器為代表，例如#article。
* 等級二：以 Class 選擇器為代表，另外包括屬性選擇器、偽類選擇器，例如 .container 、[type="radio"]、:hover。
* 等級三：以 Element 選擇器為代表，另外包括偽元素選擇器，例如 div、::before。

可以用選擇器計算機計算優先順序：[Specificity Calculator](https://specificity.keegan.st/)、[CSS Specificity calculator](https://polypane.app/css-specificity-calculator/#selector=)

## 其他注意事項

- 大部分情況，我們寫的 CSS 屬於**「Author」層級**。
- `!important` 可以提高優先度，但**不應濫用，會造成維護困難**。
- 使用者樣式存在是為了**輔助無障礙設計、暗模式等個人化需求，開發者應尊重**。
- **遵守內容和樣式分離**：盡量不要在 HTML 中另外撰寫` style` 區塊及使用 inline style ，除非你有個很好的理由，例如用 JavaScript 動態控制樣式變化的時候。
- **讓選擇器的優先級放低**：撰寫樣式時，盡量選用 `Class` 或 `Element` 的級別的選擇器，非必要不用 id 或複雜的選擇器組合，保留多一點彈性的空間給後續的樣式來覆蓋。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>