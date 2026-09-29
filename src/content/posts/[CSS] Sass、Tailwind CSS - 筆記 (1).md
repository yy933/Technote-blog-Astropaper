---
title: "[CSS] Sass、Tailwind CSS - 筆記"
pubDatetime: 2026-09-29T15:58:21.232Z
tags: ["Interview Preparation","CSS","Frontend","TailwindCSS","Sass"]
description: "Tags: Interview Preparation CSS Frontend Sass TailwindCSS T..."
---

###### Tags: `Interview Preparation` `CSS` `Frontend` `Sass` `TailwindCSS`


## Table of contents

## 前言  
現代前端開發幾乎鮮少直接撰寫純原生 CSS（Vanilla CSS），原因在於原生 CSS 在大型專案中容易面臨命名衝突、程式碼冗長、維護困難與權重打架（Specificity War）等問題。Sass（預處理器）與 Tailwind CSS（Utility-First CSS 框架）代表了兩種不同的解決思維：

- Sass/SCSS：強化傳統 CSS 的撰寫能力，透過程式化結構（變數、巢狀、Mixin）提升可重用性。

- Tailwind CSS：打破傳統 CSS 檔寫法，**透過原子化（Utility-First）Class 直接在 HTML/JSX 中快速建構 UI**，徹底解決命名與檔案切換問題。

## 為什麼選擇 Sass / SCSS？
### 解決原生 CSS 的痛點：
- Lack of Abstraction（缺乏抽象能力）：原生 CSS 過去缺乏變數、邏輯控制與函式（雖然現在有 CSS Variables，但功能仍不如 Sass 豐富）。
- 選擇器冗長（Verbose Selectors）：無法階層化，必須重複寫祖先選擇器。

### Sass 的核心優勢：
- 變數與主題管理（Variables）：統一管理專案的主色、字體、間距（$primary-color: #007bff;）。
- 巢狀結構（Nesting）：貼近 HTML 結構，提高可讀性（注意：建議不要超過 3 層，以免產出權重過高的 CSS）。
- 程式化邏輯（Mixins & Functions）：可封裝常用的樣式組合（如 RWD Breakpoint 判斷、Flexbox 置中樣式），直接 @include 呼叫。
- 模組化拆分（@use / @import）：將 CSS 切分為 _button.scss、_header.scss，方便大型專案維護。

```sass
// SCSS 範例：利用 Mixin 與 Nesting 封裝 RWD 與樣式
@mixin mobile {
  @media (max-width: 768px) { @content; }
}

.card {
  background: $bg-color;
  &__title { // BEM 命名搭配 Sass & 運算子
    font-size: 1.5rem;
    @include mobile { font-size: 1.2rem; }
  }
}
```

## 為什麼選擇 Tailwind CSS？
### 解決傳統 CSS（包含 Sass）的痛點：
- 命名疲勞（Naming Fatigue）：每天花大量時間想 `.card-wrapper-inner-title` 這種 Class 名稱。
- Context Switching（頻繁切換上下文）：在 HTML/JSX 與 SCSS 兩個檔案之間來回跳轉，影響開發流暢度。
- Dead CSS & 檔案膨脹：專案開發久了，沒人敢刪舊的 CSS，導致打包後的 CSS 檔案無限變大。

### Tailwind CSS 的核心優勢：
- 極速開發（High Development Speed）：直接在 HTML 上寫 `flex items-center justify-between p-4 bg-white shadow-md`，實現「所見即所得」。
- 打包體積極小（JIT Engine & Purging）：只會打包專案中實際用到的 Class，最終產出的 CSS 檔案通常只有幾十 KB，且不會隨專案規模成長而無限膨脹。
- 解決權重與命名衝突：完全不需要想 Class 名稱，也不會發生全域樣式覆蓋的問題。
- 一致性的設計系統（Design System Consistency）：Tailwind 預設限制了 `spacing`、`color`、`font-size` 的數值選單（如 `p-2`, `p-4`, `p-6`），強制團隊遵循統一的設計規範，防止亂寫 `px`。

```javascript
// React + Tailwind 範例：直接在 Component 內寫原子化樣式
function UserCard() {
  return (
    <div className="p-6 max-w-sm mx-auto bg-white rounded-xl shadow-lg flex items-center space-x-4">
      <div className="text-xl font-medium text-black">Dylan Israel</div>
    </div>
  );
}
```

## 回答範本 (Interview Answer Template)
### 「你在專案中會如何選擇 Sass 或 Tailwind CSS？」

「這兩種工具解決問題的角度不太一樣：

- Sass 是對原生 CSS 的增強，**主要解決程式碼重用、變數抽象與架構模組化的問題。** 如果專案是採用傳統的 HTML/Template 渲染、或是需要極度高度客製化且沒有統一 Design System 的舊專案轉型，Sass 的 BEM 架構與 Mixin 能提供很好的維護性。

- Tailwind CSS 則是**當前 Component-Based（如 React/Vue）時代的利器**。它**解決了命名障礙、檔案切換與無用 CSS 殘留的問題**。透過 Utility-First，我們能把 UI 的『結構』與『樣式』同置（Co-location）在元件內部。**搭配它的 JIT 編譯，還能拿到極小體積的 CSS 打包檔。**

### 技術選擇建議
- 新專案 + React/Vue ➔ 優先選擇 Tailwind CSS，能帶來極高的開發效率與一致的設計規範。
- 建立組件庫 (Design System / Component Library) 或傳統大型 CMS 專案 ➔ 選擇 Sass/SCSS，能更精準掌控底層 CSS 選擇器與權重輸出。」

## FAQ
### Q1: Tailwind CSS 把一長串 Class 寫在 HTML 裡面，難道不會讓程式碼變得很髒、難以維護嗎？  
這是一般人對 Tailwind 最常見的誤解。**解答關鍵在於現代前端是「元件化（Component-Based）」開發：**

- 傳統開發中，我們試圖用 CSS Class 來做「抽象化（Abstraction）」；
- 現代 React/Vue 開發中，我們是用 Component（如 `<Button/>`, `<Card/>`） 來做抽象化。  
因為樣式已經被封裝在 Component 內部，其他地方只需要呼叫 `<Button/>`，因此 HTML 並不會重複出現那一長串 Class，反而降低了維護成本。

### Q2: Tailwind CSS 可以跟 Sass 一起搭配使用嗎？  
可以，但實務上不太需要也不推薦。Tailwind 本身就支援 `@apply` 指令（可以在 CSS 中引用 Utility Class），且透過 PostCSS 就能處理變數與前綴問題。若強行引入 Sass，只會增加 Build 工具的編譯負擔與系統複雜度。