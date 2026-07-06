---
title: "[React - Testing] 在 Vite 專案中建立 Vitest 與 React Testing Library 測試環境 - 筆記"
pubDatetime: 2026-07-06T05:26:56.531Z
tags: ["cheatsheet","Vite","Vitest","React.js","Unit Test","testing"]
description: "Table of contents 安裝測試框架與工具 首先，我們需要安裝測試核心框架 vitest，以及用於渲染 R..."
hackmd_id: "rkY5_2umzx"
---

## Table of contents


 

## 安裝測試框架與工具

首先，我們需要安裝測試核心框架 `vitest`，以及用於渲染 React 元件、DOM 斷言（Matchers）與模擬使用者事件的相關工具。

### 1. 安裝 Vitest 核心

```bash
npm install -D vitest
```

### 2. 安裝 React Testing Library 家族

```bash
npm install -D @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

工具說明：
* `@testing-library/react`：負責在測試環境中渲染（render）React 元件。
* `@testing-library/jest-dom`：擴充 Vitest 的斷言庫，提供如 `.toBeInTheDocument()` 等強大且直覺的 DOM 匹配器。
* `@testing-library/user-event`：比傳統 fireEvent 更擬真的使用者互動模擬工具。

### 3. 安裝環境模擬工具  
由於測試是在 Node.js 環境運行，我們需要一個模擬的瀏覽器環境：

```bash
npm install -D jsdom
```

## 測試環境設定  
安裝完工具後，需要進行專案配置，讓 Vitest 知道如何載入這些測試工具。

### 4. 建立測試初始化設定檔  
在專案根目錄（Project Root）下建立一個名為 `test-setup.js` 的檔案，用來處理每個測試結束後的清理工作並導入 DOM 斷言庫：

```javascript
// test-setup.js
import "@testing-library/jest-dom/vitest";
import { afterEach } from 'vitest';
import { cleanup } from "@testing-library/react";

// 在每個測試案例（test case）結束後自動清理 DOM，避免測試互相干擾
afterEach(() => {
    cleanup();
});
```

### 5. 調整 Vite 配置檔  
在專案根目錄的 `vite.config.js`（或 `vite.config.ts`）中，將剛剛建立的設定檔與環境寫入 test 屬性中：

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  // 新增以下 test 設定項目
  test: {
    setupFiles: ["./test-setup.js"],
    environment: 'jsdom'
  }
})
```

## 配置腳本與執行測試
### 6. 新增指令至 `package.json`  
打開 `package.json`，在 `scripts` 區塊中加入啟動 `Vitest` 的指令：

```json
"scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint . --ext js,jsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview",
    "test": "vitest"
}
```

### 7. 啟動測試（Watch Mode）  
在終端機執行以下指令，Vitest 預設會以 Watch Mode 啟動，當專案內的程式碼或測試檔有變動時，會自動重新跑測試：

```bash
npm run test
```

## 常用指令與工具對照表

| 指令 / 工具 | 類型 | 說明 |  
| :--- | :--- | :--- |  
| `npm run test` | 指令 | 啟動本地測試環境（預設開啟 Watch Mode） |  
| `vitest` | 測試框架 | 核心測試執行器（Test Runner），速度極快 |  
| `jsdom` | 測試環境 | 模擬瀏覽器行為的純 JavaScript 專案環境 |  
| `@testing-library/react` | 測試工具 | 提供 `render`, `screen` 等方法來測試 React 元件 |

> 進階設定：MSW Mock API、Coverage 測試覆蓋率報告（待補）

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>