---
title: "[React] 概念回顧 Part1 (React元件、JSX) - 筆記"
pubDatetime: 2025-05-04T23:35:38.000Z
tags: ["JavaScript","React.js","Interview Preparation"]
description: " Table of contents 本篇回顧涵蓋的筆記範圍 [React 元件 筆記](https://hackmd...."
---

## Table of contents

## 本篇回顧涵蓋的筆記範圍
* [React 元件 - 筆記](https://hackmd.io/EsHsVOKJSha5FlGhCF9R5Q)
* [匯出與匯入 React 元件 - 筆記](https://hackmd.io/QWwQsBa_TSappy_M1QvY5A)
* [React 語法糖：JSX - 筆記](https://hackmd.io/50LZSK3yRyenSx9iSVAmaA)
* [在 JSX 中使用 JavaScript - 筆記](https://hackmd.io/4hkOiRLjQQulaCqdk1t6Rg)



## 「在 React 中，一切都是元件」代表著什麼？

在React中，元件（Component）：把 HTML、CSS 和 JavaScript 合在一起，**做成「可以重複使用」的 UI 元素。元件可以像積木一樣「組合」、「排序」、「巢狀」做成整個頁面。**

1. 元件就是函式
在 React 裡，一個元件本質上就是一個 JavaScript 函式，這個函式回傳要渲染的 UI（JSX）。
2. 可重用性
可以在不同地方重複使用同一個元件，就像 HTML 的` <button>` 一樣。
3. 組合性
元件可以巢狀（嵌套）其他元件，讓你用小元件組成大元件，大元件再組成整個頁面。
4. 邏輯與 UI 綁在一起
元件不只描述畫面長什麼樣，還可以包含邏輯、狀態（state）、事件處理等行為

:bulb: 「一切都是元件」= React 視 UI 為函式式元件的組合，這讓開發者能更簡潔、可維護、模組化地建構 UI。

:arrow_right: 參考 [React 元件 - 筆記](https://hackmd.io/EsHsVOKJSha5FlGhCF9R5Q)



## 可以將所有程式碼寫在根元件嗎？如果可以的話，優缺點是什麼？

當然可以把所有程式碼都寫在 React 的根元件（`App` 或 `Root`）裡，但不建議在實務上使用。

✅ 優點（但不多）：

* 快速上手、原型開發方便
初學者或做簡單小專案時，所有東西都寫在一個檔案中比較直觀，不需要思考「要不要拆元件」。
* 沒有檔案切割的複雜度
不用處理太多檔案引入、props 傳遞等問題，直接把畫面與邏輯寫好就能跑。

❌ 缺點：

* 難以維護與閱讀
當畫面變複雜時，所有 JSX 混在一起會變成一坨「巨型元件」，邏輯和 UI 混亂，找錯誤變得困難。
* 不利於重用
如果所有程式碼都塞在根元件裡，就不能在其他地方重用特定區塊，像是按鈕、表單、商品卡片等。
* 不利於測試與除錯
小元件可以單獨測試與排錯，根元件太大會很難除錯。
* 難以分工合作
團隊開發時，每個人都要改同一個檔案，容易衝突、難以協作。

:bulb: 小結論：可以寫在根元件，但不建議這樣做。模組化與拆元件，是維持 React 專案清晰與可維護的關鍵。

:arrow_right: 參考 [匯出與匯入 React 元件 - 筆記](https://hackmd.io/QWwQsBa_TSappy_M1QvY5A)


## JSX 語法與 HTML 語法兩者的關係是什麼？
* JSX 是 JavaScript 的語法擴充（不是 HTML）
JSX（JavaScript XML）是 React 自創的一種語法，讓我們可以在 JavaScript 中寫類似 HTML 的標記語法，但它其實會被轉換成 `React.createElement(...)` 這類 JS 語法。例如以下jsx：
```jsx
const element = <h1>Hello</h1>;
```
會被轉譯成：
```jsx
const element = React.createElement('h1', null, 'Hello');
```

* JSX 和 HTML 的差異

| 差異點             | HTML                            | JSX（React）                                  |
|------------------|---------------------------------|----------------------------------------------|
| 屬性寫法          | `class="btn"`                   | `className="btn"`（避免與 JS 關鍵字衝突）         |
| 結尾標籤          | 選擇性（如 `<input>`）            | 一定要完整（如 `<input />`）                    |
| JavaScript 內嵌  | 不支援                         | 用 `{}` 包住變數或運算式，如 `{name}`           |
| 陣列/條件渲染     | 不支援                         | JSX 支援 `{arr.map(...)}` 和 `{cond && ...}` 等語法 |
| 標籤命名          | 全部小寫                        | 自定義元件要大寫開頭，如 `<MyButton />`        |

:bulb: JSX 是長得像 HTML 的 JavaScript 語法糖，讓我們可以用寫 HTML 的方式描述 UI，但背後運作完全是 JavaScript。

:arrow_right: 參考 [在 JSX 中使用 JavaScript - 筆記](https://hackmd.io/4hkOiRLjQQulaCqdk1t6Rg)、[React 語法糖：JSX - 筆記](https://hackmd.io/50LZSK3yRyenSx9iSVAmaA)

## 為什麼 HTML 中的 class 屬性在 React element 中的對照 prop 會改名為 className ?
:bulb: 因為 class 是 JavaScript 的保留字，在 JSX（JavaScript XML）裡會造成語法衝突。

JSX 本質上是 JavaScript 的語法擴展，它會被編譯成 `React.createElement()` 語法，而不是直接是 HTML。

但在 JavaScript 中，class 是用來宣告類別的關鍵字（例如：`class MyComponent {}`），所以如果在 JSX 中寫：
```jsx
<div class="box">Hello</div>
```
編譯時會發生錯誤，因為 JavaScript 認不出這樣的語法。

* 類似的例子:

| HTML 屬性  | JSX 對應名 | 原因                  |
|------------|-------------|-----------------------|
| `class`    | `className` | 避免與 JS 的 `class` 衝突 |
| `for`      | `htmlFor`   | 避免與 JS 的 `for` 迴圈衝突 |

:arrow_right: 參考 [在 JSX 中使用 JavaScript - 筆記](https://hackmd.io/4hkOiRLjQQulaCqdk1t6Rg)、[React 語法糖：JSX - 筆記](https://hackmd.io/50LZSK3yRyenSx9iSVAmaA)

## inline CSS style 的屬性有什麼特別的撰寫規則嗎？舉例說明。
1. 使用物件語法（不是字串）
```jsx
<div style={{ color: 'red', fontSize: '20px' }}></div>
```
外層` {}` 是 JSX 表示「JavaScript 表達式」`<br>` ，內層 `{}` 是 JavaScript 的物件。

2. CSS 屬性名稱使用 camelCase（駝峰式命名）

* `background-color` → `backgroundColor`
* `font-size` → `fontSize`
* `text-align` → `textAlign`
```jsx
<div style={{ backgroundColor: 'blue', textAlign: 'center' }}></div>
```

3. 屬性值為字串（或變數）

數字不需要加單位的屬性可直接寫數字，例如 `lineHeight`, `zIndex`...

```jsx
<div style={{ lineHeight: 1.5, zIndex: 10 }}></div>
```

⚠️ 常見錯誤範例：

```jsx
// 錯誤：不是物件
<div style="color: red;"></div>

// 錯誤：CSS 名稱沒轉成 camelCase
<div style={{ 'font-size': '16px' }}></div>
```

:arrow_right: 參考 [在 JSX 中使用 JavaScript - 筆記](https://hackmd.io/4hkOiRLjQQulaCqdk1t6Rg#%E5%A6%82%E4%BD%95%E5%9C%A8-JSX-%E4%B8%AD%E5%82%B3%E5%85%A5%E7%89%A9%E4%BB%B6--%E4%BD%BF%E7%94%A8-inline-CSS-styles%E3%80%82)