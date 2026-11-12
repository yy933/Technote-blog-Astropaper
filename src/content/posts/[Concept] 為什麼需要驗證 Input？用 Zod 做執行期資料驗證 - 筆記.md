---
title: "[Concept] 為什麼需要驗證 Input？用 Zod 做執行期資料驗證 - 筆記"
pubDatetime: 2026-11-12T10:21:24.847Z
tags: ["TypeScript","Zod","Validation","Frontend","Backend","Concepts","Issue","Angular","React.js"]
description: "Tags: TypeScript Zod Validation Frontend Backend Concepts A..."
hackmd_id: "SJD__bX0Gg"
---

###### Tags: `TypeScript` `Zod` `Validation` `Frontend` `Backend` `Concepts` `Angular` `React.js` `Security` `Issue`


## Table of contents

## 問題（Issue）：為什麼需要驗證 Input？

### TypeScript 只保證「編譯期」，不保證「執行期」

TypeScript 是靜態型別語言，但**程式實際執行時拿到的資料，可能是任何東西**。我們寫的 `interface User` 只是「宣稱」資料長這樣，並不會真的檢查。

```typescript
// 這只是「告訴 TS 回傳值是 User」，並不會驗證
this.http.get<User>(`https://jsonplaceholder.typicode.com/users/${id}`);
```

當實際資料與型別不一致時，錯誤往往出現在離根源很遠的地方，例如：

```typescript
// 若 name 或 username 是 null / undefined，這行會直接丟出錯誤
`${user.name.trim()} (${user.username.toLocaleLowerCase()})`;
```

常見的情境：
* 後端欄位為 `null` 或 `undefined`
* 後端改版，但前端的型別沒有同步更新
* 前端不小心寫錯了型別
* 第三方 API 悄悄改了資料結構

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

TypeScript 給了我們一種「虛假的安全感」。問題通常要花很多時間才能找到真正原因（甚至會花好幾個小時懷疑是自己的程式寫錯）。

</blockquote>

### 後端、前端為什麼都要驗證？

| 位置 | 為什麼要驗證 | 驗證重點 |  
|---|---|---|  
| 後端 | 任何人都能送請求，來源不可信 | 驗證**來源**與**內容**，避免成為攻擊入口 |  
| 前端 | 來源已受信任 | 解析**內容**，資料與預期不符時第一時間得知 |

### 哪些地方算是「未知的 Input」？

* CLI 參數（`process.argv`）
* API 的路徑參數（`:id`）、HTTP Method、Headers、Query String、Request Body
* 表單（登入、CRUD 表單）
* WebSocket 連線
* `localStorage`（使用者可以竄改，也可能是過期資料）
* 第三方 API 的回應
* 自家後端的回應（可能發生版本漂移）

### 版本漂移（Version Drift）

使用者開著頁面沒有重新整理，期間後端部署了包含破壞性變更的新版本，前端還在用舊的資料格式去打 API，頁面就會壞掉。這種前後端版本不同步的情況，比想像中常見。

### 後端不驗證會發生什麼？（常見攻擊）

後端路由若直接解構 `req.body`，再把資料丟進資料庫查詢、檔案操作或商業邏輯，就等於**信任所有輸入**。

[OWASP API Security Top 10（2023）](https://api-security.owasp.org/editions/2023/en/0x11-t10/)中，與此直接相關的有：
* 第 3 名：Broken Object Property Level Authorization（包含 mass assignment）
* 第 4 名：Unrestricted Resource Consumption

只允許已知欄位、並限制大小的 schema，就是對這兩者的直接防禦。

| 攻擊類型 | 未驗證的輸入範例 | 後果 |  
|---|---|---|  
| SQL injection | `name: "'; DROP TABLE users;--"` | 資料庫遭破壞 |  
| 權限提升（mass assignment） | `role: "admin"` | 取得未授權的管理員權限 |  
| 阻斷服務（DoS） | `name: "A".repeat(10_000_000)` | 伺服器記憶體耗盡、當機 |  
| 型別混淆 | `age: "not a number"` | 執行期錯誤、計算出現 NaN |  
| 儲存型 XSS | `name: "<script>alert(1)</script>"` | 資料被渲染時執行腳本 |  
| Prototype pollution | `{"__proto__": {"admin": true}}` | 物件原型遭竄改 |

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

**驗證不等於防住所有攻擊。** 上表中的 SQL injection 與 XSS 字串，其實都能通過一般的字串 schema。
* SQL injection → 要靠**參數化查詢**，或會綁定參數值的 ORM
* XSS → 要靠 **渲染時跳脫（escape）** 使用者內容

驗證只是多層防禦中的其中一層。

</blockquote>

## Zod 的功能（為什麼要使用 Zod）

Zod 是一個 **TypeScript-first 的 schema 宣告與驗證函式庫**。這裡的 schema 泛指任何資料型別，從簡單的字串到複雜的巢狀物件都算。

使用 Zod 的理由：

* **解決「型別與驗證邏輯要分開維護」的問題**：宣告一次 schema，就能自動推導出 TypeScript 型別，不用維護兩份。
* **及早偵測錯誤**：資料「進入應用程式的當下」就會報錯，而不是之後才出現難以理解的崩潰。
* **清楚的錯誤訊息**：會指出是哪個欄位、為什麼不合法。
* **單一資料來源（Single source of truth）**：資料模型集中定義，確保整個應用程式一致。
* **減少防禦性程式碼**：元件可以放心使用驗證過的資料，不需要到處寫額外的檢查。
* **後端安全防線**：在請求進入商業邏輯前，擋掉格式錯誤、過大、或帶有不該出現欄位的 payload（例如防止 mass assignment、限制字串長度避免 DoS）。
* **彌補 TypeScript 的不足**：TS 說明資料「應該」長怎樣，Zod 確保它「真的」長這樣（Express 的 `req.body` 型別是 `any`，TS 在執行期不會檢查任何東西）。

## Zod 的特性

* **Schema 即型別**：用 `z.infer<typeof Schema>` 推導靜態型別，消除重複的型別宣告。
* **可組合**：簡單的 schema 可以組合成複雜的資料結構（`z.object`、`z.array`、`z.tuple`……）。
* **鏈式驗證規則**：`min`、`max`、`positive`、`int`、`email` 等可以直接串接。
* **兩種解析方式**：

| 方法 | 驗證失敗時 | 適用情境 |  
|---|---|---|  
| `parse` | **拋出** `ZodError` | 想讓錯誤直接中斷（例如開發環境） |  
| `safeParse` | 回傳結果物件，**不拋錯** | 想優雅處理錯誤（推薦的預設做法） |

`safeParse` 回傳的物件包含：
* `success`：是否驗證成功（布林值）
* `data`：驗證後的資料（成功時才有）
* `error`：錯誤資訊（失敗時才有）

* **開發者友善**：錯誤訊息包含欄位路徑（`path`）、預期型別（`expected`）、實際型別（`received`）與說明（`message`）。
* **可同時做「驗證」與「正規化」**：`.trim()`、`.toLowerCase()` 會在驗證的同時轉換資料；`.default()` 可設定預設值（如分頁）。
* **限制可接受的值**：
  * `z.enum([...])`：只允許指定的值，避免任意字串
  * `z.string().uuid()`：驗證 ID 格式，避免任意字串直接進入查詢
  * `.max()`：限制字串長度，防止超大 payload
* **`.strict()`**：遇到 schema 沒定義的多餘欄位時直接回報錯誤，而不是默默丟掉（用於防範 mass assignment）。
* **`z.coerce.number()`**：Query 參數預設是字串，可用它轉成數字再驗證。

## 程式碼範例

### 1. 建立 schema 並推導型別

```typescript
// user.model.ts
import { z } from "zod";

