---
title: "[React] 使用 props 向下傳遞資料 - 筆記"
pubDatetime: 2025-05-07T20:45:08.000Z
modDatetime: 2026-05-25T10:04:23.545Z
tags: ["JavaScript","React.js"]
description: "Table of contents 什麼是 props？ props（properties）是元件間傳遞資料的方式。..."
hackmd_id: "Bk-iRZ0Jlg"
---

## Table of contents


## 什麼是 props？

* props（properties）是元件間傳遞資料的方式。
* 和 HTML 的屬性相似，但 props 可以傳任意 JavaScript 值（物件、陣列、函式、JSX 等）。
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}

function App() {
  return <Welcome name="Alice" />;
}
```
* App 是父元件，它透過 name="Alice" 傳了一個 props 給 Welcome。
* Welcome 子元件接收 props，並顯示 Hello, Alice!。

使用「解構語法」讓程式碼更乾淨：
```jsx
function Welcome({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```
## 常見props型別
| 類型 | 範例 | 說明 |
|------|------|------|
| **字串（String）** | `title="Hello"` | 用在文字顯示、標題等 |
| **數字（Number）** | `count={5}` | 注意要用 `{}` 包住非字串型別 |
| **布林（Boolean）** | `disabled={true}` | 可控制元件狀態，例如按鈕是否可按 |
| **陣列（Array）** | `items={['a', 'b']}` | 常用於列出清單項目 |
| **物件（Object）** | `user={{ name: 'Tom', age: 30 }}` | 可傳整包資料給元件處理 |
| **函數（Function）** | `onClick={handleClick}` | 事件處理、回呼函數等 |
| **JSX 元素** | `children={<p>Hello</p>}` | 用於組合元件、Slot 概念 |
| **null / undefined** | `value={null}` | 可作為控制元件的清空狀態等 |
<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:bulb: **除了字串，其他型別的props都要用 `{}` 包住**
JSX 裡 props 只接受「JS 表達式」的話，要用 `{}`，
而「字串常值」可以直接寫成 HTML 字串格式。

</blockquote>

## 如何傳遞與讀取 props？
例如有一個沒有傳遞任何props給子元件的元件Profile:
```jsx
export default function Profile() {
  return (
    <Avatar />
  );
}
```
### 步驟一: 傳遞props給子元件
這裡的props: `person` (物件), `size` (數字):
```jsx
export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```
:pencil: 雙花括號`{{}}`只是JSX中的物件!

### 步驟二: 讀取子元件props
* 讀取方式（解構賦值）：
```jsx
function Avatar({ person, size }) {
  // person and size are available here
}
```
加入一些邏輯到Avatar中:
`App.js`
```jsx
import { getImageUrl } from './utils.js';

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <div>
      <Avatar
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi', 
          imageId: 'YfeOqp2'
        }}
      />
      <Avatar
        size={80}
        person={{
          name: 'Aklilu Lemma', 
          imageId: 'OKS67lh'
        }}
      />
      <Avatar
        size={50}
        person={{ 
          name: 'Lin Lanying',
          imageId: '1bX5QH6'
        }}
      />
    </div>
  );
}
```

`utils.js`
```jsx
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```
> 可以在父層（如 Profile）改變 props 的值（如 person 或 size），而不需要知道子元件（如 Avatar）怎麼用這些資料。反過來，子元件也能自由調整 props 的用法，而不需要修改父層程式碼。
> 這樣設計讓元件更獨立、更容易維護。

* 傳統寫法（不解構）：
```jsx
function Avatar(props) {
  let person = props.person;
  let size = props.size;
}
```


## 給予props預設值（default props）
有時會遇到某些資料缺少的情況，為了避免fallback，可以給予props預設值：
```jsx
function Avatar({ person, size = 100 }) {
  // 若未傳入 size，預設為 100
}
```
:bulb: 只有在 size 是 undefined 時才會套用預設值，null 或 0 不會。

## 使用 JSX Spread 語法傳遞 props
有時候傳遞props使得code看起來重複性很高：
```jsx
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}
```

這時可以用展開語法(spread syntax)：
```jsx
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```
<blockquote class="my-6 p-4 bg-red-50 dark:bg-red-950/30 border-l-4 border-red-500 rounded-r-md text-red-900 dark:text-red-200 blocknoted-fix">

⚠️ 展開語法(spread syntax)須小心使用，多用反而會降低可讀性，除非元件只是單純傳遞而不使用 props。

</blockquote>

## 將JSX作為子元件(children)傳遞
這樣寫的時候:
```jsx
<Card>
  <Avatar />
</Card>
```
JSX 會自動把 `<Avatar /> `當作 `Card` 的 children prop 傳進去。
相當於:
```jsx
<Card children={<Avatar ... />} />
```

在 Card 元件裡，可以用 `{children}` 來決定要把這段插進哪裡：
```jsx
function Card({ children }) {
  return (
    <div className="card">
      {children} {/* 這裡會顯示 Avatar */}
    </div>
  );
}
```

📌 用途：打造可重複使用的容器元件
這樣的設計可以讓 Card 元件不管裡面是 Avatar、文字、還是圖片都能通用，非常有彈性。適合用來做排版或視覺容器（像是 Panel、Grid）。

也就是：
```jsx
<Card>這裡放什麼都可以</Card>
```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

**也就是說!**
例如這樣定義 Card 元件：
```jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}  // 這裡會出現外部塞進來的內容
    </div>
  );
}
```

可以這樣使用：
```jsx
<Card>
  <p>Hello!</p>
  <img src="cat.jpg" />
