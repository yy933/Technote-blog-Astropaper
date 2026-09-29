---
title: "前端框架觀念：React Props - 筆記.md"
pubDatetime: 2026-09-29T04:15:32.358Z
tags: ["Interview Preparation","React","React.js","Concepts","Frontend"]
description: "Tags: Interview Preparation React React.js Concepts Fronten..."
---

###### Tags: `Interview Preparation` `React` `React.js` `Concepts` `Frontend`


## Table of contents


## 核心觀念

* **單向資料流（Unidirectional Data Flow）**：父元件透過 Props 將資料傳給子元件（Parent ➔ Child）。
* **子傳父的機制（Child ➔ Parent）**：透過傳遞 **Callback Function（回調函式）** 作為 Prop，由子元件呼叫該函式並傳入參數，實現向上傳遞。
* **Prop Drilling（屬性鑽孔）**：資料跨越過多層級元件時導致的中轉現象。
* **Props 的不可變性（Immutable / Read-Only）**：Props 是唯讀的，React 要求所有元件在對待自己的 Props 時必須像「純函式」一樣，不得直接修改。



## 1. 元件間的資料傳遞（Data Passing）

### ① 父傳子（Parent ➔ Child）
* **機制**：直接在子元件標籤上以屬性（Attribute）形式傳遞，這是 React 最基本的傳承機制。
* **速答**：透過 **Props** 向下傳遞資料。

### ② 子傳父（Child ➔ Parent）
* **機制**：父元件先定義一個**處理函式（Function/Callback）**，並將這個函式作為 Prop 傳給子元件。子元件觸發事件時呼叫該函式（可帶入參數），即可將資料向上傳遞給父元件。
* **速答**：傳遞一個 **Function Prop**，由子元件執行該函式來向父元件發送訊息。

```jsx
// 1. 父元件定義函式並傳給子元件
function Parent() {
  const handleDataFromChild = (data) => {
    console.log("收到子元件資料：", data);
  };
  return <Child onSend="{handleDataFromChild}"/>;
}

// 2. 子元件呼叫傳進來的 Function Prop
function Child({ onSend }) {
  return <button onClick={() => onSend("Hello Parent!")}>傳送資料給父元件</button>;
}
```

## 2. 什麼是 Prop Drilling（屬性鑽孔）？

-   **定義**：當資料需要從層級極高的元件（如 Grandparent）傳遞給極深層的末端元件（如 Atom/Button）時，中間無關的元件都必須被迫接收並轉發該 Props 的現象。
    
-   **情境**：`Grandparent`  ➔  `Parent`  ➔  `Child`  ➔  `Atom`。
    
-   **影響**：中間層級的元件不需要這些資料，但卻充當「中繼站」，導致程式碼偶合度過高、難以維護。
    
-   **解決方案**：可透過  **Context API**  或全域狀態管理工具（如 Redux Toolkit, Zustand）來跨層級直接存取資料。
    

## 3. 可以修改 Props 嗎？（Props 是 Read-Only 的）

**答案是：絕對不能（Props 必須是 Read-Only）。**

### 原因：純函式（Pure Functions）原則

React 規範：**所有 React 元件在處理其 Props 時，必須表現得像「純函式（Pure Function）」一樣。**

-   **純函式的定義**：相同的輸入（Inputs），永遠獲得相同的輸出（Output），且不產生任何副作用（Side Effects）。
    
-   **範例說明**：
```javascript
// 純函式範例：相同 x 與 y 永遠回傳相同結果，不改變傳入的參數
function add(x, y) {
  return x + y;
}
```

**為什麼不能修改？**：  
若直接修改 Props，會破壞資料的一致性與 UI 的可預測性。如果需要根據使用者操作而「修改/改變」資料，那是 **State（狀態）** 的職責，而非 Props。

# FAQ

### Q1: 子元件要如何傳資料給父元件？

**回答骨架**： 透過  **傳遞 Callback Function（回調函式）**  作為 Prop 來實現。

1.  父元件建立一個 Function 並將其作為 Prop 傳遞給子元件。
      
2.  子元件在特定時機（如點擊事件）呼叫這個 Function，並將資料作為參數傳入。
      
3.  父元件在該 Function 中接收參數，從而完成子對父的資料傳遞。
    

### Q2: 何謂 Prop Drilling？該如何避免？

-   **定義**：為了將資料傳給深層元件，必須將 Props 穿過許多不使用該資料的中間層元件的現象。
    
-   **避免方式**：
    
    1.  使用  **React Context API**（針對跨元件共享資料，如 Theme、User Profile）。
        
    2.  使用全域狀態管理庫（如  **Zustand**,  **Redux Toolkit**）。
        
    3.  使用  **Component Composition（元件組合）**  將子元件直接作為  `children`  帶入，減少中間轉發。
        

### Q3: Props 與 State 有何本質上的不同？


| 比較維度 | Props | State |  
| :--- | :--- | :--- |  
| **資料來源** | 由外部（父元件）傳入 | 在元件內部自行宣告與管理 |  
| **可變性** | 唯讀（Read-Only），不可直接修改 | 可變（Mutable），透過 `useState` 函式更新 |  
| **職責** | 負責元件間的資料傳遞與設定 | 負責驅動元件內部的 UI 重新渲染 |