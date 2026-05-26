---
title: "BEM 命名原則與 Sass 搭配 - 筆記"
pubDatetime: 2026-05-26T03:01:46.838Z
tags: ["Sass","CSS"]
description: " Table of contents 主流命名分為三種： OOCSS (Object oriented CSS) ：物件..."
---

## Table of contents

主流命名分為三種：

* **OOCSS (Object oriented CSS)** ：物件導向的命名法，著名的 Bootstrap 便是使用這套命名規則，當你看到 .btn 就知道這會是一個按鈕的樣式；看到 d-flex 就知道這邊是以 flex 做排版等等
* **SMACSS (Scalable and Modular Architecture for CSS)** ：明確地透過結構模組化來命名，諸如 Base、Layout、Module、State、Theme
* **BEM (Block Element Modifier)** ：BEM 命名法的最大優勢就是他人在閱讀你的 HTML 程式碼時，光是夠**透過觀察 class 名稱，就能了解你的 CSS 階層架構**，例如看到 list__item--hover 就知道這是滑鼠滑過表單子項目時的樣式。

## BEM 命名法（Block, Element, Modifier）
BEM 是一種讓 class 命名有邏輯、結構清晰的規範，特別適合模組化 CSS（Sass / SCSS）開發。

### 命名結構
#### Block（區塊）
* 定義：
指在 Web 開發中的模組，每個 Block 在邏輯和功能上都應該是互相獨立的。獨立的、功能完整的組件，可重用於不同頁面（例如按鈕、導航列、表單）。
* 命名規則：
使用小寫字母，單詞間以連字符（-）分隔。
描述功能或組件名稱，避免過於具體的視覺描述。
範例：.header, .button, .card, .menu
#### Element（元素）
* 定義：Block 的子組件，依賴於 Block，無法獨立存在（例如按鈕中的圖標、表單中的輸入框）。
* 格式：block__element（兩個下劃線 __ 連接 Block 與 Element）。
元素名稱同樣使用小寫，單詞間以連字符（-）分隔。
* 範例：.header__logo, .button__icon, .card__title
#### Modifier（修飾符）
* 定義：用於表示 Block 或 Element 的不同狀態、外觀或行為（例如按鈕的禁用狀態、尺寸變化）。
* 格式：block--modifier 或 block__element--modifier（兩個破折號 -- 連接）。
* 修飾符名稱使用小寫，單詞間以連字符（-）分隔。
* 範例：.button--disabled, .card--large, .header__link--active



```
.block {}              → 區塊：一個功能完整的元件
.block__element {}     → 元素：區塊內的子項目
.block--modifier {}    → 狀態/變化：同區塊不同樣式版本
```

### 範例
* 範例一

```html
<section class="card card--highlight">
  <h2 class="card__title">標題</h2>
  <p class="card__description">這是一張卡片</p>
</section>
```

```sass
.card {
  padding: 1rem;
  border: 1px solid #ccc;
}

.card__title {
  font-size: 1.5rem;
}

.card__description {
  color: #555;
}

.card--highlight {
  border-color: gold;
  background-color: #fffbe6;
}
```

* 範例二
```html
<form class="order-form">
  <input class="order-form__name-input">
  <input class="order-form__name-input--disabled" disabled>
</form>
```
BEMCSS 會產生相當長的類別名稱，在 Sass 中，**可以透過 & 符號來使得代碼更加簡潔**。
![ExportedContentImage_00 (2)](https://hackmd.io/_uploads/H1Mie8CC1e.png)

```sass
.card {
  // Block

  &__title {
    font-size: 1.5rem;
  }

  &--highlight {
    background-color: yellow;
  }
}
```


## 命名規則速記

| 名稱結構        | 用途    | 範例               |
|-----------------|---------|--------------------|
| `.btn`          | Block   | `.header`, `.menu` |
| `.btn__icon`    | Element | `.menu__item`      |
| `.btn--large`   | Modifier| `.header--dark`    |