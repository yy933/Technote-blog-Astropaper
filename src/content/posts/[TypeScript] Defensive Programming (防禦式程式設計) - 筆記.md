---
title: "[TypeScript] Defensive Programming (防禦式程式設計) - 筆記"
pubDatetime: 2026-05-26T03:29:26.546Z
tags: ["TypeScript"]
description: " Table of contents Defensive Programming 是什麼？ 相信大家都聽過防禦式駕駛，其..."
---

## Table of contents

## Defensive Programming 是什麼？
相信大家都聽過防禦式駕駛，其實寫程式也有防禦式程式設計！

簡單來說，Defensive Programming　就是 **「預先假設會出錯，並在程式中做好防範措施」** 的寫法。它的重點在先想最壞的情況，並讓程式依然能正常工作或適當回應。
:arrow_right: 它的核心精神是：
> ❝ 不信任任何input、使用者、API 或第三方函式，永遠預留錯誤處理的空間 ❞

搭配 TypeScript 的靜態型別檢查，可以大幅提升開發時期就發現錯誤的機率，再加上防禦式邏輯處理，讓我們在執行階段也有後盾。

## 基本範例
JavaScript
```javascript
function getFirstItem(arr) {
  return arr[0];
}

getFirstItem(undefined); // 這邊直接報錯，因為沒檢查 arr
```
加上防禦式寫法：
```javascript
function getFirstItem(arr) {
  if (!Array.isArray(arr) || arr.length === 0) return null;
  return arr[0];
}
```

## TypeScript 靜態型別檢查
TypeScript 在「開發階段」就會提醒我們哪些變數可能是 undefined、哪些函式參數型別不符，這樣可以更早防止error。

範例：
```typescript
function getFirstItem(arr: string[]) {
  return arr[0];
}
getFirstItem(undefined); // ❌ 編譯時就報錯，根本不能執行
```

## 防禦式程式設計 5 大實作原則
### 1. Input Validation（輸入驗證）
確保輸入資料是合理的，避免處理錯誤或惡意資料。
```typescript
function greet(name: string): string {
  if (!name || name.trim().length === 0) {
    throw new Error('Name cannot be empty.');
  }
  return `Hello, ${name}!`;
}
```
<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
提醒：即使 TS 已指定 name 為 string，但仍可能傳入空白或無意義的內容，這就是 runtime 防禦的重要性。
</div>

### 2. Error Handling（錯誤處理）
遇到例外情況時，適當處理錯誤並提供有意義的訊息或備案。
```typescript
function divide(a: number, b: number): number {
  if (b === 0) {
    throw new Error('Division by zero is not allowed.');
  }
  return a / b;
}

function getResult(a: number, b: number): number | null {
  try {
    return divide(a, b);
  } catch (error: unknown) {
    if (error instanceof Error) {
      console.error(error.message);
    }
    return null;
  }
}
```
好的錯誤處理能避免整個 app 崩潰，並協助 debug。

### 3. Boundary Checks（邊界檢查）
檢查 array、字串等索引值是否在合理範圍內，避免越界錯誤。
```typescript
function getArrayElement<T>(arr: T[], index: number): T | null {
  if (index < 0 || index >= arr.length) {
    console.error('Index out of bounds.');
    return null;
  }
  return arr[index];
}
```
特別適合處理 API 資料或動態 index 的情境。

### 4. Invariants & Assertions
用 assert 強化某些邏輯應該「永遠成立」的假設，違反時即刻報錯。

```typescript
import assert from 'node:assert';

function squareRoot(x: number): number {
  assert.ok(x >= 0, 'Cannot compute the square root of a negative number');
  return Math.sqrt(x);
}
```
用來防止邏輯錯誤，常見於數值運算、資料狀態切換等場合。

### 5. Fail-safe Defaults（安全預設值）
當輸入不完整、資料缺失時，提供合理預設值，避免應用中斷。

```typescript
interface User {
  name: string;
  age: number;
}

function createUser(name: string, age: number = 0): User {
  return { name, age };
}

const user = createUser('Benny'); // age 預設為 0
```
通常搭配 optional parameters 或 fallback 機制一起使用。

## TypeScript 在 Defensive Programming 中的角色
| 功能         | 類型檢查（TypeScript） | 運行期防錯（Defensive Coding）             |
| ---------- | ---------------- | ----------------------------------- |
| 發現潛在錯誤     | 編譯階段             | 執行階段                                |
| 檢查資料結構是否正確 |  例如型別 interface |  搭配 `typeof`、`Array.isArray` 等手動檢查 |
| 保證函式回傳安全結果 |  回傳型別定義         |  加入錯誤處理與預設值邏輯                      |

## Recap：設計心法
* 預期錯誤會發生：永遠預防最壞的輸入與情境。
* 不要太相信使用者輸入或外部 API。
* TS 幫忙查型別，防禦式邏輯避免執行時爆掉。
* 遇到錯誤，記得留訊息，不要讓 app 靜靜掛掉。