// 建立代表「合法 User」的 schema
export const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  username: z.string(),
  email: z.string(),
  phone: z.string(),
  website: z.string(),
});

// 由 schema 推導出型別，不用再另外寫 interface
export type User = z.infer<typeof UserSchema>;
```

### 2. 加入驗證規則

```typescript
import { z } from "zod";

const userName = z.string().min(5).max(10); // 長度 5~10 的字串
const userAge = z.number().positive().int(); // 正整數
const userEmail = z.string().email(); // 合法的 email 格式
```

### 3. parse 與 safeParse

```typescript
const invalidUser = {
  // 缺少 id
  name: "tim",
  username: "@tim",
  email: "tim@mail.com",
  phone: "",
  website: "https://timdeschryver.dev",
};

// parse：失敗時直接拋出 ZodError
UserSchema.parse(invalidUser);

// safeParse：失敗時回傳 { success: false, error }
const result = UserSchema.safeParse(invalidUser);
if (!result.success) {
  console.log(result.error.issues);
  /*
    [{
      code: "invalid_type",
      expected: "number",
      received: "undefined",
      path: ["id"],
      message: "Required"
    }]
  */
}
```

### 4. Angular：自訂 RxJS 運算子 `parseResponse`

在 HTTP service 收到回應後立刻驗證，避免汙染元件。為了不在每個請求重複寫邏輯，抽成共用運算子：

```typescript
// parse-response.operator.ts
import { tap } from "rxjs";
import type { MonoTypeOperatorFunction } from "rxjs";
import type { ZodType } from "zod";
import { environment } from "../environments/environment";

