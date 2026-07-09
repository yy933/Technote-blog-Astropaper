---
title: "[React - Design Pattern] 什麼是無頭元件 (Headless Component)？ - 筆記"
pubDatetime: 2026-07-09T09:26:41.140Z
tags: ["React.js","Design Pattern","Custom Hooks","Frontend","TailwindCSS","CSS","Shadcn","Concepts"]
description: "Table of contents 核心觀念 Headless Component 是一種將「功能邏輯」與「視覺外觀」..."
hackmd_id: "H1q-IyTmfx"
---

## Table of contents

## 核心觀念
**Headless Component** 是一種**將「功能邏輯」與「視覺外觀」完全抽離的前端設計模式**。

* **定義**：它是一個**只負責處理「邏輯、狀態與功能（大腦）」，但完全不提供任何「視覺樣式與預設 HTML 結構（外表）」** 的元件。
* **無頭（Headless）的意思**：就是該元件沒有預設的「腦袋」（也就是畫面），它把畫面的 100% 主導權完整交還給開發者。



## 1. 為什麼需要Headless Component

在傳統的 UI 元件庫（如早期的 Bootstrap、Material UI）中，邏輯與樣式是強耦合的：



### 傳統元件的缺點  
當一個切換開關（Toggle）元件內部寫死了 HTML 標籤與 CSS 樣式時，一旦設計師要求：「專案 A 的開關要長得像 iOS 的按鈕，專案 B 的開關要長得像傳統按鈕，且字要換掉。」  
這時候，即使「開與關」的邏輯、無障礙規格（A11y）完全一模一樣，你也必須重寫兩個元件，因為**無法輕易剝離原本綁定好的樣式**。



## 2. 範例對比說明

以一個純粹的「切換開關 (Toggle)」功能為例，看看 Headless 在現代 React（通常以 **Custom Hook** 呈現）中如何運作：

### 大腦：Headless邏輯（useToggle）  
這個 Hook 負責管好狀態，並把需要綁定在 HTML 元素上的屬性打包好（也就是 Props Bag）丟出來：

```javascript
// useToggle.js (純邏輯，0% 畫面)
import { useState } from "react";

export default function useToggle(initialState = false) {
  const [on, setOn] = useState(initialState);
  const toggle = () => setOn((prev) => !prev);

  return {
    on,
    toggle,
    // 預先打包好符合網頁無障礙標準 (A11y) 的屬性
    toggleProps: {
      onClick: toggle,
      "aria-pressed": on, 
      role: "button"
    }
  };
}
```

### 骨肉：自由定義 UI  
現在，不論設計師給什麼瘋狂的規格，你都能用同一個大腦（`useToggle`）並搭配喜歡的 CSS（如 TailwindCSS）來處理：

#### 專案 A：普通的按鈕切換

```javascript
// ButtonToggle.jsx
import useToggle from "./useToggle";

function ButtonToggle() {
  const { on, toggleProps } = useToggle();
  return (
    <button className="my-custom-btn" {...toggleProps}>
      狀態：{on ? "開啟" : "關閉"}
    </button>
  );
}
```

#### 專案 B：滑動動態效果（使用 TailwindCSS）

```javascript
// SliderToggle.jsx
import useToggle from "./useToggle";

function SliderToggle() {
  const { on, toggleProps } = useToggle();
  return (
    <div 
      className={`w-14 h-8 rounded-full cursor-pointer transition-colors ${on ? 'bg-green-500' : 'bg-gray-300'}`} 
      {...toggleProps}
    >
      <div className={`w-6 h-6 bg-white rounded-full m-1 transform duration-200 ${on ? 'translate-x-6' : 'translate-x-0'}`} />
    </div>
  );
}
```

## 3. Headless Component 的優缺點分析

| 優點 (Pros) | 缺點 (Cons) |  
| :--- | :--- |  
| 💅 **100% 樣式自由**：<br>完全不會有「為了覆蓋第三方套件的底色，CSS 寫得痛苦萬分」的問題。 | ⏳ **開發成本較高**：<br>因為它不附帶任何樣式，你必須自己從頭刻出所有的 HTML 結構與外觀。 |  
| ♻️ **極高的邏輯複用性**：<br>同一套複雜邏輯（如：下拉選單的鍵盤上下鍵自由選取）可以完美移植到 Web、甚至 React Native（手機端）。 | 🧠 **初學者門檻高**：<br>需要對 React 的 Props 展開、Custom Hooks 或 Function as a Child 模式有一定理解。 |  
| ♿ **內建完美的無障礙（A11y）**：<br>通常成熟的 Headless 套件都會自動幫你寫好 `aria-*` 屬性，這是一般工程師最容易忽略卻最難寫好的部分。 | |



## 4. 著名的 Headless 生態系  
在現今（2026年）的前端開發中，幾乎很少有人會從頭自己寫複雜元件（如 DatePicker, Combobox）的無頭邏輯，通常會直接引入以下著名的開源無頭套件：

* **Radix UI / Shadcn UI**：目前最頂級的Headless元件庫，Shadcn UI 的底層正是建立於它之上。它提供強大的Headless邏輯與完美的無障礙規格(a11y)，外層樣式完全交給使用者用 Tailwind 處理。
* **Headless UI**：Tailwind CSS 官方團隊親自開發的Headless庫，與 Tailwind 元件生態系完美搭配。
* **TanStack Table (React Table)**：大名鼎鼎的表格處理Headless庫。前端表格的分頁、過濾、排序、列凍結邏輯極度複雜，它把這些大腦功能全寫好，至於表格要怎麼用 `<table>` 刻、長相如何，它完全不管。
* **Downshift**：由 PayPal 團隊維護，專門用來處理極度複雜的「自動完成輸入框（Autocomplete / Combobox）」之Headless庫。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>