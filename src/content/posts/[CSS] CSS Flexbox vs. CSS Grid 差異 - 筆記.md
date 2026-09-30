---
title: "[CSS] CSS Flexbox vs. CSS Grid 差異 - 筆記"
pubDatetime: 2026-09-30T07:56:06.544Z
tags: ["Interview Preparation","CSS","Flexbox","Grid","Frontend"]
description: "Tags: Interview Preparation CSS Flexbox Grid Frontend Table..."
hackmd_id: "HyAKqN5qGx"
---

###### Tags: `Interview Preparation` `CSS` `Flexbox` `Grid` `Frontend`


## Table of contents

## 前言  
傳統 CSS 依賴 `float` 與 `inline-block` 進行排版，容易引發高度塌陷與計算複雜等痛點。現代前端開發中，**Flexbox 與 CSS Grid 並非互斥的替代品，而是相輔相成的佈局工具**。理解兩者的維度差異與運作思維，是掌握現代響應式網頁設計（RWD）的核心關鍵。



## 核心特性與觀念總覽

* **一維 (1D) vs. 二維 (2D)**：
  * **Flexbox**：**一維佈局（1D Layout）**。一次只能處理一個方向（欄 Column 或 列 Row）。適合用於元件內部的元件排列（如導覽列、按鈕群組）。
  * **CSS Grid**：**二維佈局（2D Layout）**。能夠同時控制欄（Columns）與列（Rows）。適合用於整體網頁架構（如 Dashboard、相簿網格、文章版面）。
* **內容導向 (Content-First) vs. 佈局導向 (Layout-First)**：
  * **Flexbox（內容導向）**：由內容大小決定佈局。內容增加時，元件會自動收縮或伸展。
  * **CSS Grid（佈局導向）**：由預先定義好的格子網格（Grid Tracks）決定內容的位置。先劃分網格，內容再放入對應的格子中。



## 1. Flexbox vs. Grid 核心比較與速查

| 比較維度 | Flexbox (Flexible Box) | CSS Grid |  
| :--- | :--- | :--- |  
| **佈局維度** | **一維 (1D)**：Row *或* Column | **二維 (2D)**：Row *與* Column |  
| **設計哲學** | **內容導向 (Content-driven)** | **網格/版型導向 (Layout-driven)** |  
| **重疊元素能力** | 較難（需依賴 `position: absolute`） | **極佳**（可利用 `grid-column` / `grid-row` 重疊） |  
| **軸向控制** | 主軸 (Main Axis) 與 交叉軸 (Cross Axis) | 欄軸 (Column Axis) 與 列軸 (Row Axis) |  
| **單位支援** | `flex: 1` (`flex-grow`, `flex-shrink`, `flex-basis`) | `fr` (Fractional Unit), `minmax()`, `repeat()` |  
| **典型應用場景** | Navbar、卡片內部置中、按鈕組、Tag 標籤 | 網頁整體 Layout (Header/Sidebar/Main)、相本牆 |



## 2. 佈局思維與範例程式碼對比

### 情境 A：Flexbox 一維切版（以導覽列 Navbar 為例）

Flexbox 擅長處理「彈性伸展」與「內容排成一列/一欄」的情境：

```css
.navbar {
  display: flex;
  justify-content: space-between; /* 主軸對齊：兩端對齊 */
  align-items: center;            /* 交叉軸對齊：垂直置中 */
}

.nav-item {
  flex: 1; /* 自動分配剩餘空間 */
}
```

### 情境 B：CSS Grid 二維切版（以 Dashboard 網頁大架構為例）  
Grid 擅長直接劃分區域（Areas），並將 Component 精準放入格子中：

```css
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr;              /* 左側 Sidebar 250px，右側切分剩餘空間 */
  grid-template-rows: 60px 1fr 40px;             /* Header, Main, Footer 高度 */
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
  gap: 16px;                                     /* 網格間距 */
  min-height: 100vh;
}

/* 將子元素放入指定區域 */
.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main    { grid-area: main; }
.footer  { grid-area: footer; }
```

## FAQ
### Q1: 請寫出最經典的「垂直水平置中 (Center a Div)」寫法，Flexbox 與 Grid 各要怎麼寫？

```css
/* ✅ 方式 1：Flexbox 寫法 */
.parent-flex {
  display: flex;
  justify-content: center; /* 水平置中 */
  align-items: center;     /* 垂直置中 */
}

/* ✅ 方式 2：CSS Grid 簡短寫法 (最極簡 2 行) */
.parent-grid {
  display: grid;
  place-items: center;     /* 等同於 align-items + justify-items */
}
```

### Q2: 如何不使用任何 @media 媒體查詢 (Media Queries)，用 CSS Grid 寫出自動適應螢幕寬度的卡片網格 (RWD Auto-fit Grid)？

```css
.card-container {
  display: grid;
  /* 當卡片最小不低於 250px 時自動填滿欄位，超出時自動折行 */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}
```

- `minmax(250px, 1fr)`：卡片寬度最小 250px，最大佔據 1 個剩餘空間單位（1fr）。
- `auto-fit`：當螢幕變寬時，自動擴展現有卡片以佔滿整個容器；若為 `auto-fill` 則會保留空白格子。

### Q3: Flexbox 中 flex: 1 究竟是哪三個屬性的簡寫 (Shorthand)？預設值分別是什麼？  
`flex: 1` 是 `flex-grow` / `flex-shrink` / `flex-basis` 的簡寫。

```css
/* flex: 1 的完整展開寫法： */
flex-grow: 1;   /* 當有剩餘空間時，放大比例為 1 */
flex-shrink: 1; /* 當空間不足時，縮小比例為 1 */
flex-basis: 0%; /* 計算剩餘空間時的基準大小 (設定為 0% 代表空間完全重新分配) */

/* 預設值 (Default) 為： */
flex: 0 1 auto; /* flex-grow: 0, flex-shrink: 1, flex-basis: auto */
```

## 回答策略
### 「在實務專案中，如何決定要用 Flexbox 還是 CSS Grid？」

推薦答題結構：

「這兩個工具在現代前端開發中並非『二選一』，而是 **互相搭配（Complementary）** 的關係。

- 核心原則：看佈局的『維度』

  - 如果是一維佈局（單一欄或單列），例如 Navbar、Tab 標籤頁、按鈕群組，或是卡片內部的標題與圖示對齊，優先採用 Flexbox。
  - 如果是二維佈局（同時需要控制欄與列），例如網頁總體架構（Header/Sidebar/Main）、Dashboard 數據圖表區塊，或是圖片牆（Gallery），採用 CSS Grid。

- 實務架構模式（Grid + Flexbox 組合拳）：  
在實際專案中，**通常會用 CSS Grid 來建構外層巨觀的二維框架（Macro Layout），然後在內層的各個元件（如 `.card` 內部）使用 Flexbox 來做微觀的一維元素對齊（Micro Layout）**。這樣能發揮兩者最大的優勢，讓 CSS 結構既清晰又易於維護。」