export function parseResponse<T>(schema: ZodType): MonoTypeOperatorFunction<T> {
  return tap({
    next: (value: any) => {
      if (!environment.production) {
        // 開發環境：直接拋錯，讓開發者馬上察覺
        schema.parse(value);
      } else {
        // 正式環境：不中斷使用者體驗，只記錄錯誤
        const parsed = schema.safeParse(value);
        if (!parsed.success) {
          console.log(parsed.error);
        }
      }
    },
  });
}
```

```typescript
// user.service.ts
getUser(id: number): Observable<User> {
  return this.http
    .get<User>(`https://jsonplaceholder.typicode.com/users/${id}`)
    .pipe(parseResponse(UserSchema)); // 每個請求只要加這一行
}
```

### 5. React：fetch 後用 safeParse 驗證

```tsx
import { z } from "zod";
import { useEffect, useState } from "react";

const postSchema = z.object({
  userId: z.number().positive().int(),
  id: z.number().positive().int(),
  title: z.string(),
  body: z.string(),
});

const postSchemaArray = z.array(postSchema);
type Posts = z.infer<typeof postSchemaArray>;

const ZodApi = () => {
  const [posts, setPosts] = useState<Posts>([]);
  const [error, setError] = useState("");

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/posts")
      .then((response) => response.json())
      .then((data: unknown) => {
        const validated = postSchemaArray.safeParse(data);

        if (!validated.success) {
          console.log("Validation Error:", validated.error);
          setError(validated.error.message);
          return;
        }

        // 這裡的資料已確認符合 schema，可以安全使用
        setPosts(validated.data);
      });
  }, []);

  if (error) return <div>Error fetching Data</div>;
  if (!posts.length) return <div>Loading posts...</div>;

  return (
    <ol>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ol>
  );
};

export default ZodApi;
```

### 6. 後端：驗證公開 API 的請求

```typescript
import { z } from "zod";
import { Request, Response } from "express";

const CreateUserSchema = z.object({
  body: z.object({
    // 確保 email 存在，而且格式正確
    email: z.string().email(),
  }),
  headers: z.object({
    // 確保有帶 authorization 標頭
    authorization: z.string(),
  }),
});

const handleCreateUser = (req: Request, res: Response) => {
  const result = CreateUserSchema.safeParse(req);

  // 驗證失敗：回傳 400 與易讀的錯誤訊息，而不是丟出例外
  if (!result.success) {
    res.status(400).send(result.error);
    return;
  }

  const { email } = result.data.body;
  // 建立使用者...
};
```

### 7. CLI：驗證 `process.argv`

```typescript
import { z } from "zod";

