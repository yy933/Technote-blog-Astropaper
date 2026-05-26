---
title: "for...of 和 for...in 迭代器的差別 - 筆記"
pubDatetime: 2026-05-26T03:29:26.398Z
tags: ["others"]
description: " Table of contents for...of 和 for...in 在 JavaScript 中都是用來遍歷（..."
---

## Table of contents

`for...of` 和 `for...in` 在 JavaScript 中都是用來遍歷（迴圈）物件的，但用途和行為不同。
## `for...of` 
* 遍歷 **值** : `for...of` 語法執行一個迴圈，該迴圈操作來自**可迭代物件的值序列**。([MDN Doc](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Statements/for...of))
* 適用於：陣列、字串、Map、Set、NodeList 等可迭代物件（iterable objects）。
* 作用：直接取得值（element）。
* **不適用於：物件（plain object）**。
* 範例一：陣列
```javascript
const fruits = ["🍎", "🍌", "🍇"];

for (let fruit of fruits) {
  console.log(fruit); // 依序輸出 "🍎"、"🍌"、"🍇"
}
```
* 範例二：字串
```javascript
for (let char of "Hello") {
  console.log(char); // 依序輸出 H, e, l, l, o
}
```


## `for...in` 
* 遍歷 **索引(index)或物件的鍵(key)**：迭代物件的**可列舉屬性**。
* 適用於：物件（Object）、陣列（Array）。
* 作用：
-- 如果是**物件**，取得**物件的 key（屬性名稱）**。
-- 如果是**陣列**，取得**索引值（index）**。
* 範例一：陣列（取得index）
```javascript
const colors = ["red", "blue", "green"];

for (let index in colors) {
  console.log(index, colors[index]);
}
// 輸出：
// 0 red
// 1 blue
// 2 green
```
* 範例二：遍歷物件
```javascript
const person = { name: "Alice", age: 25, city: "Taipei" };

for (let key in person) {
  console.log(`${key}: ${person[key]}`);
}
// 輸出：
// name: Alice
// age: 25
// city: Taipei
```

## 總結
📌遍歷「值」👉 `for...of`
📌遍歷「索引或鍵」👉 `for...in`