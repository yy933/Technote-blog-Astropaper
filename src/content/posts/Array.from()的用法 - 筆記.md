---
title: "Array.from()的用法 - 筆記"
pubDatetime: 2025-03-14T00:25:42.000Z
tags: ["JavaScript","Interview Preparation"]
description: "Table of contents Array.from() 是 JavaScript 中非常多功能且強大的方法，主要..."
hackmd_id: "ryOJZml21g"
---

## Table of contents


`Array.from()` 是 JavaScript 中非常多功能且強大的方法，主要用來**將類陣列或可迭代對象轉換成真正的陣列**，並且可以同時進行映射處理。

## 基本語法
`Array.from(arrayLike, mapFunction, thisArg)`
`arrayLike`：任何類陣列或可迭代對象。
`mapFunction`（optional）：類似於 `.map()`，可對每個元素進行處理。
`thisArg`（optional）：指定 `mapFunction` 中的 `this`。


## `Array.from()` 的常見用途

### 將類陣列對象轉為陣列
像 `NodeList`、`arguments` 這類物件雖然類似陣列，但並不是真正的陣列，不能直接使用陣列方法（如 `.map()`、`.filter()`）。
```javascript
const divs = document.querySelectorAll('div'); // NodeList
const divArray = Array.from(divs);
console.log(Array.isArray(divArray)); // true
```

### 將字串轉為陣列
`Array.from() `會自動將字串分割成單個字元。
```javascript
const str = 'hello';
const arr = Array.from(str);
console.log(arr); // ['h', 'e', 'l', 'l', 'o']
```

### 快速創建連續數字陣列
```javascript
const arr = Array.from({ length: 5 }, (_, i) => i);
console.log(arr); // [0, 1, 2, 3, 4]
```

### 使用 mapFunction 直接處理元素
```javascript
const arr = Array.from([1, 2, 3], x => x * 2);
console.log(arr); // [2, 4, 6]
```
<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

`Array.from()` vs `.map()`
對於純陣列，`.map() `通常效能更好。

`.map() `是針對**原生陣列**優化的方法。
`Array.from()` 在處理陣列時，還會多進行一次**類型檢查和轉換**，因此速度略慢。
但如果來源是**類陣列或可迭代對象（如 NodeList、Set、arguments）**，`Array.from()` 更方便且高效。

用 `.map()` 之前還要先轉換為真正的陣列。

</blockquote>

### 將 `Set` 轉為陣列
`Set`是可迭代的，但不是陣列，使用` Array.from()` 可以輕鬆轉換。
```javascript
const set = new Set([1, 2, 3, 3]);
const arr = Array.from(set);
console.log(arr); // [1, 2, 3]
```

### 將 Map 的鍵或值轉為陣列
```javascript
const map = new Map([['a', 1], ['b', 2]]);
const keys = Array.from(map.keys());
const values = Array.from(map.values());
console.log(keys); // ['a', 'b']
console.log(values); // [1, 2]
```

### 消除空洞陣列
`Array.from()` 會自動將空洞處理為 `undefined`，讓方法如` .map()` 能正常運作。
```javascript
const arr = Array.from(new Array(3));
console.log(arr); // [undefined, undefined, undefined]
```

### 生成自訂格式的陣列
```javascript
const arr = Array.from({ length: 5 }, (_, i) => `Item ${i + 1}`);
console.log(arr); // ['Item 1', 'Item 2', 'Item 3', 'Item 4', 'Item 5']
```

### 利用 thisArg 綁定 this
指定 `mapFunction` 中的 `this`。
```javascript
const obj = { multiplier: 2 };
const arr = Array.from([1, 2, 3], function(x) {
  return x * this.multiplier;
}, obj);
console.log(arr); // [2, 4, 6]
```

### 轉換數字為數字陣列
把數字的每個位數拆開
```javascript
const number = 12345;
const digits = Array.from(String(number), Number);
console.log(digits); // [1, 2, 3, 4, 5]
```


## `Array.from()` vs `Array.of()` vs `Array()`

| 方法                     | 用途                            | 
|--------------------------|-------------------------------- |
| `Array.from()`  | 將**可迭代對象或類陣列轉成真正的陣列** | 
| `Array.of()` | 將任意數值轉為陣列 | 
| `Array()` | 建立陣列，但`數字作為單一參數時會建立空洞陣列` | 

```javascript
Array.from('abc');    // ['a', 'b', 'c']
Array.of(3);          // [3]
Array(3);             // [ <3 empty items> ]
```