// process.argv 前兩個參數通常沒用，第三個才是使用者輸入
const ArgSchema = z.tuple([z.any(), z.any(), z.string()]);

const [, , name] = ArgSchema.parse(process.argv);
console.log(name); // name 自動推導為 string
```

### 8. 後端：AI 常見寫法 vs 加上驗證

```typescript
// ❌ 不好：直接解構 req.body，什麼都不檢查
app.post('/api/users', async (req, res) => {
  const { name, email, age, role } = req.body;
  // name 可能是 10MB 的文字
  // email 可能是 "not-an-email"
  // role 可能是 "admin"（權限提升！）
  const user = await db.insert(users).values({ name, email, age, role });
  res.json(user);
});
```

```typescript
// ✅ 好：用 Zod 驗證，敏感欄位由伺服器決定
import { z } from 'zod';

const createUserSchema = z.object({
  name: z.string().trim().min(1).max(100),
  email: z.string().email().max(255).toLowerCase(),
  age: z.number().int().min(13).max(150),
  // role 不接受 client 傳入，由伺服器端設定
});

app.post('/api/users', async (req, res) => {
  const result = createUserSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({
      error: 'Validation failed',
      details: result.error.flatten(),
    });
  }

  const { name, email, age } = result.data;
  const user = await db.insert(users).values({
    name, email, age, role: 'user', // role 永遠由伺服器決定
  });
  res.json(user);
});
```

### 9. 後端常用的 Zod 寫法

```typescript
import { z } from 'zod';

// 字串限制（先 trim，這樣 "   " 才會因 min(1) 而失敗）
const nameSchema = z.string().trim().min(1).max(100);

// Email 順便正規化
const emailSchema = z.string().email().max(255).toLowerCase();

// 用 enum 避免任意值
const roleSchema = z.enum(['user', 'editor']);

// 數字範圍
const ageSchema = z.number().int().min(0).max(150);

// ID 用 UUID
const idSchema = z.string().uuid();

// 預設值（分頁）
const pageSchema = z.number().int().min(1).default(1);
const limitSchema = z.number().int().min(1).max(100).default(20);

// 拒絕未知欄位（防止 mass assignment）
const updateSchema = z.object({
  name: z.string().min(1).max(100).optional(),
  email: z.string().email().optional(),
}).strict();
```

### 10. 後端：抽成 `validate(schema)` middleware

```typescript
import { ZodType } from 'zod';
import { Request, Response, NextFunction } from 'express';

export const validate =
  (schema: ZodType) => (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({
        error: 'Validation failed',
        details: result.error.flatten(),
      });
    }
    req.body = result.data; // 之後的 handler 拿到的是驗證（並正規化）過的資料
    next();
  };

