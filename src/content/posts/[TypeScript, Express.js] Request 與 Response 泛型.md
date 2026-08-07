---
title: "[TypeScript, Express.js] Request 與 Response 泛型"
pubDatetime: 2026-08-07T09:47:29.260Z
tags: ["Express.js","TypeScript","Node.js","Backend","JavaScript","Concepts","Type system","Best-Practices"]
description: "Table of contents 前言 在 TypeScript 中使用@types/express 時，Reque..."
hackmd_id: "SyFUGmQIfg"
---

## Table of contents


## 前言  
在 TypeScript 中使用`@types/express` 時，`Request` 與 `Response` 都支援泛型（Generics）。透過型別定義，可以讓 `req.params`、`req.body`、`req.query` 以及 `res.json()` 取得完整的型別檢查與自動補全。

## 1. Request 泛型解析
### 參數位置對應  
`Request` 一共包含 4 個按位置排序的泛型參數：

```typescript
interface Request<
  P = ParamsDictionary, // 1. req.params  (路徑參數)
  ResBody = any,        // 2. ResBody     (回應體預期型別)
  ReqBody = any,        // 3. req.body    (請求體資料)
  ReqQuery = ParsedQs   // 4. req.query   (網址查詢參數)
>
```

### 4 個位置詳細說明

| 位置 | 泛型名稱 | 對應屬性 | 說明與用途 | 常見填寫內容 |  
| :--- | :--- | :--- | :--- | :--- |  
| 第 1 位置 | `P` | `req.params` | 路徑動態變數（如 `/:id` 或 `/:category/:id`） | `{ id: string }` |  
| 第 2 位置 | `ResBody` | （回應體） | 預期回傳的 JSON 結構（通常佔位用，更多直接寫在 `Response<T>`） | `unknown` 或 `{}` |  
| 第 3 位置 | `ReqBody` | `req.body` | POST / PUT / PATCH 送入的 JSON 內容 | `{ name: string, age: number }` |  
| 第 4 位置 | `ReqQuery` | `req.query` | 網址 `?key=value` 的查詢參數 | `{ species?: string, page?: string }` |


### 核心規則：位置依賴（Position-based Matching）  
因為泛型是依序對齊的，如果「只想定義第 4 個 `ReqQuery`」，前面 1~3 個位置必須填寫佔位型別（如 `{}` 或 `unknown`），無法直接跨越。


### 範例：使用 4 個泛型的完整 GET 路由

```typescript
import type { Request, Response } from 'express';

// 1. 定義各區塊的型別
type PetParams = { id: string };
type PetResponseBody = Pet | { message: string };
type UpdatePetBody = { name: string; isAdopted: boolean };
type PetQueryParams = { verbose?: string };

// 2. 帶入 Request 與 Response 泛型
app.put(
  '/pets/:id',
  (
    req: Request<PetParams, PetResponseBody, UpdatePetBody, PetQueryParams>,
    res: Response<PetResponseBody>
  ) => {
    // req.params.id       -> string (自動推導)
    // req.body.name       -> string (自動推導)
    // req.query.verbose   -> string | undefined (自動推導)
    const { id } = req.params;
    const { name, isAdopted } = req.body;
    
    // res.json() 只能傳入符合 PetResponseBody 的結構
    res.json({ id, name, isAdopted, species: 'dog', age: 3, adopted: isAdopted });
  }
);
```


## 2. Response 泛型解析
### 參數位置對應  
`Response` 一共支援 3 個按位置排序的泛型參數：

```typescript
interface Response<
  ResBody = any,                            // 1. res.json() / res.send() 的回傳結構
  Locals extends Record<string, any> = ..., // 2. res.locals (中間件共用變數)
  StatusCode extends number = number        // 3. res.status() 允許的 HTTP 狀態碼
>
```

### 3 個位置詳細說明

| 位置 | 泛型名稱 | 對應屬性 / 函式 | 說明與用途 | 常見填寫內容 |  
| :--- | :--- | :--- | :--- | :--- |  
| 第 1 位置 | `ResBody` | `res.json(data)` | 限制 `res.json()` 或 `res.send()` 輸出的資料結構 | `Pet[]` 或 `{ message: string }` |  
| 第 2 位置 | `Locals` | `res.locals` | 跨中間件傳遞的本地變數（如 JWT 解開後的 user 資訊） | `{ user: AuthUser }` |  
| 第 3 位置 | `StatusCode` | `res.status(code)` | 限制可使用的 HTTP 狀態碼（較少用，預設為 number） | `200 \| 400 \| 404 \| 500` |

### 範例：在身分驗證中間件中使用 `Locals`

```typescript
import type { Request, Response, NextFunction } from 'express';

type AuthLocals = {
  user: { id: string; role: 'admin' | 'user' };
};

// 中間件：解開 JWT 並掛載到 res.locals
export const authMiddleware = (
  req: Request,
  res: Response<unknown, AuthLocals>,
  next: NextFunction
): void => {
  res.locals.user = { id: 'usr_101', role: 'admin' };
  next();
};

// 路由：存取強型別的 res.locals.user
export const getAdminDashboard = (
  req: Request,
  res: Response<{ data: string }, AuthLocals>
): void => {
  const { role } = res.locals.user; // 型別自動推導為 'admin' | 'user'
  
  if (role !== 'admin') {
    res.status(403).json({ data: 'Permission denied' });
    return;
  }
  
  res.json({ data: 'Welcome Admin' });
};
```

