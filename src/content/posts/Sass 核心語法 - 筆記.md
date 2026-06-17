---
title: "Sass 核心語法 - 筆記"
pubDatetime: 2025-04-17T01:46:18.000Z
modDatetime: 2026-05-25T10:04:23.726Z
tags: ["cheatsheet","CSS","Sass"]
description: "Table of contents 變數 (variable) 與運算子 (operator) 變數可以使 css 代..."
hackmd_id: "B13g4BRA1g"
---

## Table of contents


 

## 變數 (variable) 與運算子 (operator)  
變數可以使 css 代碼具備更佳的可維護性與可讀性—— Sass 使用冒號 `:` 進行**變數的指派**，並以分號 `;` **作為結尾**。此外，還可以針對數值進行各種常見的計算。

範例:
```sass
$primary-color: #3498db;
$padding: 1rem;

.button {
  background-color: $primary-color;
  padding: $padding;
}
```
![ExportedContentImage_00 (1)](/images/r1NkBrR0ke.png)



## 巢狀（Nesting）  
使用巢狀語法來編寫**組合選擇符 (nested combinator)** ，例如空白以及 > + ~ ，以達到更好的可讀性。

範例:
```sass
.nav {
  ul {
    margin: 0;
    li {
      list-style: none;
    }
  }
}
```
![ExportedContentImage_01 (1)](/images/ByXIHBRAyl.png)

✅ 只建議巢狀 2~3 層內，避免過度複雜。



## @mixins＋ @include  
用來抽離重複樣式！在物件導向程式設計中，mixin 意指一種工具形式的類別，用以附加在目標類別之上，為目標類別增添額外的方法或屬性——你可以將它視作一種更彈性的繼承 (inherit) 的實作方式。在 Sass 裡，我們使用 @mixin 進行宣告，並使用 @include 使用 mixin。

### 範例一
```sass
/* 置中對齊 */
@mixin center {
  display: flex;
  justify-content: center;
  align-items: center;
}

.box {
  @include center;
}
```
效果是` .box` 元素會變成 `flex` 容器，內容完全置中。這段程式碼可重複用在其他地方！


### 範例二:帶參數的 mixin
```sass
@mixin font($size, $weight: normal) {
  font-size: $size;
  font-weight: $weight;
}

.title {
  @include font(24px, bold);
}

.subtitle {
  @include font(18px); // 第二個參數預設為 normal
}
```

![ExportedContentImage_04 (1)](/images/Sk26LBC0Je.png)


### 範例三：RWD（響應式 mixin）  
寫一個簡單的 media query 工具：
```sass
@mixin respond($breakpoint) {
  @if $breakpoint == sm {
    @media (max-width: 600px) { @content; }
  } @else if $breakpoint == md {
    @media (max-width: 900px) { @content; }
  }
}

// 使用方法
.card {
  padding: 2rem;

  @include respond(sm) {
    padding: 1rem;
  }
}
```
`@content` 表示可以在 `mixin` 裡包裹「一段內容」，讓語法變得超直覺！



## 檔案模組化  
在 Sass 中，我們可以將檔案拆成多個，配合上妥善的命名，讓專案變得更容易管理。  
![ExportedContentImage_02 (2)](/images/H1e3PrC0Jl.png)

* 開頭為底線` _ `的檔名在 Sass 中稱之為**partial**，也就是一個低階的模組。
* 我們使用 `@use` 來引用 partial。請注意**引用名稱不包含底線，也不包含副檔名**。
* 模組有自己的命名空間 (namespace) ，**使用模組中的變數時，必須給命名空間加上前綴**，例如 `base.$primary-color` 。

```sass
// 在 _variables.scss 中
$font-size: 16px;

// 在 main.scss 中
@use 'variables' as v;

body {
  font-size: v.$font-size;
}
```


## 繼承（`@extend`）與覆寫  
繼承可以使多個類別都享有共同的屬性—— Sass 使用 `%` 來**宣告類別**，並使用 `@extend` 來**執行繼承**。如果要覆寫屬性，則直接宣告即可。

直接看範例:  
![ExportedContentImage_03 (1)](/images/SyY5OrC0ye.png)

```sass
%card-base {
  border-radius: 8px;
  box-shadow: 0 0 10px #ccc;
}

.card {
  @extend %card-base;
}
```

## 函數（Functions）  
單位轉換或計算時很實用。
```sass
@function px-to-rem($px) {
  @return $px / 16 * 1rem;
}

.title {
  font-size: px-to-rem(24);
}
```


