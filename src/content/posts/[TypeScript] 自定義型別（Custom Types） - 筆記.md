---
title: "[TypeScript] 自定義型別（Custom Types） - 筆記"
pubDatetime: 2025-05-27T21:50:40.000Z
modDatetime: 2026-05-25T10:04:23.465Z
tags: ["TypeScript","cheatsheet"]
description: "Table of contents 在 TypeScript 中，當基本型別（如 string, number, bo..."
hackmd_id: "rkn7F-4fgg"
---

## Table of contents

在 TypeScript 中，當基本型別（如 `string`, `number`, `boolean`）不足以描述我們所使用的資料結構時，可以使用「自定義型別」（Custom Types）來更準確表達物件的型別。

## 基本語法
使用 `type` 關鍵字定義：

```typescript
type Programmer = {
  name: string;
  knownFor: string[];
};
```
* Programmer 是一個型別名稱。
* name 是 string，knownFor 是 string[]。

`;`、`,` 或不加都可作為屬性分隔符。

### 使用方式：
```typescript
const ada: Programmer = {
  name: 'Lizzy',
  knownFor: ['Mathematics', 'Computing', 'First Programmer']
};
```

## 型別註解說明（TSDoc）
可用 `/** ... */` 為每個欄位加上說明，有助於 IDE 自動補全與文件生成：
```typescript
type Programmer = {
  /** The full name of the Programmer */
  name: string;
  /** This Programmer is known for what? */
  knownFor: string[];
};
```

## 型別錯誤示範
```typescript
const lizzy: Programmer = {
  name: true, // ❌ Type 'boolean' is not assignable to type 'string'. (2322)
  knownFor: ['Math']
};

const lizzy2: Programmer = {
  name: 'Ada Lovelace' // ❌ Property 'knownFor' is missing. (2741)
};

const lizzy3: Programmer = {
  name: 'Lizzy',
  knownFor: ['Math'],
  age: 36 // ❌ 'age' does not exist in type 'Programmer'. (2322)
};
```

## 巢狀型別（Nested Types）
可以將一個型別嵌入另一個型別中：
```typescript
type Person = { name: string; };
type Company = { name: string; manager: Person; };

const company: Company = {
  name: 'ACME',
  manager: {
    name: 'John Doe'
  }
};
```
:bulb: 只要物件結構吻合，即使未明確標註型別也不會出錯（型別推論）。

## 選擇性屬性（Optional Properties）
使用 `?` 宣告可選屬性：

```typescript
type Programmer = {
  name: string;
  knownFor?: string[]; // 一個只包含字串的陣列，例如 ["A", "B", "C"]，也可以寫成Array<string>
};

const lizzy: Programmer = {
  name: 'Lizzy' // ✅ knownFor 可省略
};
```

## 可索引型別（Indexable Types）
「可以用變動的 key 來存取屬性」，讓物件的結構可以彈性擴充，不需要每一個 key 都事先寫死。
有些情況下，我們無法預先知道一個物件裡會有哪些欄位名稱，或者這些欄位名稱是由使用者、API、資料動態決定的。

```typescript
type Data = {
  [key: string]: any;
};

const info: Data = {
  name: 'Lizzy',
  age: 36,
  isAlive: false
};
```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

* `key`: string 表示：這個物件可以用任何字串當 key（例如 name, age, foo, abc123...）
* `any` 表示：每個 key 的值是什麼型別都可以（可以是字串、數字、布林、物件等等）

</blockquote>
也可以可以加上固定屬性，指定某些屬性是「必備」的，同時保留其餘屬性的彈性：：

```typescript
type Data = {
  status: boolean;
  [key: string]: any;
};

const obj: Data = {
  status: true,         // ✅ 必備屬性
  name: 'Liz',          // ✅ 自由擴充
  age: 36,
  notes: ['engineer']
};
```
<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:exclamation: 陷阱題：
```typescript
type UserData = {
  isActive: boolean;
  [key: string]: string;
};

const user1: UserData = {
  isActive: true,          // ❌ 錯！isActive 是 boolean，但預設所有值都要是 string
  name: 'Ada'
};

const user2: UserData = {
  isActive: 'yes',         // ✅ 正確，因為 'yes' 是 string
  nickname: 'coder'
};
```
* 為什麼 `user1` 會噴錯?
```typescript
type UserData = {
  isActive: boolean;
  [key: string]: string;
};
```
這看起來好像是說：**「isActive 是 boolean，其他的 key 都是 string 對吧？」**
但 TypeScript 的行為並不是這樣判斷的。
```typescript
[key: string]: string;
```
這是在告訴 TypeScript：**「這個物件的所有屬性，不管是什麼名稱，都必須是 string。」**

而 `isActive` 雖然是特別指定為`boolean`，但 **TypeScript 認為它也得滿足「所有屬性都要是 string」這個規則**。 所以會檢查 boolean（isActive 的型別） ➜ 能不能視為 string？(答案是不能)。
* 正解一：放寬 value 的型別
```typescript
type UserData = {
  isActive: boolean;
  [key: string]: string | boolean;
};
```

* 正解二：移除Index Signature，改用更精確的 mapped type（進階用法）
如果只想支援特定 key，例如 name, email，但不開放所有 key 都能任意加，可以考慮用：
```typescript
type UserData = {
  isActive: boolean;
  name?: string;
  email?: string;
};
```
這樣型別更安全，不會無限制擴充。

</blockquote>


<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

⚠️ 注意事項
* 當使用` [key: string]: any`，就等於開放了「任何欄位都可以進來」，這會讓 TypeScript 的型別檢查變弱。建議搭配明確的型別限制，例如：`[key: string]: string | number`。
* 如果只接受有限的幾個動態 key，也可以使用「Mapped Type」來處理（進階）。

</blockquote>

## 陣列中至少包含幾個元素（Rest Tuple）
保證陣列最少包含兩個字串：

```typescript
type MergeStringsArray = [string, string, ...string[]];

const valid: MergeStringsArray = ['A', 'B', 'C'];
const invalid: MergeStringsArray = ['A']; // ❌ 少於兩項
```

## 型別組合（Composing Types）
### Union（聯集）：擁有任一型別皆可

```typescript
type ProductCode = number | string;

const codeA: ProductCode = 'abc123';
const codeB: ProductCode = 123;
```
### Intersection（交集）：需同時符合所有型別

```typescript
type Person = { name: string };
type Worker = { company: string };
type Employee = Person & Worker;

const staff: Employee = {
  name: 'Lizzy',
  company: 'ACME'
};
```

## Recap
| 功能       | 關鍵語法                            |
| -------- | ------------------------------- |
| 自訂型別     | `type`                          |
| 可選屬性     | `?`                             |
| 可索引屬性    | `[key: string]: any`            |
| 最少 N 項陣列 | `[string, string, ...string[]]` |
| 聯集型別     | `A \| B`                        |
| 交集型別     | `A & B`                         |