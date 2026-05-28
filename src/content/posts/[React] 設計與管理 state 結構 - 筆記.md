---
title: "[React] 設計與管理 state 結構 - 筆記"
pubDatetime: 2025-05-15T03:55:58.000Z
modDatetime: 2026-05-25T10:04:23.559Z
tags: ["JavaScript","React.js","React Hook"]
description: "Table of contents state結構設計五大原則 原則 說明 1️⃣ 群組相關的 state 若兩個變數..."
hackmd_id: "rk9RErlbeg"
---

## Table of contents


## state結構設計五大原則
| 原則                    | 說明                                                  |
| --------------------- | --------------------------------------------------- |
| 1️⃣ 群組相關的 state       | 若兩個變數總是一起改動，就該合併為一個物件（例如：x, y 改為 position）          |
| 2️⃣ 避免互相矛盾的 state     | 不要讓多個 state 可能彼此矛盾，例如：isSending 和 isSent 同時為 true   |
| 3️⃣ 避免多餘 state        | 若資料能從現有的 props 或 state 計算出來，就不用存進 state，例如 fullName |
| 4️⃣ 避免 state 重複儲存同樣資訊 | 例如 selectedItem 重複於 items 中出現，容易造成不同步               |
| 5️⃣ 避免巢狀太深的 state 結構  | 巢狀資料不易更新，儘量讓結構扁平化                                   |

## 範例
### 1. 群組相關的 state
```jsx
// 不好的設計
const [x, setX] = useState(0);
const [y, setY] = useState(0);

// 好的設計: 合併為 position 物件
const [position, setPosition] = useState({ x: 0, y: 0 });
setPosition({ x: e.clientX, y: e.clientY });

// 若只更新 x
setPosition({ ...position, x: 100 });
```

### 2. 避免矛盾 state：用單一 status 變數取代
```jsx
// ❌ 錯誤：容易出現 isSending 和 isSent 同時為 true
const [isSending, setIsSending] = useState(false);
const [isSent, setIsSent] = useState(false);

// ✅ 正確：用 status 統一管理
const [status, setStatus] = useState('typing'); // 'typing' | 'sending' | 'sent'

// 同時間還是可以這樣定義:
const isSending = status === 'sending';
const isSent = status === 'sent';
```

### 3. 移除多餘 state：從現有資料計算
```jsx
// ❌ 不必要的 fullName state
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Last name:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```
```jsx
// ✅ 直接在 render 計算
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName;

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Last name:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

:bulb: 不要以`props`初始化`state`變數
```jsx
import { useState } from 'react';

export default function Clock(props) {
  const [color, setColor] = useState(props.color);
  return (
    <h1 style={{ color: color }}>
      {props.time}
    </h1>
  );
}
```
以上code的問題在於：
* `useState()` 只在第一次 render 時取值
* 如果父元件後續傳來不同的 `messageColor`，`state` 中的` color `不會跟著更新，**造成props 改了，但畫面沒變**

:bulb: 正確做法：直接使用 props，或轉為變數
```jsx
import { useState } from 'react';

export default function Clock(props) {
  return (
    <h1 style={{ color: props.color }}>
      {props.time}
    </h1>
  );
}
```
這樣做可以確保：
* 使用的 color 永遠與父元件的 props 同步
* 不會有「畫面卡住、不更新」的問題

💡 例外情況：只想記住 props 的初始值（忽略後續更新）
```jsx
function Message({ initialColor }) {
  const [color, setColor] = useState(initialColor); // 僅用第一次的值
}
```
* 適用場景：如使用者自訂初始值、編輯器初始內容等
* 通常會用 `initialXXX` 或 `defaultXXX` 命名，表示之後的更新會被「忽略」

</blockquote>

### 4. 避免 state 重複儲存同樣資訊: 只存 id 而非整個物件
```jsx
import { useState } from 'react';

const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crispy seaweed', id: 1 },
  { title: 'granola bar', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.title}
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Choose</button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```
:bulb: 這樣會造成 `selectedItem` 和 `items` 裡的某一項是同一個物件，代表資料重複儲存在兩個地方。萬一 `items` 更新了，`selectedItem` 有可能沒跟上（或反之），容易產生「資料不同步」的 bug。

正確做法（只存 id）:
```jsx
const [selectedId, setSelectedId] = useState(0);
const selectedItem = items.find(item => item.id === selectedId);
```
這樣做的好處是：

* 資料只有一份，統一由 items 控制
* 當 items 更新時，selectedItem 會正確反映最新內容
* 程式更簡潔，避免不必要的狀態管理與記憶體浪費

### 5. 避免巢狀太深的 state 結構
巢狀太深的 state 結構（如：object 中包 object 再包 array）會造成更新某個子節點變得複雜，需一路複製父節點鏈，以及程式碼冗長、容易出錯，難以追蹤資料流動與關係等問題。
* 常見錯誤結構:
```jsx
// 每次更新都需複製從 root 到目標的整條巢狀鏈。
{
  id: 0,
  title: 'Root',
  childPlaces: [
    {
      id: 1,
      title: 'Earth',
      childPlaces: [
        { id: 2, title: 'Africa', childPlaces: [ ... ] }
      ]
    }
  ]
}
```
* 改進做法：扁平化 state（Flat/Normalized）
```jsx
// 每筆資料只存自己的 id 與 childIds，像資料表一樣管理。
{
  0: { id: 0, title: 'Root', childIds: [1, 42, 46] },
  1: { id: 1, title: 'Earth', childIds: [2, 10, 19] },
  ...
  2: { id: 2, title: 'Africa', childIds: [3, 4, 5] }
}
```
複雜的巢狀結構：
* 刪除子節點 → 一路複製父節點、祖父節點、甚至 root。

扁平化後，只需更新兩層資料：
* parent.childIds 移除該 id
* 更新 state 中該 parent 的資料
範例:
```jsx
function handleComplete(parentId, childId) {
  const parent = plan[parentId];
  const nextParent = {
    ...parent,
    childIds: parent.childIds.filter(id => id !== childId)
  };
  setPlan({
    ...plan,
    [parentId]: nextParent
  });
}
```

* 概念說明

| 概念         | 說明                                                  |
| ---------- | --------------------------------------------------- |
| 扁平化 (flat) | 讓每個資料用 id 鍵對應，彼此以 id 參照，不再層層嵌套                      |
| Normalized | 模仿資料庫 table 概念，每筆資料為獨立單元，關聯由 id 建立                  |
| 類似工具       | Redux Toolkit 中的 `createEntityAdapter` 即用此方法自動化扁平管理 |


## Recap
1. 合併總是一起更新的狀態:
如果兩個 state 總是一起變動，應該合併為一個，以避免錯誤或同步問題。

1. 避免創造「不可能的狀態」:
設計 state 時要避免出現邏輯上無法成立的組合（例如：loading=true 但 data=null 的情況）。

1. 結構設計應降低錯誤機率
用清晰、單一責任的方式設計 state 結構，減少更新時的錯誤發生。

1. 避免冗餘與重複資料
若資料可以從 props 或其他 state 計算出來，就不需要另外存一份。

1. 除非特別需求，不要把 props 放進 state
props 會隨父元件變動，不應放入 state，除非想忽略它後續更新。

1. UI 狀態（如選擇項目）建議只存 ID 或 index
儲存資料的識別碼即可，避免物件不一致或更新不易。

1. 扁平化巢狀狀態，方便更新
將巢狀資料轉為扁平結構，提升更新效能與可讀性。