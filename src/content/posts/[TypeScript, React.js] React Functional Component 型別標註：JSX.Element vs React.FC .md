---
title: "[TypeScript, React.js] React Functional Component 型別標註：JSX.Element vs React.FC"
pubDatetime: 2026-08-05T08:03:07.895Z
tags: ["Type system","React.js","Concepts","TypeScript","cheatsheet"]
description: "Table of contents React 元件的型別推導與保護機制 在 TypeScript 中，撰寫一般 Fu..."
hackmd_id: "B1TziPxUfl"
---

## Table of contents

## React 元件的型別推導與保護機制  
在 TypeScript 中，撰寫一般 Functional Component 並且回傳 JSX 時，TypeScript 預設會自動推導 (Type Inference) 元件的回傳型別為 `JSX.Element`（或 `React.JSX.Element`）。

```typescript
// TypeScript 會自動推導 Header 的回傳型別為 JSX.Element
export default function Header() {
  return <header><h1>Title</h1></header>;
}
```

### 為什麼依然建議明確標註型別？  
雖然自動推導方便，但在團隊開發或大型專案中，明確宣告元件回傳型別能發揮強大的「防禦性編程」作用。例如防止誤將不該回傳的值（如 `true`、未處理的非 JSX 物件）傳出：

```typescript
// ❌ 若未標註型別，誤回傳 boolean 時 TS 僅會將回傳型別推導為 boolean，無法第一時間報錯
export default function Header() {
  return true; 
}

// ✅ 明確標註型別後，誤回傳非 JSX 會立即引發編譯錯誤
export default function Header(): JSX.Element {
  return true; // ❌ Error: Type 'boolean' is not assignable to type 'Element'.
}
```


## 現代推薦作法：使用 `JSX.Element`  
直接標註函式的回傳型別為 `JSX.Element` 是目前 React + TypeScript 社群最為推薦的標準作法。

### 語法結構

```typescript
import type { JSX } from "react";

export default function Header(): JSX.Element {
  return (
    <header>
      <h1>Assembly: Endgame</h1>
      <p>Guess the word within 8 attempts!</p>
    </header>
  );
}
```

### 優勢
* 語法直觀：符合一般 JavaScript/TypeScript 宣告函式的習慣。
* 精準防禦：明確限制該函式的執行結果必須是合法的 JSX 元素。
* 無多餘封裝：不需透過額外的高階型別（Higher-Order Types）封裝元件。

## 傳統/舊版作法：`React.FC` (`React.FunctionComponent`)  
在早期的 React + TypeScript 專案中，常見到使用 `React.FC` 泛型型別來宣告元件變數：

```typescript
import React from "react";

// 傳統變數宣告型別作法
const Header: React.FC = () => {
  return (
    <header>
      <h1>Assembly: Endgame</h1>
    </header>
  );
};

export default Header;
```


## 為什麼現代社群更傾向 `JSX.Element` 而非 `React.FC`？  
雖然 `React.FC` 曾是主流，但社群（包含 React 官方團隊與 TypeScript 指南）逐漸轉向直接標註 `JSX.Element`，主因包含以下幾點：

### 1. 隱式 `children` 議題（React 18 前）  
在舊版 `React.FC` 中，即使沒有定義 `Props`，`React.FC` 預設也會將 `children?: ReactNode` 包裹進 `Props` 中。這導致元件即使不支援子元素，傳入 `children` 時也不會引發型別錯誤。  
(註：React 18 雖已移除隱式 `children`，但 `React.FC` 的其他缺點依然存在)。

### 2. 與預設參數 (Default Props) 的相容性問題  
使用 `React.FC` 時，ES6 的參數預設值（Default Parameters）型別推導有時無法如預期般運作。

### 3. 泛型元件 (Generic Components) 宣告繁瑣  
當需要撰寫「泛型 React 元件」時，使用 `React.FC` 的語法會變得異常複雜，而普通函式寫法（搭配 `JSX.Element`）則非常自然。

## Quick Recap 

| 寫法類型 | 程式碼範例 | 特點與優缺點 | 建議 |  
| :--- | :--- | :--- | :--- |  
| **自動推導** | `function Header() { return <header /> }` | 寫法最快，TS 會自動推導為 JSX | 🟢 小型專案或快速開發適用 |  
| **`JSX.Element`** | `function Header(): JSX.Element { ... }` | 明確限制回傳值必須為 JSX，語法乾淨 | 🟢 **現代 TS 社群推薦** |  
| **`React.FC`** | `const Header: React.FC = () => { ... }` | 傳統變數宣告型別，過去常用於自動包裹 children | 🟡 舊專案常見，現代較少主動採用 |