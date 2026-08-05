---
title: "[TypeScript] TypeScript 定義 Function（函式）型別: Function Overloads（函式過載）"
pubDatetime: 2026-08-05T09:50:34.544Z
tags: ["TypeScript","cheatsheet","Type system","Concepts"]
description: "Table of contents 什麼是 Function Overloads（函式過載）？ 在 JavaScrip..."
---

## Table of contents

## 什麼是 Function Overloads（函式過載）？  
在 JavaScript 中，一個函式可以根據傳入參數的數量或型別不同，回傳完全不同結構的結果。

但在 TypeScript 中，如果只用傳統的 Union Types（如 `string | number`）來標註，**TypeScript 無法精確掌握「特定輸入對應特定輸出」的連動關係。**

**Function Overloads（函式過載） 允許我們為同一個函式定義多個型別簽名（Overload Signatures）**，精確告訴 TypeScript：

* 「傳入 `string` 時，回傳值一定是 `string`」
* 「傳入 `number` 時，回傳值一定是 `number`」

## 為什麼需要 Function Overloads？  
假設我們想撰寫一個 `makeDate` 函式：

* 傳入 1 個 timestamp（`number`）➔ 建立 Date
* 傳入 3 個參數 `year`, `month`, `day`（`number`, `number`, `number`）➔ 建立 Date

### 嘗試用 Union Types（遭遇型別曖昧）

```typescript
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}

// 呼叫端沒問題：
makeDate(12345678);           // ✅ 正常
makeDate(1, 15, 2026);        // ✅ 正常

// ❌ 問題發生：呼叫端傳入 2 個參數，邏輯上不合理，但 TS 無法有效阻止！
makeDate(1, 15);              // ⚠️ TS 不會報錯，但會建立非預期的 Date

```

## Function Overloads 的結構與語法  
Function Overloads 的撰寫分為兩個部分：

* 重載簽名（Overload Signatures）：唯有這裡定義的簽名才對外部呼叫端公開。
* 實作簽名（Implementation Signature）：內部實作邏輯，必須相容所有重載簽名（外部無法直接看到/呼叫此簽名）。

### 語法結構範例

```typescript
// ==========================================
// 1. 重載簽名 (Overload Signatures) - 對外公開
// ==========================================
function formatInput(input: string): string;
function formatInput(input: number): number[];

// ==========================================
// 2. 實作簽名 (Implementation Signature) - 內部處理
// ==========================================
function formatInput(input: string | number): string | number[] {
  if (typeof input === "string") {
    return input.trim().toUpperCase();
  }
  return Array.from({ length: input }, (_, i) => i + 1);
}

// ==========================================
// 3. 測試呼叫（獲得極度精確的推導）
// ==========================================
const res1 = formatInput("hello"); // ✅ res1 型別精準為 string
const res2 = formatInput(5);       // ✅ res2 型別精準為 number[]

// ❌ 錯誤：不符合任何一個重載簽名
// No overload matches this call.
formatInput(true); 
```

## ⚠️ 關鍵陷阱：實作簽名外部不可見  
初學者最常犯的錯誤是誤以為外部可以呼叫實作簽名的型別組合。


```typescript
function combine(a: string, b: string): string;
function combine(a: number, b: number): number;
// 實作簽名使用了 Union Type
function combine(a: string | number, b: string | number): string | number {
  if (typeof a === "string" && typeof b === "string") return a + b;
  if (typeof a === "number" && typeof b === "number") return a + b;
  throw new Error("Invalid arguments");
}

combine("hello", "world"); // ✅ 回傳 string
combine(10, 20);           // ✅ 回傳 number

// ❌ 錯誤！即使實作簽名寫了 (string | number)，但外部「只能」使用重載簽名！
// No overload matches this call.
combine("hello", 20); 
```

## 重載 vs. 泛型 (Generics) / Conditional Types  
並非所有多型別情境都要用 Overloads，**有時使用 Generics 或 Conditional Types 會更簡潔：**

### 1. 什麼時候用 Generics？  
當輸入與輸出的型別完全同源時：

```typescript
// 🟢 適合泛型：輸入什麼型別，就回傳什麼型別
function identity<T>(arg: T): T {
  return arg;
}
```

### 2. 什麼時候用 Function Overloads？
**當輸入不同的型別/參數數量，會觸發完全不同的邏輯與輸出結構時。**

## Recap

| 比較維度 | 重載簽名 (Overload Signatures) | 實作簽名 (Implementation Signature) |  
| :--- | :--- | :--- |  
| **宣告數量** | 可以有 2 個或多個 | 只能有 1 個 |  
| **函式主體 `{}`** | ❌ 沒有函式主體，僅有型別定義 | 🟢 必須包含實際執行的程式碼主體 `{}` |  
| **對外能見度** | 🟢 **對外公開**，決定呼叫端能否通過型別檢查 | ❌ **對外隱藏**，僅供內部防禦與轉譯 |  
| **型別精度** | 極高，精確對應「特定輸入➔特定輸出」 | 較寬鬆，通常需包含各種條件的 Union |