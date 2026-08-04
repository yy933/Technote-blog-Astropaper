---
title: "[TypeScript] Generics (泛型) 基礎介紹與應用 - 筆記"
pubDatetime: 2026-08-04T10:48:54.796Z
tags: ["others"]
description: "為什麼需要 Generics（泛型）？ 在開發工具函式（Utility Funct..."
hackmd_id: "Bk2rgS18Gx"
---

## Table of contents

## 為什麼需要 Generics（泛型）？  
在開發工具函式（Utility Functions）時，我們經常會遇到「邏輯相同，但處理的資料型別不同」的情境（例如：取得陣列最後一個元素、呼叫 API 取得不同格式的 response）。

* 寫死具體型別（如 `number[]`）：程式碼無法重用，處理 `string[]` 就必須再寫一個函式。
* 使用 `any`：雖然能接受任意資料，但會完全關閉 TypeScript 的型別檢查，失去 IntelliSense 自動補全與型別安全。

**Generics（泛型） 就是為了解決這個矛盾——它允許我們將「型別本身」當成參數傳遞，在保持高彈性（Flexibility）的同時，依然擁有 100% 的型別安全（Type Safety）。**

## 什麼是 Generic？  
可以把 Generic 想成是「型別的占位符（Placeholder）」或「型別的參數」：

* 函式參數（Function Parameter）：值的占位符（傳入具體數值/字串）。
* 泛型（Generic Parameter）：型別的占位符（傳入具體型別 `number` / `string` / `User`）。

## 語法與範例
### 1. 基礎語法結構  
使用角括號 `< >` 將泛型名稱宣告在函式名稱與參數括號之間：

```typescript
function getLastItem<T>(array: T[]): T | undefined {
  return array[array.length - 1];
}
```
* `<T>`：宣告泛型參數，代表「某種尚未確定的型別」。
* `array: T[]`：限制傳入的參數必須是該型別的陣列。
* `: T | undefined`：明確標註回傳值型別（若陣列為空可能回傳`undefined`）。


<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

**💡 命名慣例（Naming Convention）**  
角括號內的名稱可自訂（如 `<Type>`、`<ItemType>`），但社群最常用的慣例是使用單大寫字母：

* T：代表 Type（最常用）
* U, V：代表第二、第三個泛型參數
* K, V：代表 Key, Value（常用於字典或物件操作）
* E：代表 Element（常用於集合或陣列元素）

</blockquote>
    
### 2. TypeScript IntelliSense 自動推導 (Type Inference)  
呼叫帶有泛型的函式時，通常不需要手動指定型別，TypeScript 會根據你傳入的引數自動推導（Infer）出 `T` 的具體型別： 

```typescript
const gameScores = [14, 21, 33, 42, 59];
const favoriteThings = ["raindrops", "whiskers", "kittens"];
const voters = [{ name: "Alice", age: 42 }, { name: "Bob", age: 77 }];

// 1. 自動推導 T 為 number => 回傳 number | undefined
const lastScore = getLastItem(gameScores); 

// 2. 自動推導 T 為 string => 回傳 string | undefined
const lastThing = getLastItem(favoriteThings); 

// 3. 自動推導 T 為 { name: string; age: number } => 回傳物件型別
const lastVoter = getLastItem(voters);
```

若有需要，也可以在呼叫時明確指定（Explicitly Pass）泛型：

```typescript
const lastScore = getLastItem<number>(gameScores);
```

## `any` vs Generics 差異比較  
以 `getLastItem` 為例，比較三種寫法的優缺點：

```typescript
// ❌ 寫死型別：缺乏彈性
function getLastNumber(array: number[]): number { ... }

// ❌ 使用 any：失去型別安全，回傳值型別變成 any
function getLastAny(array: any[]): any { ... }

// ✅ 使用 Generics：兼具彈性與型別安全，回傳值會精準對應傳入型別
function getLastGeneric<T>(array: T[]): T | undefined { ... }
```

## Recap

| 撰寫方式 | 彈性 / 重用性 | 型別安全性 | IntelliSense 自動補全 |  
| :--- | :--- | :--- | :--- |  
| **寫死具體型別** (`number[]`) | 🔴 低（只能處理單一型別） | 🟢 高 | 🟢 精準 |  
| **使用 `any`** (`any[]`) | 🟢 高 | 🔴 無（關閉 TS 檢查） | 🔴 失去提示，易有 Runtime Error |  
| **使用 Generics** (`T[]`) | 🟢 高 | 🟢 高 | 🟢 精準（自動推導對應型別） |

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>