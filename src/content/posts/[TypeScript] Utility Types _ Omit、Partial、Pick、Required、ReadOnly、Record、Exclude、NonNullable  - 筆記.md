---
title: "[TypeScript] Utility Types : Omit、Partial、Pick、Required、ReadOnly、Record、Exclude、NonNullable  - 筆記"
pubDatetime: 2025-06-02T01:21:16.000Z
tags: ["TypeScript","cheatsheet"]
description: "Tags: TypeScript cheatsheet Table of contents Omit<Type, Key..."
---

Tags: `TypeScript` `cheatsheet`
## Table of contents

## Omit<Type, Keys>
用途：從一個型別中「排除」一或多個屬性。

### 語法
```ts
Omit<T, K>
```
* T：原始型別
* K：要被排除的屬性名稱（可以是單個，也可以是聯合型別）

### 範例
```ts
type User = {
  id: number;
  username: string;
  role: string;
};

// 移除 id 屬性
type NewUser = Omit<User, "id">;

const user: NewUser = {
  username: "alice",
  role: "admin"
};
```
也可以排除多個屬性：
```ts
type SlimUser = Omit<User, "id" | "role">;

const slim: SlimUser = {
  username: "bob"
};
```

## Partial<Type>
用途：讓某個型別中所有屬性變成optional，常用於更新或初始化階段。

### 語法
```ts
Partial<T>
```

### 範例
```ts
type Profile = {
  name: string;
  age: number;
  email: string;
};

// 所有屬性都變成 optional
type PartialProfile = Partial<Profile>;

const updateData: PartialProfile = {
  email: "user@example.com"
};
```
效果等於:
```ts
type PartialProfile = {
  name?: string;
  age?: number;
  email?: string;
};
```
也可以搭配 `Required<T>` 把可選變回必填，例如：
```ts
type FullProfile = Required<PartialProfile>;
```
可以還原成：
```ts
type FullProfile = {
  name: string;
  age: number;
  email: string;
};
```
* 實用情境：更新函式
```ts
function updateProfile(profile: Profile, updates: Partial<Profile>) {
  return { ...profile, ...updates };
}
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
    
## Required<T>
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

## Readonly<T>
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

defaultSettings.theme = "dark"; // ❌ 錯誤：因為 Readonly 限制修改
```
    
## Record<K, T>
建立一個 key-value 結構，key 的型別是 K，value 是 T
```ts
type Role = "admin" | "member";
type UserInfo = { name: string };

type UserMap = Record<Role, UserInfo>;
// => { admin: UserInfo; member: UserInfo }

/*
等同於：
type UserMap = {
  admin: { name: string };
  member: { name: string };
};
*/
```
也就是:
```ts
const users: UserMap = {
  admin: { name: "Alice" },
  member: { name: "Bob" }
};
```

    
## Exclude<T, U>
從 T 中移除與 U 重疊的部分
```ts
type Status = "active" | "inactive" | "archived";

type VisibleStatus = Exclude<Status, "archived">;
// => "active" | "inactive"
```
    
## NonNullable<T>
移除 null 與 undefined
```ts
type MaybeName = string | null | undefined;

type Name = NonNullable<MaybeName>;
// => string
```
    
## Recap
| Utility Type     | 功能                         |
| ---------------- | -------------------------- |
| `Partial<T>`     | 全部屬性變成 optional (`?`)      |
| `Required<T>`    | 全部屬性變成必填                   |
| `Omit<T, K>`     | 移除指定屬性                     |
| `Pick<T, K>`     | 挑選指定屬性                     |
| `Record<K, T>`   | 建立一個 key 為 K、value 為 T 的物件 |
| `Exclude<T, U>`  | 從 T 中排除與 U 相同的型別           |
| `NonNullable<T>` | 移除 `null` 和 `undefined`    |