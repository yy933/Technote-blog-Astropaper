---
title: "[TypeScript] Optional Chaining & Nullish Coalescing  - 筆記"
pubDatetime: 2025-06-04T02:43:18.000Z
tags: ["TypeScript","cheatsheet"]
description: "Tags: TypeScript cheatsheet Table of contents 什麼是 Optional C..."
hackmd_id: "HJn9JiTfxe"
---

Tags: `TypeScript` `cheatsheet`
## Table of contents

## 什麼是 Optional Chaining？
`?.` 是一個運算子，當運算鏈中的任一部分為 null 或 undefined 時就停止執行並回傳 undefined。於 TypeScript 3.7 版加入，主要解決繁瑣的 if (a && a.b) 這類檢查。

```ts
const length = user?.profile?.name?.length;
// 等同於：user && user.profile && user.profile.name && user.profile.name.length
```
## Nullish Coalescing 是什麼？
* `??` 是一種提供「預設值」的方式，僅在運算元為 `null` 或 `undefined` 時使用預設值。
* 不等同於 `||`（`||` 也會把 `''`, `0`, `false` 當作「無效值」）。
```ts
const name = userInput ?? "default"; 
// 當 userInput 是 null/undefined 時使用 "default"
```


## 為什麼這兩個要一起學？
Optional chaining 避免因為 undefined 而程式爆炸，nullish coalescing 在空值出現時給出合理預設值。搭配起來超好用！

```ts
const nameLength = person?.name?.length ?? -1;
```

## 傳統寫法（TS 3.6 以前）
```ts
function getNameLength(name?: string | null): number {
  if (name === null || name === undefined) {
    return -1;
  }
  return name.length;
}
```
## 使用 ?. 和 ?? 改寫後
```ts
function getNameLength(name?: string | null): number {
  return name?.length ?? -1;
}
```
## Optional Chaining 的三種用法：
| 用法類型           | 範例                     |
| -------------- | ---------------------- |
| 屬性存取（Property） | `user?.profile?.name`  |
| 函式呼叫（Call）     | `user?.getProfile?.()` |
| 陣列元素存取（Index）  | `items?.[0]`           |

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

⚠️ 注意：不是所有情況都能避免錯誤
例如：
```ts
function barPercentage(foo?: { bar: number }) {
  return foo?.bar / 100;
}
// 如果 foo 是 undefined，foo?.bar 是 undefined，除以 100 -> NaN
```

</blockquote>




## 好習慣：打開 `strictNullChecks`
`strictNullChecks` 開啟後，TypeScript 會強制你處理 `null` / `undefined`。
更安全、更明確，也更能發揮 optional chaining 與 nullish coalescing 的力量。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

開啟`strictNullChecks` :
```
// tsconfig.json
{
  "compilerOptions": {
    "strictNullChecks": true
    // 或是 strict: true 也可以，那其實也已經包含了 strictNullChecks，因為它是 strict 模式的一部分。
  }
}
```

</blockquote>

## Recap
| 概念                           | 說明                                         |
| ---------------------------- | ------------------------------------------ |
| `?.`                         | 碰到 `null` 或 `undefined` 就停下，傳回 `undefined` |
| `??`                         | 僅在值為 `null` / `undefined` 時使用預設值           |
| 組合應用                         | 讓程式碼更乾淨、可讀性高，不再滿地 `if` 判斷                  |
| 搭配 `strictNullChecks` 使用效果最佳 |                                            |