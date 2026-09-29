---
title: "JavaScript undefined vs null - 筆記"
pubDatetime: 2026-09-29T07:06:13.937Z
tags: ["Interview Preparation","JavaScript","Frontend","Concepts"]
description: "Tags: Interview Preparation JavaScript Concepts Frontend Ta..."
hackmd_id: "ryZvRbWcMl"
---

###### Tags: `Interview Preparation` `JavaScript` `Concepts` `Frontend`


## Table of contents

## 前言  
「`undefined` 與 `null` 有什麼不同？」，這個問題表面上問的是型別定義，**實質上想知道的是「工程意圖（Engineering Intent）」與實務開發經驗的理解**。



## 核心特性與觀念總覽

* **`undefined`（非預期 / 未定義）**：代表**「屬性或變數根本不存在、尚未被賦值」**。通常是 JavaScript 引擎的預設行為。
* **`null`（明確的意圖 / 刻意賦值）**：代表**「此屬性確實存在，但目前被刻意設定為空值/暫無資料」**。這是開發者主動賦予的佔位符（Placeholder）。
* **核心差異在於「意圖 (Intent)」**：`null` 提供了明確的上下文資訊（Context），表明工程師「知道這個欄位存在，但目前沒有數值」。
* **型別差異**：`typeof undefined === 'undefined'`，而歷史包袱導致 `typeof null === 'object'`。



## 1. 核心觀念速查與比較

| 比較維度 | `undefined` | `null` |  
| :--- | :--- | :--- |  
| **語意 (Semantics)** | 值未定義 / 不存在 (Does not exist) | 刻意設定為空 (Intentional empty value) |  
| **產生來源** | 通常由 **JavaScript 引擎自動產生** | 由 **開發者主動賦予** |  
| **`typeof` 評估結果** | `'undefined'` | `'object'` (經典歷史 Bug) |  
| **轉型為數字** | `Number(undefined)` ➔ **`NaN`** | `Number(null)` ➔ **`0`** |  
| **JSON 序列化行為** | 物件屬性若為 `undefined` 會被**直接忽略剔除** | 會保留鍵名並序列化為 `null` |



## 2. 實務情境解析：為什麼要選擇 `null`？

### 程式碼範例與比較

```javascript
// 情境 A：使用 null 代表「明確知道屬性存在，但目前為空」
const user1 = {
  firstName: null // 開發者宣告：user1 有 firstName 欄位，但目前無資料
};

// 情境 B：沒有宣告屬性
const user2 = {};

// 讀取屬性測試
console.log(user1.firstName); // null
console.log(user2.firstName); // undefined
```

### 關鍵優勢：表達工程意圖 (Expressing Intent)
- 避免混淆（屬性不存在 vs 值未設定）：
  - 當讀取 `user2.firstName` 得到 `undefined` 時，我們無法區分是「資料庫漏掉傳送這個 Key」，還是「這是一個無效的物件結構」。
  - 當讀取 `user1.firstName` 得到 `null` 時，我們非常確定「這個欄位是合法的，只是資料暫時為空」。

- 提升程式碼可讀性與維護性：  
就像在程式碼中選擇使用 `map` 或 `filter` 一樣，選擇 `null` 能向團隊成員與未來的自己清楚表達程式的設計意圖（Intent）。

- API 與 TypeScript 整合更友善：  
在將資料透過 JSON 傳輸至後端或在 TypeScript 中定義型別時（如 `firstName: string | null`），明確使用 `null` 能夠確保資料結構的完整性，不會因為 `undefined` 而導致欄位在序列化過程中消失。

## 3. 回答策略   
回答這道題目時，建議分為三個層次遞進說明：

1. 基本型別定義：先說明 `undefined` 是系統預設未定義狀態，`null` 是開發者賦予的空值。  
2. 語意與意圖（核心重點）：強調軟體工程重在「意圖（Intent）」，說明 `null` 作為佔位符（Placeholder）如何幫助團隊明確辨識欄位狀態。  
3. 實務與工具層面：補充說明 `JSON.stringify` 處理兩者的差異，或在 TypeScript 中型別設計的便利性。

## FAQ
### Q1: 為什麼 `typeof null` 會回傳 `'object'`？  
這是 JavaScript 第一版留下來的歷史包袱（Bug）。在 JavaScript 最初的實作中，值是以 32 位元的結構存儲，型別標籤（Type Tag）佔用低位元的 1~3 位。當時 000 代表 `object`，而 `null` 的空指標表示剛好全為 0，因此被錯誤判定為 `object`。出於對舊有網站相容性的考慮，這個行為被沿用至今。

### Q2: 現代語法中的 Nullish Coalescing (`??`) 如何同時處理 `null` 與 `undefined`？  
`??`（空值合併運算子）正是專門為了這兩個值設計的，只有當左側的值為 `null` 或 `undefined` 時，才會使用預設值：

```javascript
const name1 = null ?? '預設名稱';      // '預設名稱'
const name2 = undefined ?? '預設名稱'; // '預設名稱'

// 比較：不會像 || 一樣誤判 0 或空字串
const count = 0;
console.log(count || 10); // 10 (誤判)
console.log(count ?? 10); // 0  (正確)
```