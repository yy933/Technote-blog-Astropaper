---
title: "Async/Await 語法糖 - 筆記"
pubDatetime: 2026-05-25T11:17:35.876Z
tags: ["JavaScript","asynchronous","Interview Preparation"]
description: " Table of contents Prerequisite ✨ Async/await 的底層仍然是 Promise..."
---

## Table of contents

## Prerequisite
✨ `Async/await` 的底層仍然是 `Promise`，針對 promise-based 函式再次進行語法包裝，讓程式碼讀起來更接近同步處理。因此需要先了解 `Promise`:
* 關於`Promise`，參考[這篇筆記](https://hackmd.io/l_Tx5Tg7SqaPw4o6vHIzKA)。
* 關於`Promise.all`，參考[這篇筆記](https://hackmd.io/9l_LMhZcQC66OtC3S8HYtg)。

## Async/Await 語法
![ExportedContentImage_00 (5)](https://hackmd.io/_uploads/rkZ5U6nygx.png)

## 使用原則

* 確認有 `Promise` 物件實例，也就是已經定義好 `resolve/reject`，才能使用 `async/await`
* 在流程中正確設定關鍵字：
  - 把後續流程用一個 async function 包裝起來
  - 設定好 async function 之後，在要運用非同步處理的地方加上 await 關鍵字
* `async/await` 和 `then` 不可以混搭使用

### 範例一:await 等待多個 Promise
![ExportedContentImage_01 (6)](https://hackmd.io/_uploads/SkZiD6h1xl.png)

:::warning
改寫重點:
* 插入 async 關鍵字時，不需要更動原有結構，只要是「函式」就可以插在前面
* async/await 改寫掉的是 then 的串鏈，Promise 物件本身要保留下來
* 經過 async/await 後，沒有留下任何的 then
* 原本的 return 會拿掉，因為在原本寫法中，return 的意思是回傳 Promise 物件，讓 then 的串鏈可以運作下去
:::

### 範例二
```javascript
// 模擬一個非同步操作
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function asyncTask() {
  console.log('Task started');
  await delay(2000);  // 等待 2 秒
  console.log('Task finished');
}

asyncTask();
```

### 範例三:串聯 Promise
```javascript
async function asyncChain() {
  let step1 = await fetchData('url1');
  console.log(step1);

  let step2 = await fetchData(`url2?prev=${step1}`);
  console.log(step2);

  let step3 = await fetchData(`url3?prev=${step2}`);
  console.log(step3);
}

asyncChain();
```

## 錯誤處理: try/catch
原本 Promise 寫法，會在 then 串鏈的最後加 catch，去對應 reject 情境。

而 await/async 改寫後不會有 then 串鏈，因此也不會用到 Promise 語法中的 catch，需要進行錯誤處理時，會採用 [try...catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch) ，在更外層包一層 try {}，這是同步處理中延續已久的作法。
![ExportedContentImage_02 (5)](https://hackmd.io/_uploads/Bk-FuphJeg.png)

### 範例
```javascript
async function asyncTask() {
  try {
    let result = await someAsyncFunction();
    console.log(result);
  } catch (error) {
    console.error('Error:', error);
  }
}
```