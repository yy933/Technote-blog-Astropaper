---
title: "[React - Testing] 什麼是程式碼覆蓋率 (Code Coverage)？ - 筆記"
pubDatetime: 2026-07-07T04:35:37.356Z
tags: ["Testing","Vitest","Concepts","Vite","testing","Unit Test"]
description: "Table of contents 核心觀念 程式碼覆蓋率（Code Coverage）是一種指標，用來衡量「在執行測..."
hackmd_id: "HkTwnlc7zg"
---

## Table of contents

## 核心觀念  
程式碼覆蓋率（Code Coverage）是一種指標，用來衡量「在執行測試時，專案中有多少比例的程式碼被實際執行到」。高覆蓋率通常代表測試涵蓋得越全面，未知的 bug 隱患越少。

## 1. 覆蓋率的四大核心指標 (The 4 Metrics)

當你執行覆蓋率工具時，報告通常會提供以下四種維度的百分比（%）：

| 指標名稱 | 英文 | 說明 | 舉例 |  
| :--- | :--- | :--- | :--- |  
| **敘述覆蓋率** | **Statement Coverage** | 專案中的**每一行程式碼**被執行到的比例。 | 是否每一行變數宣告、賦值都被跑過？ |  
| **分支覆蓋率** | **Branch Coverage** | 邏輯判斷（如 `if-else`、`switch`）中**每個分支**被執行到的比例。 | 若有 `if(A) else{B}`，測試是否 A 與 B 兩種情況都有模擬到？ |  
| **函式覆蓋率** | **Function Coverage** | 專案中宣告的**每個 function** 有沒有被呼叫過。 | 元件裡的 `handleChange`、`getMemeImage` 是否都有被觸發？ |  
| **行覆蓋率** | **Line Coverage** | 類似 Statement，計算實際有執行程式碼的**行數**比例。 | 通常與 Statement 數字接近，但計算基準依工具微有不同。 |

-

## 2. 實戰範例說明

以 `Main.jsx` 元件中的 `handleChange` 函式為例：

```javascript
function handleChange(event) {
    const { value, name } = event.currentTarget;
    
    // 假設我們加一個防呆邏輯（分支）
    if (value === "不雅文字") {
        alert("請勿輸入不雅文字"); // 分支 A
    } else {
        setMeme(prevMeme => ({ ...prevMeme, [name]: value })); // 分支 B
    }
}
```

### 測試案例如何影響覆蓋率？
#### 情況一： 測試只有測試打入 "Code without coffee"。

❌ 分支覆蓋率不會滿（約 50%）：因為測試從未進入 `value === "不雅文字"` 的 分支 A，那一行程式碼會被標示為紅色的「未覆蓋」。

#### 情況二： 補了第二個測試，專門模擬打入 "不雅文字"。

分支覆蓋率達到 100%：因為兩條路徑都被測試程式碼實際走過一遍了。

## 常見迷思
### 迷思一：覆蓋率 100% = 程式絕對沒有 Bug？  
完全錯誤。 **覆蓋率工具非常誠實，它只管程式碼「有沒有被執行過」，但不管測試寫得「對不對」。**

例如：寫了一個相加函式 `add(a, b) { return a - b; }`（寫錯成減法）。測試跑了 `add(1, 2)`，此時這行的覆蓋率是 100%，但測試結果依然是錯的。

### 迷思二：團隊應該強烈追求 100% 的覆蓋率？  
盲目追求 100% 往往會帶來反效果。**因為最後那 10-15% 的程式碼通常是極端邊界狀況（Edge Cases）或是第三方套件的設定，為了測試它們需要寫出極度複雜、難以維護的測試碼。**

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡一般來說，專案的整體覆蓋率達到 70% ~ 80% 就已經是非常健康且高水準的表現了。

</blockquote>

## 實作：如何在 Vitest 中檢視覆蓋率？  
Vitest 原生就支援覆蓋率計算！只要在專案中安裝官方推薦的覆蓋率工具（如 `v8`）：

```bash
npm install -D @vitest/coverage-v8
```

接著在 `package.json` 加入一行新的指令：

