---
title: "React 語法糖：JSX - 筆記"
pubDatetime: 2026-05-26T03:29:26.452Z
tags: ["JavaScript","React.js"]
description: " Table of contents JSX 是什麼？ 🧠 JSX: 把 Markup 寫進 JavaScript！ ..."
---

## Table of contents



## JSX 是什麼？
**🧠 JSX: 把 Markup 寫進 JavaScript！**
* 過去：HTML、CSS、JavaScript 各自分工
* 現在：互動性提高 → 邏輯掌控內容
* React 作法：將 邏輯與標記 整合在一起 → Component
* React 的設計哲學：
  - 同步性：標記和邏輯一起寫，修改時更容易保持一致。
  - 維護性：每個小元件自己負責自己的外觀和行為。
  - 組件化：將功能拆成小元件（Component），方便重用和管理。
* JSX是語法糖，比起原生React，JSX更直覺、可讀性高、開發效率更好。例如：
```jsx
const element = <h1>Hello, world!</h1>;
```
以上jsx其實是：
```javascript
const element = React.createElement('h1', null, 'Hello, world!');
```
比起`React.createElement`，JSX更像 HTML，直觀易上手，並且在巢狀元件、條件渲染時，結構比 `createElement` 好讀。大部分 React 專案會透過 Vite、Next.js 或 Webpack 等工具，自動設定 Babel，把 JSX 轉譯成 `createElement`，無須手動處理。



每個 React 元件是：
* 一個 JavaScript 函式
* 使用 JSX 語法來描述要顯示的內容

> 🔔 JSX 是語法擴充 (syntax extension)，React 是函式庫！它們可以分開使用。
> 
![螢幕擷取畫面 2025-04-29 133556](https://hackmd.io/_uploads/BkpoE10ygx.png)




## 從 HTML 轉成 JSX
原本 HTML 長這樣：
```html
<h1>Hedy Lamarr's Todos</h1>
<img src="https://i.imgur.com/yXOvdOSs.jpg" alt="Hedy Lamarr" class="photo">
<ul>
  <li>Invent new traffic lights</li>
  <li>Rehearse a movie scene</li>
  <li>Improve the spectrum technology</li>
</ul>
```

在 JSX 中直接貼上會錯誤！
```nginx
Adjacent JSX elements must be wrapped in an enclosing tag.
```
<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
:bulb: JSX 中不能直接回傳多個並列的元素，必須**用一個「包裹元素」把它們包起來**。
**JSX 其實會被轉換成純 JavaScript 物件，而 JavaScript 函式不能直接回傳兩個物件**，必須用一個集合（像陣列或容器）包起來。

* 在純 HTML 裡，可以隨便列很多` <h1>`, `<img>`, `<ul>`，不用特別包起來。
* 但在 React 的 JSX 中，**一個 component 的 return 只能回傳一個「單一根元素」**。
* 如果寫了像這樣：
```jsx
return (
  <h1>標題</h1>
  <img src="圖片網址" />
  <ul>...</ul>
);
```
JSX 會不知道該怎麼處理這「多個元素」，所以報錯。

用 `<div>`包起來：
```jsx
return (
  <div>
    <h1>標題</h1>
    <img src="圖片網址" />
    <ul>...</ul>
  </div>
);
```
或是用「Fragment」`<>...</>`，不增加多餘的標籤：
```jsx
return (
  <>
    <h1>標題</h1>
    <img src="圖片網址" />
    <ul>...</ul>
  </>
);
```
:pencil: `<>...</>`叫做Fragment，它不會在 HTML 結果裡產生額外的標籤，很輕量。
</div>

## JSX三個使用規則
### 1. 必須有一個單一根元素（Single Root Element）
用 `<div>` 或空的` <>...</>`（Fragment）包起來：
```jsx
<>
  <h1>Hedy Lamarr's Todos</h1>
  <img src="https://i.imgur.com/yXOvdOSs.jpg" alt="Hedy Lamarr" className="photo" />
  <ul>
    <li>Invent new traffic lights</li>
    <li>Rehearse a movie scene</li>
    <li>Improve the spectrum technology</li>
  </ul>
</>
```

### 2.標籤一定要關閉
* 單標籤要加 /：`<img />`
* 成對標籤要完整寫出：`<li>內容</li>`

### 3. 屬性命名改用 camelCase
| HTML屬性  | JSX寫法     |
|-----------|-------------|
| `class`     | `className`    |
| `stroke-width` | `strokeWidth` |

```jsx
<img 
  src="https://i.imgur.com/yXOvdOSs.jpg" 
  alt="Hedy Lamarr" 
  className="photo"
/>
```
> :bulb: `aria-*` 和 `data-*` 例外，仍然用中線命名。
<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
:bulb: 在 JSX 中，class 不是一個有效的屬性名稱，因為：
* `class` 是 JavaScript 的 保留字（例如：`class MyComponent {}`），會造成語法解析問題。
* JSX 其實是被 轉譯成 `React.createElement()` 的 JavaScript 函式，那個函式期待的屬性名稱是對應 DOM 的 property name，而不是 HTML attribute。
> JSX 使用 JavaScript 的屬性命名（DOM property），而不是 HTML 的屬性命名。

因此JSX要寫:
```jsx
<div className="container"></div>
```


:bulb: 還有哪些 HTML 屬性在 JSX 中名稱不同？
| HTML 屬性     | JSX 對應（React DOM 屬性） | 說明                              |
|---------------|----------------------------|-----------------------------------|
| `class`       | `className`                | 如上所述，避免與 JS 保留字衝突     |
| `for`         | `htmlFor`                  | JS 中 `for` 是關鍵字                |
| `onclick`     | `onClick`                  | 所有事件處理器需轉為小駝峰式命名    |
| `tabindex`    | `tabIndex`                 | 使用小駝峰格式                    |
| `maxlength`   | `maxLength`                | 也是 DOM property 的命名方式      |
| `readonly`    | `readOnly`                 | 注意 O 大寫                       |
| `contenteditable` | `contentEditable`     | 同樣是轉為 camelCase              |
| `autoplay`    | `autoPlay`                 | 注意自動播放常見於 `<video>`      |
| `crossorigin` | `crossOrigin`              | 常見於 `<img>`、`<video>`         |

✅ 特殊例外：`data-*` 與 `aria-*` 保持原樣，不需要轉換。
```jsx
<div data-user-id="123" aria-label="close button"></div>
```

</div>

## 工具推薦：JSX 轉換器
大量 HTML 要轉成 JSX？
→ 可以用 [JSX Converter](https://transform.tools/html-to-jsx) 幫忙，但還是要理解基本規則！

## Recap

* React 把邏輯和標記放在一起是有原因的: 
React 把「邏輯（JavaScript）」和「標記（HTML-like 的 JSX）」寫在一起，主要是因為現代網頁互動性大幅提升，內容已經被邏輯控制了，而不是單純靜態地寫在 HTML 裡。
* JSX ≈ HTML，但規則更嚴格 
* 常見錯誤訊息會引導你修正問題