---
title: "JavaScript Double Equals (==) vs Triple Equals (===) - 筆記"
pubDatetime: 2026-09-29T07:05:59.560Z
tags: ["Interview Preparation","JavaScript","Frontend","Concepts"]
description: "Tags: Interview Preparation JavaScript Concepts Frontend Ta..."
hackmd_id: "ByD5-1-5fl"
---

###### Tags: `Interview Preparation` `JavaScript` `Concepts` `Frontend`


## Table of contents

## 前言  
在 JavaScript 中，理解 **雙等號 `==`（Loose Equality，寬鬆相等）與三等號 `===`（Strict Equality，嚴格相等）的差異** 運作原理與隱式型別轉換（Implicit Type Coercion），是寫出無 Bug 程式碼的重中之重。



## 核心觀念總覽

* **`==`（寬鬆相等）**：僅比較**值（Value）**。若比較雙方的型別不同，JavaScript 會在背景自動進行 **隱式型別轉換（Type Coercion）** 後才進行比較。
* **`===`（嚴格相等）**：同時比較**型別（Type）與值（Value）**。只要型別不同，直接回傳 `false`，不會進行任何轉型。
* **語言趨勢**：現代 JavaScript 開發標準越來越嚴格（如 ESLint 規範、TypeScript 的普及）。
* **最佳實務**：全面強制使用 `===`（嚴格相等），避開隱式轉型帶來的不可預期邊界狀況（Edge Cases）。



## 1. 機制比較

| 比較運算子 | 名稱 | 是否進行型別轉換？ | 比較維度 | 範例結果 (`5` 與 `'5'`) |  
| :--- | :--- | :--- | :--- | :--- |  
| **`==`** | 寬鬆相等 (Loose Equality) | **會**（自動隱式轉型） | 僅比較「值」 | `5 == '5'` ➔ **`true`** |  
| **`===`** | 嚴格相等 (Strict Equality) | **不會** | 同時比較「型別」與「值」 | `5 === '5'` ➔ **`false`** |

> **底層原理說明**：
> 當執行 `5 == '5'` 時，JavaScript 引擎會在背景執行抽象相等比較演算法（Abstract Equality Comparison Algorithm），將兩者轉換為相同型別（例如呼叫 `.toString()` 或轉為 Number）後再行比對，因此會得到 `true`。



## 2. 程式碼範例

```javascript
const value1 = 5;        // Number 型別
const value2 = '5';      // String 型別

// 1. Double Equals (==) - 寬鬆相等
// 背景會進行隱式型別轉換，只檢查「值」是否相同
console.log(value1 == value2); // true

// 2. Triple Equals (===) - 嚴格相等
// 同時檢查「型別」與「值」，Number !== String
console.log(value1 === value2); // false
```

## 3. FAQ
### Q1. 為何建議一律使用 `===`？
- 避免邊界陷阱 (Edge Cases)：寬鬆相等 `==` 的轉型規則極度複雜且不直覺，容易引發隱蔽的 Bug（例如 `0 == ''` 為 `true`、`false == '0'` 為 `true`）。

- 程式碼可讀性與維護性：使用 === 能讓程式碼的意圖更明確，降低閱讀與維護成本。

- 靜態檢查與 Lint 規範：現代開發工具（如 ESLint 的 eqeqeq 規則）與 TypeScript **都將「禁止使用 ==」列為標準規範**。

### Q2. 唯一可以考慮使用 `==` 的例外情境  
在實務開發中，唯一被社群接受使用 `==` 的極少數例外，是同時檢查變數是否為 `null` 或 `undefined`：

```javascript
// 💡 使用 == null 可以一次捕捉 null 與 undefined
if (value == null) {
  // 當 value 為 null 或 undefined 時都會執行
  console.log('value 是 null 或 undefined');
}

// 等同於使用 === 的冗長寫法：
if (value === null || value === undefined) {
  console.log('value 是 null 或 undefined');
}
```

(註：在現代 JS 中，多數開發者仍偏好寫出完整的 `===` 或是利用現代語法如 Nullish Coalescing `??` 來保持極致的嚴格性。)

### Q3: 經典陷阱：`null == undefined` 與 `null === undefined` 的結果分別是什麼？
- `null == undefined` ➔ `true`：在 JavaScript 的規範中，特地定義了 `null` 與 `undefined` 在寬鬆相等時互相相等。

- `null === undefined` ➔ `false`：**兩者的型別不同**（`null` 的型別在歷史包袱中為 `object`，`undefined` 的型別為 `undefined`）。

### Q4: 經典陷阱：`[] == false` 的結果是什麼？為什麼？  
答案是 `true`！這是 `==` 最惡名昭彰的隱式轉型過程：

- 布林值 `false` 被轉為數字 `0`➔ `[] == 0`

- 物件/陣列 `[]` 透過 `ToPrimitive` 轉為原始型別，呼叫 `.toString()` 變成空字串 `""` ➔ `"" == 0`

- 空字串 `""` 被轉為數字 `0` ➔ `0 == 0`

最終結果為 `true`！

這個例子完美體現了為何在實務中必須嚴格禁止使用 `==`。