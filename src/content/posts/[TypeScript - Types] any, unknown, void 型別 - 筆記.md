---
title: "[TypeScript - Types] any, unknown, void 型別 - 筆記"
pubDatetime: 2026-08-04T08:39:59.126Z
tags: ["TypeScript","Concepts","cheatsheet"]
description: "Table of contents TypeScript的特殊型別：any, unknown, void 在 Type..."
hackmd_id: "S1O4WQ18Gx"
---

## Table of contents

## TypeScript的特殊型別：any, unknown, void  
在 TypeScript 中，any、unknown 與 void 各自代表了不同的「型別檢查政策」與「語法意圖」。

* `any` 的使命：「完全關閉型別檢查。」（逃生出口，應極力避免使用）
* `unknown` 的使命：「安全的未知型別。」（要求先檢查後使用，為 `any` 的最佳替代方案）
* `void` 的使命：「標示無回傳值的函式。」（**明確表達函式僅執行動作不產生結果**）

## any, unknown, void 特性與語法解析
### 1) any：放棄靜態檢查的黑洞  
寫上 `any` 等同於告訴編譯器「別管這個變數」。雖然能解決一時的編譯錯誤，但也會讓 TypeScript 失去防護作用。

```typescript
let data: any = 42;

// ❌ 都不會在編譯期報錯，但會在執行期（Runtime）崩潰！
data.toUpperCase();
data();
```

### 2) unknown：強制型別窄化（Type Narrowing）  
與 `any` 一樣能接收任意值，但在對它進行「型別檢查/窄化」之前，TypeScript 禁止你對它進行任何操作。

```typescript
let input: unknown = "Hello World";

// ❌ 報錯：Object is of type 'unknown'.
// input.toUpperCase();

// ✅ 正確：經過型別檢查窄化後即可安全使用
if (typeof input === "string") {
  console.log(input.toUpperCase()); // "HELLO WORLD"
}
```

### 3) `void`：無回傳值函式  
主要用於函式的回傳值型別，代表「該函式不打算回傳任何內容」。

```typescript
function logMessage(message: string): void {
  console.log(message);
  // 沒有 return 或只有 return;
}
```

| 型別 | 賦值限制 | 能否直接存取屬性/方法 | 可否指派給具體型別變數 | 安全等級 |  
| :--- | :--- | :--- | :--- | :--- |  
| **`any`** | 任意值 | 任意呼叫（不檢查） | 可以指派給任何型別 | 🔴 極低（失去 TS 保護） |  
| **`unknown`** | 任意值 | ❌ 需先Type Narrowing才可呼叫 | ❌ 只能指派給 `unknown` 或 `any` | 🟢 高（編譯期強制要求防護） |  
| **`void`** | 僅 `undefined` | ❌ 不適用 | ❌ 無法隨意給一般變數 | 🟢 高（語意明確） |


## 為什麼需要 `unknown` 替代 `any`？
### 傳統做法：濫用 `any` 導致潛在的 Runtime Crash  
在處理外部 API 資料或動態 JSON 時，若為了圖方便而使用 `any`，會讓型別錯誤一路隱藏到線上環境：

```typescript
function parseApiResponse(rawJson: string): any {
  return JSON.parse(rawJson);
}

const user = parseApiResponse('{"name": "Alice"}');

// ❌ 拼錯欄位名稱！編譯不會報錯，但 Runtime 拿到 undefined，後續邏輯可能直接崩潰
console.log(user.username.toUpperCase()); 
```

### 解法：使用 `unknown` 進行防禦性檢查  
改用 `unknown` 可以強迫開發者在存取屬性前撰寫防禦邏輯，提升系統穩定度：

```typescript
function parseApiResponse(rawJson: string): unknown {
  return JSON.parse(rawJson);
}

const user = parseApiResponse('{"name": "Alice"}');

// ✅ 經過完整的型別防禦檢查（Type Guard）
if (
  typeof user === "object" && 
  user !== null && 
  "name" in user && 
  typeof (user as any).name === "string"
) {
  console.log((user as { name: string }).name.toUpperCase()); // "ALICE"
} else {
  console.error("無效的資料格式");
}
```


## 注意事項
### 1) `void` 與 `undefined` 的語意差異  
在 JS 中未回傳值的函式預設會得到 `undefined`，但在 TS 的型別系統中兩者有明確分工：

```typescript
// ❌ 代表函式必須「明確 return undefined」
function funcA(): undefined {
  return undefined; 
}

// ✅ 代表函式「不打算回傳任何東西」（內部邏輯不關心回傳值）
function funcB(): void {
  console.log("Processing...");
}
```

### 2) 只有一種合理使用 `any` 的情境：舊專案過渡期  
`any` 唯一的合理使用情境是將大型原生 JavaScript 專案遷移至 TypeScript 時，作為暫時（Temporary）壓制警告的過渡手段，遷移完成後應全面移除並重構為具體型別或 `unknown`。

## 比較 any vs unknown vs void

| 特性 | `any` | `unknown` | `void` |  
| :--- | :--- | :--- | :--- |  
| **主要情境** | 關閉型別檢查（極力避免） | 處理外部未知資料 (API/JSON/`catch`) | 標示函式執行動作且不回傳結果 |  
| **型別檢查** | 徹底繞過 | 強制寫條件判斷窄化型別 | 回傳值檢查 |  
| **程式碼防護** | 無，容易引發 Runtime 錯誤 | 高，於編譯期攔截潛在危險 | 高，防止誤用無回傳值的結果 |


### 什麼時候不要使用？  
這些時候請不要用：

* 純變數宣告：不要寫 `let x: void`，這幾乎沒有任何實用價值。
* 因為型別太難寫就用 `any`：遇到複雜型別時，應優先使用工具型別（如 `Partial`、`Omit`）或 `unknown`，而非直接掛上 `any` 放棄檢查。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>