---
title: "[TypeScript, React.js] useState 搭配 TypeScript 的型別標註與泛型 (Generics)"
pubDatetime: 2026-08-05T05:08:02.068Z
tags: ["TypeScript","cheatsheet","Type system","Concepts","React Hook","React.js"]
description: "Table of contents React 中 useState 的型別推導機制 在 React 搭配 TypeS..."
hackmd_id: "SkwVKVeLfe"
---

## Table of contents

## React 中 useState 的型別推導機制  
在 React 搭配 TypeScript 開發時，`useState` 預設會根據提供的初始值 (Initial State) 自動進行型別推導 (Type Inference)。

```typescript
// TypeScript 會自動推導 currentWord 型別為 string
const [currentWord, setCurrentWord] = useState("apple");

// ❌ 嘗試寫入不同型別會直接觸發編譯錯誤：
// Argument of type 'boolean' is not assignable to parameter of type 'SetStateAction<string>'.
setCurrentWord(true);
```

### ⚠️ 空陣列與 null 的推導陷阱  
然而，自動推導並非萬能。當初始值無法提供足夠的型別資訊時（例如空陣列 `[]` 或 `null`），TypeScript 會推導出不理想的型別：

```typescript
// ❌ 被推導為 any[]，失去了型別安全保護！
const [guessedLetters, setGuessedLetters] = useState([]);

// ❌ 被推導為 null，後續無法 setUser 物件！
const [user, setUser] = useState(null);
```

## useState 的泛型 `<S>` 設計原理  
為什麼我們可以在 `useState` 後面加上角括號 `<>` 來指定型別？  
因為在 React 的型別定義檔（`@types/react`）中，`useState` 本身就是一個泛型函式 (Generic Function)。

簡化後的 React 底層 `useState` 如下：

```typescript
// React 底層簡化定義
function useState<S>(
  initialState: S | (() => S)
): [S, Dispatch<SetStateAction<S>>];
```

### 運作解析
* `<S>` 泛型占位符：`S` 代表 State 的型別（可以傳入任何型別，如 `string`、`User`、`string[]`）。
* Getter 的型別：解構出來的第一個值（State 本身）型別會被設定為 `S`。
* Setter 的型別：解構出來的第二個值（更新函式）會被限制只能接收型別為 `S` 的新值（或接收回傳 `S` 的更新 Callback）。

當我們寫下 `useState<string[]>([])` 時，我們實際上是在顯式帶入泛型參數（Explicit Generic Type Argument），將底層所有的 `S` 替換為 `string[]`。


## 實務用法與語法說明
### 1. 顯式指定泛型 (Explicit Type Argument)  
當初始值為空陣列或需要嚴謹的型別控管時，推薦使用角括號語法：

```typescript
// ✅ 強制限制 State 為字串陣列，避免被推導為 any[]
const [guessedLetters, setGuessedLetters] = useState<string[]>([]);

// ✅ 多型別聯集 (Union Type)：適合初始為 null 的情況
const [user, setUser] = useState<User | null>(null);
```

### 2. Lazy Initialization 與箭頭函式回傳值標註  
當 State 的初始值需要經過複雜計算時，我們會傳入一個 Callback Function。除了可以在 `useState` 指定泛型外，也可以明確標註箭頭函式的回傳值型別：


```typescript
// 語法格式：(): ReturnType => value
const [currentWord, setCurrentWord] = useState<string>(
  (): string => getRandomWord() // 箭頭函式的回傳值型別是string
);
```

## Recap

| 寫法範例 | 狀態 (Getter) 型別 | 更新函式 (Setter) 限制 | 評語與適用時機 |  
| :--- | :--- | :--- | :--- |  
| `useState("apple")` | `string` | 僅能傳入 `string` | 🟢 **自動推導**：初始值明確時最簡潔 |  
| `useState([])` | `any[]` | 無限制 (任意資料) | 🔴 **不安全**：初始空陣列易失去型別保護 |  
| `useState<string[]>([])` | `string[]` | 僅能傳入 `string[]` | 🟢 **推薦**：處理陣列 State 之最佳實踐 |  
| `useState<User \| null>(null)` | `User \| null` | 可傳入 `User` 或 `null` | 🟢 **推薦**：適用於資料非即時取得（如 API） |