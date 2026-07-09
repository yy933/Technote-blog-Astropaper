---
title: "[React.js] React 的 Context API - 跨元件狀態共享的神器 - 筆記"
pubDatetime: 2026-07-09T09:47:20.721Z
tags: ["React.js","Frontend","State Management","Context API","Design Pattern","React Hook","Compound Components"]
description: "Table of contents 核心觀念 在 React 中，資料預設是透過 props 「單向向下傳遞」的。當元..."
hackmd_id: "By4y9y6QMg"
---

## Table of contents

## 核心觀念  
在 React 中，資料預設是透過 props 「單向向下傳遞」的。當元件樹變得很深時，為了把資料傳給底層元件，中間不需要該資料的元件也必須幫忙傳遞，這種情況稱為 **Props Drilling**。

**React Context** 提供了一種新機制：**「它允許元件架設一個廣播電台（Provider），讓樹狀結構中任何深度的子元件都能直接收聽（useContext），藉此繞過中間人、實現跨元件的狀態共享。」**



## 1. 結構圖解

以下以 `Menu` 元件群組為例，Context 在幕後的資料流向長這樣：


* **Menu (Provider 電台)**：負責掌控 `open`（開關狀態）、`toggle`（切換函式）與 `menuId`（無障礙 ID）。
* **MenuButton & MenuDropdown (Consumers 收聽者)**：不論它們中間被隔了多少層 `<div>` 或是其他裝飾性標籤，都能直接拉一條無形的線到 `Menu` 拿資料，中間完全不驚動任何 Props。



## 2. 範例程式碼拆解

以下範例是「複合元件（Compound Components）結合 Context」的實作。拆解三個核心步驟：

### 步驟一：建立電台與廣播（`Menu.jsx`）  
首先，必須使用 `createContext()` 建立一個通訊管道，並用 `.Provider` 將資料廣播出去。

```javascript
// Menu.jsx
import { createContext, useState, useId } from "react";

// 1. 建立一個專屬於 Menu 的通訊管道（Context 物件）
const MenuContext = createContext();

export default function Menu({ children }) {
  const [open, setOpen] = useState(false);
  const menuId = useId(); // 產生唯一的 ID 供無障礙網頁規格 (A11y) 使用

  function toggle() {
    setOpen((prevOpen) => !prevOpen);
  }

  return (
    // 2. 使用 Provider 標籤，並透過 value 屬性把「想要共享的寶箱」丟出去
    <MenuContext.Provider value={{ open, toggle, menuId }}>
      <div className="menu" role="menu">
        {children}
      </div>
    </MenuContext.Provider>
  );
}

// 3. 將電台導出，讓子元件可以引入
export { MenuContext };
```

### 步驟二：子元件直接收聽（`MenuButton.jsx`）  
子元件不需要在參數列等待父元件傳 props 進來，而是主動調用 `useContext` 去對應的電台拿取所需的資源。

```javascript
// MenuButton.jsx
import { useContext } from "react";
import Button from "../Button/Button";
import { MenuContext } from "./Menu"; // 引入電台

export default function MenuButton({ children }) {
  // 3. 用 useContext 默默拿出 Menu 廣播出來的 open, toggle, menuId
  const { open, toggle, menuId } = useContext(MenuContext);
  
  return (
    <Button 
      onClick={toggle} 
      aria-expanded={open} 
      aria-haspopup="true" 
      aria-controls={menuId} // 完美的無障礙網頁串接！
    >
      {children}
    </Button>
  );
}
```

### 步驟三：另一個子元件同步收聽（`MenuDropdown.jsx`）  
同樣的管道，`MenuDropdown` 也可以去拿同一個 `menuId` 與 `open` 狀態，達到高度同步的協同運作。

```javascript
// MenuDropdown.jsx
import { useContext } from "react";
import { MenuContext } from "./Menu";

export default function MenuDropdown({ children }) {
  // 4. 一樣去拿同一個電台的資料
  const { open, menuId } = useContext(MenuContext);
  
  return (
    <div className="menu-dropdown" aria-hidden={!open} id={menuId}>
      {open && children} {/* 當 open 為 true 時，才把下拉的選項內容倒出來 */}
    </div>
  );
}
```

## 3. 帶來的巨大優勢  
透過 Context 改造後，我們在外部（例如 `index.jsx`）使用這個下拉選單元件時，程式碼會乾淨得不可思議：

```javascript
import React from "react";
import ReactDOM from "react-dom/client";
import Menu from "./Menu/index";


function App() {
  const sports = ["Tennis", "Pickleball", "Racquetball", "Squash"];
  return (
    <Menu>
      <Menu.Button>Sports</Menu.Button>
      <Menu.Dropdown>
        {sports.map((sport) => (
          <Menu.Item key={sport}>{sport}</Menu.Item>
        ))}
      </Menu.Dropdown>
    </Menu>
  );
}

ReactDOM.createRoot(document.getElementById("root")).render(<App />);
```

### 核心優點
* 零 Props 痕跡：外層看不到任何 `open={open}` 或 `onClick={toggle}` 的手動傳遞，這被稱為隱式狀態共享（Implicit State）。
* 無痛無障礙串接（A11y）：使用了 `useId` 產出的 `menuId`。藉由 Context 的傳播，`<button>` 的 `aria-controls` 與 `<div>` 的 `id` 拿到了「完全一模一樣的值」，這在不用 Context 的情況下極難優雅地完成。

## Context API 的優缺點與使用時機

| 優點 (Pros) | 缺點 (Cons) / 注意事項 |  
| :--- | :--- |  
| 🚀 **終結 Props Drilling**：<br>直接跨級傳遞，拯救中間充當快遞員的無辜元件。 | 🔄 **效能隱憂（不適合高頻更新）**：<br>當 Context 的 `value` 改變時，所有使用 `useContext` 的子元件**通通會被迫重新渲染 (Re-render)**。 |  
| 📦 **內建免安裝**：<br>React 原生自带，不需要像 Redux 或 Zustand 額外安裝第三方套件。 | 🧩 **元件依賴性提高**：<br>`MenuButton` 如果被單獨拿到 `<Menu>` 外面使用會噴錯，因為它找不到電台（Context 為 `undefined`）。 |

### 什麼時候該用 Context？
* 適合：低頻率更新的全域資料（如：切換深色/淺色主題 Theme、使用者登入語系 Locale、或是像本例中結構固定的 UI 元件組內部狀態）。
* 不適合：高頻率更新、極度複雜的商務數據（如：每秒都在變動的即時股價、複雜的購物車數量與複雜過濾邏輯），這類場景建議使用 Zustand 或 Redux Toolkit。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>