---
title: "[TypeScript] 還不確定型別的時候，應該用Generics (泛型) 或是 unknown？"
pubDatetime: 2026-08-04T11:13:15.407Z
tags: ["TypeScript","cheatsheet","Type system","Issue"]
description: "Table of contents 前言 許多人在剛學泛型時，都會有一樣的疑問：「既然 unknown 和 Gener..."
hackmd_id: "BkD0BrkIGl"
---

## Table of contents

## 前言  
許多人在剛學泛型時，都會有一樣的疑問：「既然 `unknown` 和 `Generics`（泛型）都是用在『還不確定型別』的時候，那不確定型別時直接寫泛型不就好了嗎？」

答案是：不行，它們的用途和思考邏輯截然不同。

## 核心差異
* **`unknown`：「我真的不知道（或不在乎）這是什麼型別」**
  - 適用於外部資料輸入（API 回傳、使用者輸入、JSON.parse、catch error）。
  - 核心精神：安全防禦（強迫你在使用前做型別窄化 / Type Guard）。

* **`Generics`：「我現在不知道，但『呼叫者』知道，而且我要把輸入和輸出的型別『綁定』在一起」**
  - 適用於通用工具函式、資料結構、API 元件。
  - 核心精神：型別連結與自動推導（建立輸入與輸出之間的對應關係）。

## 關鍵判斷：有無「型別連結 (Type Linkage)」需求？  
要決定用哪個，問自己這個關鍵問題：**「輸入的型別，會決定回傳值的型別（或影響其他參數）嗎？」**

* 有鏈結需求 ➔ 用 `Generics`
* 沒有鏈結需求 ➔ 用 `unknown`

## 實際範例
### 情況 A：需要型別連結 ➔ 必須用 Generics  
假設有一個函式 `identity` 或 `getLastItem`，傳入什麼型別，就應該回傳什麼型別：

```typescript
// ❌ 用 unknown：型別連結斷掉！
function getLastItem(arr: unknown[]): unknown {
  return arr[arr.length - 1];
}

const num = getLastItem([1, 2, 3]); 
// num 的型別變成 unknown！
// 明明傳入 number[]，拿出來卻不能直接當 number 用，還要寫 typeof 檢查，超麻煩！


// ✅ 用 Generics：成功建立型別連結！
function getLastItem<T>(arr: T[]): T | undefined {
  return arr[arr.length - 1];
}

const num = getLastItem([1, 2, 3]); 
// TypeScript 會自動推導出 T 是 number，num 的型別就是 number！
```

### 情況 B：完全沒有連結需求 ➔ 應該用 `unknown`  
假設有一個印出 `Log` 的函式，或是解析 API 回傳值的函式，它只負責印出或檢查，不關心輸入跟輸出之間的型別綁定：

```typescript
// ✅ 用 unknown：語意明確，這就是一個接受任何東西的防禦性函式
function logValue(value: unknown): void {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  } else if (typeof value === "number") {
    console.log(value.toFixed(2));
  }
}

// ⚠️ 若硬寫成泛型（無意義的泛型）：
function logValue<T>(value: T): void {
  // 這裡的 T 完全沒有跟任何其他參數或回傳值做鏈結，寫成泛型只是多此一舉！
}
```


## 什麼時候「不能/不適合」直接寫泛型？  
有些新手會想把所有不確定的地方都宣告成泛型 `<T>`，這常會踩到以下兩個坑：

### 1. 內部實作無法預測泛型 T 的操作  
當宣告 `function doSomething<T>(x: T)` 時，TypeScript 會認為 `T` 可以是世界上任何型別（例如 `number`、`string`、`HTMLElement`、甚至 `{}`）。

如果在函式內部試圖直接存取某些屬性，TypeScript 會直接報錯：

```typescript
function processData<T>(data: T) {
  // ❌ 錯誤：Property 'name' does not exist on type 'T'.
  // 因為 T 可能是 number，number 沒有 .name 屬性！
  console.log(data.name); 
}
```

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

💡 補充：如果想讓泛型具備某些屬性，就必須搭配 `extends`，例如` <T extends name: string { }>`。

</blockquote>


### 2. 外部動態資料無法被泛型自動推導  
泛型的自動推導（Inference）依賴於 JavaScript 執行時傳入的已知實體。  
如果資料是來自外部 API（例如 `fetch`），TypeScript 在編譯期根本不可能知道 API 會回傳什麼，這時寫泛型也不會自動變成正確型別：

```typescript
// 情況 1：API 撈回來的資料 (通常用 unknown)
async function fetchData(): Promise<unknown> {
  const res = await fetch("/api/user");
  return res.json(); // TypeScript 找不到實體推導，回傳 unknown 最安全，強迫後續做驗證
}

// 情況 2：API 套件封裝 (用泛型讓呼叫者手動指定)
async function apiGet<T>(url: string): Promise<T> {
  const res = await fetch(url);
  return res.json() as T; // 呼叫時手動傳入 apiGet<User>("/api/user")
}
```

## Recap 

| 比較項目 | `unknown` | `Generics` (`<T>`) |  
| :--- | :--- | :--- |  
| **核心語意** | 「我不知道這是什麼，請先檢查再用」 | 「我現在不知道，但傳進來時就會知道」 |  
| **型別連結** | ❌ 無（輸入與輸出無關） | 🟢 有（參數與回傳值/其他參數綁定） |  
| **型別窄化** | 必須在函式內部寫 `if (typeof...)` 防禦 | 在呼叫端由 TypeScript 自動推導具體型別 |  
| **最佳情境** | 1. API / JSON / catch error 資料<br>2. 僅需接收任意值的通用邏輯（如 Log） | 1. 工具函式（如 `map`, `filter`, `pop`）<br>2. 通用資料結構（如 `List<T>`, `Response<T>`） |