---
title: "[React - Hooks] 什麼是 useId？ - 筆記"
pubDatetime: 2026-07-09T10:04:06.068Z
tags: ["React.js","A11y","SSR","Next.js","React18","React Hook"]
description: "Table of contents 核心觀念 useId 是 React 18 推出的一個內建 Hook。它的唯一職責..."
hackmd_id: "ByuDCyp7fg"
---

## Table of contents

## 核心觀念
**`useId`** 是 React 18 推出的一個內建 Hook。它的唯一職責是：**「生成一個在元件生命週期中固定、且在整個 Web 應用程式中絕對唯一的 ID 字串。」**

它輸出的字串格式通常帶有冒號（例如：`:r0:`、`:r1:`），這能有效防止與一般手寫的 CSS 選擇器（如 `#username`）產生命名衝突。



## 1. 為什麼我們需要`useId`？

在過去，若多個相同的 UI 元件同時渲染在畫面上，手寫寫死 ID 會導致嚴重衝突；而使用 `Math.random()` 則會在伺服器端渲染（SSR）時引發前後端不一致的災難。



### 情境一：重複元件導致的 ID 衝突 (破壞A11y)  
若手寫 `<input id="username">`，當這個表單元件在同一頁面被重複呼叫三次，畫面上就會出現三個完全一樣的 `id`。這違反了 HTML 規範，也會導致螢幕閱讀器錯亂、使用者點擊第二個 `label` 卻跳到第一個輸入框。

### 情境二：伺服器端渲染（SSR）的前後端不一致（Hydration Mismatch）  
若使用 `Math.random()` 企圖產生隨機 ID，在 Next.js 等 SSR 框架中：  
1. **伺服器端** 跑了 `Math.random()` 產出 `0.123`，渲染出 `<input id="0.123">` 傳給瀏覽器。  
2. **瀏覽器端** 進行 Hydration時又跑了一次，產出 `0.789`。  
3. **結果**：React 偵測到兩邊 HTML 的 ID 對不起來，直接在終端機噴出錯誤（Hydration Mismatch Warning）。

> **`useId` 的解法**：它不僅能確保同一個頁面中的唯一性，還能**保證在伺服器端與客戶端（瀏覽器）渲染出來的 ID 字串百分之百一模一樣**！


## 2. 實際應用場景

### 場景一：串接 HTML 表單的 `<label>` 與 `<input>`  
最標準的無障礙網頁規格（A11y）實作，確保點擊標籤時能正確聚焦到對應的輸入框：

```javascript
import { useId } from "react";

function UsernameInput() {
  const inputId = useId(); // 自動生成全域唯一 ID，例如 ":r0:"

  return (
    <div className="form-group">
      <label htmlFor={inputId}>使用者名稱</label>
      <input id={inputId} type="text" placeholder="請輸入名稱..." />
    </div>
  );
}
```

### 場景二：複合元件內部關聯（如 Menu, Modal, Accordion）  
利用 `useId` 配合 Context API，可以讓按鈕與下拉選單的 `aria-*` 屬性完美無痛綁定：

```javascript
// Menu.jsx
import MenuButton from "./MenuButton";
import MenuDropdown from "./MenuDropdown";
import { createContext, useState, useId } from "react";

const MenuContext = createContext();
export default function Menu({ children }) {
  const [open, setOpen] = useState(false);
  const menuId = useId(); 

  function toggle() {
    setOpen((prevOpen) => !prevOpen);
  }

  return (
    <MenuContext.Provider value={{ open, toggle, menuId }}>
      <div className="menu" role="menu">{children}</div>
    </MenuContext.Provider>
  );
}

export { MenuContext };


// MenuButton.jsx
const { open, toggle, menuId } = useContext(MenuContext);
return (
  <button onClick={toggle} aria-expanded={open} aria-controls={menuId}>
    展開選單
  </button>
);

// MenuDropdown.jsx
const { open, menuId } = useContext(MenuContext);
return (
  <div id={menuId} aria-hidden={!open}>
    {open && children}
  </div>
);
```

## 3. 史詩級地雷：絕對不要拿 `useId` 當作列表的 `key`！  
在處理陣列遍歷`（.map()）`時，React 規定必須傳入 `key` 屬性。千萬不能因為手邊沒有現成的 ID，貪圖方便調用 `useId`。

這是極度錯誤的危險寫法：

```javascript
// ❌ 錯誤示範！
export default function ItemList({ items }) {
  return (
    <ul>
      {items.map((item) => {
        const uniqueId = useId(); // 警告：會導致嚴重效能與渲染 Bug
        return <li key={uniqueId}>{item.name}</li>;
      })}
    </ul>
  );
}
```

### 為什麼不能這樣用？
* 設計初衷不同：`useId` 的目的是為了**解析元件在「DOM 樹結構中的實體位置標識」，它是固定的。**
* 遺失節點狀態：當陣列順序改變、新增或刪除項目時，React 需要透過 `key` 來追蹤「資料本體」的移動。如果使用 `useId`，當元件重新渲染，React 會誤判或遺失原本 DOM 節點的內部狀態（例如 `input` 裡輸入到一半的文字會莫名消失或錯位）。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡 正確做法：列表的 key 應該永遠來自於資料庫帶來的唯一欄位（如 `item.id`、`item.uuid`），或在資料結構保證絕對不會變動、不會排序的極端情況下，才勉強使用陣列的 index。

</blockquote>

## 4. 常見觀念問答

### Q: `useId` 可以用來當作 CSS 選擇器（querySelector）嗎？

A: 不行。因為 `useId` 產出的字串包含冒號（如 `:r0:`），在原生 CSS 中冒號是偽類（Pseudo-class）的保留字，直接使用 `document.querySelector('#:r0:')` 會噴錯，必須做複雜的反斜線轉義（Escape），因此不建議拿來做 CSS 抓取。

### Q: 可以在一個元件裡呼叫多次 `useId` 嗎？

A: 可以，但通常沒必要。如果一個元件內有五個輸入框，你只需要呼叫一次 `const baseId = useId()`，然後在綁定時手動加上後綴即可（例如 `id={${baseId}-email}、id={${baseId}-password}）`，這樣更省效能。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>