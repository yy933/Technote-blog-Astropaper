---
title: "[React] Pure Function - 筆記"
pubDatetime: 2025-05-07T20:37:06.000Z
tags: ["JavaScript","React.js"]
description: " Table of contents 什麼是 Pure Function？ Pure Function 的特徵 Same..."
---

## Table of contents

## 什麼是 Pure Function？
**Pure Function 的特徵 - Same inputs, same output.**
* 不更動外部變數或物件: 不影響函式外的世界
* 同樣輸入 ➝ 同樣輸出（可預測性高）
* 像數學公式：y = 2x，x=3 ➝ y永遠是6

React 假設所有元件都是純函式。也就是說：
```jsx
function Recipe({ drinkers }) {
  return (
    <ol>
      <li>Boil {drinkers} cups of water.</li>
      <li>Add {drinkers} spoons of tea and {0.5 * drinkers} spoons of spice.</li>
      <li>Add {0.5 * drinkers} cups of milk and sugar to taste.</li>
    </ol>
  );
}
```
無論何時傳入 drinkers={2}，都會得到相同的 JSX 結果。**元件 = 純函式 ➝ 相同 props 永遠產出相同 JSX**

## 避免 Side Effects
不可在 render 階段修改外部狀態，直接看範例:
```jsx
let guest = 0;
function Cup() {
  guest += 1; // ❌ 改變了外部變數
  return <h2>Guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```
以上會印出:
Tea cup for guest #2
Tea cup for guest #4
Tea cup for guest #6

每次呼叫這個元件的時候會產出不同的JSX，如果修改guest的值也會產生不同的JSX，使得這個元件不可預測。

正確做法:用 props 傳值
```jsx
function Cup({ guest }) {
  return <h2>Guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

以上會印出:
Tea cup for guest #1
Tea cup for guest #2
Tea cup for guest #3

此時這個元件是純粹的(pure)，會產出的JSX可以由props推測。

<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
:bulb: 元件應各自獨立，不依賴渲染順序：
* 你不能假設某個元件會先 render、某個後 render
* 每個元件應該「自己算自己的 JSX」，不與其他元件協調
範例:(錯誤寫法)
```jsx
let guestNumber = 1;

function Cup() {
  const current = guestNumber;
  guestNumber++;
  return <p>Guest #{current}</p>;
}

export default function TeaParty() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```
🧨 問題在哪？
* `Cup` 元件透過共用的 `guestNumber` 變數來決定內容
* 若 React 開啟嚴格模式、或元件渲染順序變動，結果會錯亂
* 這樣的寫法是「不純」的，因為 JSX 不再只由 `props` 決定

正確寫法:
```jsx
function Cup({ guest }) {
  return <p>Guest #{guest}</p>;
}

export default function TeaParty() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```
優點：
* 每個 `Cup` 元件都靠傳入的 `props` 自行計算 JSX
* 不管誰先 render、誰後 render，結果都是可預期的
* 更容易測試、除錯、優化（如 React.memo）
</div>

## 如何發現不純的元件？
* 開啟 `<React.StrictMode>`：
* 開發模式中會 render 兩次 ➝ 偵測不純元件（結果會錯亂）
ex: 原本顯示 Guest #2/#4/#6 ➝ 修正後為 #1/#2/#3

## 可以「局部修改」的情況（Local Mutation）
OK 的範例（不影響外部）：
```jsx
let cups = [];
for (let i = 1; i <= 12; i++) {
  cups.push(<Cup key={i} guest={i} />);
}
```

只要在函式裡建立並修改的變數不會洩漏到外部，就屬於安全的Local Mutation。

## 什麼時候可以讓 side effects 發生?
* 事件處理函式內（如 onClick） ➝ 不會在 render 階段執行

* 若無法使用事件處理器，必要時可使用` useEffect` ➝ 等 render 完再執行side effects（但應視為最後手段）
```jsx
useEffect(() => {
  // 執行side effects（例如：資料請求、動畫）
}, []);
```

## 為何 React 重視純粹性？
* 可以預渲染（伺服器端渲染，Server Side Rendering）或多端共享。
* 更容易做快取（memoization） 與效能最佳化。
* 當中斷或重新渲染時能保持正確性。

## Recap
* 元件必須是pure的，也就代表：
   - 不更改外部變數或物件
   - 相同input一定產生相同 JSX
* 不依賴其他元件的 render 順序
* 不可以直接修改（mutate）元件用來渲染的資料來源，而應該使用 `setState` 或其他機制來更新畫面。[[註1]](###註1)
* 盡量把元件的邏輯表現在 JSX 裡:
  - 把邏輯寫在 render 階段
  - 讓畫面 JSX 自然呈現 state 或 props
* 需要「改變某些東西」時，優先選擇`event handler`，真的找不到事件觸發的時機（例如頁面載入時要 fetch 資料），可以用 `useEffect`  這個專門處理side effects的 Hook　[[註2]](###註2)

<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
### 註1
不可以直接修改（mutate）元件用來渲染的資料來源，而應該使用 `setState` 或其他機制來更新畫面。

直接舉例說明：
#### 錯誤做法：直接修改 state
```jsx
const [user, setUser] = useState({ name: "Alex", age: 25 });

function updateAge() {
  // ❌ 錯誤：直接改變 user.age（mutate）
  user.age = 30;
}
```
問題：
* user 是 React 用來渲染畫面的資料（state）
* 直接改它，React 不會知道需要重新 render
* 所以畫面不會更新，或出現難以追蹤的 bug

#### 正確做法：用 setState 更新
```jsx
const [user, setUser] = useState({ name: "Alex", age: 25 });

