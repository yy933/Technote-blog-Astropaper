---
title: "[TypeScript] 陣列型別（Array Type） - 筆記"
pubDatetime: 2026-05-26T03:01:47.007Z
tags: ["TypeScript"]
description: " Table of contents 基本語法 寫法 範例 說明 T[] string[], number[] 常見、簡..."
---

## Table of contents

## 基本語法
| 寫法         | 範例                     | 說明           |
| ---------- | ---------------------- | ------------ |
| `T[]`      | `string[]`, `number[]` | 常見、簡潔的語法     |
| `Array<T>` | `Array<string>`        | 與泛型語法一致，語意清楚 |

```typescript
let names: string[] = ['Ada', 'Grace']; // 只有字串的陣列
let scores: Array<number> = [95, 88]; // 只有數字的陣列
let scores: (string | number)[]; // 混合類型的陣列
scores = ['Programming', 5, 'Software Design', 4];
```

兩種寫法的使用場景:
| 場景        | 推薦語法                 |
| --------- | -------------------- |
| 日常宣告、變數型別 | ✅ `T[]`（簡潔）          |
| 函式泛型參數    | ✅ `Array<T>`（統一泛型語法） |
```ts
function getFirst<T>(arr: Array<T>): T {
  return arr[0];
}
```

### 新增值的方式
```ts
let skills: string[] = [];
skills[0] = "Problem Solving";
skills[1] = "Programming";
skills.push("Software Design");

// 一旦定義了類型為 string[]，則無法插入非 string 值
skills.push(100); // ❌ Error: Argument of type 'number' is not assignable to parameter of type 'string'.
```

## 不能修改的陣列（ReadonlyArray）
若陣列不該被修改，使用 `readonly` 保護：

```ts
const readonlyList: readonly string[] = ['a', 'b'];
// readonlyList.push('c'); ❌ 錯誤：不能修改

const genericReadonly: ReadonlyArray<number> = [1, 2, 3];
```

## 二維 / 多維陣列
```ts
const matrix: number[][] = [
  [1, 2],
  [3, 4],
];
```
等同於：
```ts
const matrix: Array<Array<number>>;
```

## 陣列型別推論
TypeScript 能自動推論，但明確指定型別可增加可讀性與錯誤提示力：

`const tags = ['typescript', 'javascript']; // 推論 string[]`

可明確指定型別防止意外：

`const tags: string[] = ['typescript', 'javascript'];`

## Array 的屬性和方法
TypeScript array 和 JavaScript array 共用一樣的屬性與方法:

```typescript
let series = [1, 2, 3];
console.log(series.length); // 3

let doubleIt = series.map(e => e * 2);
console.log(doubleIt); // [2, 4, 6]

const nums: number[] = [1, 2, 3];

nums.map(n => n * 2);        // number[]
nums.filter(n => n > 1);     // number[]
nums.reduce((acc, n) => acc + n, 0); // number
```

常用方法: `forEach()`, `map()`, `reduce()`, `filter()` 等等

## 陣列與物件型別結合

```ts
type User = { name: string; age: number };

const users: User[] = [
  { name: 'Ada', age: 36 },
  { name: 'Alan', age: 45 }
];
```
也可以這樣寫:
```ts
const users: Array<User> = [
  { name: 'Ada', age: 36 },
  { name: 'Alan', age: 45 }
];
```

## Tuple
Tuple是一種固定長度、固定類型順序的陣列，可以把它想成「每個位置都規定好資料型別」的陣列。
範例：
```ts
const rgb: [number, number, number] = [255, 0, 128]; // 只能有三個元素，順序固定
```
* 只能有三個元素
* 第一個必須是 number，第二個也是 number，第三個還是 number
* 如果寫 [255, 0] 或 [255, '0', 128] 都會錯！

## 延伸閱讀：泛型（Generic） 函式

```ts
function reverseArray<T>(input: T[]): T[] {
  return input.slice().reverse();
}

reverseArray<number>([1, 2, 3]); // [3, 2, 1]
```
`function reverseArray<T>(input: T[]): T[]`
這是在宣告一個泛型函式 `reverseArray`：

* `T` 是一個泛型參數，代表任何資料型別（可以是 number、string、User 等等）。
* `input: T[]` 表示這個函式的參數是「T 類型的陣列」。
* `: T[]` 表示這個函式會回傳「T 類型的陣列」。

> :bulb: 換句話說，這是一個可以接受「任何型別陣列」並回傳「同樣型別陣列」的函式。

## Recap
| 目的     | 寫法範例                                          |
| ------ | --------------------------------------------- |
| 宣告基本陣列 | `string[]` / `Array<string>`                  |
| 泛型函式參數 | `Array<T>`                                    |
| 防止修改   | `readonly string[]` / `ReadonlyArray<string>` |
| 二維陣列   | `number[][]`                                  |
| 固定長度   | Tuple，如 `[string, number]`                    |