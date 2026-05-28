---
title: "[TypeScript] Narrowing - 筆記"
pubDatetime: 2025-05-29T00:35:52.000Z
modDatetime: 2026-05-25T10:04:23.574Z
tags: ["TypeScript","cheatsheet"]
description: "Tags: TypeScript cheatsheet Table of contents 為什麼需要 Type Nar..."
hackmd_id: "SyRNV5Szxe"
---

Tags: `TypeScript` `cheatsheet`
## Table of contents

## 為什麼需要 Type Narrowing？
當變數類型是 union type（例如 number | string），我們無法直接對變數使用該型別的方法。這時需要 **narrowing** 來讓 TypeScript 確認你正在處理哪一種型別。

## Type Guard 是什麼？
Type Guard 是讓 TypeScript 根據條件判斷變數的實際型別，從而正確推斷型別的方式。

## 基本範例

```ts
function padLeft(padding: number | string, input: string): string {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```
* `typeof padding === "number"` 是 type guard，TypeScript 在此行內會判定 padding 為 `number`，否則就視為 string。
* Narrowing 會依照程式邏輯路徑，自動推導變數的具體型別。

## 1. 使用 typeof 判斷原始型別
```ts
function roughAge(age: number | string) {
  if (typeof age === 'number') {
    console.log(Math.round(age)); // age 是 number
  } else {
    console.log(age.split(".")[0]); // age 是 string
  }
}
```
可用 `typeof` 判斷的型別有：

```
* "number"
* "string"
* "boolean"
* "object"
* "undefined"
* "function"
* "symbol"
* "bigint"
```
> 「function」不是一個 type，而是 JS 的一種 object 實體。
雖然 `typeof fn === "function"` 是可以的，但 function 不是 TypeScript 中的型別，而是語言層級的概念。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡 注意：`typeof null === "object"` 是 JavaScript 歷史的奇怪特例！
```ts
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    // Type: string[] | null (因為 typeof null 是 "object")
  }
}
```
實際上 `null` 並不是物件，只是 `typeof` 的行為如此。

更安全的處理方式：
```ts
if (Array.isArray(strs)) {
  // 明確為 string[]，排除null
} else if (typeof strs === 'string') {
  // 是 string
} else {
  // 是 null
}
```

</blockquote>

## 2. `if-else` 形式的 Type Guard
當 if 判斷某一型別後，else 區塊會自動推斷為剩餘的型別。

```ts
function processAnswer(answer: number | boolean) {
  if (typeof answer === 'number') {
    console.log(choices[answer]); // answer 是 number
  } else {
    console.log(answer ? 'YES' : 'NO'); // answer 是 boolean
  }
}
```
## 3. if + return 形式的 Type Guard
若 `if` 裡有 `return`，之後的區塊會自動被視為處理剩下的型別。
```ts
function formatAge(age: number | string) {
  if (typeof age === 'number') {
    return age.toFixed(); // age 是 number
  }
  return age; // age 是 string
}
```
## 4. 使用 in 判斷物件屬性
對 union 型別中的物件，可以用 in 判斷屬性來進行 type narrowing。
```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(pet: Fish | Bird) {
  if ('swim' in pet) {
    return pet.swim(); // pet 是 Fish
  }
  return pet.fly(); // pet 是 Bird
}
```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

`swim: () => void` 表示`swim` 是一個「不接受參數」且「不回傳任何東西」的函式。
:bulb: 延伸範例：有參數、有回傳值
```ts
type Talker = {
  speak: (words: string) => string;
};

const person: Talker = {
  speak: (words) => `你說了：${words}`
};

console.log(person.speak('哈囉')); // 你說了：哈囉
```

* React中常見的函式型別:
```ts
onClick: () => void
```
表示這是一個點擊事件的 handler，不接受參數，也沒有回傳值。


</blockquote>
## Truthiness 檢查
```ts
if (value) { ... }
```
以下值會視為 false：

* `0`
* `NaN`
* `""`（空字串）
* `0n` （BigInt 型別的值，代表「值為 0 的 BigInt」）
* `null`
* `undefined`

範例：
```ts
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```
> ⚠️ 小心！`if (strs)` 可能會誤判空字串為無效值。

## Equality narrowing
```ts
if (x === y) { ... }
```

範例：
```ts
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // x 和 y 都 narrow 為 string，因為 string | number 和 string | boolean 的交集是 string。TypeScript 會根據比較後的結果自動推斷
    x.toUpperCase();
    y.toLowerCase();
  }
}
```
比較 literal 值也可用：

```ts
if (strs !== null) {
  // 排除 null，剩下 string | string[]
}
```
`== null` 可以同時排除 null 和 undefined：

```ts
if (container.value != null) {
  // value: number
}
```
## Recap
| Narrowing方式          | 適用型別                   | 範例語法                       |
| --------------- | ---------------------- | -------------------------- |
| `typeof`        | 原始型別（number, string 等） | `typeof x === 'number'`    |
| `in`            | 物件                     | `'prop' in obj`            |
| `if` + `return` | 任意 union               | `if (條件) return ...`       |
| `if` + `else`   | 任意 union               | `if (A) {...} else {B...}` |