---
title: "前端框架觀念：JSX 的本質與 Element vs. Component - 筆記.md"
pubDatetime: 2026-09-29T04:20:54.019Z
tags: ["Interview Preparation","React","React.js","Concepts","Frontend"]
description: "Tags: Interview Preparation React React.js Concepts Fronten..."
---

###### Tags: `Interview Preparation` `React` `React.js` `Concepts` `Frontend`


## Table of contents


## JSX特性與觀念總覽

* **JSX 不是 HTML**：它是一種讓你在 JavaScript 中使用「類 HTML（HTML-like）」的**樣板語法（Template Syntax）**，最後會被轉譯為 JavaScript 物件。
* **JSX 的產出**：JSX 語法執行後會產生代表 UI 的 **React Element（物件）**。
* **不強制依賴 JSX**：React 完全可以不使用 JSX 開發，其底層是透過 `React.createElement()` 來運作。


## 1. 什麼是 JSX？

### 運作機制與常見誤解
* **常見誤解**：許多人常說「JSX 就是把 HTML 寫在 JavaScript 裡面」或「它只是一串插入 DOM 的字串」，**這是錯誤的觀念**。
* **正確定義**：JSX 是 JavaScript 內部的樣板語法（Template Syntax）。
* **背後行為**：JSX 樣板經過轉譯後，會產生代表 UI 結構的 **JavaScript 物件（Elements）**。



## 2. Element vs. Component 的根本差異

| 比較面向 | React Element | React Component |  
| :--- | :--- | :--- |  
| **本質定義** | 一個描述 UI 結構的 **JavaScript 物件** | 一個回傳 Element 的 **JavaScript 函式** |  
| **建立方式** | 直接撰寫 JSX，例如 `<div />` | 宣告函式並回傳 JSX，例如 `function App() { return <div />; }` |  
| **角色定位** | UI 的最小渲染單位（產出結果） | 封裝邏輯與 UI 的重用單元（產出機制） |

### 程式碼範例對照

#### (1) React Element（純物件）  
直接宣告 JSX 語法，此時它只是一個 React Element 物件：

```javascript
const reactElement = <div>hello</div>;

const domElement = document.getElementById('root');
ReactDOM.render(reactElement, domElement);
```

#### (2) React Component（函式）

將其包裝成函式並回傳 Element，即成為一個 Component：

```javascript
 // Component 是一個回傳 Element 的函式
function MyComponent() {
  return <div>hello</div>;
}

// 使用時以組件標籤呼叫
const domElement = document.getElementById('root');
ReactDOM.render(<MyComponent/>, domElement);
```

##  3. 可以不使用 JSX 寫 React 嗎？

**答案是：可以。**

### 轉譯底層機制

JSX 只是  `React.createElement()`  的**語法糖（Syntactic Sugar）**。即使完全不寫 JSX，也能直接呼叫原生 API 來建立 Element 與 Component：

```javascript
// 使用 JSX 寫法
const elementWithJSX = <div>Hello!</div>;

// 不使用 JSX 的等價寫法 (React 底層行為)
const elementWithoutJSX = React.createElement(
  'div',      // 1. HTML 標籤或 Component 類型
  null,       // 2. 屬性 (Props)
  'Hello!'    // 3. 子元素 (Children)
);
```

**反思思考題**：既然不使用 JSX 也能寫 React，那為什麼幾乎所有 React 專案與開發者都選擇使用 JSX？（提示：可讀性、開發效率、HTML-like 的直覺結構）

# FAQ

### Q1: 什麼是 JSX？

**回答**：

1.  **名稱與定義**：JSX 是 JavaScript XML 的縮寫，是一種在 JavaScript 中撰寫類 HTML 語法的**樣板語法**。
      
2.  **澄清誤解**：它不是真正的 HTML，也不是普通的字串，而是會被轉譯器（如 Babel）編譯成  `React.createElement()`  的呼叫。
      
3.  **最終產出**：JSX 執行的結果會產生一個純 JavaScript 物件，也就是  **React Element**，用來描述預期的 UI 結構。
    

### Q2: 請簡單說明 Element 與 Component 的關係？

-   **Element**  是 UI 的藍圖與資料結構（**JavaScript Object**）。
    
-   **Component**  是建立藍圖的工廠（**Function**），它的職責就是「接收 input (props) 並回傳 Element」。
    
-   簡單來說：**Component 是一個回傳 Element 的函式**。