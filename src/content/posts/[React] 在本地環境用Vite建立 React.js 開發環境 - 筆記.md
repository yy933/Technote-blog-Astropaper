---
title: "[React] 在本地環境用Vite建立 React.js 開發環境 - 筆記"
pubDatetime: 2026-05-26T03:29:26.530Z
tags: ["Node.js","cheatsheet","React.js"]
description: " Table of contents 1. 準備 Node.js 環境 用nvm安裝Node.js，確認Node版本:n..."
---

## Table of contents


 

### 1. 準備 Node.js 環境
* 用nvm安裝Node.js，確認Node版本:`node -v`
* 同時也確認npm安裝: `npm -v`
### 2. 用 Vite 快速建置專案
用 [Vite](https://vite.dev/guide/) 來建立 React 專案
```bash
npm create vite@latest my-chat-app --template react
// my-chat-app 為資料夾名稱，可以改成自己喜歡的名字
```
根據提示選：
* 專案名稱：my-chat-app
* 選擇框架：React
* 選擇語言：JavaScript

超快速建立完成，終端機顯示：
![螢幕擷取畫面 2025-04-23 170751](https://hackmd.io/_uploads/SJLdTX8kge.png)
照做就是了。

* 安裝dependencies
```bash
cd my-chat-app
npm install
```
### 3. 啟動開發伺服器
```bash
npm run dev
```
:exclamation: 出現錯誤如下:
```bash
SyntaxError: The requested module 'node:fs/promises' does not provide an export named 'constants'
```
因為 Node.js 版本太舊，導致不支援 `import { constants } from 'node:fs/promises'` 這種寫法。

升級Node.js至v18，確認版本:
```bash
$ node -v
v18.20.8
```
再重啟一次伺服器:
```bash
npm run dev
```
![螢幕擷取畫面 2025-04-23 171448](https://hackmd.io/_uploads/S1XZyNUJxe.png)
打開瀏覽器，到`http://localhost:5173/`確認，出現以下畫面就成功了：
![螢幕擷取畫面 2025-04-23 171543](https://hackmd.io/_uploads/Hk8EkV8Jll.png)

### 4. 專案架構
開啟VScode看一下專案架構：
![螢幕擷取畫面 2025-04-23 171756](https://hackmd.io/_uploads/SyAygNLJle.png)

* 打開 `src/App.jsx`，先改成這樣，建立一個 Hello Component：

```jsx
function App() {
  return (
    <div style={{ textAlign: 'center', marginTop: '3rem' }}>
      <h1>Hello, Vite + React!</h1>
    </div>
  );
}

export default App;
```
看一下瀏覽器畫面:
![螢幕擷取畫面 2025-04-23 172121](https://hackmd.io/_uploads/rk9tlVL1gl.png)


### :toolbox: React Developer Tools
下載[React Developer Tools](https://chromewebstore.google.com/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en&pli=1)這個瀏覽器套件幫助除錯

![螢幕擷取畫面 2025-04-23 174125](https://hackmd.io/_uploads/ByxrBNUkxl.png)
當 React Developer Tools 感應到網頁是用 React 打造時，icon 會自動亮起來，這時候可以看到DevTools面板上多了兩個有 React 圖示的區塊：Components & Profiler，Components 區塊會看到元件的 state 和方法，Profiler 區塊是評估渲染效能時會用到的工具。




<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>