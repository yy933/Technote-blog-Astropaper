---
title: "前端框架觀念：React Context API - 筆記"
pubDatetime: 2026-09-29T04:21:49.270Z
tags: ["Interview Preparation","React","React.js","Concepts","Frontend","React Hook"]
description: "Tags: Interview Preparation React React.js Concepts React H..."
hackmd_id: "ByunPhk5Mg"
---

###### Tags: `Interview Preparation` `React` `React.js` `Concepts` `React Hook` `Frontend`


## Table of contents

## 前言  
在 React 專案中，當資料需要跨越層層元件傳遞時，傳統的 **Prop Drilling（屬性鑽孔）** 往往會導致程式碼冗長且難以維護。React 提供了 **Context API** 作為跨層級共享資料的機制。然而，若缺乏權衡地過度使用 Context，容易造成元件不必要的 **Re-render（重新渲染）**，因此理解其與 Prop Drilling 的差異及使用時機是面試中的關鍵考點。



## 核心特性與觀念總覽

* **跨層級隱式傳遞**：在頂層提供資料（Provider），元件樹內部的任何子元件皆可直接存取（Consumer / `useContext`）。
* **避免過度依賴頂層**：僅將真正全域、跨元件共享的資料放在頂層 Context，避免單一 Context 膨脹引發效能問題。
* **小心效能問題**：Context 的值一旦發生變動，所有訂閱該 Context 的元件都會被觸發重新渲染（Re-render）。
* **權衡使用**：絕大多數常態資料仍適合透過 Props 明確傳遞，或切分成多個小型、區域性的 Context。



## 1. Context API 與 Prop Drilling 的差異比較

| 特性 / 項目 | Prop Drilling（屬性鑽孔） | Context API |  
| :--- | :--- | :--- |  
| **資料傳遞方式** | **顯式（Explicit）**：需一層層透過 Props 手動傳給子元件 | **隱式（Implicit）**：頂層宣告後，樹狀結構下的子元件可直接存取 |  
| **程式碼可讀性** | 層級過深時易造成「屬性鑽孔」，中間元件需傳遞不屬於自己的資料 | 簡化層級結構，中間元件無需處理不相關的 Props |  
| **資料追蹤與除錯** | 清楚知道資料從何處傳入，但中間層級有機會誤改資料 | 資料來源統一於 Provider，但較不易從 JSX 直接看出資料來源 |  
| **效能影響** | 僅影響接收該 Prop 的元件更新 | 當 Context Value 改變時，所有訂閱該 Context 的元件都會重新渲染 |



## 2. 何時不該使用 Context API？（與最佳實務）

面試時被問到「何時不該使用 Context API？」，關鍵回答在於 **「切勿過度使用（Overuse）」**。

### 不建議使用的情境  
1. **頻繁變動的高頻資料**：例如追蹤滑鼠軌跡、點擊次數或即時輸入框文字。因為每一次更新都會觸發所有使用該 Context 的子元件 Re-render。  
2. **非全域共享的區域狀態**：如果資料只在兩三個相鄰的元件間傳遞，使用 Props 是更直覺且安全的做法。

### Context API 的最佳適用情境  
適合放入 Context 的，通常是**變動頻率低、且全域/大範圍元件皆需存取**的資料：
* **使用者身分驗證（Authentication State）**：登入狀態、權限 Token。
* **網站主題/風格（UI Theme）**：深色/淺色模式（Dark/Light Mode）。
* **語系與國際化（i18n / Localization）**：網站語言設定。



## 3. 在 Function Component 中使用 Context 的做法

透過 `createContext` 建立上下文，並利用 `useContext` Hook 在子元件中存取資料：

```javascript
import { createContext, useContext, useState } from 'react';

// 1. 建立 Context
const ThemeContext = createContext('light');

function App() {
  const [theme, setTheme] = useState('dark');

  return (
    // 2. 使用 Provider 於頂層注入資料
    <ThemeContext.Provider value="{theme}">
      <Toolbar/>
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  // 中間元件不需要傳遞任何 theme Prop
  return <ThemeButton/>;
}

function ThemeButton() {
  // 3. 使用 useContext 直接取得頂層的資料
  const theme = useContext(ThemeContext);
  return <button className={theme}>目前主題：{theme}</button>;
}
```

## FAQ
### Q1: 為什麼 Context API 容易造成不必要的重新渲染（Unnecessary Re-renders）？  
當 Context Provider 的 value 發生變動時，React 會通知所有呼叫了 `useContext(MyContext)` 的子元件進行更新。即使子元件只用到 Context 物件中的其中一個小屬性，只要整個 Context 物件重新產生，該子元件就會被強制 Re-render。

### Q2: 如何避免 Context API 造成的重新渲染問題？
* 拆分 Context（Context Splitting）：將不相關的狀態拆分成多個獨立的 Context（例如將 `UserContext` 與 `ThemeContext` 分開），避免單一狀態改變影響全元件。

* 結合 `useMemo` 與 `useCallback`：將傳給 Provider 的 `value` 物件透過 `useMemo` 進行快取，防止父元件重新渲染時產生新的物件參考。

* 退回使用 Props 或專用的狀態管理庫：對於複雜且高頻變動的狀態，考慮使用如 Redux Toolkit、Zustand、Jotai 等具備精準訂閱（Selector）機制的狀態庫。