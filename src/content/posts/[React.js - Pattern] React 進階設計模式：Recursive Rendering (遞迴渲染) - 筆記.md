---
title: "[React.js - Pattern] React 進階設計模式：Recursive Rendering (遞迴渲染) - 筆記"
pubDatetime: 2026-07-13T03:14:05.533Z
tags: ["React.js","Design Pattern","Recursive Rendering","Advanced React","Frontend","Data Structure"]
description: "Table of contents Recursive Rendering 核心觀念 Recursive Render..."
hackmd_id: "rkhnm0WVMx"
---

## Table of contents

## Recursive Rendering 核心觀念
**Recursive Rendering (遞迴渲染)** 是前端開發（特別是像 React 這類組件化框架）中處理複雜資料結構的一種強大技巧。  
它的核心思想是：**「元件在它自己的模板（JSX）裡面，再次呼叫元件自己。」**

這就像是程式設計中的遞迴函式（Recursive Function），只是它最後產出的不是一個單純的計算數值，而是一整層一整層具有嵌套關係的 **UI 畫面**。

簡單來說，**當資料本身具備「圖形結構」或「樹狀結構」（不知道總共有幾層，且每一層的資料格式都極為相似）時，我們就可以利用組件自體呼叫的特性，用最少、最乾淨的程式碼去處理結構無限複雜的畫面。**



## 1. 什麼時候我們需要 Recursive Rendering？

在開發複雜的系統或網頁介面時，我們經常會遇到資料層級「無限延伸」的場景。如果使用傳統的 `map()` 迴圈，我們就必須在程式碼中寫死（Hardcode）多層嵌套，這在層級不固定時是完全無法實現的。

### 常見的應用場景  
1. **樹狀目錄 / 檔案瀏覽器**：資料夾裡面還有資料夾，最內層才是檔案（如：VS Code 的側邊欄）。  
2. **多層級選單 / 麵包屑導航**：電商網站的無限分類（手機防護 -> Apple 週邊 -> iPhone 機型 -> 手機殼）。  
3. **留言板的嵌套回覆**：像是 Reddit、PTT 或 Facebook 的留言系統，一則留言下方可以有無數層的「回覆的回覆」。



## 2. 資料結構與運作機制  
遞迴渲染的運作完全建立在資料的 **樹狀結構（Tree Structure）** 上。每一個節點（Node）通常會包含一個陣列屬性（如 `children` 或 `subItems`），裡面存放著下一層的節點資料。

### 遞迴渲染三大核心原則  
1. **必須有終止條件（Base Case）**：必須設定一個條件（例如沒有子資料了），讓遞迴適時停止。如果漏掉這個條件，瀏覽器會陷入無窮迴圈，導致記憶體溢位（Stack Overflow）而崩潰。  
2. **記得提供唯一的 `key`**：由於是在 `map()` 循環中重複呼叫元件自身，每一層的子元件都必須給予唯一的 `key`，React 才能正確追蹤 DOM 節點以維持渲染效能。  
3. **注意大型資料的效能**：如果樹狀資料非常龐大，一次性全部遞迴渲染會拖慢網頁速度。實務上通常會搭配 `state` 做「點擊才展開（Lazy Loading）」，或是導入虛擬列表（Virtual List）。

```
[FileNode 元件 - 根目錄] (渲染 📁 我的專案)
│
▼ 檢查：發現有 children 陣列
[map 循環呼叫] ───► [FileNode 元件 - 子項目 1] (渲染 📄 index.html) ──► 無 children (遞迴終止)
│
▼
[map 循環呼叫] ───► [FileNode 元件 - 子項目 2] (渲染 📁 src)
│
▼ 檢查：發現有 children 陣列
[map 循環呼叫] ───► [FileNode 元件] (渲染 📄 App.js)
```

以下以實際程式碼說明。

## 範例程式碼架構  
以下展示如何將一段未知層級的 JSON 資料，透過 React 的遞迴組件完美渲染成檔案樹。

### 原始資料：`fileData.js`
```javascript
const fileData = {
  name: "我的專案",
  type: "folder",
  children: [
    { name: "index.html", type: "file" },
    {
      name: "src",
      type: "folder",
      children: [
        { name: "App.js", type: "file" },
        { name: "styles.css", type: "file" }
      ]
    }
  ]
};
```

### 遞迴元件：FileNode.jsx  
在這個元件中，當判斷當前節點是資料夾（`isFolder`）且擁有子項目時，就會在 `map` 中直接呼叫 `<FileNode />` 自身。

