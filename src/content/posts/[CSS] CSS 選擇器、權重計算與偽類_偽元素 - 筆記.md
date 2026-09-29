---
title: "[CSS] CSS 選擇器、權重計算與偽類/偽元素 - 筆記"
pubDatetime: 2026-09-29T10:51:24.510Z
tags: ["Interview Preparation","CSS","Frontend"]
description: "Tags: Interview Preparation CSS Frontend Table of contents..."
---

###### Tags: `Interview Preparation` `CSS` `Frontend`


## Table of contents


## 核心觀念總覽

* **Class vs. ID 的設計意圖 (Design Intent)**：
  * **Class (`.`)**：具備**可重用性（Reusable）**，用於多個元素的通用樣式（如 `.btn`、`.text-green`）。
  * **ID (`#`)**：具備**唯一性（Unique）**，原則上在同一個 HTML 頁面中只能出現一次，且擁有極高的權重。
* **選擇器權重（CSS Specificity）陷阱**：
  * 權重決定了當樣式發生衝突時，瀏覽器優先套用誰。
  * **高權重的選擇器會覆蓋低權重的選擇器**。例如：`#red` (ID) 的權重高於 `h1:hover` (標籤 + 偽類)，因此單純寫 `h1:hover` 無法改變帶有 ID `#red` 的元素顏色。
* **進階延伸觀念（Impress the Interviewer）**：
  * **偽類 (Pseudo-classes，單冒號 `:`)**：根據使用者互動或元素狀態套用樣式（如 `:hover`, `:focus`, `:nth-child()`）。
  * **偽元素 (Pseudo-elements，雙冒號 `::`)**：插入或修飾元素的特定部分（如 `::before`, `::after`, `::placeholder`）。
  * **選擇器鏈結 (Chaining Selectors)**：提高權重以覆蓋樣式（如 `#red:hover`）。



## 1. CSS 權重計算階層與速查 (Specificity Hierarchy)

瀏覽器計算權重時，可簡化為四個數字等級 `(Inline, ID, Class/Pseudo-class, Element/Pseudo-element)`：

| 選擇器類型 | 範例 | 權重分數 (簡化表示) | 說明 |  
| :--- | :--- | :--- | :--- |  
| **行內樣式 (Inline Style)** | `style="color: red"` | **(1, 0, 0, 0)** | 直接寫在 HTML 標籤上，權重極高 |  
| **ID 選擇器** | `#red`, `#header` | **(0, 1, 0, 0)** | 具備高優先權，通常用於特定唯一元件 |  
| **Class / 偽類 / 屬性** | `.green`, `:hover`, `[type="text"]` | **(0, 0, 1, 0)** | 最常用的樣式封裝單位，可重複套用 |  
| **元素標籤 / 偽元素** | `h1`, `div`, `::before` | **(0, 0, 0, 1)** | 最基礎的標籤選擇器 |  
| **通配符 / 繼承** | `*`, `body { color: red }` | **(0, 0, 0, 0)** | 權重最低 |

> ⚠️ **`!important`**：能打破所有權重規則，強制覆蓋樣式。但實務上應**極力避免使用**，否則會破壞 CSS 瀑布流維護性。

> [CSS Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Specificity) (MDN Doc)



## 2. 實務選擇器權重衝突與鏈結範例 (Code Example)

### 情境：當高權重 ID 碰上低權重 偽類 (`:hover`)

```html
<h1 id="red" class="example-text">這是一段文字</h1>
```

```css
/* 1. Class 選擇器：權重 (0,0,1,0) */
.green {
  color: green;
}

/* 2. ID 選擇器：權重 (0,1,0,0) - 文字呈現紅色 */
#red {
  color: red;
}

/* ❌ 失敗的 Hover 效果：權重 (0,0,1,1) [h1 + :hover] */
/* 因為 (0,0,1,1) 小於 ID 的 (0,1,0,0)，滑鼠移上去時顏色不會變成藍色！ */
h1:hover {
  color: blue;
}

/* ✅ 正確的 Hover 效果：選擇器鏈結 (Chaining) 權重 (0,1,1,0) [#red + :hover] */
/* 權重超過原本的 #red，成功改變顏色 */
#red:hover {
  color: blue;
}
```

## 3. 回答策略 (Senior Response Strategy)  
建議採用「基礎回答 ➔ 權重延伸 ➔ 主動延伸進階話題」的三段式答題術：

### 破題說明 Class 與 ID 的基礎差異（What）：  
簡述 Class 用於可重用樣式，ID 用於單一元件，且兩者在 CSS 選擇器語法（`.` vs `#`）的不同。

### 切入權重機制（Specificity Deep-dive）：  
主動舉出權重例子（如 ID (100) > Class (10) > Element (1)）。說明若一個元素帶有 ID，單純寫 `h1:hover` 會因為權重不夠而無法生效，必須寫成 `#red:hover` 進行選擇器鏈結。

### 主動展現廣度（Show Mastery - 主動帶出進階主題）：
- **談偽元素**：提到 `::before` 與 `::after` 可以用來在不增加 DOM 結構的前提下修飾 UI 或動態插入內容（`content: ''`）。
- **談 CSS 預處理器 (Sass/SCSS)**：提到 Sass 的巢狀（Nesting）寫法能簡化選擇器鏈結，但也提醒巢狀過深（超過 3 層）會導致產出的 CSS 權重過高且難以重用，展現思考高度。

## FAQ
### Q1: 什麼是偽類 (Pseudo-class) 與偽元素 (Pseudo-element)？語法上有什麼差別？
#### 偽類 (Pseudo-class，使用單冒號 `:`)：用於定義元素的特殊狀態（State）。

- 範例：`:hover`（懸停）、`:focus`（聚焦）、`:nth-child(2)`（次序）。

#### 偽元素 (Pseudo-element，使用雙冒號 `::`)：用於設定元素的特定部位樣式，或建立虛擬 DOM 節點。
- 範例：`::before`（元素前插入內容）、`::after`（元素後插入內容）、`::placeholder`（輸入框佔位文字）。  
(註：CSS3 規範以 `::` 區分偽元素，但為了相容舊版瀏覽器，寫單冒號 `:before` 瀏覽器通常也能解析。)

### Q2: 為什麼在大型專案中，許多前端團隊（如 BEM 命名法）建議完全不要在 CSS 中寫 ID 選擇器？
- **權重過高（Too High Specificity）**：ID 的權重高達 (0,1,0,0)，未來若想改寫或覆蓋樣式，很容易陷入「必須用更高權重或 !important 互相覆蓋」的惡性循環（Specificity War）。

- **缺乏重用性（Non-reusable）**：HTML 中 ID 必須唯一，導致該 CSS 樣式無法套用在其他元件上，違背了 CSS 模組化的初衷。