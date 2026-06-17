---
title: "[React] 渲染陣列清單：key 屬性- 筆記"
pubDatetime: 2026-06-17T08:16:31.661Z
tags: ["JavaScript","React.js","Frontend"]
description: "Table of contents :memo: 簡介 在 React 開發中，我們經常需要處理動態資料，最常見的情境..."
hackmd_id: "HkD_XRJGzx"
---

## Table of contents

## :memo: 簡介  
在 React 開發中，我們經常需要處理動態資料，最常見的情境就是從 API 取得一個陣列（Array）後，利用 JavaScript 的 `.map()` 陣列方法將資料逐一渲染成網頁上的 HTML 標籤或元件。

然而，當我們開開心心地寫完 `.map()` 渲染出清單時，打開瀏覽器的 Console 主控台，往往會跳出一行紅色的經典警告：
```text
Warning: Each child in a list should have a unique "key" prop.
```

這個 `key` 屬性到底是什麼？為什麼 React 如此依賴它？**提升網頁效能與維持畫面正確性，這小小的屬性扮演了重要的角色。**


## :memo: 為什麼 React 需要 key？  
這必須從 React 的底層核心機制 —— **虛擬 DOM 的比較機制（Diffing Algorithm）**說起。

**當陣列資料發生變動（例如：新增項目、刪除某個欄位、或是將清單重新排序）時，React 會重新執行渲染，並產生一組「新狀態的虛擬 DOM 物件」，接著與「舊狀態的虛擬 DOM 物件」進行前後比對，最終只將「有改變的部分」更新到瀏覽器的真實網頁上。**

### 1. 如果「沒有加 key」會怎樣？
**如果沒有提供 key，React 無法辨識個別元素的身份，只能盲目地按照陣列的 索引順序（Index） 依序進行一對一的死板比對：**

* 舊的第 1 項 vs 新的第 1 項、舊的第 2 項 vs 新的第 2 項...
* **潛在 Bug 致命傷**： 假設你把陣列的「第一項項目」刪除了，React 透過順序比對，會誤以為是「前面每一項的內容都改變了，最後一項被刪除了」。  
這會導致 React 重新銷毀並重建大量真實的 DOM 節點，不僅造成嚴重的效能損耗，更可能引發畫面狀態錯位（例如：輸入框內的文字、核取方塊的勾選狀態對不上）的靈異現象。

### 2. 如果「有加唯一值的 key」
**key 就像是 React 幫每個動態生成的元件發放的 「身份證字號」。**

* 當資料順序改變時，React 只要看一眼 key，就能立刻認出：「喔！原來只是從第 3 名移動到第 1 名，本體沒有變，那我只要在畫面上挪動位置就好，不需要拆掉重做！」
* **透過這種方式，React 能夠達到最小幅度的網頁更新（Minimal DOM manipulation），極致優化渲染效能。**


## :memo: 什麼時候需要加 key？什麼時候不用？  
辨別是否需要加上 key 的黃金法則只有一個：「**當你在 JSX 中使用陣列、迴圈（迭代）動態產生一組並列的元件或 HTML 標籤時。**」

| 情境狀態 | 是否需要 key | 說明與範例 |  
| :--- | :---: | :--- |  
| **透過 `.map()` 產生清單** | **YES** | 屬於動態迭代渲染，必須加在 `.map()` 內部最外層的標籤上。 |  
| **手動寫死並列元素** | **NO** | 例如：`<a>首頁</a><a>關於</a>`。數量固定，React 已掌控結構，免加。 |  
| **單一元件內部的最外層** | **NO** | 元件本身內部不是迴圈，如 `<article>` 內部結構不需要自己加上 key。 |


## :memo: 實作分享
### 常見錯誤：在元件內部加上 `key`  
初學者常犯的錯誤是，將 `key` 寫在元件檔案內部的最外層標籤上，或者是誤以為 `key` 是一種可以用 `props.key` 讀取的屬性。

```jsx
// components/Entry.jsx (下層顯示元件)

export default function Entry(props) {
  // ❌ 錯誤：在這裡寫 key 是完全無效的！因為它不在 .map() 迴圈裡。
  // ❌ 備註：React 內部會沒收 key，因此 props 裡面根本拿不到 id (除非上層有特地傳入 id={...})
  return (
    <article className="journal-entry" key={props.id}> 
      <h2>{props.title}</h2>
    </article>
  );
}
```

### 正確寫法：`key` 必須在外層 `.map()` 的現場  
key 的特殊規則是：**必須寫在呈現陣列迭代的最外層元素上**。

```jsx
// App.jsx (上層管理元件)
import Entry from "./components/Entry";
import data from "./data";

export default function App() {
  return (
    <main className="container">
      {data.map((entry) => (
        // ✅ 正確：key 寫在 .map() 迭代出來的最外層 Entry 元件上
        <Entry
          key={entry.id} 
          title={entry.title}
          img={entry.img}
        />
      ))}
    </main>
  );
}
```


### 進階小技巧（Fragment 的例外狀況）  
如果在 `.map()` 內部不想產生多餘的 `<div>` 標籤，而選擇使用 Fragment 包裹，此時不能使用空標籤縮寫` <>...</>`（因為它沒辦法寫屬性），必須寫成完整的 React 元件寫法，才能順利帶入 key：

```jsx
{data.map((item) => (
  <React.Fragment key={item.id}>
    <h2>{item.title}</h2>
    <p>{item.text}</p>
  </React.Fragment>
))}
```

## 綁定 key 的三大金律  
為了確保 React 運作正常，選擇 key 的值時必須遵守以下規範：

* **同層唯一（Unique）**： 在「同一個陣列」中，每個元素的 `key` 必須是獨一無二的，不同陣列之間的 `key` 則不相干。
* **穩定不變（Stable）**： 絕對不能使用 `Math.random()` 這種每次渲染都會重新亂數產生的值，否則 React 每次都會誤以為是全新元件而全面銷毀重組，效能會變得更極致糟糕。
* **避免使用 index（陣列索引）**： 除非該陣列是「純靜態顯示」，永遠不會發生新增、刪除、或重新排序。否則一旦順序變動，`index` 帶來的對比錯覺一樣會引發嚴重的畫面渲染 Bug。**最佳首選永遠是資料庫自帶的 id。**

## 小結
* key 不是給開發者用的，而是 **給 React 認親、追蹤虛擬 DOM 身分證用的**。
* 請永遠將 `key` 綁在 `.map()` 內部的最外層節點。
* 好的 `key` 必須具備 唯一性、穩定性，並盡可能避免使用 `index`。


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>