```javascript
// FileNode.jsx
import React from 'react';

export default function FileNode({ item }) {
  const isFolder = item.type === 'folder';

  return (
    <div style={{ paddingLeft: '20px', fontFamily: 'monospace' }}>
      {/* 1. 渲染當前節點的 UI */}
      <div>
        {isFolder ? '📁' : '📄'} {item.name}
      </div>
      
      {/* 2. 關鍵點：滿足條件時，組件在內部「自己呼叫自己」 */}
      {isFolder && item.children && (
        <div>
          {item.children.map((child, index) => (
            // 遞迴呼叫，並記得帶入唯一的 key
            <FileNode item="{child}" key="{`${child.name}-${index}`}"/>
          ))}
        </div>
      )}
    </div>
  );
}
```

### 主元件：App.jsx

```javascript
import React from 'react';
import FileNode from './components/FileNode';
import { fileData } from './data/fileData';

function App() {
  return (
    <div className="app-container">
      <h3>專案檔案瀏覽器</h3>
      <hr />
      {/* 傳入根節點，讓元件開啟自動遞迴渲染 */}
      <FileNode item="{fileData}"/>
    </div>
  );
}

export default App;
```

## 常見觀念問答
**Q: 遞迴渲染跟一般的 `map` 渲染有什麼本質上的差別？**

A: 一言以蔽之：**「動態層級」與「靜態層級」的差別。**

* 一般 `map` 渲染：**適合處理「一維陣列」或「固定層級」的資料（例如商品列表、單層表格）。** 你必須明確知道有幾層，並寫死對應數量的 `map`。
* 遞迴渲染：**適合處理「縱向無限延伸」的樹狀結構。** 你只需要寫「一層」的元件邏輯，它就會根據資料本身的深度，自動長出對應數量的層級，彈性極高。

**Q: 在實務上，如果遇到幾萬條資料的超大樹狀結構，遞迴渲染該如何優化？**

A: 實務上會採用以下兩種主流優化策略：

* **動態展開（Lazy Loading / Collapsible）**：在 `FileNode` 內部加入一個 `isOpen` 的 state。預設不渲染 `children`，只有當使用者點擊資料夾時，才將 `isOpen` 設為 `true` 並觸發下一層的遞迴。這樣可以避免網頁初始化時一次建立太多 DOM 節點。
* **扁平化資料 + 虛擬列表（Virtualization）**：如果必須全部展開，通常會先在後端或前端將樹狀資料「扁平化（Flatten）」成一維陣列，並計算好每個節點的深度（Indent）。接著搭配 react-window 等虛擬列表套件，只渲染畫面看得到的節點，這才是應對大數據的終極解法。

## Recursive rendering的優勢
### 1. 程式碼體積優化 (Bundled Size & Maintenance)
* 省去冗餘的程式碼：如果你需要支援 10 層的樹狀結構，用傳統寫法你需要複製貼上 10 層極為相似的 UI 邏輯。這會導致打包出來的 JavaScript 檔案體積變大，進而影響網頁的首次載入速度 (First Load Time)。
* 以逸待勞：遞迴渲染只需要一份組件的程式碼（例如只寫了 50 行），就能動態應付 10 層、100 層甚至無限層的資料。對瀏覽器來說，需要解析的 JS 語法變少了。

### 2. 極易結合「狀態封裝」，達成天然的Lazy Rendering  
遞迴渲染最強大的地方，在於**它非常容易與組件的內部狀態（State）結合**。當你把「展開/收合」的邏輯寫在遞迴組件內部時，它能帶來巨大的效能好處：

**核心原理：**
**如果子層級沒有被點擊展開，遞迴就不會繼續向下執行，底下的 JSX 就不會被觸發，React 也就完全不會建立那些尚未展開的 DOM 節點。**
* 減少初始 DOM 節點數量：假設你有一萬個檔案的樹狀目錄，如果一開始全塞給瀏覽器會直接卡死。但使用遞迴搭配 `isOpen` 控制，初始畫面可能只需要渲染根目錄的 5 個節點。**剩下的 9,995 個節點，只有在使用者點擊時才會「動態遞迴渲染」出來。** 這極大程度優化了首次內容繪製 (FCP, First Contentful Paint)。

### 3. 記憶體開銷與資料結構的完美契合  
React 的 Virtual DOM 在進行 Diffing 演算法（比較前後畫面差異）時，**是採用深度優先搜尋 (DFS, Depth-First Search) 的樹狀走訪。**

遞迴渲染的組件結構，完美對應了 React 內部的 Fiber 樹架構。**當某個深層節點的資料改變時，React 可以非常精準地只針對該遞迴子樹進行 Re-render，而不會波及到其他不相干的平行分支**，這在局部資料更新時能維持良好的執行期（Runtime）效能。

## 小結
**遞迴渲染本質上是為了解決「複雜度」而非「速度」**。它真正的價值在於 **「用最精簡的程式碼，提供完美的動態擴充性」**。

當它與組件內的 state 結合實現 **「點擊才展開」** 時，就能化被動為主動，成為避免瀏覽器一次性載入大量 DOM 節點的關鍵效能利器！

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>