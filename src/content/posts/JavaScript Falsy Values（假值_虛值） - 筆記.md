---
title: "JavaScript Falsy Values（假值/虛值） - 筆記"
pubDatetime: 2026-09-29T07:05:38.407Z
tags: ["Interview Preparation","JavaScript","Frontend","Concepts"]
description: "Tags: Interview Preparation JavaScript Concepts Frontend Ta..."
---

###### Tags: `Interview Preparation` `JavaScript` `Concepts` `Frontend`


## Table of contents

## 前言  
在 JavaScript 中，**Falsy Values（假值/虛值）** 是一個非常核心的觀念。當 JavaScript 在進行邏輯判斷（例如 `if` 條件式或布林值轉譯）時，有些非布林型別的值會被自動強制轉型（Type Coercion）為 `false`。


## 核心特性與觀念總覽

* **定義**：在布林值情境（Boolean Context）中被評估（Evaluate）為 `false` 的特定值。
* **經典 6 大 falsy values**：`false`、`0`（包含 `-0` 與 `0n`）、`""`（空字串）、`null`、`undefined` 以及 `NaN`（Not-a-Number）。
* **優點**：可簡寫條件判斷，讓程式碼更精簡（例如判斷字串或陣列是否非空）。
* **缺點與陷阱**：過度依賴隱式轉換容易引發 Edge Cases（邊界條件錯誤），例如將數值 `0` 誤判為「沒有資料」。
* **最佳實務**：在實務開發中，建議採取**顯式判斷（Explicit Checking）**（如 `word.length > 0` 或 `num !== undefined`），配合 ESLint 規範以提升程式碼可讀性與安全性。



## 1. JavaScript 中的 6 大 Falsy Values

當將以下值放入 `if (value)` 判斷式中時，程式碼區塊**完全不會被執行**：

| 假值 (Falsy Value) | 型別 (Type) | 說明 / 評估結果 |  
| :--- | :--- | :--- |  
| **`false`** | Boolean | 布林值的假值本身 |  
| **`0`** | Number | 數字零（包含 `-0`, `0n` BigInt） |  
| **`""`** | String | 長度為 0 的空字串（`''` 或 `""`） |  
| **`null`** | Null | 代表「無/空」的物件值 |  
| **`undefined`** | Undefined | 未定義或未賦值的變數 |  
| **`NaN`** | Number | 代表「非數值（Not-a-Number）」的計算結果 |

> **特別注意（常見陷阱）**：
> 空陣列 `[]` 與 空物件 `{}` 在 JavaScript 中**都是 Truthy（真值）**！放入 `if ([])` 仍然會被評估為 `true`。



## 2. 實務使用優缺點與潛在陷阱（Caveats）

### 1. 使用假值的好處：程式碼簡短 (Concise Code)  
在簡單的情境下，利用假值可以減少冗長的比較：
```javascript
let inputWord = '';

// 利用空字串是 Falsy 的特性，快速過濾沒輸入內容的情況
if (!inputWord) {
  console.log('使用者未輸入內容');
}
```

### 2. 使用假值的陷阱 (Caveats & Edge Cases)  
當數值 0 或空字串 "" 在業務邏輯中代表「有意義的資料」時，使用假值判斷會導致 Bug：

範例：數量設定為 0 時的誤判

```javascript
let itemQuantity = 0; // 數量為 0 是合理的業務數值

// ❌ 不推薦：0 被當作 Falsy，導致誤以為使用者「未設定數量」
if (!itemQuantity) {
  itemQuantity = 10; // 錯誤地覆蓋了原本設定的 0
}

// ✅ 推薦：明確判斷 undefined 或 null (顯式判斷)
if (itemQuantity === undefined || itemQuantity === null) {
  itemQuantity = 10;
}
```

## 3. 程式碼實作與重構比較  
依賴 Falsy 轉型的寫法 (Implicit)

```javascript
/**
 * 依賴 Falsy 進行條件評估
 */

if (null) console.log('null');           // 不會執行
if (undefined) console.log('undefined'); // 不會執行
if (false) console.log('false');         // 不會執行

const arr = [];
// ⚠️ 雖然空陣列 [] 是 Truthy，但 arr.length 是 0 (Falsy)
if (arr.length) {
  console.log('陣列有內容');
}

const word = '';
// ⚠️ word 長度為 0 (Falsy)
if (word.length) {
  console.log('字串有內容');
}
```

### 顯式判斷的最佳實務寫法 (Explicit & Defensive)  
為了避免 Edge Cases 並滿足嚴格的 Linting 規範，實務上推薦明確寫出比較條件：

```javascript
/**
 * 防禦性與顯式判斷寫法 (Explicit Checking)
 */

const arr = [];

// ✅ 明確檢查長度是否大於 0
if (arr.length > 0) {
  console.log('陣列有內容');
}

const word = '';

// ✅ 明確檢查字串長度
if (word.length > 0) {
  console.log('字串有內容');
}
```

## FAQ
### Q1: `null` 與 `undefined` 在轉型布林值時都是 Falsy，兩者在語意上有何不同？
- `undefined`：代表 **「變數尚未被賦值」或「未定義」** 。例如宣告了變數但未給值、函式沒有回傳值（Return）時的預設結果。

- `null`：代表「**開發者主動賦予的空值**」，表示該變數目前沒有指向任何物件值或實體。

### Q2: 實務上如何只針對 `null` 與 `undefined` 做判斷，而不誤判 `0` 或 `""`？  
可以**使用 Nullish Coalescing Operator（空值合併運算子 `??`）**：

```javascript
const count = 0;

// ❌ 使用邏輯或 ||：0 會被視為 Falsy，而使用預設值 10
const result1 = count || 10; // 結果為 10

// ✅ 使用空值合併 ??：只有當 count 為 null 或 undefined 時才會使用預設值
const result2 = count ?? 10; // 結果為 0 (符合預期)
```