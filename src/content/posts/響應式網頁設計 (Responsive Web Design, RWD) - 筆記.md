---
title: "響應式網頁設計 (Responsive Web Design, RWD) - 筆記"
pubDatetime: 2026-09-30T09:56:24.499Z
tags: ["Interview Preparation","CSS","RWD","Responsive Design","Media Queries","Frontend"]
description: "Tags: Interview Preparation CSS RWD Responsive Design Media..."
hackmd_id: "S1ClfB5cGg"
---

###### Tags: `Interview Preparation` `CSS` `RWD` `Responsive Design` `Media Queries` `Frontend`


## Table of contents


## 核心觀念總覽

* **RWD 的核心定義**：確保網頁應用程式在各種不同螢幕尺寸（Desktop, Tablet, Mobile，甚至是穿戴裝置如手錶）下，都能**維持完整的視覺美觀與功能可用性（Functional & Visually Correct）**。
* **技術演進與現代佈局工具**：
  * **過去**：依賴繁瑣的 `width` 計算、`float` 與 `clear float`（主要為了相容舊版 Internet Explorer）。
  * **現在**：全面採用原生 CSS **Flexbox** 與 **CSS Grid**，極大地簡化了版型對齊與折行的動態調整。
* **RWD 的三大技術支柱**：
  1. **媒體查詢（Media Queries）**：監聽螢幕寬度（Viewport Width），切換不同斷點（Breakpoints）的樣式。
  2. **流體佈局與相對單位（Fluid Layout & Relative Units）**：字體與間距使用 `rem`、`em`，區塊與圖片使用 `%` 或 `vw/vh`，取代死板的固定像素（`px`）。
  3. **彈性網格與媒體（Flexible Grids & Media）**：結合 Flexbox/Grid 與 `max-width: 100%`，防止圖片或元件破版溢出。



## 1. RWD 核心技術與相對單位速查

| 技術 / 單位 | 運作原理與機制 | 實務應用情境 |  
| :--- | :--- | :--- |  
| **Media Queries** | `@media (min-width: 768px) { ... }` | 根據螢幕寬度觸發對應斷點，切換佈局結構 |  
| **`rem` 單位** | 相對根元素（`<html>`）的 `font-size`（預設 16px） | 專案字體大小、Margin/Padding 彈性縮放 |  
| **`em` 單位** | 相對父級元素（Parent Element）的 `font-size` | 需要隨父容器尺寸連動的元件（如按鈕內距） |  
| **百分比 `%`** | 相對父級容器的寬高 | 流體容器（Fluid Containers）、欄位寬度分配 |  
| **Flexbox / Grid** | 彈性一維與二維網格佈局 | 自動折行（`flex-wrap`）、動態網格（`auto-fit`） |



## 2. 實務 Mobile-First（行動優先）範例程式碼

在現代 RWD 開發中，推薦採取 **Mobile-First（行動優先）** 策略：先撰寫行動端的基礎樣式，再透過 `min-width` 媒體查詢逐漸疊加桌面端的樣式，這樣能產出更乾淨、效能更高的 CSS。

```html
<!-- HTML Viewport Meta Tag（RWD 必備基礎設定） -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

```css
/* --------------------------------------------------
   1. 行動端基礎樣式 (Mobile Default) - 無需任何 @media
   -------------------------------------------------- */
.container {
  width: 100%;
  padding: 1rem;            /* 使用 rem 保持彈性 */
  display: flex;
  flex-direction: column;   /* 行動端卡片垂直單欄排列 */
}

.responsive-img {
  max-width: 100%;          /* 確保圖片不超出父容器 */
  height: auto;
}

/* --------------------------------------------------
   2. 平板端斷點 (Tablet Breakpoint: >= 768px)
   -------------------------------------------------- */
@media (min-width: 768px) {
  .container {
    flex-direction: row;     /* 平板以上轉為水平多欄排列 */
    justify-content: space-between;
  }
}

/* --------------------------------------------------
   3. 桌面端斷點 (Desktop Breakpoint: >= 1024px)
   -------------------------------------------------- */
@media (min-width: 1024px) {
  .container {
    max-width: 1200px;       /* 限制桌面端最大寬度 */
    margin: 0 auto;          /* 水平居中 */
  }
}
```

## 回答策略 (Senior Response Strategy)
### 「請談談你對 Responsive Web Design (RWD) 的理解。」

推薦 90 秒黃金答題架構：

- 核心定義（What）：  
RWD 的核心目標是讓 Web App 在 Desktop、Tablet 到 Mobile 等不同裝置下，都能保持完美的功能性與視覺體驗。

- 技術演進與工具選擇（How）：  
過去我們可能需要依賴繁瑣的 `float` 或精確計算 `px` 寬度，但現代前端我們全面轉向以 Flexbox 與 CSS Grid 為核心的流體佈局，搭配 `max-width: 100%` 處理媒體資源。

- 相對單位與 Mobile-First 思維（Best Practices）：  
在單位選擇上，會避免硬編碼（Hardcode）固定 `px`，而是選用 `rem` 與 ``%`` 來實現排版與字體的縮放。實務上偏好採用 Mobile-First（漸進增強） 的策略，用 `min-width` 媒體查詢進行堆疊，這不僅讓 CSS 結構更簡潔，也更符合現代極致追求行動端效能的趨勢。

## FAQ
### Q1: 為什麼在 RWD 中，推薦使用 rem 而不是 px 來設定字體與邊距？
- `px` (Absolute Unit)：死板的絕對單位。如果視障或高齡使用者修改了瀏覽器的預設字體大小（例如調大成 24px），設定死 `px` 的網頁字體將無法隨之放大幅度，破壞無障礙性（Accessibility / A11y）。
- `rem` (Relative Unit)：相對於根元素（`<html>`）字體大小的相對單位（預設 1rem = 16px）。當使用者調整瀏覽器預設字體時，使用 `rem` 的網頁能自動成比例縮放，具備極佳的彈性與無障礙支援。

### Q2: 什麼是 Desktop-First 與 Mobile-First？兩者有何差異？
- Desktop-First（桌面優先 / 降級思維）：先寫桌面端複雜樣式，再用 `@media (max-width: ...)` 逐步遮蔽或縮小元件。容易導致行動端下載無用的 CSS 代碼。
- Mobile-First（行動優先 / 漸進增強）：先寫行動端最精簡的單欄樣式，再用 `@media (min-width: ...)` 隨著螢幕變大逐步添加欄位與裝飾。這是現代前端開發的首選實踐。