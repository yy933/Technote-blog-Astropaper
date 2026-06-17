---
title: "[Node.js] 模組系統：CommonJS (CJS) vs ES Module (ESM) - 筆記"
pubDatetime: 2026-06-04T09:35:16.816Z
tags: ["Node.js"," JavaScript","Backend"]
description: "Table of contents :memo: CJS 與 ESM 是什麼？ 在 JavaScript 的世界裡，隨..."
hackmd_id: "r1nXzTCxGe"
---

## Table of contents

## :memo: CJS 與 ESM 是什麼？  
在 JavaScript 的世界裡，隨者專案規模擴大，我們需要將程式碼拆分成多個檔案管理，這就是**模組化（Modularity）**。而在 Node.js 與前端開發生態中，最核心的兩大模組標準就是 **CommonJS (CJS)** 與 **ES Module (ESM)**。

* **CommonJS (CJS)**：Node.js 誕生初期的預設模組系統，歷史包袱最重。使用 `require()` 引入、`module.exports` 導出。
* **ES Module (ESM)**：ECMAScript 官方於 2015 年推出的國際標準，是現代 JavaScript 的統一規格。使用 `import` 引入、`export` 導出。


### 為什麼以前 CJS 變數前面要加雙底線 `__`？

在 CommonJS 環境中，我們常看到 `__dirname`、`__filename` 這類前後加上雙底線（俗稱 Dunder）的變數。

1. **隱式注入的參數**：Node.js 在執行 CJS 檔案前，會悄悄用一個外殼函式（Module Wrapper）把程式碼包起來：

```javascript
   (function(exports, require, module, __filename, __dirname) {
       // 你的程式碼其實在函式內部執行
   });
```

2. **避免命名空間污染**：加上 `__` 是為了明確告知開發者：**「這是系統注入的環境變數，不是你自己寫的全域變數」，防止開發者自己宣告同名變數而導致覆蓋。**

3. **ESM 的轉變**：到了 ESM 環境，Node.js 不再使用外殼函式注入。因此 `__dirname` 與 `__filename` 在 ESM 中是完全不存在的，必須統一改用官方標準的 `import.meta` 物件（如 `import.meta.dirname`）來動態獲取元資料（Metadata）。


## :memo: CJS和ESM核心機制三大本質差別  
除了語法上 require 與 import 的不同外，這兩者在底層運行機制上有著根本性的巨大差異：

1. 運行時機：動態同步 vs. 靜態編譯
  - **CommonJS (CJS) 是「執行時（Runtime）載入」**：當程式碼執行到 `require()` 那一行時，Node.js 才會同步去讀取檔案並執行內容。因此，`require()` 可以靈活地寫在 `if` 判斷式或函式內部。
  - **ES Module (ESM) 是「編譯時（Compile-time）載入」**：在程式碼真正執行前，JavaScript 引擎會先掃描所有的 `import` 建立依賴圖。**因此，`import` 必須寫在檔案最頂層，不能寫在條件句中（除非使用動態 `import()`）。**


2. 導出本質：值的複製 vs. 值的引用
  - **CommonJS 導出的是「值的複製（Value Copy）」**：一旦模組被 `require` 進來，原模組內部變數再怎麼改變，你拿到的那個複製品都不會變。
  - **ES Module 導出的是「值的動態唯讀引用（Live Read-only Binding）」**：你拿到的是一個指向原模組記憶體位置的指標。**若原模組內部變數更新，你這邊讀取到的值也會同步更新**。

3. 非同步支援（Top-level Await）
* **ESM 天生支援 Top-level Await**，在檔案最頂層可以直接寫 `await fs.readFile()`。
* **CJS 則完全不支援**，要使用 `await` 就必須用 `async function` 把他包起來。

## :memo: CJS 和 ESM 的核心差別與環境支援

| 特性  | CommonJS (CJS) | ES Module (ESM) |  
| :--- | :--- | :--- |  
| **主要語法** | `require()` / `module.exports` | `import` / `export` |  
| **路徑環境變數** | 有 `__dirname`, `__filename` | 無，改用 `import.meta.dirname` |  
| **載入時機** | 執行時（Runtime），同步載入 | 編譯時（Compile-time），非同步 |  
| **導出機制** | 值的複製品（彼此獨立） | 值的引用連結（即時同步更新） |  
| **前端（瀏覽器）** | **原生完全不支援**（需透過 Webpack/Vite 打包）。 | **原生支援**（`<script type="module">`）。 |  
| **後端（Node.js）** | **原生支援**（歷史預設標準）。 | **原生支援**（Node.js 12+ 後趨於成熟）。 |

## :memo: 實務應用：Node.js 如何判斷與選擇？  
當我們在後端執行 `node app.js` 時，Node.js 主要透過以下兩套機制來判斷該以 CJS 還是 ESM 運行：

1. 透過 package.json 中的 "type" 欄位（全域影響）
* 若未設定，或設定為 `"type": "commonjs"`：專案內所有 `.js` 檔案皆視為 CJS。
* 若設定為 `"type": "module"`：專案內所有 `.js` 檔案皆視為 ESM。

2. 透過檔案副檔名（最高優先權）

副檔名可以強制覆蓋 `package.json` 的設定：
* `.cjs`：強制將該檔案視為 CommonJS。
* `.mjs`：強制將該檔案視為 ES Module。
* `.js`：聽從 `package.json` 裡 `"type"` 的決定。

### 💡 舊專案 CJS 想重構升級成 ESM，會很麻煩嗎？  
老實說，會有一點痛苦，因為會遇到一連串的連鎖報錯：

1. **路徑報錯**：所有的 `__dirname` 必須手動改寫為 `import.meta.dirname`。  
1. **副檔名限制**：在 ESM 中引入本地檔案，`import user from './user.js'` 必須補上 `.js` 副檔名，否則會噴 `ERR_MODULE_NOT_FOUND`。  
1. **相容性地獄**：CJS 檔案完全無法去 `require` 一個已經升級成 ESM 的檔案或現代套件，這會逼得你必須把整條依賴鏈一口氣全改掉。  
1. **引入 JSON 變嚴格**：不能直接 `import JSON`，必須改用新語法：`import data from './data.json' with { type: 'json' };`。

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

安全重構建議：如果專案龐大，不要直接改 `package.json`。可以先採取 **「漸進式搬家」**，將部分獨立的 API 檔案副檔名改為 `.mjs` 局部試水溫；或是直接引進 TypeScript / Bun / Vite-node 等現代建置工具，讓編譯器在底層幫你處理掉這些語法相容性的折磨。

</blockquote>


## 小結
* **前端開發**：因為現代工具鏈（如 Vite）與效能優化（Tree-shaking 自動刪除未動用代碼）的需求，**已經全面走向 ESM**。
* **後端（Node.js）**：處於轉型過渡期。**新專案、新開源套件一律優先選擇 ESM**；但因為龐大的歷史遺留代碼，舊專案維護中依然會持續看到 CJS 的身影。

長痛不如短痛，如果是未來需要持續頻繁維護、新增功能的商業專案，非常建議抽空將其重構成 ESM 標準，以享受更現代、效能更好的生態圈支援！

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>