</Card>
```

實際畫面就會變成這樣：
```jsx
<div class="card">
  <p>Hello!</p>
  <img src="cat.jpg" />
</div>

```

</blockquote>


### 不定量或大量 props 要傳給子元件
使用 展開運算子（spread operator）`...`，它可以乾淨且有彈性地傳遞所有 props。
```jsx
function ProfileCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.job}</p>
    </div>
  );
}

function App() {
  const userInfo = {
    name: 'Alice',
    job: 'Designer',
    age: 28,  // 即使 ProfileCard 沒用到也沒關係
  };

  return <ProfileCard {...userInfo} />;
}
```

**:bulb: 在子元件中解構有用的 props**
```jsx
function ProfileCard({ name, job }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{job}</p>
    </div>
  );
}
```

## Props 是唯讀的、會隨時間更新
* **Props 是來自父元件的「資料快照」，子元件不能直接修改 props。** 
* 每次重新 render 會重新取得新的 props。
* 但 props 是 immutable（不可改變） 的，若想變更資料，應透過父元件重新傳入或使用 state。

:warning:　重要！可參考[[React] Pure Function - 筆記](https://hackmd.io/qIh7AQhiSoSdnFeuOxdC6w#Recap)或以下簡單錯誤示範：
```jsx
function Navbar(props) {
    // ❌ DON'T DO THIS
    props.logoIcon = "some-other-icon.png"
}
```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

:arrow_right: 這行程式碼**違反了 React 的設計原則。** 在 React 中，props 是從父元件傳進來的資料，應該是 **唯讀的（read-only）並且不可變（immutable）。** 

可以「使用」它們，但不能「修改」它們。如果修改props，會讓程式變得難以預測與除錯。若父元件重新 render，傳入的 props 會被覆蓋，改動會瞬間消失。

**好的做法是在子元件中用自己的 state 來儲存副本。**
```jsx
function Navbar(props) {
    const [logoIcon, setLogoIcon] = useState(props.logoIcon)

    // 然後再根據情況更新 logoIcon
    setLogoIcon("some-other-icon.png")
}
```

</blockquote>

##  比較兩種 props 結構設計的寫法
### 寫法一
假設有一個Entry元件：
```jsx
// Entry.jsx
export default function Entry({ img, country, googleMapsLink, title, dates, text }) {
  return (
    <article className="journal-entry">
      <div className="main-image-container">
        <img 
          className="main-image"
          src={img.src}
          alt={img.alt}
        />
      </div>
      <div className="info-container">
        <img 
          className="marker"
          src="../images/marker.png" 
          alt="map marker icon"
        />
        <span className="country">{country}</span>
        <a href={googleMapsLink} target="_blank" rel="noopener noreferrer">
          View on Google Maps
        </a>
        <h2 className="entry-title">{title}</h2>
        <p className="trip-dates">{dates}</p>
        <p className="entry-text">{text}</p>
      </div>
    </article>
  );
}
```
在App.jsx中這樣寫:
```jsx
import Header from "./components/Header"
import Entry from "./components/Entry"
import data from "./data"

export default function App() {
    
    const entryElements = data.map((entry) => {
        return (
            <Entry
                key={entry.id}
                {...entry}  // 使用展開運算子 {...entry} 
            />
        )
    })
    
    return (
        <>
            <Header />
            <main className="container">
                {entryElements}
            </main>
        </>
    )
}
```
這種寫法的優點:
* 最常見、可讀性高，一目了然 Entry 需要哪些資料。
* 在 Entry 中不需要 `props.`，更簡潔。
* 若 data 陣列來源已確定結構，這種寫法最符合 React 元件粒度思維。

缺點：
* 如果 prop 太多，Entry 的參數列會變長。
* 需要確保每個欄位命名一致，容易耦合。

### 寫法二
```jsx
// Entry.jsx
export default function Entry({ entry }) {
  return <img src={entry.img.src} />
}

// 或是沒解構的寫法
export default function Entry(props) {
    return <img src={props.entry.img.src} />
}
```
```jsx
// App.jsx
<Entry key={entry.id} entry={entry} />  // 傳入 entry 整包物件
```

這種寫法的優點：
* 如果資料結構複雜、深層，或需整包傳遞，比較方便。
* 可以快速調整只靠 entry 一個 prop。

缺點：
* 可讀性稍差，不清楚 Entry 實際會用到什麼欄位。
* 增加 .entry. 的層級，程式變冗長。

:arrow_right: 如果 Entry 是一個清楚且穩定的 UI 元件，建議使用寫法一，這是目前 React 團隊和社群最常見的寫法，清楚聲明元件需要什麼資料，也更容易搭配 TypeScript 或 PropTypes 進行型別檢查。