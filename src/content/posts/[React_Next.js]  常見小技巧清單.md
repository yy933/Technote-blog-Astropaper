---
title: "[React/Next.js]  常見小技巧清單"
pubDatetime: 2025-08-25T22:28:18.000Z
tags: ["JavaScript","cheatsheet","TypeScript","Next.js","React.js"]
description: " Table of contents 陣列處理 移除空值 / 假值 ts const arr = [\"a\", \"\", n..."
---

## Table of contents


 

## 陣列處理
### 移除空值 / 假值

```ts
const arr = ["a", "", null, undefined, 0, "b"];
const clean = arr.filter(Boolean); 
// ["a", "b"]
```


### 展開合併

const a = [1, 2];
const b = [3, 4];
const all = [...a, ...b]; 
// [1, 2, 3, 4]


### 唯一值 (去除重複)

```ts
const nums = [1, 2, 2, 3];
const unique = [...new Set(nums)];
// [1, 2, 3]
```


### 取最後一個元素

```ts
const arr = [1, 2, 3];
const last = arr.at(-1); 
// 3
```

## 布林判斷

### 雙驚嘆號 (!!) → 把值轉成布林

```ts
!!"hello"   // true
!!0         // false
!!null      // false
```


### 短路運算

```ts
const isLogin = true;
isLogin && console.log("已登入"); // 只有 true 才執行
```

## 物件處理

### 物件展開 (merge)

```ts
const user = { name: "Alex", age: 20 };
const extra = { role: "admin" };
const merged = { ...user, ...extra };
// { name: "Alex", age: 20, role: "admin" }
```


### 選擇性屬性 (??) → Nullish coalescing

```ts
const name = null ?? "Guest";
// "Guest"
const age = 0 ?? 18;
// 0 （因為 0 不是 null/undefined）
```


### 可選鏈 (?.)

```ts
const user = { profile: { name: "Alex" } };
console.log(user.profile?.name); // "Alex"
console.log(user.account?.email); // undefined (不會報錯)
```

## 字串處理

### Template Literal (字串插值)

```ts
const name = "Alex";
const msg = `Hello, ${name}!`;
// "Hello, Alex!"
```


### 陣列轉字串

```ts
const tags = ["nextjs", "react", "typescript"];
tags.join(", "); 
// "nextjs, react, typescript"
```

## React / Next.js 常用

### 條件 className

```ts
const active = true;
<div className={`menu-item ${active ? "text-bold" : ""}`} />
```


### 條件渲染

`{isLogin ? <Dashboard /> : <Login />}`


### 短路渲染

`{messages.length > 0 && <MessageList />}`

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>