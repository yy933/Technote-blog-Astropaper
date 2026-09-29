---
title: "前端框架觀念：其他 React 面試題  - 筆記"
pubDatetime: 2026-09-29T04:22:09.931Z
tags: ["Interview Preparation","React","React.js","Concepts","Frontend","React Hook"]
description: "Tags: Interview Preparation React React.js Concepts React H..."
hackmd_id: "SkHwahyqMl"
---

###### Tags: `Interview Preparation` `React` `React.js` `Concepts` `React Hook` `Frontend`


## Table of contents



## 名詞解釋

* **Fragment**：用作空的封裝節點，解決 JSX 必須傳回單一根節點（Single Root Node）的問題，避免產出多餘的 DOM 標籤。
* **Class Component 的最後陣地：Error Boundary**：捕捉下方子元件樹的 JavaScript 渲染錯誤並顯示備用 UI；目前 Hooks 尚無替代方案。
* **HOC (Higher-Order Component)**：接收元件並傳回新元件的設計模式，用於邏輯重用。
* **Portal**：將子元件渲染至父元件 DOM 階層之外的實體 DOM 節點（常用於 Modal、Tooltip）。
* **Controlled vs Uncontrolled**：決定表單元件的狀態是由 React State 驅動（受控），還是由 DOM 本身存取（非受控）。



## 1. 觀念速查

| 主題概念 | 一句話定義 | 主要用途 / 解決的問題 |  
| :--- | :--- | :--- |  
| **Fragment** (`<React.Fragment>` 或 `<>...</>`) | 一個不產生任何實際 DOM 節點的虛擬包裹容器 | 避免過度嵌套 `<div>`，解決 JSX 必須回傳單一根節點的限制 |  
| **Error Boundaries** | 一種捕捉子元件樹 JavaScript 錯誤的 Class 元件 | 避免局部渲染錯誤導致整個應用程式白屏，並提供降級 UI (Fallback UI) |  
| **Higher-Order Component (HOC)** | 接收一個元件並回傳一個新元件的**高階函式模式** | 跨元件複用邏輯（如：權限驗證 `withAuth`、樣式注入） |  
| **Portal** (`ReactDOM.createPortal`) | 將子元件渲染到指定 DOM 節點（如 `document.body`）的機制 | 解決 `overflow: hidden` 或 `z-index` 導致 Modal 被截斷的樣式問題 |  
| **Controlled vs Uncontrolled** | 表單資料是由 React State 控制（受控）或由 DOM 自行維護（非受控） | 控制元件的值、及時驗證，或簡化不需要同步狀態的傳統表單 |



## 2. 深入探討與常見情境

### 1. 什麼時候還需要使用 Class Component？  
在現代 React 開發中，幾乎 99% 的情境都已全面轉向 Function Component + Hooks。目前**唯一必須使用 Class Component 的情境**就是建立 **Error Boundary（錯誤邊界）**。
* **原因**：因為捕捉錯誤需要的生命週期方法（`getDerivedStateFromError` 與 `componentDidCatch`）目前尚無對應的 Hook 實現。

```javascript
// 建立一個 Error Boundary (一定要是 Class Component)
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  // 捕捉下方元件的錯誤，更新 State 顯示降級 UI
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      // 這裡就是「備用 UI (Fallback UI)」
      return <h2>Oops! 這個區塊出了點問題，但不影響其他功能。</h2>;
    }
    return this.props.children;
  }
}

// 使用方式：用它把可能出錯的元件包起來
function App() {
  return (
    <div>
      <Header />
      <ErrorBoundary>
        <UserProfile /> {/* 如果 UserProfile 崩潰，只有它會變備用 UI，Header 與 Footer 依然正常 */}
      </ErrorBoundary>
      <Footer />
    </div>
  );
}
```

### 3. HOC (Higher-Order Component，高階元件)

HOC 就是一個「加工廠」函式：輸入一個元件，包裝後，輸出一個功能更強大的新元件。

**程式碼範例**

```javascript
// HOC 本身是一個函式，接收一個 Component 作為參數
function withAuth(WrappedComponent) {
  // 回傳一個全新的元件
  return function ProtectedComponent(props) {
    const isAuthenticated = checkUserLogin(); // 檢查登入狀態

    if (!isAuthenticated) {
      return <div>請先登入才能查看此頁面！</div>;
    }

    // 登入通過，渲染原本的元件，並將 props 傳遞下去
    return <WrappedComponent {...props} />;
  };
}

// 原始元件：單純顯示個人頁面
function UserProfile() {
  return <h1>歡迎來到個人頁面！</h1>;
}

// 使用 HOC 加工，產出具備「權限驗證」的新元件
const ProtectedUserProfile = withAuth(UserProfile);
```

(註：在 Hooks 出現後，很多 HOC 的邏輯重用已被 Custom Hooks 替代，但 HOC 在許多 UI 框架或套件中依然常見。)

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

為什麼 Custom Hooks 大幅取代了 HOC？
- **避免「嵌套地獄（Wrapper Hell）」**：使用 HOC 時，如果一個元件需要多個功能（如權限 + 主題 + API 資料），會寫成 `withAuth(withTheme(withData(MyComponent)))`，導致 React DevTools 出現一層又一層無意義的包裝元件；Custom Hook 可以在元件內部平鋪呼叫。
- **Prop 命名衝突問題**：HOC 是透過 Props 將資料注入元件，若多個 HOC 注入相同名稱的 Prop（例如都注入 `data`），後者會直接覆蓋前者；Custom Hook 則是以變數解構方式接收，名稱可以自由指定
- **TypeScript 型別推導更友善**：HOC 的泛型型別推導非常複雜且容易報錯；Custom Hook 的輸入與輸出變數型別非常清晰直覺。

