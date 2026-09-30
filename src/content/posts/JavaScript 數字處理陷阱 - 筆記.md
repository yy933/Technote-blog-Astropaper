---
title: "JavaScript 數字處理陷阱 - 筆記"
pubDatetime: 2026-09-30T10:59:02.925Z
tags: ["Interview Preparation","JavaScript","Frontend","Concepts"]
description: "Tags: Interview Preparation JavaScript Concepts Frontend Ta..."
hackmd_id: "B124a89cze"
---

###### Tags: `Interview Preparation` `JavaScript` `Concepts` `Frontend`


## Table of contents

## 前言  
「JavaScript 在處理數字時有什麼古怪的特性（Quirks）？為什麼 `0.1 + 0.2 !== 0.3`？在實務（例如處理金融金額）中你會如何防範？」這是前端面試中極常見的 **JavaScript 基礎與底層運算考題**。

JavaScript 被公認為語言特性較為靈活且偶有古怪（Quirky）的語言。面試官提出這個問題，目的在於評估開發者是否了解 **IEEE 754 雙精度浮點數（Double-Precision Floating-Point）** 的底層運算機制、安全的整數範圍限制（`Number.MAX_SAFE_INTEGER`），以及在面對貨幣與精確計算時的工程化防範手段。



## 核心觀念總覽

* **浮點數精確度陷阱（Floating Point Precision Issue）**：
  * 在 JS 中，所有一般數字（`Number` 型別）皆以 **64 位元浮點數（IEEE 754 標準）** 儲存。
  * 由於十進位的小數（如 `0.1` 和 `0.2`）轉為二進位時會變成**無限循環小數**，而在有限位元下截斷後，相加就會產生微小誤差：
    ```javascript
    0.1 + 0.2 // 回傳 0.30000000000000004
    0.1 + 0.2 === 0.3 // 回傳 false
    ```
* **大數安全極限與 BigInt**：
  * **`Number` 的安全整數範圍**：超出 `Number.MAX_SAFE_INTEGER`（$2^{53} - 1$，即 `9007199254740991`）之後，整數計算會失去精確度（Over-precision loss）。
  * **ES2020 解法（`BigInt`）**：新增的原生型別，可在整數末尾加上 `n`（如 `9007199254740992n`），允許表示任意大小的整數。
* **實務情境（Real-World Impact）**：
  * 一般 UI 顯示數字影響不大，但在處理**金額（Money）、薪資計算（Compensation）、加密貨幣或金融交易**等需要絕對精確度（Exact Precision）的場景，直接使用原生浮點數運算會造成嚴重 Bug。



## 1. 常用解法與防範機制速查

| 解決方案 / 工具 | 運作原理與機制 | 實務應用情境 | 注意事項 / 缺點 |  
| :--- | :--- | :--- | :--- |  
| **`Number.EPSILON`** | 比較兩數差值是否小於機器極小值（`Number.EPSILON`） | 自訂浮點數相等的判斷函式 | 適用於比較，而非直接格式化 |  
| **`toFixed()` / `Math.round()`** | 截斷小數點或四捨五入（`toFixed` 會回傳字串） | 適用於 UI 畫面呈現（如顯示金額至小數第二位） | `toFixed()` 在某些邊界情況會有四捨五入不精準問題 |  
| **轉為最小單位整數** | 將金額乘以 100（例如 10.5 元轉為 1050 分）進行計算 | 電商購物車結帳、金額運算 | 計算完畢後需再除回原單位 |  
| **高精度數學庫** | 使用 `decimal.js` / `big.js` / `bignumber.js` | 金融、證券、加密貨幣交易系統 | 需額外引入外部 Library |  
| **原生 `BigInt`** | ES2020 原生型別，支援任意長度整數 | 超大整數運算（如 64-bit ID、高精度計數器） | **不能**與一般 `Number` 直接混合進行算術運算，且不支援小數 |



## 2. 實務範例程式碼

### 情境 A：安全的浮點數比較與格式化

```javascript
// ❌ 錯誤方式：直接比較浮點數
console.log(0.1 + 0.2 === 0.3); // false

// ✅ 正確方式 1：利用 Number.EPSILON 進行安全比較
function areFloatsEqual(a, b) {
  return Math.abs(a - b) < Number.EPSILON;
}
console.log(areFloatsEqual(0.1 + 0.2, 0.3)); // true

// ✅ 正確方式 2：使用 toFixed() 處理顯示用字串（記得注意轉回 Number）
const result = Number((0.1 + 0.2).toFixed(2)); // 0.3
```

### 情境 B：超大整數處理（BigInt）

```javascript
// ❌ 超出安全整數範圍會失去精確度
const maxSafe = Number.MAX_SAFE_INTEGER; // 9007199254740991
console.log(maxSafe + 1 === maxSafe + 2); // true (精確度遺失！)

// ✅ 使用 BigInt 解決大數問題
const big1 = 9007199254740991n;
const big2 = big1 + 2n;
console.log(big2); // 9007199254740993n
```

## 回答策略
### 「JavaScript 在數字處理上有哪些常見的問題？在實務中你會如何保護你的程式碼？」

推薦答題架構：

- 說明原因（The Why）：  
JavaScript 的 Number 型別採用 IEEE 754 雙精度浮點數標準。這會帶來兩個主要的陷阱：第一是小數精確度問題（最經典的就是 `0.1 + 0.2 !== 0.3`），因為十進位小數轉二進位時會產生無限循環；第二是大數安全極限（超出 `Number.MAX_SAFE_INTEGER` 會遺失精確度）。

- 說明實務防範與解決方案（How & Best Practices）：
  - 針對大整數：現代 JavaScript（ES2020）提供了原生的 BigInt，可以輕鬆處理超過 53-bit 的超大整數（如 64-bit 資料庫 ID）。
  - 針對 UI 渲染與簡單計算：我們可以使用 `Math.round()`、`Number.EPSILON`，或是 `toFixed()` 來進行格式化。
  - 針對金融與精確金額計算（Money & Compensation）：如果是電商或金融專案，絕對不能直接用原生浮點數相加。實務上採用兩種做法：將單位化為最小整數進行運算（例如美金轉成美分 Cent），或者直接導入專門的高精度套件如 big.js / decimal.js，從根本上杜絕計算誤差。

## FAQ
### Q1: BigInt 可以和普通的 Number 直接相加嗎？例如 10n + 5？  
不行。 JavaScript 為了避免隱式型別轉換導致精度損失，不允許 BigInt 與 Number 直接進行混合算術運算（會拋出 `TypeError: Cannot mix BigInt and other types`）。必須先進行顯式轉換（例如 `BigInt(5)` 或 `Number(10n)`）。

### Q2: `Number.isNaN()` 和全域的 `isNaN()` 有什麼不同？
- `isNaN(value)`（舊版全域函式）：會先嘗試將傳入的參數隱式轉換（Coercion）為數字，再判斷是否為 `NaN`。例如 `isNaN("hello")` 會回傳 `true`，容易造成誤判。
- `Number.isNaN(value)`（ES6 嚴格版本）：不進行型別轉換。只有當傳入值型別為 `Number` 且值確實為 `NaN` 時才會回傳 `true`（例如 `Number.isNaN("hello")` 會回傳 `false`），是實務上推薦的防禦性寫法。