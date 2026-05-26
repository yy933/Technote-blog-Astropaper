---
title: "[TypeScript] Generics (泛型)  - 筆記"
pubDatetime: 2026-05-26T03:01:46.999Z
tags: ["cheatsheet","TypeScript"]
description: "Tags: TypeScript cheatsheet Table of contents 什麼是Generics? G..."
---

Tags: `TypeScript` `cheatsheet`
## Table of contents

## 什麼是Generics?
Generics（泛型） 是一種讓函式、介面或類別在定義時可以「保留型別變數」，直到實際使用時再指定具體型別的語法工具。它的目的是讓我們寫出更通用、可重複使用、且保有型別安全的程式碼。

## 為什麼需要 Generics？
假設寫一個回傳參數本身的函式（identity function）：
```ts
function identity(arg: any): any {
  return arg;
}
```
這樣雖然能運作，但 TypeScript 失去了型別資訊（傳進去什麼都變成 any）：
```ts
const result = identity("hello"); // Type 是 any ❌
```

### 用 Generics 改寫
```ts
function identity<T>(arg: T): T {]
  return arg;
}
```
這裡的 `<T>` 就是泛型參數，我們在函式中保留了 T，直到呼叫的時候才決定它是什麼型別。
```ts
const str = identity<string>("hello");  // T 是 string 
const num = identity<number>(42);       // T 是 number 
```
TS 甚至可以自動推斷型別，不需要手動寫 `<string>`：
```ts
const str = identity("hello");  // 自動推斷 T 為 string
```

## 更實用的例子：Generic Array 函式
```ts
function getFirstElement<T>(arr: T[]): T {
  return arr[0];
}

const firstNumber = getFirstElement([1, 2, 3]);     // number
const firstString = getFirstElement(["a", "b", "c"]); // string
```
可以理解為：

* `<T>`：定義一個泛型變數 T，代表任意型別（但會根據使用時的情境自動決定）。
* `arr: T[]`：參數是 T 型別的陣列。
* 回傳值`: T`：回傳陣列的第一個元素，也會是 T 型別。
### 使用範例：

```ts
const firstNumber = getFirstElement([1, 2, 3]); // 推論 T 是 number
// typeof firstNumber → number

const firstString = getFirstElement(["a", "b", "c"]); // T 是 string
// typeof firstString → string
```
這裡的 T 是一個「佔位符」(placeholder)，但一旦被推論為某種型別（如 number），整個函式內所有出現的 T 都會一致套用那個型別。

## 泛型介面
```ts
interface ApiResponse<T> {
  data: T;
  success: boolean;
}

const response1: ApiResponse<string> = {
  data: "Hello!",
  success: true
};

const response2: ApiResponse<number[]> = {
  data: [1, 2, 3],
  success: true
};
```

## Recap
| 語法                          | 說明                  |
| --------------------------- | ------------------- |
| `<T>`                       | 定義一個泛型變數 T          |
| `function fn<T>(arg: T): T` | 參數與回傳都是同一型別         |
| `interface Box<T>`          | 泛型介面                |
| `Array<T>`                  | TypeScript 原生泛型陣列寫法 |

* T 是泛型的代稱（也可以叫 U, K, V，取決於上下文）
* 泛型讓程式碼更通用、可重複使用、有型別保護
* 它們最常出現在：函式、介面、class、React 的 props/useState 等等地方

### 跟 any 有什麼不一樣？
|      | `any` | `T` (Generic) |
| ---- | ----- | ------------- |
| 靜態檢查 | 不檢查   | 有型別保護         |
| 型別推斷 | 沒有    | 根據傳入值自動推斷     |
| 安全性  | 不安全   | 安全            |
| 可讀性  | 模糊    | 更容易維護、看得懂傳入型別 |