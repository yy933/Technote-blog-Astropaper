---
title: "[React - Testing] 自動化測試的3A 模式：Arrange-Act-Assert - 筆記"
pubDatetime: 2026-07-06T10:10:14.230Z
tags: ["React.js","Vitest","cheatsheet","Unit Test","testing","Vite"]
description: "Table of contents 前言 3A 模式是撰寫自動化測試中最經典、最被廣泛推薦的結構框架。不論測試邏輯多複..."
hackmd_id: "BkvRjltQGe"
---

## Table of contents


 

## 前言
**3A 模式**是撰寫自動化測試中最經典、最被廣泛推薦的結構框架。不論測試邏輯多複雜，只要將步驟拆解為 **Arrange（準備）**、**Act（操作）**、**Assert（驗證）**，程式碼就會變得極具條理且易於維護！

## 什麼是 3A 模式？

一個完整的測試案例，通常由以下三個階段依序組成：

```markdown
```text  
┌──────────────────────────────────────────────┐  
│                  Arrange                     │ ➡️ 準備舞台 (渲染元件、初始化工具、抓取元素)  
└──────────────────────┬───────────────────────┘
                       │
                       ▼  
┌──────────────────────────────────────────────┐  
│                    Act                       │ ➡️ 模擬操作 (點擊按鈕、清空輸入框、鍵入文字)  
└──────────────────────┬───────────────────────┘
                       │
                       ▼  
┌──────────────────────────────────────────────┐  
│                  Assert                      │ ➡️ 驗證結果 (檢查畫面狀態是否符合預期)  
└──────────────────────────────────────────────┘
```

## 範例程式碼拆解

以下用模擬使用者更改 `App` 元件中輸入框文字的測試來做 3A 拆解：

```javascript
import { test, expect } from "vitest";  
import { userEvent } from "@testing-library/user-event";  
import { render, screen } from "@testing-library/react";  
import App from "./App";

test("updates the top text", async () => {
  // ────────────────────────────────────────────────────────
  // 1. Arrange (安排 / 準備環境)
  // ────────────────────────────────────────────────────────
  // 把測試所需要的前置舞台搭好。包括設置模擬使用者、渲染元件、撈出目標 DOM 元素。
  const user = userEvent.setup();
  render(<App />);
  const topTextbox = screen.getAllByRole("textbox")[0]; 

  // ────────────────────────────────────────────────────────
  // 2. Act (行動 / 執行操作)
  // ────────────────────────────────────────────────────────
  // 實際執行你要測試的核心行為。因為模擬使用者互動多為非同步操作，故搭配 await。
  await user.clear(topTextbox);                             // 清空輸入框
  await user.type(topTextbox, "A coder does not simply");   // 輸入新文字

  // ────────────────────────────────────────────────────────
  // 3. Assert (斷言 / 驗證結果)
  // ────────────────────────────────────────────────────────
  // 收網階段！驗證經過上述操作後，畫面上的狀態或文字有沒有變成我們預期的樣子。
  expect(screen.getByText("A coder does not simply")).toBeInTheDocument();  
});  
```

## 3A 步驟詳細說明
### 1. Arrange（安排環境）
* 核心任務：建立測試基本環境、Mock 資料、宣告變數、宣告模擬使用者、渲染（render）元件，並找出待會要操作的 DOM 元素。
* 白話比喻：想像要測試「自動販賣機投幣功能」，Arrange 就是「在實驗室裡放一台空的自動販賣機，並讓測試員手上拿著一塊 10 元硬幣」。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡 為什麼 `screen.getAllByRole` 是放在 Arrange 而不是 Act？
--> 因為 **「尋找目標」這個動作本身並沒有改變任何元件的狀態，它只是為了接下來的 Act 做準備**，所以歸類在 Arrange。

</blockquote>

### 2. Act（執行操作）
* 核心任務：觸發你要測試的目標行為。在前端測試中，通常是指模擬使用者的互動（如：點擊按鈕 `user.click()`、鍵入文字 `user.type()` 等）。
* 白話比喻：測試員實際上「把 10 元硬幣放進投幣孔」的這個動作。

### 3. Assert（驗證結果）
* 核心任務：使用斷言（如：`expect(...).toBe...`）來檢查測試結果是否符合預期。如果符合，測試通過（Pass）；不符合，測試失敗（Fail）。
* 白話比喻：檢查自動販賣機「有沒有掉出對應的飲料，且螢幕上的餘額有沒有正確扣除 10 元」。


## 總結整理

| 階段 | 英文 | 核心操作內容 | 在 React Testing Library 中常見的 API |  
| :--- | :--- | :--- | :--- |  
| **1. 準備** | **Arrange** | 設置環境、Mock 數據、抓取元素 | `userEvent.setup()`, `render()`, `screen.getBy...` |  
| **2. 操作** | **Act** | 模擬使用者互動行為（常搭配 `await`） | `user.click()`, `user.type()`, `user.clear()` |  
| **3. 驗證** | **Assert** | 使用 `expect` 進行斷言驗證 | `expect(...).toBeInTheDocument()`, `expect(...).toBe()` |

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>