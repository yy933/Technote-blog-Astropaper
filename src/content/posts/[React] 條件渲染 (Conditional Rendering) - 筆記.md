---
title: "[React] 條件渲染 (Conditional Rendering) - 筆記"
pubDatetime: 2026-05-26T03:01:46.985Z
tags: ["JavaScript","React.js"]
description: " Table of contents 在 Component 組成的網頁裡，Component 會需要根據不同條件顯示不..."
---

## Table of contents

在 Component 組成的網頁裡，Component 會需要根據不同條件顯示不同內容。在 React 的實作中，可以在特定的狀況下，使用 JavaScript 語法來決定最終呈現的 JSX，讓元件呈現預期的內容。

## 透過 if/else 條件式決定畫面渲染

假設有個元件如下:
```jsx
function Item({ name, isPacked }) {
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

想要幫`isPacked={true}`加上✅:
```jsx
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

完整程式碼:
```jsx
function Item({ name, isPacked }) {
  if (isPacked) {
    return <li className="item">{name} ✅</li>;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

但是這樣寫似乎很多重複的code，不符合DRY原則。這時就可以使用使用三元運算子` ? :`如下範例:
### 使用三元運算子` ? :`
```jsx
return (
  <li className="item">
    {isPacked ? name + ' ✅' : name}
  </li>
);
```
用法就和在JavaScript中一樣。也可進一步放入HTML tag包裝成：
```jsx
{isPacked ? <del>{name} ✅</del> : name}
```
:::warning
這種寫法的:
✅ 優點：語法簡潔，適合簡單條件。
⚠️ 注意：如果條件變多或內容複雜，JSX 會變得難讀，這時建議：
* 把內容抽成變數
* 包成子component
:::
:::success
**⚠️ 以上兩種寫法相同嗎？**
JSX 元素不是 OOP 中的實例（instance）！
若學過物件導向程式語言（ object-oriented programming ），可能會以為` <li> `的兩種寫法會產生不同的「實例」（instance）。其實不是，JSX 元素只是輕量描述，像是一張藍圖，**不是真實 DOM 或有內部狀態的實例。**
舉例來說，以上兩種寫法:
```jsx
// 寫法 1：
<li>{isPacked ? <del>{name}</del> : name}</li>

// 寫法 2：
{isPacked ? <li><del>{name}</del></li> : <li>{name}</li>}
```
可能會直覺以為：「我寫了兩個 `<li>`，是不是等於創造了兩個實例（instance）？會不會導致 React 多渲染？」答案是不會。

**:bulb: JSX 實際上只是 JavaScript 的語法糖，它會被轉換成像這樣的程式碼：**
```jsx
React.createElement("li", null, isPacked ? React.createElement("del", null, name) : name)
```
這種轉換出來的東西**只是一份描述（也稱為 virtual DOM blueprint），它不是一個真實 DOM，也不是 class 實例**。
React 不會因為在 JSX 中「寫了兩個 `<li>`」，就真的建立兩個 `<li>` DOM 節點。
它會聰明地比對這些 blueprint（即 JSX 結果），判斷哪些需要更新、哪些可以重用。
:::

## 回傳 null 來不顯示內容
```jsx
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```
> ⚠️ 不常見，較推薦在父層用條件判斷是否要渲染這個 component。較常見的做法是用`&&`只渲染條件為 true 的情況，如下方的說明。

## 使用 && 運算子

```jsx
return (
  <li>
    {name} {isPacked && '✅'}
  </li>
);

```
**:bulb: 以上寫法可以理解為：如果isPacked為true，顯示後方內容(✅)；為false則什麼都不顯示。**
:::warning
❗注意：不要把數字放左側，例如 `0 && <p>New</p>` 會渲染出 `0`！
* JavaScript 的 `&&` 運算會回傳第一個 falsy 值（如 0）或最後一個 truthy 值。
* 所以 `messageCount && <p>New</p>`，當 `messageCount = 0 `時，其實會回傳 0，React 就會真的渲染出「0」。
* 正確寫法應該是用布林判斷：`messageCount > 0 && <p>New</p>`。
:::

## 將 JSX 存進變數
在條件越來越複雜時，可以避免 `<li>{條件 ? xxx : yyy}</li>` 這類難讀的巢狀寫法，提升可讀性。

### 重點整理
* 先用 let 宣告一個變數，例如 itemContent，給它一個預設值。
* 用 if 條件式改寫變數的值（也可以是 JSX 表達式）。
* 在 JSX 裡用 {} 引用變數，讓邏輯與標記清楚分開。

### 範例
```jsx
let itemContent = name;

if (isPacked) {
  itemContent = name + " ✅";
}

return (
  <li className="item">
    {itemContent}
  </li>
);
```

比起直接把邏輯塞進 JSX，更清楚易懂，很適合條件較多、判斷較複雜的情境。
變數 `itemContent` 可以放字串、JSX 元素，甚至函式結果，靈活又直觀。例如以下把JSX放進`itemContent`的範例:

```jsx
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = (
      <del>
        {name + " ✅"}
      </del>
    );
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

## Recap
| 條件渲染方式             | 語法範例                            | 行為說明                                                  | 備註                         |
|------------------------|-------------------------------------|----------------------------------------------------------|----------------------------|
| `if` 條件判斷           | `if (cond) return <A />`            | 如果條件為 true，回傳 `<A />`                            | 適合較複雜邏輯               |
| 變數儲存 JSX            | `let content = <A />`<br/>`return {content}` | 先把 JSX 存進變數，再插入 JSX 結構中                        | 易讀性高                     |
| 三元運算子              | `{cond ? <A /> : <B />}`            | 如果條件為 true，渲染 `<A />`，否則渲染 `<B />`           | 適合簡單切換                 |
| 邏輯與運算子 (`&&`)     | `{cond && <A />}`                   | 如果條件為 true，渲染 `<A />`；false 時顯示 nothing       | ⚠️ `0 && <A />` 會渲染 `0` |

:bulb: 以上簡寫很常見，但複雜的情況也可以直接用`if`判斷式:
```jsx
let content = name;
if (isPacked) {
  content += " ✅";
}
return <li>{content}</li>;
```
不同情況的渲染：
```jsx
function Greeting(props) {
  let content;
  
  if (props.isLoggedIn) {
    content = <p>Welcome back!</p>;
  } else {
    content = <p>Please log in.</p>;
  }

  return <div>{content}</div>;
}
```
不需要寫else：
```jsx
function Greeting(props) {
  if (!props.isLoggedIn) {
    return <p>Please log in.</p>;
  }
  return <p>Welcome back!</p>;
}
```
用變數與 `if` 組合，易除錯、結構更清楚。