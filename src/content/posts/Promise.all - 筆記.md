---
title: "Promise.all - 筆記"
pubDatetime: 2025-04-28T01:08:00.000Z
tags: ["JavaScript","asynchronous","Interview Preparation"]
description: "Table of contents ✨ 關於Promise，先參考[這篇筆記](https://hackmd.io/l..."
hackmd_id: "r1zU-ankxe"
---

## Table of contents


✨ 關於Promise，先參考[這篇筆記](https://hackmd.io/l_Tx5Tg7SqaPw4o6vHIzKA)。

## 基本原理：
在有多個 Promise 的時候，使用 `Promise.all` 可以確保 **「所有的 Promise 都執行完以後，才進入 then」**
![ExportedContentImage_00 (4)](https://hackmd.io/_uploads/B1uabah1gg.png)

* 當所有的 `Promise` 都成功完成時，`Promise.all()` 會回傳一個解析後的值，這個值是一個陣列，包含所有 `Promise` 解決的結果。
* 如果其中有任何一個 `Promise` 失敗（reject），`Promise.all()` 會立即拒絕並返回錯誤。


### 範例一
![ExportedContentImage_01 (5)](https://hackmd.io/_uploads/BJaWf6n1xe.png)

### 範例二
```javascript
const promise1 = new Promise((resolve, reject) => setTimeout(resolve, 1000, 'First'));
const promise2 = new Promise((resolve, reject) => setTimeout(resolve, 2000, 'Second'));
const promise3 = new Promise((resolve, reject) => setTimeout(resolve, 1500, 'Third'));

Promise.all([promise1, promise2, promise3])
  .then(results => {
    console.log(results);  // ['First', 'Second', 'Third']，所有 Promise 都成功完成
  })
  .catch(error => {
    console.error('Error occurred:', error);
  });
```

如果其中一個 Promise 失敗：
`Promise.all() `會馬上進入 `catch()`，並顯示錯誤訊息。
```javascript
const promise1 = new Promise((resolve, reject) => setTimeout(resolve, 1000, 'First'));
const promise2 = new Promise((resolve, reject) => setTimeout(reject, 500, 'Error in Second'));
const promise3 = new Promise((resolve, reject) => setTimeout(resolve, 1500, 'Third'));

Promise.all([promise1, promise2, promise3])
  .then(results => {
    console.log(results);  // 這行永遠不會執行
  })
  .catch(error => {
    console.error('Error occurred:', error);  // 'Error in Second'
  });

```

## 注意事項
### Promise.all 期待接受的參數是陣列形式
可以用 `.map` 來處理資料：
```javascript
Promise.all(
    users.map((user, user_index) => {
      return UserModel.create({ ...user }).then(...)
  )
```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

注意在 `UserModel.create` 之前要加 `return`，才會將 `UserModel.create` 這個 `Promise` 物件向上回傳給 `Promise.all`。

</blockquote>

### 不需要再用條件式來檢查終止條件
如果改成 `Promise.all` 的語法，則會直接將兩次的 `UserModel.create` 放進陣列參數，`Promise.all` 會確保陣列中所有的項目都執行完以後，才進入 `then`。因此就不需要原本的條件式了。

## 什麼時候要用 `Promise.all`
**當有多個 `Promise` 要並行處理，這些 `Promise` 之間沒有明確的先後順序，但一定需要「全都執行完」，才能進入後續流程**，此時，就可以考慮使用 `Promise.all`。當操作須按順序執行時，應考慮使用 `async/await` 來確保順序執行。