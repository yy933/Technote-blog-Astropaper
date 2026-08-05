---
title: "[TypeScript] 泛型介面 (Generic Interfaces) 與 泛型別名 (Generic Type Aliases) - 筆記"
pubDatetime: 2026-08-05T10:09:29.115Z
tags: ["TypeScript","Type system","Concepts","cheatsheet"]
description: "Table of contents 簡介 在處理 API 回傳值、資料結構（如列表、樹狀圖）或通用工具函式時，資料的內..."
hackmd_id: "rk-I_Fx8Gg"
---

## Table of contents

## 簡介  
在處理 API 回傳值、資料結構（如列表、樹狀圖）或通用工具函式時，資料的內容型別往往不是固定的。

如果寫死成單一型別，會失去程式碼的複用性；如果用 `any`，則會失去 TypeScript 的型別安全檢查。泛型介面 (Generic Interfaces) 與 泛型別名 (Generic Type Aliases) 允許我們在定義結構時建立「型別占位符 (Placeholder)」，讓使用者在使用時才決定具體的型別。

## 1. 泛型介面 (Generic Interfaces)  
在 `interface` 名稱後方加上 `<T>`（或多個泛型參數，如 `<T, U>`），就能在介面內部將 T 當作一般型別使用。

### 範例：API 回傳值封裝  
最常見的應用就是封裝統一格式的 API Response：

```typescript
// 定義通用的 API 回傳結構
interface ApiResponse<Data> {
  code: number;
  message: string;
  data: Data; // data 的型別由使用時傳入的 Data 決定
  timestamp: number;
}

// 實體資料型別
interface User {
  id: number;
  name: string;
  email: string;
}

interface Product {
  id: string;
  title: string;
  price: number;
}

// 1. 使用者 API 回傳
const userResponse: ApiResponse<User> = {
  code: 200,
  message: "Success",
  data: { id: 1, name: "Alice", email: "alice@example.com" },
  timestamp: 1718000000
};

// 2. 商品 API 回傳
const productResponse: ApiResponse<Product> = {
  code: 200,
  message: "Success",
  data: { id: "p-101", title: "Wireless Mouse", price: 890 },
  timestamp: 1718000000
};
```

## 2. 泛型別名 (Generic Type Aliases)  
語法與 `interface` 類似，在 type 名稱後方加上 `<T>`。與 `type` 相比 `interface` 能處理更多元的型別組合（例如 Union Types、Tuple、Mapped Types 等）。

### 範例：異步處理與狀態封裝

```typescript
// 1. 狀態結果包裹 (Union Type + Generics)
type Result<T, ErrorType = Error> = 
  | { success: true; data: T }
  | { success: false; error: ErrorType };

// 使用範例
function parseJson<T>(jsonString: string): Result<T, string> {
  try {
    const data = JSON.parse(jsonString) as T;
    return { success: true, data };
  } catch (err) {
    return { success: false, error: "Invalid JSON format" };
  }
}

// 2. 鍵值對結構 (KeyValue Pairs)
type KeyValuePair<K, V> = {
  key: K;
  value: V;
};

const setting: KeyValuePair<string, boolean> = {
  key: "darkMode",
  value: true
};
```

## 3. 進階用法：預設值 (Default Type) 與 約束 (Generic Constraints)
### A. 泛型預設值 (Default Generic Types)  
當使用者沒有指定泛型型別時，可以使用 `=` 指定預設型別：

```typescript
// 當未指定 T 時，預設為 string
interface PaginatedList<T = string> {
  page: number;
  total: number;
  items: T[];
}

// 未傳入泛型參數 ➔ items 自動為 string[]
const tagList: PaginatedList = { page: 1, total: 2, items: ["React", "TS"] };

// 有傳入泛型參數 ➔ items 為 number[]
const idList: PaginatedList<number> = { page: 1, total: 2, items: [101, 102] };
```

### B. 泛型約束 (Generic Constraints - extends)  
使用 `extends` 強制傳入的泛型型別必須符合特定結構（例如必須包含 `id` 欄位）：

```typescript
interface Identifiable {
  id: number | string;
}

// 強制 T 必須擁有 id 屬性
type EntityItem<T extends Identifiable> = {
  entity: T;
  createdAt: Date;
};

// ✅ 正確：User 含有 id 屬性
type UserEntity = EntityItem<{ id: 101; name: "Bob" }>;

// ❌ 錯誤：不符合 Identifiable（缺少 id 屬性）
// Type '{ name: string; }' does not satisfy the constraint 'Identifiable'.
type InvalidEntity = EntityItem<{ name: "Bob" }>;
```

## 4. Generic Interface vs Generic Type Alias 比較

| 比較項目 | Generic Interface | Generic Type Alias |  
| :--- | :--- | :--- |  
| **定義方式** | `interface Box<T> { value: T; }` | `type Box<T> = { value: T; };` |  
| **自動合併 (Declaration Merging)** | 🟢 支援（同名介面會自動考量泛型合併） | ❌ 不支援（同名會報錯） |  
| **表達能力 (Expressiveness)** | 專注於 **物件、類別與函式結構** | 能定義 **Union (`\|`)、Intersection (`&`)、Tuple、Mapped Types** 等複雜型別 |  
| **OOP 繼承/實作** | `class Foo implements Box<string>` | `class` 較少直接 `implements` 純 Union 的 `type` |

## Recap

| 情境 | 推薦寫法 | 程式碼範例 |  
| :--- | :--- | :--- |  
| **通用 API/物件結構** | Generic Interface | `interface Response<T> { data: T; }` |  
| **狀態處理 (Success/Fail Union)** | Generic Type Alias | `type Result<T> = Success<T> \| Failure` |  
| **有預設備用型別** | Generic Default | `type List<T = string> = T[]` |  
| **限制傳入型別範圍** | Generic Constraint | `interface Model<T extends { id: string }> { }` |