---
title: "[TypeScript] Generics Constraints（泛型約束 extends）與 keyof 應用"
pubDatetime: 2026-08-05T10:45:24.897Z
tags: ["Type system","TypeScript","Concepts","Advanced Types"]
description: "Table of contents 為什麼需要 Generics Constraints（泛型約束）？ 預設情況下，泛..."
hackmd_id: "SJ-7uKlUMg"
---

## Table of contents

## 為什麼需要 Generics Constraints（泛型約束）？

預設情況下，泛型 `<T>` 可以代表世界上任何型別（例如 `number`、`string`、`boolean` 或是任意物件）。

但也因為它的涵蓋範圍太廣，當我們試圖在函式內部存取某些特定屬性時，TypeScript 會因為無法確定 `T` 是否擁有該屬性而直接報錯：

```typescript
function getLength<T>(arg: T): number {
  // ❌ 錯誤：Property 'length' does not exist on type 'T'.
  // 因為 T 可能是 number 或 boolean，它們沒有 .length 屬性！
  return arg.length; 
}
```

為了解決這個問題，我們需要使用 `extends` 關鍵字對泛型加上約束條件（Constraints）——限制傳入的 `T` 必須滿足某個特定的型別結構。

## 1. 使用 extends 限制泛型結構  
藉由 `<T extends { length: number }>`，我們告訴 TypeScript：「`T` 可以是任何型別，但它必須包含 `length` 屬性，且`length`的型別為 `number`」。

基本語法: `變數名稱` + `extends` + `條件`

### 語法與範例

```typescript
interface HasLength {
  length: number;
}

function getLength<T extends HasLength>(arg: T): number {
  return arg.length; // ✅ 安全存取！TypeScript 確定 arg 一定有 length 屬性
}

// 測試呼叫：
getLength("Hello World");                  // ✅ string 有 length 屬性
getLength([1, 2, 3]);                      // ✅ Array 有 length 屬性
getLength({ length: 10, value: "data" });  // ✅ 物件只要有 length 屬性即可

// ❌ 錯誤：Argument of type 'number' is not assignable to parameter of type 'HasLength'.
getLength(123);
```

## 2. 搭配 keyof 確保物件 Property 安全  
在開發中，我們經常需要撰寫一個「從物件取值」的通用工具函式（例如 getProperty）。如果只寫 `<T, K>`，無法保證 `K` 一定是 `T` 裡存在的屬性名稱。

搭配 `keyof` 運算子，可以建立強大的型別連鎖：`<K extends keyof T>`。

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

**`keyof T` 是什麼？**  
它會取得物件型別 `T` 的所有屬性名稱（Keys），並組合成長為一個字串聯集型別（String Union）。例如 `T` 是 `{ name: string; age: number }`，則 `keyof T` 就是` "name" | "age"`。

</blockquote>


### 語法與範例

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]; // ✅ 回傳值型別會精準鎖定為 T[K] (Indexed Access Type)
}

const user = {
  id: 101,
  name: "Alice",
  isVIP: true
};

// 1. 正確取值（TypeScript 自動推導 K 的精確型別）
const userName = getProperty(user, "name");  // userName 型別精準為 string
const userVIP = getProperty(user, "isVIP");  // userVIP 型別精準為 boolean

// 2. 傳入不存在的 key 會直接在編譯期被攔截！
// ❌ 錯誤：Argument of type '"email"' is not assignable to parameter of type '"id" | "name" | "isVIP"'.
getProperty(user, "email");
```


## 3. 合併與更新物件 (Merge & Update)
**泛型約束也很常用於限制傳入參數必須是「物件結構」**（`<T extends object>`）：

```typescript
// 限制 T 與 U 都必須是物件
function mergeObjects<T extends object, U extends object>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const person = mergeObjects({ name: "Bob" }, { age: 30 });
console.log(person.name); // ✅ 型別為 string
console.log(person.age);  // ✅ 型別為 number

// ❌ 錯誤：Argument of type 'number' is not assignable to parameter of type 'object'.
mergeObjects(123, { name: "Bob" });
```


## Recap

| 語法 Pattern | 說明 / 語意 | 實務應用情境 |  
| :--- | :--- | :--- |  
| `<T extends Interface>` | `T` 必須包含該介面定義的屬性 | 確保傳入的資料擁有特定欄位（如 `id`、`length`） |  
| `<K extends keyof T>` | `K` 必須是物件 `T` 內實際存在之 Key | 打造強型別的物件取值/設定工具函式（如 `getProperty`） |  
| `<T extends object>` | `T` 必須是物件型別（排除原始型別） | 處理物件合併、擴充或設定檔組裝 |