// 使用方式
app.post('/api/users', validate(createUserSchema), handleCreateUser);
```

## 實作流程

1. **安裝 Zod**
   ```bash
   npm install zod
   ```
2. **定義 schema**：用 `z.object()`、`z.string()`、`z.number()` 等描述預期的資料結構，取代原本的 `interface`。  
3. **推導型別**：`type X = z.infer<typeof XSchema>`，其他程式碼幾乎不需要修改。  
4. **選擇解析方式**：
   * 想讓錯誤中斷流程 → `parse`
   * 想自己處理錯誤 → `safeParse`（推薦）  
5. **在「資料進入應用程式的邊界」驗證**：
   * 前端：HTTP service 收到回應之後
   * 後端：request handler 一開始  
6. **處理驗證失敗的情況**：
   * 後端：回傳 `400` 與錯誤訊息
   * 前端：用 state 保存錯誤，顯示友善的錯誤畫面（Loading / Error / Success 三種狀態）
   * 正式環境：記錄錯誤（log）以便追蹤  
7. **（Optional）抽成共用邏輯**：例如 Angular 的 `parseResponse` 運算子，讓每個請求只需加一行。

**後端補充流程（API 專案）**

8. 建立 `schemas/` 目錄，**每個資源一個 schema 檔**（如 `users.schema.ts`、`posts.schema.ts`）。  
9. 寫一個 `validate(schema)` middleware，內部用 `safeParse(req.body)`，失敗就回 `400`。  
10. 重要的 schema 加上 `.strict()`，並確保**敏感欄位（如 `role`）不從輸入取得**，一律由伺服器端決定。  
11. 不只驗證 body，**Query 參數與 URL 參數（如 `:id`）也要驗證**。  
12. 用 `z.infer` 推導型別，讓執行期與編譯期共用同一份定義。  
13. 檢查現有專案有沒有漏掉驗證的路由（特別是 AI 產生的程式碼）。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

### 為什麼放在 service 層？  
只有 service 會接觸 Zod 的 API，其他部分（元件等）不需要知道 Zod 的存在，仍然照原本方式運作，導入成本很低。

</blockquote>

## 其他注意事項

### 什麼時候該用、什麼時候不用？（Matt Pocock 的觀點）

| 輸入類型 | 範例 | 建議 |  
|---|---|---|  
| **不信任** | 公開 API、表單、CLI、WebSocket、`localStorage` | **一定要用** |  
| **有點信任（信任但無法控制）** | 第三方 API、GitHub / YouTube API、其他團隊維護的 API | **建議驗證** |  
| **信任且可控制** | 自家全端框架（Remix、Next.js、SvelteKit、Nuxt）的前後端 | **通常不驗證**（有爭議） |

* 第三種情境不驗證的理由：問題通常只要重新整理頁面就能解決，且前端不會暴露安全敏感資料，bug 的影響範圍較小。
* 但若想在版本漂移發生時就即時報錯並提示使用者重新整理，驗證也是合理的選擇。
* 「不可能出現的資料」是最常見的 bug 來源，所以大多數情況下，會把**穩健性**擺在第一位。

### 後端驗證的常見錯誤

| 錯誤做法 | 為什麼不行 | 正確做法 |  
|---|---|---|  
| 只在 client 驗證 | 用 curl、Postman 就能繞過 | 一定要在伺服器端驗證；client 驗證只是改善使用體驗 |  
| 用 TS 型別當作執行期保護 | 型別編譯後就被移除，沒有任何保護 | 用 Zod 驗證，並由 schema 推導型別 |  
| 只檢查必填欄位 | 多餘欄位會混進資料（mass assignment） | 用 `.strict()` 拒絕未知欄位 |  
| 字串沒設最大長度 | 攻擊者可送出數 MB 的文字 | 一律設定 `.max()` |

### `.strict()` 與預設行為的差別

Zod 的 `z.object` **預設會移除（strip）未知欄位**，所以像 `role` 這種沒定義的欄位，本來就不會出現在 `result.data` 裡。  
`.strict()` 的差別是：遇到多餘欄位時會**直接回報錯誤**，而不是默默丟掉。要不要這麼嚴格，可以依需求決定（想讓呼叫端知道自己送錯東西，就用 `.strict()`）。

### Query、URL 參數與檔案上傳

* **Query 參數**：預設是字串，需要轉型與驗證，數字可用 `z.coerce.number()`。
* **URL 參數（`:id`）**：應驗證為 UUID 或數字，不要直接傳進資料庫查詢。
* **檔案上傳**：Zod 只處理 JSON，檔案要另外驗證：
  * MIME type 與 magic bytes（不能只看副檔名）
  * 檔案大小
  * 檔名（不要信任 client 傳來的檔名與 content type）
  * 搭配 multer 或 busboy，並設定明確的上限

### 效能與套件大小

* Zod 約 **12kb（gzip）**，對部分重視套件大小的應用可能偏大。
* 驗證一定會比不驗證稍慢，若很在意關鍵路徑（critical path）的效能，可以考慮略過。
* 小型資料集的影響可忽略，以下是 Angular 文章提供的 benchmark（解析一般大小的 user 物件）：

| 筆數 | 耗時 |  
|---|---|  
| 1 | 1ms |  
| 100 | 4ms |  
| 1,000 | 8ms |  
| 10,000 | 52ms |  
| 100,000 | 488ms |  
| 1,000,000 | 5,109ms |

* 後端的角度：據 VibeDoctor 文章的說法，驗證一般的 request body 不到 0.1ms，而後續的資料庫查詢通常要 1～50ms，驗證的開銷不到整體請求時間的 1%（此為該文章的估計，實際數字會依 schema 複雜度與環境而異）。

### 開發環境與正式環境採用不同策略

* **開發環境**：用 `parse`，直接拋錯，讓開發者馬上知道需要處理。
* **正式環境**：用 `safeParse`，資料照常放行並記錄錯誤，避免影響使用者體驗。

### 小心「型別標註」不等於「驗證」

```typescript
// 這個 `: Posts` 只是「宣稱」，並不會檢查資料
.then((posts: Posts) => { ... })

