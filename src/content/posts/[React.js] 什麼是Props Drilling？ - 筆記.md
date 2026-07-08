---
title: "[React.js] 什麼是Props Drilling？ - 筆記"
pubDatetime: 2026-07-08T08:33:20.660Z
tags: ["React.js","Frontend","State Management","Context API","architecture"]
description: "Table of contents 核心觀念 Props Drilling 是 React 單向資料流架構下常見的一種..."
hackmd_id: "r16UOtimfl"
---

## Table of contents

## 核心觀念
**Props Drilling** 是 React 單向資料流架構下常見的一種現象。當你需要把最上層元件（Parent）的某個狀態（State）傳遞給底層的某個深層子元件（Deeply Nested Child）時，你**必須將該資料透過 props 一層一層、像接力賽一樣穿過中間所有的元件**，即使中間層元件根本不需要這些資料。


## 1. 結構圖解與運作機制

在複雜的 React 應用中，元件樹可能會有非常多層級。以下為例：

* **App 元件**：擁有全域狀態（例如：`user` 登入資訊）。
* **Header 與 Navigation 元件**：自身完全不使用 `user` 資料，但因為子元件需要，它們被迫成為「中介快遞員」接收並轉傳 props。
* **UserProfile 元件**：位於元件樹深處，才是真正需要消費（Consume）`user` 資料的終點。

這種讓資料像鑽孔機一樣橫跨多個完全無關元件的過程，不僅污染了中間元件的 props 介面，也增加了程式碼的耦合度。



## 2. 範例說明

以一個常見的導覽列使用者資訊顯示為例：

```javascript
// 1. 最上層元件：App.jsx (資料源頭)
function App() {
    const [user, setUser] = useState({ name: "Alex", role: "Admin" });
    return <Header user={user} />; 
}

// 2. 中間層元件：Header.jsx (純轉手)
function Header({ user }) {
    return (
        <header>
            <h1>My Dashboard</h1>
            <Navigation user={user} /> {/* 幫忙轉交 */}
        </header>
    );
}

// 3. 次中間層元件：Navigation.jsx (純轉手)
function Navigation({ user }) {
    return (
        <nav>
            <ul><li>首頁</li></ul>
            <UserProfile user={user} /> {/* 再次幫忙轉交 */}
        </nav>
    );
}

// 4. 最底層元件：UserProfile.jsx (實際使用者)
function UserProfile({ user }) {
    return <span className="profile">歡迎回來，{user.name}！</span>;
}
```

## 這會造成什麼問題？
### 情況一： 欄位名稱變更  
如果今天後端調整 API 規格，要把 `user.name` 改成 `user.displayName`。你不能只改 `UserProfile.jsx`，必須沿著鑽孔的路徑，逐一檢查並修改 `App`、`Header`、`Navigation` 當中所有的 prop 串接名稱。

### 情況二： 元件重構與複用  
如果想把 `Header` 元件搬移到另一個不需要顯示使用者資訊的獨立頁面，你會發現它因為強制綁定了 `user prop`，導致抽離時會噴錯，大幅降低了 UI 元件的可重用性 (Reusability)。

## 常見迷思
### 迷思一：Props Drilling 是一種 Bug 或錯誤語法？  
並非如此。**Props Drilling 是 React 刻意設計的「單向資料流（Unidirectional Data Flow）」之必然產物。**  
顯式的 props 傳遞能讓資料流向非常透明、好追蹤。**只有當元件層級過深（通常大於 3-4 層以上），或者中間層元件多到讓維護成本爆炸時，它才會從「顯式的好處」變成「維護的壞處」。**

### 迷思二：只要有 Props Drilling 就要立刻引入 Redux / Zustand 全域狀態管理？  
盲目引入複雜工具會殺雞用牛刀。**在架構優化上，通常會優先考慮「元件組合（Component Composition）」，再來才是內建的「Context API」，最後才是「全域狀態管理套件」**。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡 不要過早最佳化（Premature Optimization）。**若只是 2 到 3 層的簡單傳遞，直接使用 Props 依然是效能最好、最直覺的作法。**

</blockquote>

## 最佳實踐：如何解決 Props Drilling？  
為了解救充當快遞員的中間層元件，React 生態系提供了以下三種主流解決方案：

### 方案一：元件組合 (Component Composition)  
不需要任何新套件，單純改變元件的架構。**直接在最上層將底層元件包好，並透過 children 傳下去**：

```javascript
// App.jsx 直接把 UserProfile 塞給 Header
function App() {
    const [user, setUser] = useState({ name: "Alex" });
    return (
        <Header>
            <Navigation>
                <UserProfile user={user} />
            </Navigation>
        </Header>
    );
}
```

* 效果：`Header` 和 `Navigation` 只需要渲染 `{children}`，完全不需要知道 `user` 的存在，完美解耦！

### 方案二：React Context API (內建輕量方案)  
React 內建的 Context 就像架設一個「全域廣播電台」。上層負責 Provider 廣播，底層用 `useContext` 直接收聽：

```javascript
// 1. 建立廣播電台
const UserContext = React.createContext();

// 2. 頂層發射訊號
function App() {
    const [user] = useState({ name: "Alex" });
    return (
        <UserContext.Provider value={user}>
            <Header /> {/* 中間元件完全不用傳 props */}
        </UserContext.Provider>
    );
}

// 3. 底層直接收聽
function UserProfile() {
    const user = useContext(UserContext); // 繞過中間人直接取得
    return <span>{user.name}</span>;
}
```

### 方案三：引進外部全域狀態管理 (如 Zustand)  
當專案規模龐大、商業邏輯複雜時，將狀態移出元件樹，存放在獨立的 `Store` 中：

```javascript
import { create } from 'zustand'

// 1. 建立獨立狀態倉庫
const useUserStore = create((set) => ({
  user: { name: "Alex" },
}))

// 2. 任何深度的元件直接去倉庫拿資料
function UserProfile() {
  const user = useUserStore((state) => state.user);
  return <span>{user.name}</span>;
}
```

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>