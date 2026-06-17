---
title: "[TypeScript] 型別系統：Union、Intersection、Literal、Alias - 筆記"
pubDatetime: 2025-05-29T00:04:36.000Z
modDatetime: 2026-05-25T10:04:23.490Z
tags: ["TypeScript"]
description: "Table of contents 1. Union Type（聯集型別） 讓一個變數同時接受多種型別，只要符合其中之..."
hackmd_id: "SJCzMcrfxg"
---

## Table of contents

## 1. Union Type（聯集型別）  
讓一個變數同時接受多種型別，只要符合其中之一即可。
### 基本語法：
```ts
let value: string | number;

value = "Hello"; // ✅ 
value = 123;     // ✅ 
value = true;    // ❌ 錯！boolean 不是 string 或 number
```

### 使用場景範例  
1. 表單輸入值可能是字串或數字：
```ts
function handleInput(input: string | number) {
  console.log("你輸入了", input);
}
```

2. API 回傳可能為多種狀況（例如成功或錯誤）：
```ts
type APIResponse = string | { error: string };

const res: APIResponse = "成功"; // ✅
const err: APIResponse = { error: "失敗" }; // ✅
```

3. 搭配字面值（Literal）：
```ts
type Direction = "left" | "right" | "up" | "down";

function move(dir: Direction) {
  console.log(`往 ${dir} 移動`);
}

move("left");  // ✅
move("top");   // ❌ 錯！不是指定的字面值
```

### 常見用法補充
* string | undefined：可以是字串或未定義（常見於可選參數）
* number | null：可能是數字，也可能是空值
* boolean | "yes" | "no"：可以是布林值，也可以是 "yes"/"no" 字串

## 2. Intersection Type（交集型別）  
合併多個型別，同時滿足所有條件。
```ts
type A = { name: string };
type B = { age: number };
type Person = A & B;

const p: Person = {
  name: "Ada",
  age: 30
}; // ✅ name 和 age 都要有
```
--> 用途：組合多個介面，例如物件擴充。

## 3. Literal Type（字面值型別）  
指定值只能是某個特定字串或數字。

```ts
type Direction = "up" | "down" | "left" | "right";

let move: Direction;
move = "up";      // ✅
move = "jump";    // ❌
```
--> 用途：限制選項、模擬 enum。

## 4. Type Alias（型別別名）  
為一個型別取個名字，讓重複使用更方便。

```ts
type UserID = string;
type User = {
  id: UserID;
  name: string;
};
```
--> 好處：簡潔、有語意、易維護。

## Bonus：可以結合使用

```ts
type Status = "idle" | "loading" | "success" | "error";
type User = { name: string };
type APIResponse = User | { error: string };
```