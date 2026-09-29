---
title: "JavaScript ES6 Destructuring Assignment (解構賦值) - 筆記"
pubDatetime: 2026-09-29T08:05:32.939Z
tags: ["Interview Preparation","JavaScript","ES6","Clean Code","Frontend","Concepts"]
description: "Tags: Interview Preparation JavaScript Concepts ES6 Clean C..."
hackmd_id: "BkXe51Fqzl"
---

###### Tags: `Interview Preparation` `JavaScript` `Concepts` `ES6` `Clean Code` `Frontend`


## Table of contents

## 前言  
解構賦值是ES6 的語法糖，也是**對 Clean Code（潔淨程式碼）、程式碼可讀性（Readability）與自文件化（Self-Documenting Code）的理解**。靈活運用解構賦值，能大幅減少冗長的索引寫法，並清晰展現開發者的「設計意圖（Intent）」。



## 核心特性與觀念總覽

* **語法定義**：ES6 提供的語法糖，允許從陣列（Array）或物件（Object）中直接萃取資料，並賦予獨立變數。
* **核心價值（Why use it?）**：
  * **提升可讀性與自文件化**：擺脫無意義的陣列索引（如 `dob[0]`）或重複的物件點記法（如 `user.fName`）。
  * **重新命名與語意修復（Aliasing）**：改善不良的 API 命名（例如將簡寫欄位 `f` 重新映射為 `firstName`）。
  * **按需萃取（Selective Extraction）**：僅提取當前函式或模組需要的屬性，避免傳遞龐大不透明的完整物件。
* **陣列 vs 物件解構機制**：
  * **陣列解構**：依據 **位置順序（Order/Index）** 提取。
  * **物件解構**：依據 **屬性名稱（Key Match）** 提取。



## 1. 陣列與物件解構核心比較與速查

| 比較面向 | 陣列解構 (Array Destructuring) | 物件解構 (Object Destructuring) |  
| :--- | :--- | :--- |  
| **匹配依據** | **位置順序 (Order / Index-based)** | **屬性名稱 (Key Matching)** |  
| **宣告語法** | `const [a, b] = array;` | `const { a, b } = object;` |  
| **重新命名 (Alias)** | 直接在括號內給予變數名稱（如 `[month, day]`） | 使用冒號語法 `const { f: firstName } = obj;` |  
| **預設值設定** | `const [a = 10] = [];` | `const { role = 'User' } = {};` |  
| **典型應用** | `useState` Hook、座標點/日期拆解、變數數值交換 | API 回傳資料拆解、React Props 接收、模組匯入 |



## 2. 實務使用情境與程式碼範例

### 1. 陣列解構 (Array Destructuring)

#### A. 提供語意上下文 (Providing Context)  
擺脫 `dob[0]`、`dob[1]` 這種缺乏語意的數字索引：

```javascript
const dob = [12, 25, 1995]; // [月, 日, 年]

// ❌ 舊式寫法：缺乏語意，容易混淆月與日
const month = dob[0];
const day = dob[1];
const year = dob[2];

// ✅ 現代解構寫法：自文件化 (Self-documenting)，清楚表達變數意圖
const [month, day, year] = dob;

console.log(`${year}-${month}-${day}`); // '1995-12-25'
````

### 2. 物件解構 (Object Destructuring)
#### A. 重新命名與改善不良 API 資料結構 (Aliasing / Renaming)  
當後端或舊系統傳回命名極差的欄位時（如 f 代表 firstName），可以在解構時直接重新賦予有意義的變數名稱：

```javascript
// 舊系統傳回的不良資料結構
const rawUser = {
  f: 'Dylan',
  l: 'Israel'
};

// ❌ 舊式寫法：後續程式碼充滿無意義的 .f 與 .l
console.log(rawUser.f);

// ✅ 現代解構寫法：提取時順便進行別名映射 (Alias)
const { f: firstName, l: lastName } = rawUser;

console.log(firstName); // 'Dylan'
console.log(lastName);  // 'Israel'
```

#### B. 設定預設值 (Default Values)  
當提取的屬性可能是 `undefined` 時，直接在解構中指定預設值：

```javascript
const settings = {
  theme: 'dark'
};

// 若 role 不存在，則自動帶入預設值 'Guest'
const { theme, role = 'Guest' } = settings;

console.log(role); // 'Guest'
```

## 3. 回答策略 (Senior Response Strategy)  
在回答面試官時，建議採取「定義 ➔ 價值（Why）➔ 實務亮點」三層式答題結構：

* 定義（What）：  
切入重點：解構賦值是 ES6 提煉陣列與物件資料的語法糖，陣列看順序，物件看鍵名。

* 核心價值（Why - 展示軟體工程思維）：  
強調這不僅是寫法變短，更是為了 Clean Code 的自文件化（Self-documenting Code） 與 表達程式意圖（Intent）。

* 實務與框架結合 (加分項)：
  - 提到 React 中 `useState` Hook 的陣列解構（如 `const [count, setCount] = useState(0)`）就是利用陣列解構能自由自訂變數名稱的特性。

  - 提到 React 的 Props 解構（如 `function Card({ title, content })`），能簡化 `props.title` 的重複書寫並公開組件需要的依賴。

## FAQ
### Q1: JavaScript 中如何利用陣列解構進行「變數值交換 (Variable Swapping)」？  
傳統寫法需要透過暫存變數 temp，利用陣列解構一行就能搞定：

```javascript
let a = 1;
let b = 2;

// ❌ 舊式寫法
// let temp = a; a = b; b = temp;

// ✅ 利用陣列解構進行數值交換
[a, b] = [b, a];

console.log(a); // 2
console.log(b); // 1
```

### Q2: 如果解構的物件本身是 `null` 或 `undefined` 會發生什麼事？該如何防禦？  
如果直接對 `null` 或 `undefined` 進行解構，會拋出 `TypeError: Cannot destructure property ... of 'null' as it is null.`。

- 防禦寫法：搭配預設空物件 `{}`

```javascript
function getProfile(user) {
  // 防禦寫法：若傳入的 user 是 null/undefined，則對空物件 {} 進行解構
  const { name = 'Anonymous', age = 0 } = user || {};
  console.log(name, age);
}

getProfile(null); // 'Anonymous' 0 (不會 Crash!)
```