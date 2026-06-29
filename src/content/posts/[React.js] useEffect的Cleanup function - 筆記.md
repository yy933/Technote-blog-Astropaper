---
title: "[React.js] useEffect的Cleanup function - 筆記"
pubDatetime: 2026-06-29T04:49:59.059Z
tags: ["JavaScript","React.js","Frontend","React Hook"]
description: "Table of contents 什麼是 Cleanup Function（清除函式）？ 在 React 的 use..."
hackmd_id: "B1gqfukXMx"
---

## Table of contents


## 什麼是 Cleanup Function（清除函式）？  
在 React 的 useEffect 中，如果我們在 callback 函式中回傳了另一個函式，這個被回傳的函式就被稱為 Cleanup Function（清除函式）。

它的核心目的很簡單：「有借有還，再借不難」。當我們用 useEffect 對外部世界製造了某些影響（也就是Side Effects）時，Cleanup Function 就是用來把這些影響「擦乾淨」的橡皮擦。

```jsx
React.useEffect(() => {
  // 1. 這裡執行副作用（例如：訂閱、設定計時器）
  console.log("效應開始");

  return () => {
    // 2. 這裡寫清除邏輯（Cleanup）
    console.log("清理戰場");
  };
}, [dependencies]);
```

## 什麼時候會觸發 Cleanup Function？  
React 會在以下兩個時間點自動執行 Cleanup Function：

1. **元件被卸載（Unmount）時**：當這個元件從畫面上徹底消失時。  
1. **下一次執行 useEffect 之前**：當依賴（Dependencies）改變，useEffect 準備要重新執行新一輪的程式碼之前，會先執行上一次留下來的 Cleanup Function。

什麼時候需要它？（常見情境與範例）  
只要你的副作用如果不手動關閉，就會一直活在背景、佔用記憶體，或是重複疊加，你就絕對需要 Cleanup Function。

以下是三個經典的範例：

### 範例 1：計時器（setInterval 或 setTimeout）  
如果在元件裡啟動了一個每秒執行的計時器，當元件消失時，計時器不會自動停止，它會繼續在背景執行，導致記憶體洩漏（Memory Leak）甚至操作到已經不存在的 State。

```jsx
import React from "react";

export default function TimerComponent() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    // 建立計時器
    const intervalId = setInterval(() => {
      setCount(prev => prev + 1);
      console.log("計時器跳動中...");
    }, 1000);

    // ❌ 如果不寫 cleanup，當元件切換隱藏時，這個計時器會永遠在背景瘋狂執行！
    return () => {
      clearInterval(intervalId); // ✅ 乾淨溜溜！元件卸載時自動清除
      console.log("計時器已被成功清除");
    };
  }, []);

  return <h2>秒數：{count}</h2>;
}
```

### 範例 2：監聽全域事件（addEventListener）  
假設元件需要監聽使用者的視窗大小變化（resize）或是滑鼠點擊。

```jsx
React.useEffect(() => {
  const handleResize = () => {
    console.log("當前視窗寬度：", window.innerWidth);
  };

  window.addEventListener("resize", handleResize);

  return () => {
    // ✅ 必須移除監聽，否則每當元件重新渲染或卸載，背景就會堆疊越來越多監聽器
    window.removeEventListener("resize", handleResize);
  };
}, []);
```

如果不寫 Cleanup，當每次元件重新渲染，瀏覽器就會多綁定一個 resize 事件，最後網頁就會變得越來越卡。

### 範例 3：取消非同步請求（防止 Race Condition 競態條件）  
這是一個進階但極度重要的情境。假設在頁面 A 點擊按鈕切換到頁面 B，但頁面 A 的 API 請求非常慢，在換到頁面 B 之後，頁面 A 的請求才成功回傳。這時它會試圖去更新已經死掉的頁面 A 的 State，這就是 Race Condition。

我們可以用 `AbortController` 來取消未完成的 fetch：

```jsx
React.useEffect(() => {
  const controller = new AbortController();
  const signal = controller.signal;

  async function fetchData() {
    try {
      const res = await fetch("https://swapi.dev/api/people/1", { signal });
      const data = await res.json();
      setStarWarsData(data);
    } catch (error) {
      if (error.name === 'AbortError') {
        console.log('成功取消了上一次未完成的 Fetch！');
      }
    }
  }

  fetchData();

  return () => {
    // ✅ 當元件卸載，或者 id 改變導致重新 fetch 前，把上一次沒跑完的網路請求直接「斷線」
    controller.abort();
  };
}, [id]); // 假設 id 改變時會重新抓取
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

#### 甚麼時候 API 請求需要用 Cleanup 取消?
**核心的判斷標準是：「這個 API 請求的結果，會不會因為『時間差』或『元件消失』而導致嚴重的side effect或 Bug？」**

**1. 絕對需要 Cleanup 取消的情境**
* 情況 A：dependencies（如 ID、關鍵字）頻繁改變的「搜尋/切換」功能  
這是最容易發生 Race Condition（競態條件）的地方。
  - 場景：使用者在搜尋框快速輸入 A --> AB --> ABC。
  - 危機：瀏覽器會同時發出 3 個請求。如果搜尋 A 的伺服器回應最慢，最後才傳回來，畫面就會在使用者明明想看 ABC 的情況下，突然被蓋成 A 的搜尋結果。

* 情況 B：元件會被頻繁切換、隱藏或卸載（Unmount）
  - 場景：使用者在分頁 A（例如「儀表板」）點擊抓取大量數據，但等不及加載完，就立刻切換到分頁 B（例如「設定」）。
  - 危機：雖然現代 React 在元件卸載後呼叫 `setState` 不再跳出紅字警告，但讓一個已經不存在的元件在背景默默處理龐大的 JSON 資料、浪費使用者的網路頻寬與 CPU 效能，依然是非常不健康的行為。

**2. 「不需要」或「不需要特別」取消的情境**
* 情況 A：元件只在初次載入（Mount）時抓取一次，且不會輕易消失
  - 原因：它只會執行一次，只要使用者不關掉網頁，這個元件就不會消失，也沒有dependencies改變的問題，因此絕對不會發生 Race Condition。

* 情況 B：非同步的「操作型」請求（如：新增、刪除、修改）
  - 範例：使用者點擊「送出表單」、「刪除文章」、「按讚」。
  - 原因：**這種請求的目的通常是「改變資料庫的狀態」**。即使使用者點完按鈕立刻跳轉頁面，我們通常也希望這個請求在背景默默完成，而不是把它取消（如果取消了，使用者的表單或按讚可能就失敗了）。

#### 一秒判斷法  
可以問自己以下兩個問題，只要其中一個答案是 「會」，就建議加上 Cleanup 取消：
* 「使用者在資料還沒回來前，有沒有可能切換頁面或隱藏這個區塊？」
* 「觸發這個 API 的參數（如 ID），有沒有可能在短時間內連續改變？」

#### 現代前端的偷懶解法  
如果覺得每次都要手寫 `AbortController` 太痛苦，現代資料抓取套件例如 React Query (TanStack Query) 或 SWR 預設就處理好所有的 Race Condition 和自動取消機制！當dependencies的 `queryKey` 改變時，套件會自動把舊的、還沒完成的請求斷線（Abort），一行 Cleanup 都不用寫。


</blockquote>

##  總結  
只要在 `useEffect` 裡寫了：
* `setInterval` --> 就要搭配 `clearInterval`
* `addEventListener` --> 就要搭配 `removeEventListener`
* `WebSocket` 連線 --> 就要搭配 `socket.close()`  
把它當作一種習慣，就不容易出現莫名其妙的效能問題或 Bug 囉！



<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>