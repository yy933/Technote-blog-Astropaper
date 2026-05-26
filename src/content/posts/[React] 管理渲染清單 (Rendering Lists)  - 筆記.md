---
title: "[React] 管理渲染清單 (Rendering Lists)  - 筆記"
pubDatetime: 2026-05-26T03:29:26.538Z
tags: ["JavaScript","React.js"]
description: " Table of contents 基本概念 常見需求：從一筆資料清單中渲染多個類似元件（如留言列表、圖庫等）。 使用..."
---

## Table of contents

## 基本概念
* 常見需求：從一筆資料清單中渲染多個類似元件（如留言列表、圖庫等）。
* 使用 `map()` 搭配 JSX 渲染每一筆資料。
* 可用 `filter()` 篩選特定條件的資料後再渲染。

## 透過 `map()` 渲染資料

```jsx
const people = [
  'Creola Katherine Johnson: mathematician',
  'Mario José Molina-Pasquel Henríquez: chemist',
  'Mohammad Abdus Salam: physicist',
  'Percy Lavon Julian: chemist',
  'Subrahmanyan Chandrasekhar: astrophysicist'
];

export default function List() {
  const listItems = people.map(person =>
    <li>{person}</li>
  );
  return <ul>{listItems}</ul>;
}
```

出現了一個Warning：
```
Warning: Each child in a list should have a unique “key” prop.
```
稍後會說明key。

## 透過 `filter()` 篩選後渲染資料
有資料如下：
```jsx
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
  accomplishment: 'spaceflight calculations',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
  accomplishment: 'discovery of Arctic ozone hole',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'physicist',
  accomplishment: 'electromagnetism theory',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'chemist',
  accomplishment: 'pioneering cortisone drugs, steroids and birth control pills',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
  accomplishment: 'white dwarf star mass calculations',
  imageId: 'lrWQx8l'
}];

```
篩選出職業是chemist的人，再用map產生陣列：
```jsx
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const chemists = people.filter(person =>
    person.profession === 'chemist'
  );
  const listItems = chemists.map(person =>
    <li>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        known for {person.accomplishment}
      </p>
    </li>
  );
  return <ul>{listItems}</ul>;
}
```
<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
:bulb: 補充：箭頭函式的隱式回傳（Implicit Return）
箭頭函式（`=>`）在 沒有大括號 `{}` 包住的情況下，會「自動回傳」等號右邊的表達式，不需要寫 return。

```jsx
const listItems = chemists.map(person =>
  <li>{person.name}</li> // 隱式回傳，省略 return
);
```

但只要用了大括號 `{}`，就會變成「區塊主體（block body）」，這時就必須明確寫出 return：

```
const listItems = chemists.map(person => {  // 區塊主體
  return <li>{person.name}</li>;            // 需要寫 return
});
```
如果你忘了寫 `return`，函式會默默回傳 `undefined`，導致 `<li>` 不會被渲染！

```jsx
const listItems = chemists.map(person => {
  <li>{person.name}</li>; // 沒有 return，什麼都不會渲染
});
```
✅ 什麼時候用哪種？
* 只有一行 JSX：可以用簡潔的隱式回傳，少打一些字。
* 有多行邏輯或條件判斷：用 `{} `和 `return `比較清楚。
</div>

## 重點概念：Key 的使用
為什麼要加 key？
* key 是 React 用來追蹤每一筆項目是否變動的依據。
* 有了 key，React 在渲染時可以更聰明地比對新舊項目，避免整個列表重渲染。
* 如果列表內容有排序、過濾、增刪等操作，沒有 key 或 key 選得不好，會導致畫面更新不正確或效能低落。有好的 key，React 可以只更新有改變的項目，而不是整個清空重建。

```jsx
const listItems = people.map(person => (
  <li key={person.id}>{person.name}</li>
));
```
<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
在 `map()` 裡渲染多個 JSX 元素時，每一個元素都必須有 `key`，否則 React 會發出警告。
</div>

**不建議用 index 當 key：**
```jsx
list.map((item, index) => <li key={index}>{item}</li>);
```
>雖然這可以讓警告消失，但不適用於會排序、插入或刪除的情境，可能會導致錯誤的 DOM 對應。

直接把key放在資料中：
`data.js`
```jsx
export const people = [{
  id: 0, // Used in JSX as a key
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
  accomplishment: 'spaceflight calculations',
  imageId: 'MK3eW3A'
}, {
  id: 1, // Used in JSX as a key
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
  accomplishment: 'discovery of Arctic ozone hole',
  imageId: 'mynHUSa'
}, {
  id: 2, // Used in JSX as a key
  name: 'Mohammad Abdus Salam',
  profession: 'physicist',
  accomplishment: 'electromagnetism theory',
  imageId: 'bE7W1ji'
}, {
  id: 3, // Used in JSX as a key
  name: 'Percy Lavon Julian',
  profession: 'chemist',
  accomplishment: 'pioneering cortisone drugs, steroids and birth control pills',
  imageId: 'IOjWm71'
}, {
  id: 4, // Used in JSX as a key
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
  accomplishment: 'white dwarf star mass calculations',
  imageId: 'lrWQx8l'
}];

```

### 顯示多個 DOM 節點的情境

當渲染的每個列表需要顯示多個 DOM 節點（例如：`<h1>` 和 `<p>`），如果直接將它們放在 `map()` 裡會遇到問題，因為 JSX 元素本身是需要包裹在單一父元素中的。

不建議這樣（Fragment 無法加 key）：
```jsx
people.map(person => (
  <>
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </>
));
```

正確寫法:
```jsx
import { Fragment } from 'react';

people.map(person => (
  <Fragment key={person.id}>
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </Fragment>
));
```

或是用 `<div>` 來包裹每一個列表內的多個 DOM 節點：

```jsx
const listItems = people.map(person =>
  <div key={person.id}>
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </div>
);
```

### 不同的資料來源提供不同的 key：
* 來自資料庫的資料：
如果你的資料來自資料庫，通常可以使用資料庫中的唯一鍵或 ID，這些本身就是唯一的。

* 本地生成的資料：
如果資料是本地生成的，可以使用遞增計數器、`crypto.randomUUID()` 或使用像是 uuid 這樣的套件來生成唯一的key。

### Key的規則
* 必須是唯一的：
每個兄弟元素的 key 必須不同，但不同數組中的 key 可以相同。

* 不能變動：
如果key改變就失去了它的功能。不要在渲染過程中動態生成 key。key的穩定性很重要。
>不要動態生成 key，例如 `key={Math.random()}`。這會導致每次渲染時 key 都不同，進而讓所有的元件和 DOM 元素重新渲染，不僅會影響性能，還會讓user input消失。

* 不會作為 prop 傳遞給元件
key 只是 React 用來識別元素用的，並不會作為 prop 傳遞。如果元件需要一個 ID，必須額外傳遞一個 prop，像這樣：
```jsx
<Profile key={id} userId={id} />
```

## Recap
* 常見陷阱與注意事項

| 錯誤 | 解釋 |
|------|------|
| 忘記 `return` | 使用 `{}` 包裹時必須明確 `return`，否則會渲染 `undefined` |
| key 使用 `index` 或 `random()` | React 無法追蹤正確項目，會造成效能與邏輯錯誤 |
| JSX 中 `key` 不會變成 `props` | 若需要 `id` 傳進子元件，請另設 `userId={person.id}` |

* `map()`：將資料轉成 JSX 清單。
* `filter()`：根據條件過濾資料。
* `key`：讓 React 更高效地更新 DOM。
* `Fragment` 用法：當每項目需多個 DOM 元素時使用。