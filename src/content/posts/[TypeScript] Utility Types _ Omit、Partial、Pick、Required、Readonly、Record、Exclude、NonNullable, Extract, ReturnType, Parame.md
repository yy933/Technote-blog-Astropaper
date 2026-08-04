---
title: "[TypeScript] Utility Types : Omit、Partial、Pick、Required、Readonly、Record、Exclude、NonNullable, Extract, ReturnType - 筆記"
pubDatetime: 2026-08-04T09:32:54.797Z
tags: ["TypeScript","cheatsheet"]
description: "TypeScript 的內建 Utility Types 主要可分為兩大類..."
---

Tags: `TypeScript` `cheatsheet`
## Table of contents

## Utility Types分類

TypeScript 的內建 Utility Types 主要可分為兩大類：  
1. **物件型別操作**：`Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record`  
2. **聯集/原始型別操作**：`Exclude`, `Extract`, `NonNullable`, `ReturnType`, `Parameters`

## 1. 物件型別操作 (Object Types)
### `Partial<Type>`  
用途：讓某個型別中所有屬性變成 optional (`?`)，常用於更新 (Update) 或初始化階段。

#### 語法與範例

```ts
type Profile = {
  name: string;
  age: number;
  email: string;
};

// 所有屬性都變成 optional
type PartialProfile = Partial<Profile>;
/*
等同於：
type PartialProfile = {
  name?: string;
  age?: number;
  email?: string;
};
*/

// 實用情境：更新函式
function updateProfile(profile: Profile, updates: Partial<Profile>) {
  return { ...profile, ...updates };
}
```


### `Required<Type>`  
用途：把型別中所有 Optional (`?`) 的屬性都變成必填。

#### 語法與範例
```ts
type Profile = {
  name?: string;
  age?: number;
  email?: string;
};

// 所有屬性都變成必填
type FullProfile = Required<Profile>;
/*
等同於：
type FullProfile = {
  name: string;
  age: number;
  email: string;
};
*/
```

    
## Pick<T, K>  
功能：從物件類型 T 中「挑出」指定的屬性 K 組成新的型別。
```ts
type User = {
  id: number;
  name: string;
  email: string;
};

// 只挑出 id 和 name
type PublicUser = Pick<User, "id" | "name">;
/*
等同於：
type PublicUser = {
  id: number;
  name: string;
};
*/
```
    
### `Required<T>`  
功能：把類型 T 中所有Optional（?）的屬性都變成必填。

```ts
type Profile = {
  name?: string;
  age?: number;
  email?: string;
};

// 所有屬性都變成必填
type FullProfile = Required<Profile>;
/*
等同於：
type FullProfile = {
  name: string;
  age: number;
  email: string;
};
*/
```

### `Readonly<T>`  
功能：讓類型 T 中所有屬性變成唯讀（不可修改）。

```ts
type Settings = {
  theme: string;
  fontSize: number;
};

const defaultSettings: Readonly<Settings> = {
  theme: "light",
  fontSize: 14
};

// ❌ 錯誤：Cannot assign to 'theme' because it is a read-only property.
defaultSettings.theme = "dark";
```

### `Pick<Type, Keys>`  
用途：從物件型別中「挑選」指定的 key 組成新的型別。

#### 語法與範例
```typescript
type User = {
  id: number;
  name: string;
  email: string;
};

// 只挑出 id 和 name
type PublicUser = Pick<User, "id" "name" |>;
/*
等同於：
type PublicUser = {
  id: number;
  name: string;
};
*/
```


### `Omit<Type, Keys>`  
用途：從一個物件型別中「排除」一或多個屬性（Key）。

#### 語法與範例
```typescript
type User = {
  id: number;
  username: string;
  role: string;
};

// 移除 id 屬性
type NewUser = Omit<User, "id">;
// 也可以移除多個：Omit<User, "id" "role" |>

const user: NewUser = {
  username: "alice",
  role: "admin"
};
```

    
### `Record<Keys, Type>`  
用途：建立一個 Key-Value 的字典結構。`Keys` 代表 Key 的型別集合，`Type` 代表 Value 的型別。

>⚠️ 注意：`Keys` 必須能指派給 `string | number | symbol`。

#### 語法與範例
```ts
type Role = "admin" | "member";
type UserInfo = { name: string };

type UserMap = Record<Role, UserInfo>;

/*
等同於：
type UserMap = {
  admin: { name: string };
  member: { name: string };
};
*/

const users: UserMap = {
  admin: { name: "Alice" },
  member: { name: "Bob" }
};
```


## 聯集與其他型別操作 (Unions & Function Types)
### `Exclude<UnionType, ExcludedMembers>`  
用途：從 聯集型別 (Union) 中剔除與指定型別重疊的部分。

#### 語法與範例

```typescript
type Status = "active" | "inactive" | "archived";

type VisibleStatus = Exclude<Status, "archived">;
// => "active" | "inactive"
```

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

💡 `Omit` vs `Exclude` 差別：
* `Omit` 用於 **物件的屬性刪減** (`Omit<{ a: 1, b: 2 }, "a">`)
* `Exclude` 用於 **聯集的成員過濾** (`Exclude<"a" | "b", "a">`)

</blockquote>


### Extract<Type, Union>  
用途：從 聯集型別 中「提取」可指派給指定型別的成員（與 Exclude 相反）。

#### 語法與範例

```ts
type Event = "click" | "hover" | 200 | 404;

type StringEvents = Extract<Event, string>;
// => "click" | "hover"
```

### NonNullable  
用途：從型別中移除 `null` 與 `undefined`。

#### 語法與範例

```ts
type MaybeName = string | null | undefined;

type Name = NonNullable<MaybeName>;
// => string
```

### ReturnType  
用途：取得某個 **函式型別** 的回傳值型別。

#### 語法與範例

```ts
function getUser() {
  return { id: 101, name: "John Doe" };
}

// 透過 typeof 取得函式型別，再用 ReturnType 拿到回傳值型別
type UserResponse = ReturnType<typeof getUser>;
// => { id: number; name: string; }
```
    
## Recap

| Utility Type | 作用對象 | 核心功能 |  
| :--- | :--- | :--- |  
| `Partial<T>` | 物件 | 全部屬性變為可選 (`?`) |  
| `Required<T>` | 物件 | 全部屬性變為必填 |  
| `Readonly<T>` | 物件 | 全部屬性變為唯讀 |  
| `Pick<T, K>` | 物件 | 挑選指定的 Keys |  
| `Omit<T, K>` | 物件 | 剔除指定的 Keys |  
| `Record<K, T>` | 物件 | 快速建立字典/對照表物件 |  
| `Exclude<T, U>` | 聯集 | 從聯集 T 中移除 U 成員 |  
| `Extract<T, U>` | 聯集 | 從聯集 T 中抽取 U 成員 |  
| `NonNullable<T>` | 任意 | 剔除 `null` 與 `undefined` |  
| `ReturnType<T>` | 函式 | 取得函式的回傳值型別 |