## 控制結構（`@if`, `@for`, `@each`, `@while`）
### `@if / @else`  
條件判斷，就像 JavaScript 的 `if...else`，用來根據變數或條件產生不同的樣式。
```sass
$theme: dark;

body {
  @if $theme == dark {
    background: #111;
  } @else {
    background: #fff;
  }
}
```

### `@for`  
循環指定次數（適合自動產生 class）  
有兩種語法：
* `from ... through ...`：包含結尾
* `from ... to ...`：不包含結尾
```sass
// 不包含 3
@for $i from 1 to 3 {
  .to-#{$i} {
    width: 10px * $i;
  }
}

// 包含 3
@for $i from 1 through 3 {
  .through-#{$i} {
    width: 10px * $i;
  }
}

```
產出：
```css
/* 不包含 3（只有 .to-1 和 .to-2） */
.to-1 { width: 10px; }
.to-2 { width: 20px; }

/* 包含 3（有 .through-1 到 .through-3） */
.through-1 { width: 10px; }
.through-2 { width: 20px; }
.through-3 { width: 30px; }

```

### `@each`  
Array or Map  
適合用來產生有顏色、類型、尺寸等變化的 class。

#### 語法（針對 list）：

```sass
@each $color in red, blue, green {
  .text-#{$color} {
    color: $color;
  }
}
```
 #### 語法（針對 map）：

```sass
$colors: (
  primary: #3498db,
  danger: #e74c3c,
  success: #2ecc71
);

@each $name, $hex in $colors {
  .bg-#{$name} {
    background-color: $hex;
  }
}
```

### `@while`：條件迴圈（少用）  
根據一個條件不斷重複，直到條件不成立為止。

#### 語法：
```sass
$i: 1;

@while $i <= 3 {
  .padding-#{$i} {
    padding: #{$i}rem;
  }
  $i: $i + 1;
}
```

#### 輸出：
```css
.padding-1 { padding: 1rem; }
.padding-2 { padding: 2rem; }
.padding-3 { padding: 3rem; }
```


## 比較：`@mixin` vs `@extend` vs `@function`

| 功能 | `@mixin` | `@extend` | `@function` |  
| --- | --- | --- | --- |  
| **用來做什麼？** | 封裝一段樣式，帶或不帶參數 | 繼承現有選擇器的樣式 | 回傳一個單一值（顏色、數字、單位） |  
| **使用方式** | `@include mixin-name()` | `@extend .class-name` or `%placeholder` | `property: function-name()` |  
| **可帶參數？** | ✅ 可以 | ❌ 不行 | ✅ 可以 |  
| **可用條件邏輯（if/each）？** | ✅ 可以 | ❌ 不行 | ✅ 可以 |  
| **會產生多份 CSS？** | ✅ 每次使用會複製樣式 | ❌ 共用同一份樣式 | ❌ 回傳單一值，不產生樣式 |  
| **維護性** | 高，彈性大，適合元件 | 中，寫法簡潔但耦合高 | 高，簡潔、明確，用於計算 |  
| **最佳使用場景** | 按鈕樣式、RWD 模組、陰影等 | reset、卡片樣式、標題樣式共用 | px→rem 轉換、顏色處理、比例計算 |

###  什麼時候用 @mixin
* 需要參數化樣式（像是顏色、間距、尺寸）
* 想抽出可重複使用的設計模組（按鈕、card、modal）
* 有多個地方會用類似樣式但又不完全一樣
* 有條件邏輯、媒體查詢（media query）
* 範例:
```sass
@mixin button($bg-color) {
  background-color: $bg-color;
  border-radius: 4px;
  padding: 0.5rem 1rem;
}

.btn-primary {
  @include button(blue);
}
```
### 什麼時候用 @extend
* 樣式幾乎完全相同，不需要傳參數
* 希望產生的 CSS 乾淨（但容易耦合）
* 建議使用 %placeholder selector，避免污染 HTML class。
* 缺點是：@extend 會把選擇器合併（可能導致意外耦合）
* 範例：
```sass
%card-style {
  border: 1px solid #ddd;
  padding: 1rem;
  border-radius: 8px;
}

.card, .profile, .popup {
  @extend %card-style;
}
```

### 什麼時候用 @function
* 要回傳一個計算後的值（不是整段樣式）
* 常用於單位轉換（px → rem）、顏色調整、比例縮放
* 範例：

```sass
@function px-to-rem($px) {
  @return $px / 16 * 1rem;
}

.title {
  font-size: px-to-rem(24);
}
```


### 總結一句話

> `@mixin` 是模組、`@extend` 是共用、`@function` 是算式。


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>