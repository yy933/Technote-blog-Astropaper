---
title: "[CSS] Grid 佈局系統 - 筆記"
pubDatetime: 2025-04-13T02:10:02.000Z
tags: ["CSS","Interview Preparation"]
description: " Table of contents CSS Grid 是一套強大的二維布局系統，可以同時處理 行（row）與列（col..."
---

## Table of contents


CSS Grid 是一套強大的二維布局系統，可以同時處理 **行（row）與列（column）**，適合用來製作整體版面結構或複雜的卡片排列。

## Flexbox vs Grid
* **Flexbox 用於一維排版**，如果已經有了現成的內容，想要微調他們的間距或對齊方式 (所謂 content-out 排版)，適合用 flexbox；
* **Grid 可以同時操作欄與列兩個維度**，如果先有排版設計，打算從零開始把元素放進去 (所謂 layout-in 排版)，適合使用 CSS grid。

## 宣告 Grid 元素

### 定義 Grid Container
```html
<div class="container"> 
  <div class="item1"></div>
  <div class="item2"></div>
  <div class="item3"></div>
  <div class="item4"></div>
  <div class="item5"></div>
  <div class="item6"></div>
</div>
```

```css
.container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr; /* 三欄，每欄等寬，也可以寫成repeat(3, 1fr); /* 四等分 */ */
  grid-template-rows: auto auto;      /* 兩列，高度由內容決定 */
  gap: 16px 5px;                      /* 行間距16px，列間距5px */
}
```

## 操作 item 的配置
![image_7](https://hackmd.io/_uploads/B1bpEVLCyx.png)
item1 「從第 1 條線橫跨到第 4 條線」：

```css
.item1 {
  grid-column-start: 1;
  grid-column-end: 4;
}
```

也可以直接用 grid-column 縮寫：

```css
.item1 {
  grid-column: 1 / 4;
}
```
跨整列的item (例如banner)也可這樣寫：
```css
.banner {
  grid-column: 1 / -1; /* 跨整列 */
}
```
<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
在 Grid 裡，每條欄線（grid line）都有編號：

* 正數是從 左到右 編號（1、2、3...）
* 負數是從 右到左 倒數編號（-1 是最右邊）

所以 grid-column: 1 / -1 就等於：
👉「從第一條線開始，到最右邊的那條線結束」
</div>


第 3 列的高度改變了，這是因為` .item1 `佔據了整個第一列，把 `.item5` 和 `.item6` 推向第三列。因此，需要在 `'grid-template-rows'` 屬性內給新的這行指定高度 75px 。

```css
.wrapper {
  display: grid;
  grid-template-columns: 100px 100px 100px;
  grid-template-rows: 75px 75px 75px
  gap: 16px 5px; 
}
```

## 常見單位 
| 單位       | 意義                         |
|------------|------------------------------|
| `fr`       | 區域分配比例（fraction）      |
| `px` / `em` | 固定寬度                     |
| `%`        | 相對父層寬度                 |
| `auto`     | 根據內容自動決定             |



## Grid小技巧
| 技巧名稱              | 說明與範例                                      |
|-----------------------|-------------------------------------------------|
| `repeat()`            | 減少重複，例如 `repeat(12, 1fr)`               |
| `minmax()`            | 自動調整範圍，例如 `minmax(200px, 1fr)`        |
| `auto-fit` / `auto-fill` | 配合 `repeat()` 自動填滿欄位數              |
| `place-items`         | 快速置中，等於 `align-items + justify-items`   |

```css
.container {
  place-items: center;
}
```
### `grid-auto-flow`
控制當沒有宣告子元素要被擺在網格的特定位置時，子元素將根據特定的流向被自動擺放到網格當中。`grid-auto-flow` 的預設值為 `row` ，意思是子元素將逐列被擺放到網格中，如果放滿了就往下一列開始擺。
#### 語法
`grid-auto-flow: row | column | row dense | column dense;`

#### 範例
HTML:
```html
<div class="container">
  <div>1</div>
  <div>2</div>
  <div>3</div>
  <div>4</div> <!-- 自動落到下一行 -->
</div>
```
CSS:
```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  grid-auto-rows: 100px; /* 定義自動產生的格子高度 */
  grid-auto-flow: row; /* 預設值是row */
}
```

上述CSS中，只規定了三個column，而`grid-auto-flow`的值是`row`，因此當三個column擺滿之後，就會自動換行，跳到下一個row。

如果只定義了一個column：
![ExportedContentImage_04](https://hackmd.io/_uploads/ryr9FZK0ke.png)

如果`grid-auto-flow: column`：
![ExportedContentImage_05](https://hackmd.io/_uploads/SyLl5bKC1l.png)

只定義了兩個row，當兩行擺滿時，新增一個item，就會自動新增一個column：
![ExportedContentImage_06 (1)](https://hackmd.io/_uploads/BJWUqWK0Jg.png)

如果只定義一個row，就會產生類似一行的排列效果：
![ExportedContentImage_07](https://hackmd.io/_uploads/r1Y9qZKA1x.png)

#### 注意事項：
* `grid-auto-flow` 只影響「沒有手動指定 `grid-column` 或 `grid-row`」的元素。
* `dense` 模式會重新排列順序，不建議用在有語意順序需求的內容（例如文章段落）。


### 直覺好用的`grid-template-areas`
用「文字圖案」的方式定義區塊位置。非常直覺好懂，像在畫格子圖！
直接看範例：
![ExportedContentImage_02](https://hackmd.io/_uploads/SJy6iWFCkg.png)
HTML:
```html
<div class="container">
  <div class="hen"></div>
  <div class="wheat"></div>
  <div class="carrot"></div>
</div>
```
CSS:
```css
.container {
  /* 宣告grid容器 */
  display: grid;
  /* 畫格子 */
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  /* 用名字排位子 */
  grid-template-areas:
    'hen hen hen'
    'wheat . carrot'
    'wheat . carrot';
}
/* 為grid items取名 */
.hen {
  grid-area: hen;
}
.wheat {
  grid-area: wheat;
}
.carrot {
  grid-area: carrot;
}
```

#### 使用規則
| 規則             | 說明                                                                 |
|------------------|----------------------------------------------------------------------|
| 區塊名稱要一致   | `grid-area` 與 `grid-template-areas` 中的名稱必須一致                 |
| 區塊要形成矩形   | 區塊名稱在排版中必須形成 **完整矩形**（不能斜斜的或缺角）              |
| 空格可以用句點 `.` | 表示這個格子不屬於任何命名區塊                                        |
| 可讀性高         | 非常適合初學者與大型網頁排版設計，閱讀起來像「排版地圖」               |




<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>