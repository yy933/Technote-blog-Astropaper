---
title: "Pass by Value vs. Pass by Reference (傳值與傳址) - 筆記"
pubDatetime: 2026-10-02T03:40:17.146Z
tags: ["Interview Preparation","JavaScript","Frontend","Concepts"]
description: "Tags: Interview Preparation JavaScript Frontend Concepts Ta..."
hackmd_id: "HJ0d4i39zx"
---

###### Tags: `Interview Preparation` `JavaScript` `Frontend` `Concepts`


## Table of contents

## 核心特性與觀念總覽

* **JavaScript 的底層傳參真相（Evaluation Strategy）**：
  * **嚴格來說，JavaScript 永遠都是 Pass by Value（Call by Value）**！
  * 當傳遞物件時，它傳遞的是 **「記憶體位址/引用的複製值（Pass by Copy of Reference / Pass by Sharing）」** 。 
* **按值傳遞 (Pass by Value)**：
  * **適用型別**：基本型別（Primitive Types），如 `Number`, `String`, `Boolean`, `null`, `undefined`, `Symbol`, `BigInt`。
  * **運作機制**：將變數的值 **複製** 一份傳入函式。函式內對參數的任何修改，**完全不會影響**外部原有的變數。
* **按址傳遞 (Pass by Reference)**：
  * **適用型別**：物件型別（Non-Primitive / Objects），如 `Object`, `Array`, `Function`。
  * **運作機制**：將變數指向記憶體空間的**位址/引用（Reference）**傳入函式。函式內修改該物件的屬性（Properties），**會直接影響並突變（Mutate）**外部原本的物件。



## 1. 核心觀念速查

| 比較面向 | Pass by Value (傳值) | Pass by Reference (傳址 / 傳引用) |  
| :--- | :--- | :--- |  
| **資料型別 (Data Types)** | **基本型別 (Primitives)**<br>(Number, String, Boolean...) | **物件型別 (Non-Primitives)**<br>(Object, Array, Function) |  
| **記憶體儲存型態** | 值直接存在 **棧記憶體 (Stack)** | 實際資料在 **堆記憶體 (Heap)**，Stack 僅存位址 |  
| **函式傳參行為** | 複製「**數值本身**」傳入 | 複製「**指向 Heap 的位址/引用的副本**」傳入 |  
| **函式內修改副作用** | **無副作用**（外部原始變數保持不變） | **有副作用**（內部修改屬性會導致外部原始物件同步改變） |  
| **常見錯誤/風險** | 無 | 造成無預期的 Side Effects，破壞資料不可變性 (Immutability) |



## 2. 實務範例程式碼剖析 (Code Examples)

### 範例 A：基本型別 (Pass by Value) - 無副作用

```javascript
const prim = 5;

function add(value) {
  value++; // 僅修改函式內部的獨立副本 (Copy)
  return value;
}

console.log(add(prim)); // 6 (內部計算結果)
console.log(prim);      // 5 (外部原始變數完全不受影響！)
```


### 範例 B：物件型別 (Pass by Reference) - 產生副作用 (Side Effect)

```javascript
const ref = { count: 5 };

function add2(value) {
  // value 與外部的 ref 指向 Heap 記憶體中的同一塊區域！
  value.count++; // 直接改寫了記憶體中的物件屬性
  return value.count;
}

console.log(add2(ref)); // 6
console.log(ref.count); // 6 (外部原始物件被修改了！這就是 Side Effect)
```

### 範例 C：進階陷阱（重新賦值 Re-assignment）  
這是經典陷阱題：若在函式內將傳入的物件「重新指向（Re-assign）」一個新物件，外部變數會改變嗎？

```javascript
let user = { name: 'Alice' };

function changeUser(obj) {
  obj.name = 'Bob'; // ✅ 修改屬性：外部 user.name 會變成 'Bob'
  
  obj = { name: 'Charlie' }; // ❌ 重新賦值：切斷了引用鏈！obj 指向了新的記憶體位址
  obj.name = 'David';
}

changeUser(user);
console.log(user.name); // 輸出 'Bob' (而非 'David' 或 'Charlie')
```

- 解析：重新賦值 `obj = { ... }` 只是改變了內部區域變數 `obj` 的記憶體指向，不會改變外部 `user` 的指向，這也是為什麼說 JS 本質是 Pass by Copy of Reference 的原因。

## 回答策略 (Senior Response Strategy)
### 「請說明 Pass by Value 與 Pass by Reference 的差別，以及在 JavaScript 中如何避免物件的副作用？」

推薦答題架構：

- 破題與核心差異（What & Difference）：  
「Pass by Value 與 Pass by Reference 的根本差異在於傳遞資料時是『複製值』還是『分享記憶體位址』。

  - 基本型別（如 Number, String）是 Pass by Value，傳入函式的是資料副本，內部修改不會影響外部。
  - 物件型別（如 Object, Array）在 JS 中本質是 Pass by Copy of Reference（Call by Sharing），傳入的是指向 Heap 的引用位址。因此，在函式內修改物件屬性會直接突變（Mutate）外部的原始資料。

- 說明實務風險（The Risk - Side Effects）：  
在大型專案中，這種無意間修改原始物件行為（Side Effects）非常容易引發難以除錯的 Bug，尤其是在 React/Vue 等強調 **資料不可變性（Data Immutability）** 的框架中。

- 給出工程化防範方案（Best Practices - Senior Detail）：  
在實務撰寫函式時，為了遵循純函式（Pure Function）與不可變思維，我會避免直接修改傳入的物件，而是透過以下方式處理：
  - 淺拷貝（Shallow Copy）：使用 ES6 展開運算子 `const newObj = { ...value }` 或 `Object.assign()`。
  - 深拷貝（Deep Copy）：使用原生 `structuredClone(value)`（現代瀏覽器支援）或 Lodash 的 `cloneDeep()` 複製一份新物件後再進行操作。

## FAQ
### Q1: 在 React 或 Vue 等現代框架中，為什麼「不可變性（Immutability）」與 Pass by Reference 這麼重要？  
React 等框架依賴引用位址的比較（Shallow Comparison, 如 `prevProps === nextProps`）來判斷元件是否需要重新渲染（Re-render）。  
如果直接用 `obj.count++` 突變舊物件，因為位址完全沒變，React 會誤以為資料沒有更動而拒絕更新畫面 (UI)。因此實務上必須建立一個全新的物件位址（如 `setObj({ ...obj, count: obj.count + 1 })`），React 才能正確偵測到狀態改變。

### Q2: 深度拷貝（Deep Copy）物件時，為什麼不推薦用 JSON.parse(JSON.stringify(obj))？  
雖然 `JSON.parse(JSON.stringify(obj))` 是經典簡便寫法，但有重大限制：

- 會遺失（Drop） `undefined`、`Symbol` 與 `Function` 屬性。
- 會將 `Date` 物件轉為字串、`RegExp` 轉為空物件。
- 若物件有循環引用（Circular Reference）會直接丟出 `TypeError` 報錯。
- 現代建議：優先使用現代原生 API `structuredClone()`，或成熟的 Lib 如 `lodash.cloneDeep`。