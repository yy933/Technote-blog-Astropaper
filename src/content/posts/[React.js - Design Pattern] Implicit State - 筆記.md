---
title: "[React.js - Design Pattern] Implicit State - 筆記"
pubDatetime: 2026-07-08T11:08:01.219Z
tags: ["React.js","Design Pattern","Compound Components","Context API","Frontend","React Hook"]
description: "Table of contents 核心觀念 Implicit State 是 React 複合元件（Compound..."
hackmd_id: "SyX4jjsXGx"
---

## Table of contents

## 核心觀念
**Implicit State** 是 React 複合元件（Compound Components）設計模式中的靈魂。

它是指：**「存在於父元件內部，但子元件不需要被手動傳入 props，就能默默共享並使用的狀態。」**

在標準的 React 寫法中，狀態傳遞是「顯式（Explicit）」的（例如：`<MenuButton onClick={toggle} />`）。而Implicit State的目的是**將狀態封裝在元件群組內部**，讓外部使用者的代碼保持極度乾淨、直覺，同時避免 Props Drilling。



## 1. 情境：被困在父層的狀態

假設我們要做出一個可以自由開關的下拉選單（Menu），我們在 `index.jsx` 期望的使用方式如下：

```javascript
// index.jsx
function App() {
  return (
    <Menu>
      <MenuButton>Sports</MenuButton>
      <MenuDropdown>
        <MenuItem>Tennis</MenuItem>
        <MenuItem>Pickleball</MenuItem>
      </MenuDropdown>
    </Menu>
  );
}
```

### 原始元件的問題（狀態無法直達）  
如果在最外層的 `Menu.jsx`宣告了 `open` 狀態與 `toggle` 函式，普通的 `{children}` 寫法會讓狀態卡在父元件內部，底下的 `MenuButton`（需要 `toggle`）與 `MenuDropdown`（需要 `open`）根本拿不到任何資料：

```javascript
// Menu.jsx (尚未調整)
export default function Menu({ children }) {
  const [open, setOpen] = React.useState(true);
  const toggle = () => setOpen(prev => !prev);

  return <div className="menu">{children}</div>; // ❌ children 沒拿到任何狀態
}
```

為了打破這個僵局，React 社群演化出了兩種主流的隱式狀態共享寫法。

## 2. 解法
### 解法一：Context API （全域廣播）  
利用 React Context 在父元件架設電台，讓子元件無視嵌套深度、直接收聽。

#### 程式碼實作
* 父元件：`Menu.jsx`

```javascript
import React from "react";

// 1. 建立廣播電台
const MenuContext = React.createContext();

export default function Menu({ children }) {
  const [open, setOpen] = React.useState(true);
  const toggle = () => setOpen((prev) => !prev);

  return (
    // 2. 透過 Provider 將狀態隱式廣播出去
    <MenuContext.Provider value={{ open, toggle }}>
      <div className="menu">{children}</div>
    </MenuContext.Provider>
  );
}

export { MenuContext };
```

* 子元件：`MenuButton.jsx` & `MenuDropdown.jsx`

```javascript
// MenuButton.jsx
import { MenuContext } from "./Menu";

export default function MenuButton({ children }) {
  const { toggle } = React.useContext(MenuContext); // 🟢 默默收聽 toggle
  return <button onClick={toggle}>{children}</button>;
}

// MenuDropdown.jsx
import { MenuContext } from "./Menu";

export default function MenuDropdown({ children }) {
  const { open } = React.useContext(MenuContext); // 🟢 默默收聽 open
  return open ? <div className="menu-dropdown">{children}</div> : null;
}
```

### 解法二：Clone Element   
不使用 Context，而是利用 React 的 `React.Children` API中的方法 `React.Children.map` 遍歷子項目，並用 `React.cloneElement` 在渲染時，**強制把狀態當作 `props` 複製一份塞給第一層子元件**。

#### 程式碼實作
* 父元件：`Menu.jsx`

```javascript
import React from "react";

export default function Menu({ children }) {
  const [open, setOpen] = React.useState(true);
  const toggle = () => setOpen(prevOpen => !prevOpen);

  return (
    <div className="menu">
      {/* 安全地遍歷每一項第一層 child，並複製它們、強行加入屬性 */}
      {React.Children.map(children, (child) => {
        return React.cloneElement(child, {
          open,
          toggle
        });
      })}
    </div>
  );
}
```

* 子元件：`MenuButton.jsx` & `MenuDropdown.jsx`  
因為狀態是被當作 `props` 硬塞進來的，子元件不需要調用 Context，直接從 `props` 接收即可：

```javascript
// MenuButton.jsx
export default function MenuButton({ children, toggle }) { // 🟢 直接從 props 拿
  return <button onClick={toggle}>{children}</button>;
}

// MenuDropdown.jsx
export default function MenuDropdown({ children, open }) { // 🟢 直接從 props 拿
  return open ? <div className="menu-dropdown">{children}</div> : null;
}
```

## 兩種實作模式比較  
這兩種方法都能完美呈現 `index.jsx` 的高可讀性，但技術本質上有顯著的差異：

| 比較項目 |  Context API  |  Clone Element  |  
| :--- | :--- | :--- |  
| **穿透深度** | **無限深度**。<br>子元件不管藏在多深的 `<div>` 裡都能用 `useContext` 拿到。 | ❌ **僅限第一層（Direct Children）**。<br>如果外部在 `MenuButton` 外層包了別的標籤，狀態就會斷掉。 |  
| **程式碼複雜度**| **稍高**。<br>需要額外宣告 Context、Provider。 | **極低**。<br>不需要額外宣告任何變數，一個 `map` 搞定。 |  
| **擴充與彈性** | **極高**。<br>外部開發者可以隨意穿插自訂的 UI 結構，完全不影響元件運作。 | **低**。<br>嚴格限制了 HTML 的結構順序，容易因為外部包裝而壞掉。 |  
| **適用場景** | 適合大型、結構複雜、子元件可能被深層嵌套的架構（如：Nav、Modal、Form）。 | 適合結構極度簡單、固定只有一層子元件的場合（如：Tabs、ButtonGroup）。 |


## 常見迷思
* **迷思：既然 Context API 比較強，是不是就該廢棄 Clone Element 寫法？**  
並非如此。`React.cloneElement` 雖然有層級限制，但它具有 「顯式 props 覆蓋」 的特性。如果父元件想要動態根據子元件的 props 去修改子元件（例如：自動為沒有加上 `id` 的子元件補上索引 index），`cloneElement` 是非常強大且直覺的武器，這點是 Context 較難直接做到的。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>