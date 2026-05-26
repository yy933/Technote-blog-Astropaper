---
title: "[TypeScript] 函式（Function） - 筆記"
pubDatetime: 2026-05-25T11:17:36.984Z
tags: ["TypeScript","cheatsheet"]
description: "Tags: TypeScript cheatsheet Table of contents 1. 參數類型註解 (Par..."
---

Tags: `TypeScript` `cheatsheet`
## Table of contents

## 1. 參數類型註解 (Parameter Type Annotations)
在函式參數後面加冒號 `:` 和類型，確保呼叫時參數型別正確。
```ts
function greet(noun: string) {
  console.log(`Hello, ${noun}!`);
}
greet('World'); // 正確
greet(2020);    // 錯誤：number 不能指派給 string
```

## 2. Optional Parameter
用 ? 表示參數可省略，若未傳入，參數值為 undefined。
```ts
function greet(name?: string) {
  console.log(`Hello, ${name || 'stranger'}!`);
}
greet(); // 輸出: Hello, stranger!
```

## 3. Default Parameters
可直接給參數預設值，TS 會推斷參數型別為預設值型別。

```ts
function exponentiation(power = 1) {
  console.log(4 ** power);
}
exponentiation();   // 輸出: 4
exponentiation(4);  // 輸出: 256
exponentiation(true); // 錯誤：boolean 不能指派給 number
```

## 4. 回傳值類型推斷 (Inferring Return Types)
TS 根據 return 語句的值自動推斷函式回傳型別。
```ts
function factOrFiction() {
  return Math.random() >= 0.5 ? 'true' : 'false';
}
const myAnswer: boolean = factOrFiction(); 
// 錯誤：string 不能指派給 boolean
```

## 5. 無回傳值 (Void Return Type)
函式不回傳值時，可標註回傳類型為 void。
```ts
function sayHello(): void {
  console.log('Hello!');
}
```

## 6. 明確指定回傳型別 (Explicit Return Types)
可在函式定義時，明確標註回傳型別，確保回傳值符合預期。
```ts
function trueOrFalse(value: boolean): boolean {
  if (value) {
    return true;
  }
  return 'false'; // 錯誤：string 不能指派給 boolean
}
```
也可指定回傳自定義型別:
```ts
type UserRole = "guest" | "member" | "admin"

type User = {
    username: string
    role: UserRole
}

const users: User[] = [
    { username: "john_doe", role: "member" },
    { username: "jane_doe", role: "admin" },
    { username: "guest_user", role: "guest" }
];

// 指定回傳User型別
function fetchUserDetails(username: string): User {
    const user = users.find(user => user.username === username)
    if (!user) {
        throw new Error(`User with username ${username} not found`)
    }
    return user
}
```
:::warning
### `ReturnType<T>`
`ReturnType<T>` 是 TypeScript 提供的一個工具型別（Utility Types），可以幫你「推斷某個函式的回傳型別」。
```ts
// Define a function that returns a string
function greet(name: string): string {
    return Hello, ${name}!;
}

// Use ReturnType<Type> to capture
// the return type of the function
type GreetReturnType = ReturnType<typeof greet>;

// Create a variable with the return type
const greeting: GreetReturnType = "Hello, GeeksforGeeks!";

// Outputs: "Hello, GeeksforGeeks!"
console.log(greeting);
```
* `typeof greet` 👉 拿到函式 `greet` 的型別（不是執行結果，是它的類型定義：`(name: string) => string`）。
* `ReturnType<...>` 👉 從這個函式型別中，取出它的回傳型別（這裡是 string）。

所以:
```ts
type GreetReturnType = string;
```

#### 這麼做有什麼用？
* 重複使用型別時更 DRY（Don't Repeat Yourself）
* 當函式的回傳值未來修改，使用 ReturnType 的地方會自動更新，不怕漏改。

#### Note: `ReturnType<T>` 只能接收「函式型別」作為 T
```ts
type R = ReturnType<() => number>; // ✅ R = number

const add = (a: number, b: number): number => a + b;
type AddReturn = ReturnType<typeof add>; // ✅ AddReturn = number

type MyFunc = (value: string) => boolean;
type MyFuncReturn = ReturnType<MyFunc>; // ✅ MyFuncReturn = boolean

type NotAFunction = number;
type R = ReturnType<NotAFunction>; // ❌ Error: Type 'number' does not satisfy the constraint '(...args: any) => any'.
```

#### 其他工具型別
| 工具型別                       | 用途描述                |
| -------------------------- | ------------------- |
| `ReturnType<T>`            | 取得函式的回傳型別           |
| `Parameters<T>`            | 取得函式的參數型別（是個 tuple） |
| `ConstructorParameters<T>` | 取得建構子參數型別           |
| `InstanceType<T>`          | 取得建構子函式所產生的實例型別     |

:::