```json
"scripts": {
  "test": "vitest",
  "test:coverage": "vitest run --coverage"
}
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡 這裡用 `vitest run --coverage` 的 `run` 代表 **「只跑一次測試就結束並產出報告」（不會卡在 Watch Mode）**，這在看覆蓋率或未來做 CI/CD 自動化部署時非常常用。

</blockquote>

把測試覆蓋率的設定加進 `vite.config.js`:

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vite.dev/config/
export default defineConfig({
  plugins: [react()],
  test: {
    setupFiles: ["./test-setup.js"],
    environment: 'jsdom',
    // 新增以下 coverage 設定項目
    coverage: {
      provider: 'v8', // 指定使用剛剛安裝的 v8 引擎
      reporter: ['text', 'json', 'html'], // 輸出的報告格式：終端機文字、JSON、HTML 網頁
    }
  }
})
```

執行 `npm run test:coverage` 後，終端機就會直接噴出一張漂亮的覆蓋率表格，還會在專案根目錄生成一個 `coverage/` 資料夾，打開裡面的 `index.html` 就能用瀏覽器看「哪幾行程式碼沒被測到」的視覺化報告。

終端機顯示Coverage report如下：

```bash
 Test Files  3 passed (3)
      Tests  10 passed (10)
   Start at  12:11:06
   Duration  6.40s (transform 704ms, setup 4.02s, collect 1.16s, tests 2.17s, environment 4.24s, prepare 1.21s)


 % Coverage report from v8

----------------|---------|----------|---------|---------|-------------------
File            | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s 
----------------|---------|----------|---------|---------|-------------------

All files       |    93.9 |     90.9 |   83.33 |    93.9 |                   
 src            |   64.28 |       50 |      50 |   64.28 |                   
  App.jsx       |     100 |      100 |     100 |     100 |                   
  index.jsx     |       0 |        0 |       0 |       0 | 1-6               
 src/components |     100 |      100 |     100 |     100 |                   
  Header.jsx    |     100 |      100 |     100 |     100 |                   
  Main.jsx      |     100 |      100 |     100 |     100 |                   
----------------|---------|----------|---------|---------|------------------
```

報告裡有一個比較刺眼的數字：

`index.jsx | 0 | 0 | 0 | 0 | 1-6`

這導致整體的 `src` 資料夾總覆蓋率被拉低到了 64.28%。以下說明原因與如何排除它：

### 為什麼 `index.jsx` 的覆蓋率是 0%？  
`index.jsx`（有些專案叫 `main.jsx`）是整個 React 專案的進入點（Entry Point）。它的主要功能只有一行：把 React 的 `<App />` 元件渲染到 HTML 的 `id="root"` 節點上。

程式碼通常長這樣：

```javascript
ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

**為什麼不需要測它？**
* **單元測試的職責**：單元測試（Unit Test）的目的是測試「元件內部的邏輯與互動」。**我們已經直接針對 `App.jsx` 進行了全面的測試，而 `index.jsx` 只是負責掛載，裡面沒有任何商業邏輯。**
* **測試環境限制**：在 `jsdom` 模擬環境中，有時候這類與真實瀏覽器 DOM 根節點強綁定的進入點檔案，本來就很難（也沒必要）透過一般的 render 跑過去。

### 最佳實踐：在設定中「排除」不需要測試的檔案  
像 `index.jsx`、打包設定檔、或是純定義型別的檔案，我們通常會在外層配置中將它們排除（Exclude）在外，這樣產出的覆蓋率報告才會是乾淨、真正具備參考價值的 100%。

直接修改 `vite.config.js`，加入 `exclude` 排除名單，並且為了避免根目錄的設定檔（如 `eslint.config.js`、`vite.config.js`）或專案進入點拉低覆蓋率分數，應在 `vite.config.js` 中同時設定 `include` 與 `exclude`：：

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    setupFiles: ["./test-setup.js"],
    environment: 'jsdom',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      // 1. 強制限定只計算 src 內的檔案（防止根目錄設定檔被納入）
      include: ['src/**/*'],
      // 2. 排除 src 內無商業邏輯的進入點與測試配置
      exclude: [
        'src/index.jsx',
        'src/main.jsx',
        'src/test-setup.js',
        '**/*.test.{js,jsx}'
      ]
    }
  }
})
```

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>