</blockquote>

### 3. 受控元件（Controlled）與非受控元件（Uncontrolled）

#### 受控元件 (Controlled Component)
* **機制**：表單的值綁定至 React 的 State（例如 `<input value={value} onChange={e => setValue(e.target.value)} />`）。
* **優點**：React 完全掌控元件狀態，方便進行即時表單驗證、動態停用按鈕或格式化輸入。

```javascript
function ControlledInput() {
  const [text, setText] = useState('');

  return (
    <input 
      value={text} 
      onChange={(e) => setText(e.target.value)} // 每次打字都更新 State，驅動 Re-render
    />
  );
}
```

#### 非受控元件 (Uncontrolled Component)
* **機制**：表單的值由 DOM 節點本身維護，需要讀取時透過 `useRef` 存取 DOM 的 `ref.current.value`。
* **優點**：程式碼較簡單、不需要為每個按鍵觸發 State 更新與重新渲染，適合簡單且一次性讀取的表單。

```javascript
function UncontrolledInput() {
  const inputRef = useRef(null);

  const handleSubmit = () => {
    // 按下送出時，才主動去 DOM 節點抓資料
    alert('輸入的文字是：' + inputRef.current.value);
  };

  return (
    <>
      <input ref={inputRef} defaultValue="預設值" /> {/* 狀態由 DOM 自己管 */}
      <button onClick={handleSubmit}>送出</button>
    </>
  );
}
```


### 4. Portal（傳送門）

在 DOM 樹的結構中，Modal（彈窗）元件的程式碼可能寫在某個深深的子元件裡面。但因為父元件設定了 `overflow: hidden`（超出範圍隱藏）或 CSS `z-index` 層級混亂，導致彈窗被遮擋或裁切。

Portal 能把這個彈窗的「畫面」，直接傳送到 DOM 樹最外層的 `<body>` 底下渲染，但在 React 的邏輯架構裡，它依然屬於該子元件。

**程式碼範例**

```javascript
import ReactDOM from 'react-dom';

function Modal({ isOpen, children }) {
  if (!isOpen) return null;

  // 使用 ReactDOM.createPortal(要渲染的JSX, 要掛載的實體DOM節點)
  return ReactDOM.createPortal(
    <div className="modal-overlay">
      <div className="modal-content">{children}</div>
    </div>,
    document.getElementById('modal-root') // 這個節點可以直接放在 index.html 的 body 下
  );
}
```


## 實作重構：Class Component 轉 Function Component

面試中常見的實作題：將傳統的 Class Counter 元件重構為現代的 Function Component。

### 原始 Class Component 程式碼

```javascript
import React from 'react'; 
import ReactDOM from 'react-dom';

class Counter extends React.Component {
  constructor() {
    super();

    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <button onClick={() => {
          this.setState({ count: this.state.count - 1 });
        }}>-</button>
        {this.state.count}
        <button onClick={() => {
          this.setState({ count: this.state.count + 1 });
        }}>+</button>
      </div>
    );
  }
}

const domElement = document.getElementById('root');
ReactDOM.render(<Counter/>, domElement);
```

### 重構後的 Function Component 程式碼

```javascript
import React, { useState } from 'react';
import ReactDOM from 'react-dom';

function Counter() {
  // 1. 使用 useState 替換 this.state 與 constructor 宣告
  const [count, setCount] = useState(0);

  return (
    <div>
      {/* 2. 使用 setCount 替換 this.setState，並使用 count 替換 this.state.count */}
      <button onClick={() => setCount(count - 1)}>-</button>
      {count}
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}

const domElement = document.getElementById('root');
ReactDOM.render(<Counter/>, domElement);
```

**重構核心重點說明：**
- 擺脫 `this`：Function Component 中沒有 `this` 指向問題，不再需要 `this.state` 或 `this.setState`。

- 用 `useState` 替代 `constructor`：宣告 `const [count, setCount] = useState(0)`，直接取得狀態與更新函式。

- 取消 `render()` 方法：Function Component 本身的回傳值（Return）即是 JSX 結構。

## FAQ
### Q1: 為什麼使用 `<React.Fragment>` 比使用 `<div>` 包裹更好？
- 保護 DOM 結構：過多的 `<div>` 會導致 DOM 階層過深（DOM Nesting），可能破壞 CSS Flexbox / Grid 的排版邏輯，或影響特定的 HTML 語意（如 `<table>` 內包含多個 `<td>`）。

- 優化效能與記憶體：少產生額外的 DOM 節點可以減少瀏覽器排版（Reflow）與渲染（Repaint）的負擔。

### Q2: 什麼是 React Portal？常見的使用情境有哪些？  
`ReactDOM.createPortal(child, container)` 可以將子元件渲染到父元件 DOM 階層之外的任意指定 DOM 節點。

- 常見情境： Modal（對話框）、Tooltip（提示框）、Notification（通知列）。

- 解決的問題：當父元件帶有 CSS 屬性 `overflow: hidden` 或 `z-index` 層級混亂時，Modal 容易被父元件截斷或遮擋。使用 Portal 將 Modal 直接掛載到 `document.body` 即可完美避開樣式限制。