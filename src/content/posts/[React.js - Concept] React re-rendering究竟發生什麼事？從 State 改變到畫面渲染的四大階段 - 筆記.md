---
title: "[React.js - Concept] React re-rendering究竟發生什麼事？從 State 改變到畫面渲染的四大階段 - 筆記"
pubDatetime: 2026-07-13T03:49:40.362Z
tags: ["React.js","Reconciliation","Diffing Algorithm","Virtual DOM","Advanced React","Frontend","Concepts","React18"]
description: "Table of contents React 渲染流水線 在 React 的世界裡，畫面的更新並不是「一有變化就立刻..."
hackmd_id: "r1irnR-4Gg"
---

## Table of contents

## React 渲染流水線  
在 React 的世界裡，畫面的更新並不是「一有變化就立刻塗改瀏覽器」。為了追求極致的效能，React 設計了一套精密的流水線（Render Pipeline）。

自 React 18 的 Fiber 架構後，這整套更新流程被更嚴格地劃分為兩個主要階段：  
1. **Render Phase（包含觸發與比對，純 JS 計算，可被中斷、異步執行，對使用者隱形）**  
2. **Commit Phase（實際操作 DOM，必須同步執行且不可中斷，使用者可見）**

為了更好理解，我們可以將 React 的更新機制比喻為　**「連鎖披薩店的自動化廚房」**：
* **State (狀態)** = 點單內容（客戶要修改的配料）
* **Component (元件)** = 披薩食譜與製作手冊
* **Virtual DOM** = 桌上的設計圖、草稿
* **真實 DOM** = 烤箱裡正在烤、準備送給客人的實體披薩



## 1. State Change (觸發階段：訂單變更)

### 實際上發生了什麼事？  
當我們呼叫 `setState`、`dispatch` 或因為 Context 改變時，React 並不會**立刻**去翻動真實畫面。

**React 會把這個「狀態變更的請求」包裝成一個名為 **Update (更新物件)** 的資料結構，並排進該元件對應的 Fiber 節點的更新隊列（Update Queue）中**。隨後，React 會向瀏覽器調度（Schedule）一個更新任務，等待下一個 CPU 空檔開始執行。

### 🍕 披薩店生活化比喻
> 客人原本點了總匯披薩，突然打電話跟櫃台說：「不好意思，我的總匯披薩**不要加青椒，改加雙倍起司**！」
> 櫃台人員聽到了（`setState`），他沒有立刻衝進廚房把正在烤的披薩拉出來塗改，而是**在點單系統上敲入修改需求，產生一張新的修改通知（Update Queue）**。



## 2. Render (渲染階段：繪製新草稿)

### 實際上發生了什麼事？
**「Render」在 React 裡不等於塗改畫面，它只是「執行元件函式，算出新的 JSX」。**

React 會從觸發更新的元件開始，順著樹狀結構往下執行該元件的 Function。這個 Function 執行後會回傳 JSX，React 會將這些 JSX 轉換成最新的 **Virtual DOM (虛擬 DOM) 結構**。此時，**新版的虛擬樹已經在記憶體中誕生了**。

### 🍕 披薩店生活化比喻
> 廚房的顯示器亮起了修改通知。廚師（React）拿出了總匯披薩的標準食譜（Component），根據客人剛剛修改的單子，**在桌上的白紙上，重新畫了一張「不要青椒、加滿起司」的全新披薩設計圖（新 Virtual DOM）**。
> 注意！這時候真正的烤箱（真實 DOM）連碰都還沒碰，一切都還只是紙上談兵（純 JavaScript 計算）。



## 3. Reconciliation (協調/比對階段：找出差異)

### 實際上發生了什麼事？  
React 此時在記憶體中同時擁有兩棵樹：
* **Current Tree**：上一次渲染好、目前對應到畫面的「舊虛擬樹」。
* **Work-in-progress Tree**：剛剛在步驟 2 算出來的「新虛擬樹」。

在此階段，React 會啟動其核心的 **Diffing 演算法**，從根節點開始逐層比對這兩棵樹。它的目的非常單純：**「找出新舊樹之間，到底有哪些地方不一樣？」** 
**React 會把這些不一樣的地方（例如：節點類型改變、屬性變更、文字修改）標記起來，打包成一張「變更特效清單（Effect List）」。**

