---
title: "document.createDocumentFragment() 如何減少 DOM 操作並提高效能？ - 筆記"
pubDatetime: 2026-05-25T11:17:35.937Z
tags: ["JavaScript","asynchronous","Interview Preparation","axios"]
description: " Table of contents document.createDocumentFragment() 在 JavaS..."
---

## Table of contents


## `document.createDocumentFragment()`
在 JavaScript 操作 DOM 時，每次 `appendChild()` 或 `innerHTML `都會觸發 重新渲染（Repaint） 和 重新排版（Reflow），這會影響效能，特別是當有大量的 DOM 操作時。

`document.createDocumentFragment()`（文檔片段）是一個 **輕量級的虛擬 DOM 容器**，它不會直接改變 真實的DOM 結構，而是 **先在記憶體中暫存所有變更，最後一次性加入，減少不必要的渲染，從而提升效能。**
## 範例：不使用 DocumentFragment
**❌ 不使用 `DocumentFragment`（每次都更新 DOM，效能較差）**
```javascript
const container = document.querySelector("#container");

for (let i = 0; i < 100; i++) {
  const div = document.createElement("div");
  div.textContent = `Item ${i}`;
  container.appendChild(div); // 直接插入 DOM，觸發 100 次重繪
}
```
✅ 這段程式碼每次` appendChild(div)` 都會讓**瀏覽器重新計算佈局（Layout）並重繪（Repaint），導致性能下降。**

## 範例：使用 DocumentFragment
**✅ 使用` DocumentFragment`（效能更好，僅更新 DOM 一次）**
```javascript
const container = document.querySelector("#container");
const fragment = document.createDocumentFragment(); // 建立一個 DocumentFragment

for (let i = 0; i < 100; i++) {
  const div = document.createElement("div");
  div.textContent = `Item ${i}`;
  fragment.appendChild(div); // 先加到 fragment，記憶體操作，無 DOM 重繪
}

container.appendChild(fragment); // 一次性插入 DOM，只觸發 1 次重繪

```
✅ 這樣的做法，所有 DOM 變更都發生在**記憶體中的fragment**，`DocumentFragment` 不會立刻附加到 DOM，因此不會影響畫面渲染，直到 `appendChild(fragment)` 這一步才一次性更新 UI。最後 **一次性插入 container**，只會**觸發一次重繪（Repaint），效能更佳。**



## 總結
所以，在需要**批量新增元素**的情境下，`document.createDocumentFragment()` 能有效提升效能，特別是對於 **大量 DOM 操作（如動態渲染列表）** 的應用場景非常有幫助！

| 方法                     | 操作次數                             | 效能                       |
|--------------------------|--------------------------------|--------------------------|
| **直接 `appendChild`**   | N 次（每個元素都觸發 DOM 操作） | 差（頻繁 Repaint & Reflow） |
| **使用 `DocumentFragment`** | 1 次（記憶體內部操作 + 最後一次插入） | 佳（只觸發 1 次 DOM 操作）  |