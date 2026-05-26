---
title: "[CSS] 偽元素 vs 偽類 - 筆記"
pubDatetime: 2026-05-25T11:17:36.259Z
tags: ["cheatsheet","CSS"]
description: " Table of contents 在 CSS 中，「偽元素」與「偽類」都能幫助我們選取、操作一些 原本無法直接選取的..."
---

## Table of contents

在 CSS 中，「偽元素」與「偽類」都能幫助我們選取、操作一些 **原本無法直接選取的部分或狀態**。 
 

## 偽元素（Pseudo-elements）
### 定義： 
**偽元素**是畫面看起來好像有這個元素，實際上卻不屬於一個真的元素，打開 DevTools 無法檢查到它，因為偽元素不存在於 DOM 節點中，但偽元素卻如真的元素一樣可受 CSS 操控樣式。

### 語法：使用 `::`（雙冒號）
```css
p::first-line {
  font-weight: bold;
}
```
### 常見偽元素：
| 偽元素            | 功能說明                                     |
|-------------------|----------------------------------------------|
| `::before`        | 在元素內部的**最前面**插入內容               |
| `::after`         | 在元素內部的**最後面**插入內容               |
| `::first-line`    | 選取第一行文字（依瀏覽器顯示決定）           |
| `::first-letter`  | 選取第一個字母（常用於排版設計）             |
| `::placeholder`   | 選取 `<input>` 的 placeholder 樣式           |
### 範例
```css
.card::before {
  content: "🔥";
  margin-right: 4px;
}

```

## 偽類（Pseudo-classes）

### 定義
選取處於某種「狀態」或「條件」下的元素，例如滑鼠 hover、被點選的 input 等。

### 語法：使用 `:`（單冒號）
範例：
```css
a:hover {
  color: red;
}
```

### 常見偽類
| 偽類             | 功能說明                             |
|------------------|--------------------------------------|
| `:hover`         | 滑鼠移到元素上                       |
| `:active`        | 滑鼠點下去那一刻                     |
| `:focus`         | 表單元素被聚焦                       |
| `:checked`       | radio/checkbox 被選取               |
| `:first-child`   | 第一個子元素                         |
| `:last-child`    | 最後一個子元素                       |
| `:nth-child(n)`  | 第 n 個子元素                        |
| `:not(selector)` | 選取不是某個條件的元素               |


:::warning
* 單數的子元素 `p:nth-child(odd)` 也可以寫作 `p:nth-child(2n+1)`
* 雙數的子元素 `p:nth-child(even)` 也可以寫作 `p:nth-child(2n)` 
:::

範例：
```css
input:focus {
  border-color: blue;
}
li:nth-child(odd) {
  background: #f2f2f2;
}
```


## 小筆記
* CSS2 可以用單冒號寫偽元素（像 `:before`），但 CSS3 建議用雙冒號 來區分語意。
* 偽元素不是真實存在於 DOM 的元素（JS 抓不到），但可以用來 製作裝飾性內容、動畫、標籤等小工具。
* 偽類則很適合做互動效果、表單變化、列表選取等。



::: success
:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！
:::