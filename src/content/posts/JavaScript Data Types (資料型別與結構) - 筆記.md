---
title: "JavaScript Data Types (資料型別與結構) - 筆記"
pubDatetime: 2026-09-29T07:06:31.055Z
tags: ["Interview Preparation","JavaScript","Frontend","Concepts"]
description: "Tags: Interview Preparation JavaScript Concepts Frontend Ta..."
hackmd_id: "SylUBzZqGg"
---

###### Tags: `Interview Preparation` `JavaScript` `Concepts` `Frontend`


## Table of contents

## 前言  
「JavaScript 有哪些資料型別？」看似是一道非常基礎開放的題目，但它能直接看出面試者的**知識深度**與**語言更新程度**。單純回答出 `string` 或 `number` 只能算及格；若能清晰劃分 **Primitive（原始型別）** 與 **Non-Primitive（非原始型別/物件型別）**，並主動展現對現代 JS 特性（如 `BigInt`、`Symbol`、`Map`、`Set`）的理解，便能展現資深工程師的技術廣度。



## 核心特性與觀念總覽

* **Primitive Types（原始型別）**：不可變（Immutable）、傳值（Pass by Value）、直接存在 Stack（堆疊）記憶體中。
* **Non-Primitive / Objects（非原始型別/物件型別）**：可變（Mutable）、傳址/傳參考（Pass by Reference）、存在 Heap（堆積）記憶體中。
* **現代 JS 延伸資料結構**：除了基礎陣列與物件外，MDN 規範中的內建物件（Built-in Objects）如 `Map` 與 `Set` 提供了更靈活且專門的集合管理功能。



## 1. 核心資料型別分類與速查

| 大類 | 型別 / 結構 (Data Type / Structure) | 說明與特點 | 範例 / 語法 |  
| :--- | :--- | :--- | :--- |  
| **Primitives** | **Boolean** | 布林值 | `true`, `false` |  
| | **String** | 字串 | `'Hello'`, `"World"` |  
| | **Number** | 雙精度浮點數（所有數字包含整數與小數） | `42`, `3.14`, `NaN`, `Infinity` |  
| | **Null** | 刻意設定的空值 | `null` |  
| | **Undefined** | 未定義 / 未賦值 | `undefined` |  
| | **BigInt** *(現代進階)* | 可安全表示超過 `2^53 - 1` 的任意精度大整數 | `9007199254740991n` 或 `BigInt()` |  
| | **Symbol** *(現代進階)* | 獨一無二且不可變的唯一識別值，常用於物件私有屬性 Key | `Symbol('id')` |  
| **Non-Primitives** | **Object** | 鍵值對（Key-Value Pairs）的基礎物件結構 | `{ name: 'Alice', age: 25 }` |  
| *(Objects & Collections)* | **Array** | 有序的資料列表（底層亦為 Object） | `[1, 2, 3]` |  
| | **Map** *(現代進階)* | 鍵值對集合，**鍵（Key）可以是任何型別**（包含物件/函式） | `new Map()` |  
| | **Set** *(現代進階)* | 元素不重複的唯一值集合 (Unique Values) | `new Set([1, 2, 2, 3])` ➔ `{1, 2, 3}` |



## 2. 程式碼範例與進階特性展現

```javascript
// --------------------------------------------------
// 1. Primitives: 展現對現代 JS 特性的熟悉度
// --------------------------------------------------

// BigInt：解決超大數字精度遺失問題
const maxSafeInt = Number.MAX_SAFE_INTEGER; // 9007199254740991
const bigNum = 90071992547409919999n;      // 注意末尾的 n

// Symbol：保證值一定是獨一無二的 (Unique Value)
const sym1 = Symbol('description');
const sym2 = Symbol('description');
console.log(sym1 === sym2); // false (即使描述相同，依然是不同實體)


// --------------------------------------------------
// 2. Non-Primitives & Built-in Objects: Map 與 Set
// --------------------------------------------------

// Set：自動過濾重複值的列表中介（Array 替代/補充）
const uniqueNumbers = new Set([1, 2, 2, 3, 4, 4]);
console.log([...uniqueNumbers]); // [1, 2, 3, 4]

// Map：鍵名不限於字串的鍵值對（Object 替代/補充）
const userMap = new Map();
const userObjKey = { id: 1 };

// 使用「物件」作為 Key
userMap.set(userObjKey, { role: 'Admin' });
console.log(userMap.get(userObjKey)); // { role: 'Admin' }
```

## 3. 回答策略 (Interview Strategy)
- 大架構劃分：  
先說明 JavaScript 的型別分為 Primitive（原始型別） 與 Non-Primitive（物件/引用型別） 兩大類，並簡單說明記憶體存取模式的不同（Pass by Value vs Pass by Reference）。

- 列舉基礎型別：  
迅速列出常見的 `Boolean`, `String`, `Number`, `Null`, `Undefined`。

- 主動提及現代 JS 型別 (加分項)：  
強調 ES6+ 引入的 `Symbol`（用於唯一識別標籤）與 `BigInt`（用於處理大數運算），證明自己的知識庫保持在最新狀態。

- 拓展至專門的資料結構 (展現工程深度)：  
補充說明除了基本 `Object` 與 `Array` 外，MDN 規範中實務常用的 `Map`（鍵名型別不限的鍵值對）與 `Set`（不重複值的陣列結構），說明它們如何簡化許多常見的開發情境（如陣列去重）。

## FAQ
### Q1: `typeof` 運算子可以用來檢測所有型別嗎？有哪些特例與陷阱？  
`typeof` 適合用來檢查大部分 Primitive 型別，但有以下經典陷阱：

`- typeof null` ➔ `'object'`（歷史 Bug）。
- `typeof []` ➔ `'object'`（陣列本質是物件；要檢查陣列應使用 `Array.isArray(arr)`）。
- `typeof NaN` ➔ `'number'`（`NaN` 代表「非數值」，但其型別分類屬於 `Number`）。

### Q2: 實務上什麼時候該用 `Map` 而不是一般的 `Object`？
- 需要非字串 Key 時：當你需要用物件、DOM 元素或數字作為 Key 時。
- 頻繁增刪資料與效能需求：`Map` 在頻繁插入與刪除鍵值對時，效能表現優於一般 `Object`。
- 需要保持順序與大小：`Map` 會嚴格保留 Key 插入的順序，且可以直接透過 `map.size` 取得長度，無須呼叫 `Object.keys(obj).length`。