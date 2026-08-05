---
title: "[TypeScript] TypeScript 定義 Function（函式）型別的 4 種常用方法"
pubDatetime: 2026-08-05T09:21:39.167Z
tags: ["TypeScript","cheatsheet","Type system","Concepts"]
description: "Table of contents 簡介 在 TypeScript 中，函式的型別標註主要涵蓋兩大核心：傳入參數 (P..."
hackmd_id: "B1UjnulIze"
---

## Table of contents

## 簡介  
在 TypeScript 中，函式的型別標註主要涵蓋兩大核心：**傳入參數 (Parameters) 與 回傳值 (Return Value)。**

根據不同開發情境（如單一函式、重複使用的 Callback、物件方法或元件 Props），TypeScript 提供了 4 種最常見的定義方式。

## 方法 1：直接在函式宣告時標註 (Inline Type Annotation)  
最直觀、最基礎的寫法。直接在每個參數後方註明型別，並於括號外標註回傳值型別。

### 語法與範例

```typescript
// 1. 普通函式宣告 (Function Declaration)
function add(a: number, b: number): number {
  return a + b;
}

// 2. 箭頭函式 (Arrow Function)
const multiply = (a: number, b: number): number => {
  return a * b;
};
```

* 優點：語法簡單直接，適合不需要重複利用型別定義的單一函式。
* 適用情境：常規的工具函式、單一獨立邏輯。

## 方法 2：使用 Type Alias 定義函式簽名 (Function Signatures)  
當函式型別需要**重複使用**（例如傳入多個元件的 Event Handler 或 Callback 函式）時，推薦使用 `type` 宣告獨立的函式型別。

### 語法與範例

```typescript
// 1. 定義函式型別簽名：(參數: 型別) => 回傳值型別
type MathOp = (x: number, y: number) => number;

// 2. 將型別套用到變數宣告上（TypeScript 會自動推導內部參數型別）
const add: MathOp = (a, b) => a + b;
const subtract: MathOp = (a, b) => a - b;

// 3. 常用於 Callback 函式
type OnSelectHandler = (selectedId: string) => void;

function handleItemClick(callback: OnSelectHandler) {
  callback("item-123");
}
```

* 優點：將「型別定義」與「程式碼實作」完全解耦，大幅提升程式碼重用度。
* 適用情境：Callback 函式、高階函式 (Higher-Order Functions)、React Event Handlers。

## 方法 3：使用 Interface 定義呼叫簽名 (Call Signature)  
`interface` 也可以用來定義純函式型別。它的語法類似物件結構，但直接以 `(參數): 回傳型別` 呈現。

### 語法與範例

```typescript
// 定義函式介面 (Call Signature)
interface SearchFunc {
  (source: string, subString: string): boolean; // 回傳值的型別是boolean
}

// 套用介面
const mySearch: SearchFunc = (src, sub) => {
  return src.includes(sub);
};
```
### 進階：帶有自訂屬性的函式 (Callable Object)  
在 JavaScript 中，函式本質上也是物件，可以擁有自訂屬性（如 `fn.version`）。此情境必須使用 `interface`：

```typescript
interface Counter {
  (start: number): string; // 呼叫簽名
  interval: number;        // 自訂屬性
  reset(): void;           // 自訂方法
}
```

* 優點：可擴充性高，支援物件型別融合與帶屬性的函式。
* 適用情境：傳統 JavaScript 函式庫封裝、帶有額外屬性的複雜函式物件。

## 方法 4：在 Interface / Type 中定義物件方法 (Method Signatures)  
當函式作為物件的其中一個屬性（Method）時，有兩種標準寫法：

### 語法與範例

```typescript
type UserProfile = {
  id: number;
  name: string;
  
  // 寫法 A：方法語法 (Method Syntax)
  greet(message: string): void;
  
  // 寫法 B：屬性語法 (Property Syntax - 箭頭函式形式)
  logout: () => void;
};

const user: UserProfile = {
  id: 101,
  name: "Alice",
  greet(msg) {
    console.log(`${this.name} says: ${msg}`);
  },
  logout: () => {
    console.log("Logged out");
  }
};
```

* 適用情境：定義資料模型 (Data Models)、Service 物件、API Client 介面。

## 延伸補充：參數技巧速查

```typescript
// 1. 選填參數 (Optional Parameter) - 必須放在必填參數後面
function buildName(firstName: string, lastName?: string): string {
  return lastName ? `${firstName} ${lastName}` : firstName;
}

// 2. 預設參數 (Default Parameter)
function multiply(a: number, factor: number = 1): number {
  return a * factor;
}

// 3. Rest 參數 (Rest Parameter)
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}
```

## Quick Recap 

| 定義方法 | 語法結構範例 | 最佳適用情境 |  
| :--- | :--- | :--- |  
| **1. Inline 標註** | `function fn(a: number): string` | 獨立函式、簡單單一邏輯 |  
| **2. Type Alias** | `type Fn = (a: number) => string` | 可重複使用的 Callback、Event Handlers |  
| **3. Interface Call Signature** | `interface Fn { (a: number): string }` | 需要定義可呼叫物件或擴充屬性時 |  
| **4. Object Method Signature** | `{ method(a: number): string }` | 物件裡的方法、Service 介面宣告 |