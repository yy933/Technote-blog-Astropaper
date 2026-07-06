---
title: "[React - Testing] 撰寫 React Testing Library 測試的關鍵好習慣與進階心法 - 筆記"
pubDatetime: 2026-07-06T09:50:11.964Z
tags: ["React.js","Vitest","cheatsheet","Unit Test","testing","Vite"]
description: "Table of contents 1. 減少重複程式碼：善用 beforeEach 💡 觀念說明 當多個測試案例（..."
hackmd_id: "BybfvgFmzl"
---

## Table of contents


 

## 1. 減少重複程式碼：善用 `beforeEach`

### 💡 觀念說明  
當多個測試案例（`test`）都需要先渲染同一個元件時，重複寫 `render(<Component />)` 會讓程式碼顯得冗長。可以使用 Vitest 的生命週期鉤子 `beforeEach`，在每個測試執行前自動處理渲染。

### 🛠 範例對照
* **優化前（重複渲染）：**

```javascript
  test("測試 A", () => {
    render(<Main />);
    // 斷言...
  });
  test("測試 B", () => {
    render(<Main />);
    // 斷言...
  });
```

* **優化後（使用 `beforeEach`）：**

```javascript
import { test, expect, describe, beforeEach } from "vitest";
import { render, screen } from "@testing-library/react";
import Main from "./Main";

describe("Main component", () => {
  // 在底下的每個 test 執行前，都會先跑這個區塊
  beforeEach(() => {
    render(<Main />);
  });

  test("display the text", () => {
    expect(screen.getByText("One does not simply")).toBeInTheDocument();
  });

  test("display the image", () => {
    expect(screen.getByRole("img")).toBeInTheDocument();
  });
});
```

## 2. 提升 DOM 斷言的語意化：使用 toHaveAttribute
### 💡 觀念說明  
測試圖片 `src` 或超連結 `href` 時，雖然直接讀取 DOM 節點屬性（如 `.src`）可以通，但使用 `@testing-library/jest-dom` 提供的專用匹配器 `toHaveAttribute`，能讓測試失敗時的錯誤報告（Error Log）更精準易讀。

### 🛠 範例對照

```javascript
// ❌ 較不推薦：語意較像純 JS 測試
expect(screen.getByRole("img").src).toBe("[https://i.imgflip.com/1bij.jpg](https://i.imgflip.com/1bij.jpg)");

//  推薦：具備 DOM 測試語意化
expect(screen.getByRole("img")).toHaveAttribute("src", "[https://i.imgflip.com/1bij.jpg](https://i.imgflip.com/1bij.jpg)");
```

## 3. 防範未然：避免 getByRole 的多元素衝突
### 💡 觀念說明
**`getByRole("img")` 或 `getByRole("button")` 的背後邏輯是：在畫面上尋找「唯一一個」該角色的元素。** 如果未來網頁升級，新增了 Logo 圖片或第二個按鈕，測試就會直接拋出 `Found multiple elements with the role...` 的錯誤。

### 🛠 推薦解法（利用 `name` 屬性精準定位）

```javascript
// 透過指定 name (通常是圖片的 alt 或是按鈕上的文字)，精準鎖定特定元素
expect(screen.getByRole("img", { name: /meme/i })).toBeInTheDocument();

// 測試按鈕時，也比比對 .textContent 更符合使用者視角
expect(screen.getByRole("button", { name: /Get a new meme image/i })).toBeInTheDocument();
```

## 4. 增強測試彈性：善用「正規表達式 (Regex)」
### 💡 觀念說明  
使用精確字串比對（如 `"Top Text"`），只要畫面文字多了一個空格、大小寫改變，測試就會壞掉。**使用正規表達式並加上 `i` 旗標（忽略大小寫），可以讓測試更有彈性，專注在「文字是否有正確呈現」的核心上。**

### 🛠 範例對照

```javascript
// ❌ 嚴格字串比對：大小寫、空格必須完全一致
screen.getByText("Top Text")

//  推薦正規表達式：包含 "top text" 即可，且忽略大小寫
screen.getByText(/top text/i)
```

## 總結

| 撰寫邏輯 | 思考出發點 | 優點 |  
| :--- | :--- | :--- |  
| **代碼視角** (較不推薦) | 檢查 DOM 節點的 `textContent`、`.src` 屬性。 | 直覺，但測試與實作細節綁得太死。 |  
| **使用者視角** (強烈推薦) | 模擬真人操作：找畫面上的圖案（看 alt）、找按鈕（看文字）。 | 程式碼重構時測試不易壞，具備高信心水準。 |

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>