## 3. 常見觀念與最佳實踐
### 1. Middleware的提早返回（Early Return）寫法  
在寫Middleware時，如果不需處理 `body`/`query`，前三個位置可以寫 `unknown` 做強型別占位：

```typescript
const validateId = (
  req: Request<{ id: string }, unknown, unknown, unknown>,
  res: Response<{ message: string }>,
  next: NextFunction
): void => {
  if (!/^\d+$/.test(req.params.id)) {
    res.status(400).json({ message: 'ID 必須為數字' });
    return; // 提早結束，避免繼續執行 next()
  }
  next();
};
```

### 2. 什麼時候不用寫全部泛型？
* 簡單 `GET` 路由：通常只需要定義 `Request<{ id: string }>` 或 `Response<Pet>`，不需要的尾部泛型直接省略，TS 會自動代入預設值。

* 省略 ResBody 佔位：若覺得 `Request<{}, unknown, {}, QueryType>` 太長，也可以定義一個自訂的 Helper 類型或介面。

## 4. 最佳實踐（Best Practices）  
在實際專案中，**寫得太詳細往往會變成一種「型別負擔（Type Friction）」，降低開發效率。** 如果專案已經使用 Zod / Yup / Joi 做 Runtime 資料驗證，手動再寫一套 TypeScript Interface 給 `req.body` 就是雙重維護。

通常採用「重點抽換與實用主義」的策略：**只在最容易出錯的地方進行型別約束。** 

### 什麼時候一定要寫?
#### 1. 處理 `req.body` 與複雜的 `req.query`  
預設的 `req.body` 型別是 `any`，直接存取會導致 `any` 擴散到整個應用程式，失去型別檢查能力。

#### 2. 開發自訂中間件（Middleware）  
當Middleware會在 `res.locals` 注入資料（如 JWT 解析後的 `user` 資訊）或驗證 `req.headers` 時，必須明確定義型別，否後續的路由 Handle 無法自動取得型別提示。

#### 3. 重點 API 的 Request Params 驗證  
路徑動態參數（如 `/:id`）如果需要確保只能傳入特定結構，適度定義 `Request<{ id: string }>` 可防止手誤寫錯 key（例如將 `req.params.id` 誤寫為 `req.params.petId`）。

### 主流作法（Best Practices）  
為了兼顧「型別安全」與「開發速度」，現代 Express 專案通常採用以下兩種做法：

#### 做法一：自訂工具型別（Utility Types）簡化語法  
建立一個全域或共用的 `types/express.ts`，將繁瑣的 4 個位置封裝成易讀的語法：

```typescript
import type { Request } from 'express';
import type { ParamsDictionary } from 'express-serve-static-core';

// 只定義 Body 的 Helper
export type ReqBody<T> = Request<ParamsDictionary, any, T>;

// 只定義 Query 的 Helper
export type ReqQuery<T> = Request<ParamsDictionary, any, any, T>;

// 只定義 Params 的 Helper
export type ReqParams<T> = Request<T>;
```

使用時:

```typescript
// 簡潔許多，不再需要寫任何佔位符
app.post('/pets', (req: ReqBody<CreatePetDto>, res) => {
  const { name, age } = req.body; // 自動獲得 CreatePetDto 型別
});
```

#### 做法二：結合 Schema 驗證工具（如 Zod，最推薦）  
現代 TypeScript 專案最主流的做法是**採用 Zod 自動推導型別，同時完成 Runtime 驗證與 Compile-time 型別導出**：

```typescript
import { z } from 'zod';

// 1. 定義 Schema（單一真相來源）
export const CreatePetSchema = z.object({
  name: z.string(),
  species: z.string(),
  age: z.number().positive(),
});

// 2. 自動推導出 TypeScript 型別
export type CreatePetDto = z.infer<typeof CreatePetSchema>;

// 3. 在 Express 中使用驗證套件（如 express-zod-api 或自訂驗證中間件）自動掛載型別
```

#### 做法三：使用 Declaration Merging 擴充全域 `req.user`  
對於身份驗證Middleware，通常不會寫在 `res.locals` 的泛型，而是直接擴充 Express 原生的 `Request` 介面：

```typescript
// @types/express/index.d.ts
declare global {
  namespace Express {
    interface Request {
      user?: {
        id: string;
        role: 'admin' | 'user';
      };
    }
  }
}

export {};
```

擴充後，所有路由可以直接調用 `req.user`，完全無需在每個路由手動寫泛型：

```typescript
app.get('/profile', (req, res) => {
  // req.user 自動擁有 id 與 role 型別
  if (!req.user) return res.status(401).send();
  console.log(req.user.id);
});
```

## Quick Recap

| 實作情境 | Request 泛型 | Response 泛型 |  
| :--- | :--- | :--- |  
| **取得列表 (GET /pets)** | `Request` | `Response<Pet[]>` |  
| **帶 ID 讀取 (GET /pets/:id)** | `Request<{ id: string }>` | `Response<Pet \| { message: string }>` |  
| **帶 Query 篩選 (GET /pets?species=dog)** | `Request<{}, unknown, {}, { species?: string }>` | `Response<Pet[]>` |  
| **新增資料 (POST /pets)** | `Request<{}, unknown, CreatePetDto>` | `Response<Pet>` |  
| **驗證中間件 (寫入 res.locals)** | `Request` | `Response<unknown, { user: User }>` |

* 不用追求 100% 寫滿 4 個泛型位置。
* `req.body` 必做型別約束（透過自訂 Helper 或 Zod）。
* `res` 泛型較少使用，保持 `res.json()` 自然輸出即可。
* 全域擴充 `req.user` 處理身份驗證資訊。