// 真正的保證來自 safeParse
// 建議把尚未驗證的資料標成 unknown，再交給 schema 處理
.then((data: unknown) => postSchemaArray.safeParse(data))
```

### Schema 盡量保持簡單

* 只檢查資料型別，就足以解決「後端回傳格式與預期不符」的問題。
* 若在 schema 裡加入複雜的內容驗證，容易把**商業邏輯**混進 schema，不適合用於「驗證回應格式」的情境。
* （範例中的 `min`、`max` 等規則，主要用於表單、API 請求這類需要檢查內容的輸入。）

### 缺點

* 使用 Zod 後，無法再用 IDE 的重構功能，一次重新命名整個程式碼庫中的介面屬性（因為 schema 是變數，不是 interface）。

## 小結

* TypeScript 只能保證編譯期的型別安全，**執行期的資料仍然是未知的**，因此需要在資料進入應用程式的邊界做驗證。
* Zod 讓我們只需宣告一次 schema，就同時擁有**執行期驗證**與**靜態型別**，並能在資料不符時提供清楚的錯誤訊息。
* 判斷是否使用 Zod，取決於對輸入的**信任程度與掌控程度**：
  * 不信任 → 使用 Zod
  * 信任但無法控制 → 建議用 Zod 驗證
  * 信任且可控制 → 視情況（通常不驗證，但仍可用來偵測版本漂移）
* 實務上優先用 `safeParse` 優雅處理錯誤；若想區分環境，可在開發環境用 `parse`、正式環境用 `safeParse` 並記錄錯誤。
* 導入成本低（定義 schema → 推導型別 → 在邊界驗證），換來的是更容易除錯、更穩健的應用程式。
* **後端是最需要驗證的地方**：沒驗證的 API 路由，會暴露在 mass assignment、超大 payload、型別混淆等風險下。AI 產生的程式碼特別容易漏掉這一步，需要人工檢查補上。
* 後端驗證的重點：**限制長度、限制可接受的值（enum / UUID）、拒絕或移除未知欄位、敏感欄位由伺服器端決定**。
* **驗證不是萬能**：SQL injection 要靠參數化查詢、XSS 要靠輸出時跳脫，Zod 只是其中一層防禦。



## 參考資料

* [Why we should verify HTTP response bodies, and why we should use zod for this](https://timdeschryver.dev/blog/why-we-should-verify-http-response-bodies-and-why-we-should-use-zod-for-this)
* [When should you use Zod?](https://www.totaltypescript.com/when-should-you-use-zod)
* [How to Use Zod for React API Validation（freeCodeCamp）](https://www.freecodecamp.org/news/how-to-use-zod-for-react-api-validation/)
* [Zod vs No Validation: What AI Code Skips and Why It Matters](https://vibedoctor.io/blog/zod-input-validation-ai-generated-api-routes)
* [Zod 官方文件](https://zod.dev)