### 🍕 披薩店生活化比喻
> 廚師拿著剛剛畫好的**新設計圖**，跟**前一分鐘畫的舊設計圖**放在一起左右比對（Diffing）。
> 廚師開啟火眼金睛檢查：
> * 「洋蔥圈一樣有，不用動。」
> * 「臘腸片一樣有，不用動。」
> * 「啊！看到了，舊圖有青椒，新圖沒有；而且新圖的起司量從 1 變成 2！」
> 廚師最後在便條紙上寫下結論：**「指令：把青椒拿掉，補上一層起司。」** 這張便條紙就是變更清單。



## 4. Commit to DOM (提交階段：真正動工)

### ⚙️ 實際上發生了什麼事？  
一旦 Reconciliation 結束，React 確定了所有的變更，就會進入 **Commit Phase**。

React 會拿著剛剛打包好的變更清單，一口氣調用瀏覽器的原生 DOM API（例如 `appendChild`、`removeChild`、`setAttribute`），**以最少、最精準的步驟去修改畫面上真正的 HTML 元素**。

當 DOM 更新完成後，瀏覽器才會觸發重繪（Repaint），使用者才會看到畫面的改變。緊接著，React 會在背後依序觸發 `useLayoutEffect`、佈局重繪、以及非同步的 `useEffect` 等生命週期 Hook。

### 🍕 披薩店生活化比喻
> 廚師拿著那張精準的便條紙（變更清單），終於走向工作檯和烤箱（真實 DOM）。
> 他完全不需要把整片披薩倒掉重做，而是非常精準地**用夾子把披薩上的青椒一片片夾起來丟掉，然後抓起一把起司灑上去**。
> 動作一氣呵成！隨後把披薩送進去烤（瀏覽器 Repaint），香噴噴的改版披薩正式呈現在客人面前。

---

## 核心技術總結表

| 階段名稱 | 屬於哪個 Phase | 瀏覽器看得到變化嗎？ | React 在做什麼？ | 披薩店術語 |  
| :--- | :--- | :--- | :--- | :--- |  
| **1. State Change** | Trigger Phase | ❌ 看不到 | 收到變更請求，排入更新隊列 | 櫃台修改點單系統 |  
| **2. Render** | Render Phase | ❌ 看不到 (純 JS 計算) | 執行元件 Function，產出新 Virtual DOM | 廚師在白紙上畫新設計圖 |  
| **3. Reconciliation** | Render Phase | ❌ 看不到 (純 JS 計算) | 新舊 Virtual DOM 大比對，抓出差異 (Diffing) | 廚師左右對照新舊圖，寫下便條紙 |  
| **4. Commit** | Commit Phase | **⭕ 看得到！** | 呼叫原生 DOM API，精準修改真實網頁 | 廚師走向工作檯，動手夾走青椒、灑上起司 |



## 常見觀念問答
**Q: 為什麼 React 要大費周章把 Render 和 Commit 分開？直接改 DOM 不是更直接？**

A: 因為 **DOM 操作是極度昂貴且緩慢的**，而 JavaScript 的物件計算非常快。  
如果每次 state 一有小變動（例如一秒觸發 60 次的滑鼠追蹤）就直接去改真實 DOM，網頁一定會卡死。**React 透過 Render Phase 在記憶體裡進行大量的「預判」與「打包」，最後在 Commit Phase 只對真實 DOM 進行「一擊必殺」的精準修改，這就是 React 效能強大的祕密**。

**Q: 什麼是 Fiber 架構？它跟這四個階段有什麼關係？**

A: **Fiber 是 React 18 支援「可中斷渲染（Concurrent React）」的核心基礎。**  
在舊版 React 中，一旦開始 Render（步驟 2 + 3），就必須一口氣把整棵樹算完，期間會卡住瀏覽器的 UI 主執行緒（Main Thread）。  
而在 Fiber 架構下，**Render Phase 的工作被拆分成許多微小的任務單元（Fiber）**。React 在算完一個節點後，會抽空看一下瀏覽器有沒有更高等級的任務（例如使用者正在打字、點擊），如果有，React 會**暫停、放棄或延後**目前的 Render 計算，先讓瀏覽器回應使用者，等空閒了再重新計算。這種高彈性的調度，全部都發生在進入 Commit Phase 之前。
> 延伸閱讀：[A deep dive into React Fiber](https://blog.logrocket.com/deep-dive-react-fiber/)

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>