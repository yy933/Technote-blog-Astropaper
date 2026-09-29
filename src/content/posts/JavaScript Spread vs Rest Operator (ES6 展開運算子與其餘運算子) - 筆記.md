---
title: "JavaScript Spread vs Rest Operator (ES6 展開運算子與其餘運算子) - 筆記"
pubDatetime: 2026-09-29T07:06:53.964Z
tags: ["Interview Preparation","JavaScript","ES6","Frontend","Concepts"]
description: "Tags: Interview Preparation JavaScript Concepts ES6 Fronten..."
hackmd_id: "rJYh3G-9Mx"
---

###### Tags: `Interview Preparation` `JavaScript` `Concepts` `ES6` `Frontend`


## Table of contents

## 前言  
自 ES6 (ECMAScript 2015) 發布以來，`...`（三個點）運算子已成為現代 JavaScript 最常見且不可或缺的語法糖。對 **Spread Operator（展開運算子）** 與 **Rest Operator（其餘/其餘參數運算子）** 的精通，是評估開發者是否具備現代 JS 標準與語法潔癖（Syntactic Sugar）的基本門檻。兩者語法外觀相同，但**運作邏輯與使用場景恰好相反**。



## 核心特性與觀念總覽

* **三點語法雙雄**：
  * **Spread Operator（展開運算子）**：**拆解 / 解包（Unwrap / Unpack）**。將陣列或物件「展開」成獨立的元素或屬性。
  * **Rest Operator（其餘運算子）**：**組裝 / 打包（Collect / Pack）**。將多個獨立的元素或參數「收集」組合成一個陣列或物件。
* **淺拷貝（Shallow Copy）特性**：Spread Operator 在複製物件或陣列時執行的是**淺拷貝**，僅複製第一層資料結構。
* **屬性覆蓋機制**：在物件展開時，後定義的屬性會 **自動覆蓋（Override）** 先定義的同名屬性，常用於設定預設值（Default Options）。
* **替代傳統 `arguments`**：Rest Parameter 可在函式中接收不確定數量的參數，直接轉為真正的陣列（Array），完美取代傳統類陣列（Array-like）的 `arguments` 物件。



## 1. Spread vs Rest 比較

| 特性比較 | Spread Operator (展開運算子) | Rest Operator (其餘運算子) |  
| :--- | :--- | :--- |  
| **主要動作** | **解包 / 展開 (Unwrap / Spread)** | **打包 / 收集 (Collect / Gather)** |  
| **資料方向** | `[1, 2]` ➔ `1, 2` (一個變多個) | `1, 2, 3` ➔ `[1, 2, 3]` (多個變一個) |  
| **常見使用位置** | 陣列字面值 `[...]`、物件字面值 `{...}`、函式呼叫的引數 `fn(...arr)` | 函式定義的參數列表 `function(...args)`、解構賦值 `{ a, ...rest }` |  
| **替代傳統語法** | `Array.prototype.concat` / `Object.assign` | `arguments` 物件 |



## 2. 實務使用情境與程式碼範例

### 1. Spread Operator (展開運算子) 範例

#### A. 陣列串接與複製 (Array Merging & Copying)

```javascript
const users = ['Dylan', 'Per', 'Dolan'];

// 展開陣列並合併新元素 (不改變原陣列)
const allUsers = ['Olivia', ...users];

console.log(allUsers); // ['Olivia', 'Dylan', 'Per', 'Dolan']
```

#### B. 物件組合與預設值覆蓋 (Object Merging & Default Overriding)  
物件展開時，放置順序決定覆蓋結果：後者會覆蓋前者的同名 Key。

```javascript
// 預設設定
const defaultSettings = {
  theme: 'light',
  channel: 'Coding Tutorials 360'
};

// 使用者自訂設定
const userCustom = {
  channel: 'My Custom Channel' // 欲覆蓋的屬性
};

// 合併物件：userCustom 的 channel 會覆蓋 defaultSettings 的 channel
const fullUser = {
  ...defaultSettings,
  ...userCustom
};

console.log(fullUser); 
// { theme: 'light', channel: 'My Custom Channel' }
```

### 2. Rest Operator (其餘運算子) 範例
#### A. 函式可變參數收集 (Rest Parameters)  
在函式定義中使用 `...nums`，能將傳入的無限個引數打包成真正的陣列，可直接呼叫 `reduce`、`map` 等陣列方法：

```javascript
// 使用 Rest Operator 接收不確定數量的參數
function addNums(...nums) {
  // nums 在內部是一個真正的陣列
  return nums.reduce((total, current) => total + current, 0);
}

console.log(addNums(1, 2, 3)); // 6
console.log(addNums(1, 2));    // 3
console.log(addNums(10, 20, 30, 40)); // 100
```

#### B. 配合解構賦值 (Destructuring Assignment)  
擷取特定屬性後，將剩下的屬性打包為一個新的物件：

```javascript
const userProfile = {
  firstName: 'Dylan',
  lastName: 'Israel',
  channel: 'Coding Tutorials 360',
  subscriberCount: 100000
};

// 解構：取出 channel，並將剩餘的屬性打包進 remainder 物件
const { channel, ...remainder } = userProfile;

console.log(channel);   // 'Coding Tutorials 360'
console.log(remainder); // { firstName: 'Dylan', lastName: 'Israel', subscriberCount: 100000 }
```

## 3. 回答策略 (Senior Response Strategy)
- 區分動詞（Unwrap vs Pack）：  
一開始先用一句話總結：**Spread 是將集合拆解成個別元素（Unwrapping），Rest 是將多個元素收集成一個集合（Packing into Array/Object）。**

- 提及替代的舊語法：  
說明 Spread 替代了舊式的 `concat()` 與 `Object.assign()`；Rest 替代了沒有陣列原生方法且不靈活的 `arguments` 物件。

- 強調淺拷貝注意事項 (Shallow Copy Notice)：  
主動提醒 Spread Operator 在進行陣列/物件複製時僅為第一層的淺拷貝，若第二層包含引用型別（如巢狀物件），修改複製項依然會影響原物件。

## FAQ
### Q1: 為什麼使用 Rest Parameters (`...args`) 比使用傳統的 `arguments` 物件更好？
- `arguments` 不是真正的陣列：`arguments` 只是「類陣列物件（Array-like Object）」，無法直接呼叫 `map()`、`filter()`、`reduce()` 等陣列原生方法，必須先透過 `Array.from(arguments)` 轉型。

- 不支援箭頭函式 (Arrow Functions)：箭頭函式內部沒有自己的 `arguments` 物件，必須依賴 Rest Parameters。

- 語意更精準：Rest Parameters 可以選擇性地只收集「部分參數」（例如 `function(first, second, ...rest)`），彈性更高。

### Q2: 使用 Spread Operator 複製物件時，如何避免淺拷貝 (Shallow Copy) 的問題？  
Spread Operator 只能完成第一層的複製：

```javascript
const nestedObj = { a: 1, b: { c: 2 } };
const copiedObj = { ...nestedObj };

copiedObj.b.c = 99;
console.log(nestedObj.b.c); // 99 (原物件的第二層也被修改了！)
```

* 解決方案 1 (原生深拷貝)：使用現代瀏覽器內建的 `structuredClone(nestedObj)`。
* 解決方案 2 (第三方套件)：使用 Lodash 的 `_.cloneDeep()`。
* 解決方案 3 (簡易 JSON 轉換)：使用 `JSON.parse(JSON.stringify(nestedObj))`（但注意不支援 `Date`、`Function` 與 `undefined` 等特殊型別）。