function updateAge() {
  // ✅ 正確：透過 setUser 建立一個新物件
  setUser({ ...user, age: 30 });
}
```
說明：
* **React 是靠「比較新舊 state」來決定要不要重新渲染，所以要給它一個新的物件或陣列**
* 用 `setUser` 是一個安全且可預期的方式
</div>

<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
### 註2
需要「改變某些東西」時，優先選擇`event handler`，真的找不到事件觸發的時機（例如頁面載入時要 fetch 資料），可以用 `useEffect`
#### 優先選擇：事件處理器 (event handler)
React 希望只在事件發生後改變 state
```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1); // 透過 event handler 改變 state
  }

  return <button onClick={handleClick}>Clicked {count} times</button>;
}
```
#### 最後手段：`useEffect`
真的找不到事件觸發的時機（例如頁面載入時要 fetch 資料），可以用 `useEffect`：
```jsx
useEffect(() => {
  fetchData();
}, []);
```
`useEffect` 不是第一選擇，因為它發生在 render 之後、比較難控管。
</div>

## 挑戰範例
### 範例一
```jsx
export default function Clock({ time }) {
  let hours = time.getHours();
  if (hours >= 0 && hours <= 6) {
    document.getElementById('time').className = 'night';
  } else {
    document.getElementById('time').className = 'day';
  }
  return (
    <h1 id="time">
      {time.toLocaleTimeString()}
    </h1>
  );
}
```
以上這段code的問題：
#### 問題一：直接操作 DOM（document.getElementById(...)）
React 是透過**虛擬 DOM（Virtual DOM）來管理真實 DOM**，這樣直接去改` <h1>` 的 class，等於繞過 React 的控制，會讓元件難以追蹤、無法預期行為，甚至未來難以除錯。

#### 問題二：在「渲染期間」造成side effects
```jsx
document.getElementById('time').className = ...
```
這段操作真實 DOM是side effect，它改變了 React 外部的東西，不是單純「輸入（props, state） ➜ 輸出（JSX）」
> 每次執行渲染都不應有side effect（例如改資料、操作 DOM、發 API）

:bulb:怎麼分辨side effect？
只要程式碼：
* 改變了外部狀態（像 DOM、全域變數、本地儲存、API 等）
* 依賴外部狀態（像 DOM 的狀態或 URL）

那就可能是side effect。side effect應該寫在` useEffect` 裡，或盡量轉成「渲染邏輯」來處理（像用 `className={...}`）。

#### 正確做法：透過狀態與 className 控制
```jsx
export default function Clock({ time }) {
  const hours = time.getHours();
  const timeClass = (hours >= 0 && hours <= 6) ? 'night' : 'day';

  return (
    <h1 className={timeClass}>
      {time.toLocaleTimeString()}
    </h1>
  );
}
```

### 範例二
```jsx
export default function StoryTray({ stories }) {
  stories.push({
    id: 'create',
    label: 'Create Story'
  });

  return (
    <ul>
      {stories.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```
#### 問題：直接改 props 是錯誤的
```jsx
stories.push({
  id: 'create',
  label: 'Create Story'
});
```

#### 正確寫法：創造一個新的陣列
```jsx
export default function StoryTray({ stories }) {
  const allStories = [...stories, {
    id: 'create',
    label: 'Create Story'
  }];

  return (
    <ul>
      {allStories.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```
或是[用`slice()`做淺拷貝](https://ithelp.ithome.com.tw/articles/10224915):
```jsx
export default function StoryTray({ stories }) {
  // Copy the array!
  let storiesToDisplay = stories.slice();

  // Does not affect the original array:
  storiesToDisplay.push({
    id: 'create',
    label: 'Create Story'
  });

  return (
    <ul>
      {storiesToDisplay.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```
:exclamation: 不管哪種解法，重點都是**不要修改原本資料** !!!

### 範例三
```jsx
import Panel from './Panel.js';
import { getImageUrl } from './utils.js';

let currentPerson;

export default function Profile({ person }) {
  currentPerson = person;
  return (
    <Panel>
      <Header />
      <Avatar />
    </Panel>
  )
}

function Header() {
  return <h1>{currentPerson.name}</h1>;
}

function Avatar() {
  return (
    <img
      className="avatar"
      src={getImageUrl(currentPerson)}
      alt={currentPerson.name}
      width={50}
      height={50}
    />
  );
}
```

#### 問題：使用元件外的變數（`currentPerson`）來傳遞資料
```jsx
let currentPerson;

export default function Profile({ person }) {
  currentPerson = person;
  return (
    <Panel>
      <Header />
      <Avatar />
    </Panel>
  )
}
```
這裡把 person 存進了模組層級的變數 `currentPerson`，然後其他元件（Header、Avatar）直接從外部變數讀資料。這會造成以下問題：

* 問題 1：元件變成「不純」（impure）
`Header()` 和 `Avatar()` 都不再是純函式（pure functions），因為它們依賴外部變數currentPerson。如果 currentPerson 被別的地方改掉，這些元件就會有預期外的行為。

* 問題 2：React 無法追蹤資料變化
React 是依賴 props / state 的變化來觸發 re-render。
但這裡直接改了一個外部變數，**React 不會知道資料變了，所以畫面可能不會更新**。

#### 正確做法：透過 Props 傳遞資料
```jsx
export default function Profile({ person }) {
  return (
    <Panel>
      <Header person={person} />
      <Avatar person={person} />
    </Panel>
  )
}

function Header({ person }) {
  return <h1>{person.name}</h1>;
}

function Avatar({ person }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={50}
      height={50}
    />
  );
}

```