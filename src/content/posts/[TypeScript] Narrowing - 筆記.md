---
title: "[TypeScript] Narrowing - 筆記"
pubDatetime: 2025-05-29T00:35:52.000Z
modDatetime: 2026-08-04T09:52:09.598Z
tags: ["TypeScript","cheatsheet","Type system"]
description: "為什麼需要 Type Narrowing？ 當變數為 Union Type（聯集型別）..."
hackmd_id: "SyRNV5Szxe"
---

## Table of contents

## 為什麼需要 Type Narrowing？  
當變數為 **Union Type（聯集型別）**（例如 `number | string`）時，TypeScript 在編譯期無法確定變數到底是什麼型別，因此會限制你只能呼叫兩者共同的屬性與方法。

**Narrowing（型別窄化）** 就是透過程式碼邏輯判斷，讓 TypeScript 自動推導出變數在特定區塊內的精確型別。



## 1. 基礎控制流窄化 (Control Flow Analysis)

### 1) `typeof` 窄化（原始型別）  
用於判斷 JavaScript 的 7 種原始型別與函式：  
`"string"` | `"number"` | `"bigint"` | `"boolean"` | `"symbol"` | `"undefined"` | `"object"` | `"function"`

```ts
function processValue(val: number | string) {
  if (typeof val === "number") {
    console.log(val.toFixed(2)); // val 窄化為 number
  } else {
    console.log(val.toUpperCase()); // val 窄化為 string
  }
}
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡 注意：`typeof null === "object"`  
JavaScript 的歷史遺留問題導致 `typeof null` 會回傳 `"object"`，若只寫 `typeof x === "object"` 無法排除 `null`！

```typescript
function printAll(strs: string[] | null) {
  if (typeof strs === "object") {
    // ⚠️ 此處 strs 依然是 string[] | null！
  }
}

// ✅ 正確做法：使用 Array.isArray 或判斷 Truthiness
if (Array.isArray(strs)) {
  // strs 明確為 string[]
}
```

</blockquote>

### 2) in 運算子窄化（物件屬性）  
用於檢查物件是否擁有特定屬性，特別適合區分不同的介面或型別。

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(pet: Fish | Bird) {
  if ("swim" in pet) {
    return pet.swim(); // pet 窄化為 Fish
  }
  return pet.fly(); // pet 窄化為 Bird
}
```

### 3) instanceof 窄化（類別與內建物件）  
用於檢查值是否為某個建構函式或 `Class` 的實例（如 `Date`, `Error`, `RegExp` 或自訂 `Class`）。

```typescript
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString()); // x 窄化為 Date
  } else {
    console.log(x.toUpperCase()); // x 窄化為 string
  }
}
```

### 4) 相等性窄化 (Equality Narrowing)  
透過 `===`、`!==`、`==`、`!=` 進行比對。

```typescript
// 1. 交集推導
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // x 與 y 的交集只有 string，因此兩者皆窄化為 string
    x.toUpperCase();
    y.toLowerCase();
  }
}

// 2. 使用 == null 同時排除 null 與 undefined
function check(value: string | null | undefined) {
  if (value != null) {
    // 排除 null 和 undefined，剩餘 string
    console.log(value.toUpperCase());
  }
}
```


## 2. 進階實務窄化技巧 (Advanced Narrowing)
### 1) Discriminated Unions（可辨識聯集 / 標籤聯集）
**實務上最推薦的物件窄化方式！**  
在每個型別中設計一個相同的唯讀字面值欄位（通常稱為 kind 或 type），作為辨識標籤。


```typescript
type NetworkLoadingState = { state: "loading" };
type NetworkFailedState = { state: "failed"; code: number };
type NetworkSuccessState = { state: "success"; response: { title: string } };

type NetworkState = NetworkLoadingState | NetworkFailedState | NetworkSuccessState;

function handleResponse(status: NetworkState) {
  switch (status.state) {
    case "loading":
      return "Downloading...";
    case "failed":
      return `Error ${status.code}`; // status 窄化為 NetworkFailedState
    case "success":
      return `Data: ${status.response.title}`; // status 窄化為 NetworkSuccessState
  }
}
```


### 2) Custom Type Guards (自訂型別謂詞 `is`)  
當窄化邏輯太複雜，需要抽成獨立函式時，必須使用 `parameterName is Type` 作為回傳值型別，否則 TypeScript 無法跨函式傳遞窄化結果。

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };

// 使用 is 關鍵字定義 Type Guard
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function handlePet(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim(); // pet 成功窄化為 Fish
  } else {
    pet.fly(); // pet 窄化為 Bird
  }
}
```

### 3) Exhaustive Checking（使用 `never` 做完整性檢查）  
利用 `never` 型別的特性，確保在處理 Discriminated Unions 時沒有漏掉任何一種分支狀況。

```typescript
type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    // ⚠️ 如果漏寫了 "triangle" 的處理邏輯：
    default:
      // 編譯期會直接報錯：Type 'Triangle' is not assignable to type 'never'
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

## Recap

| Narrowing 方式 | 適用對象 | 範例語法 | 特殊注意 / 實務建議 |  
| :--- | :--- | :--- | :--- |  
| **`typeof`** | 原始型別 | `typeof x === "string"` | `typeof null` 是 `"object"` 需額外處理 |  
| **`in`** | 物件與介面 | `'swim' in pet` | 檢查物件是否包含特定屬性 key |  
| **`instanceof`** | Class 與內建物件 | `x instanceof Date` | 適合判斷 `Error`、`Date`、`RegExp` |  
| **相等性 (== / ===)** | 任意聯集 | `x != null` | `!= null` 可同時過濾 `null` 和 `undefined` |  
| **Discriminated Union** | 多重物件聯集 | `switch(obj.type)` | 💡 **大型專案與 API 狀態管理首選** |  
| **Type Guard (`is`)** | 自訂複雜邏輯 | `fn(x): x is Fish` | 用於將判定邏輯抽成獨立函式時 |  
| **`never` 完整性檢查** | Switch 分支保護 | `const _check: never = val` | 防止新增 Union 成員時少寫邏輯 |

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>