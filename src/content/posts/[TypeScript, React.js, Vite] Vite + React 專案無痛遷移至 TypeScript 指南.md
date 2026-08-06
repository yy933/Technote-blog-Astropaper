---
title: "[TypeScript, React.js, Vite] Vite + React 專案無痛遷移至 TypeScript 指南"
pubDatetime: 2026-08-06T04:31:45.675Z
tags: ["Project","React.js","TypeScript","Frontend","Vite","cheatsheet","JavaScript","Migration"]
description: "Table of contents 說明：Vite + React (JS to TS Migration) 本篇說明..."
hackmd_id: "BJPzOFb8Ge"
---

## Table of contents

## 說明：Vite + React (JS to TS Migration)  
本篇說明如何將現有的 Vite + React (JavaScript) 專案遷移至 TypeScript，包含dependencies安裝、編譯器設定、進入點調整及型別檢查指令設置。

## 1. 遷移流程圖 (Architecture Flow)

```text
[階段 1：環境dependencies配置]
  1. 安裝 TypeScript 與 @types/react, @types/react-dom
  2. 建立 tsconfig.json 與 tsconfig.node.json
          │
          ▼
[階段 2：進入點與構建設定檔改名]
  3. 將 vite.config.js 重命名為 vite.config.ts
  4. 修改 index.html 中的模組引入路徑 (main.jsx ➔ main.tsx)
  5. 將 src/main.jsx 重命名為 src/main.tsx
          │
          ▼
[階段 3：元件與工具函式漸進式重構]
  6. 工具函式 / Helper：.js ➔ .ts (補上參數與回傳型別)
  7. React 組件 / Pages：.jsx ➔ .tsx (補上 Props、State 與 Event 型別)
          │
          ▼
[階段 4：腳本設置與型別驗證]
  8. package.json 新增 tsc 型別檢查指令 (npm run typecheck)
  9. 執行 npm run typecheck 確認零 Build Error
 ```
 
## 2. 四階段拆解
### 階段 1：安裝 TypeScript 與基礎設定  
在專案根目錄執行以下指令，安裝 TypeScript 本體與對應的型別定義檔：

```bash
npm install -D typescript @types/react @types/react-dom @types/node
```

在專案根目錄建立 `tsconfig.json` 設定檔：

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,

    /* Path Aliases */
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"]
}
```

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

若要對 Vite 設定檔進行型別檢查，可補上 `tsconfig.node.json`：

```json
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}
```

</blockquote>

### 階段 2：修改 Vite 設定與進入點
* 將根目錄的 `vite.config.js` 重新命名為 `vite.config.ts`。
* 更新 `index.html` 中的 `script` 引入路徑：

```html
<!-- 修改前 -->
<script type="module" src="/src/main.jsx"></script>

<!-- 修改後 -->
<script type="module" src="/src/main.tsx"></script>
```

將 `src/main.jsx` 重新命名為 `src/main.tsx`，並為 HTML root 節點補充斷言：

```typescript
// src/main.tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App/>
  </React.StrictMode>,
)
```

### 階段 3：組件與工具函式轉換檔名與補強型別  
將 `src/` 目錄下的檔案逐步轉換：
* 包含 JSX 的檔名：`.jsx` --> `.tsx`
* 純邏輯 / 工具函式：`.js` --> `.ts`
*   
常見型別重構範例 (`Button.tsx`)：

```typescript
import React from 'react';

// 1. 定義 Props 介面
interface ButtonProps {
  label: string;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

// 2. 於組件參數標註型別
export const Button: React.FC<ButtonProps> = ({ 
  label, 
  onClick, 
  variant = 'primary', 
  disabled = false 
}) => {
  return (
    <button 
      className={`btn btn-${variant}`} 
      onClick={onClick} 
      disabled={disabled}
    >
      {label}
    </button>
  );
};
```

### 階段 4：設定 Build 與 Typecheck 腳本  
Vite 在開發與打包（`vite build`）時不會主動中斷型別錯誤。為了確保 CI/CD 或部署時能抓出 TypeScript 錯誤，需更新 `package.json` 的 `scripts`：

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "typecheck": "tsc --noEmit",
    "preview": "vite preview"
  }
}
```

執行型別檢查：

```bash
npm run typecheck
```

## 3. 其他注意事項
* Vite 的 Transpile 與 Type Check 機制Separation：Vite 使用 Esbuild 進行快速語法轉譯（Strip Types），但不做 Type Checking。因此必須在打包時透過 `tsc && vite build` 來強制執行型別卡關。
* 路徑別名 (Path Alias) 同步：若專案在 `vite.config.ts` 設定了 `@/` 別名，必須同步在 `tsconfig.json` 的 `paths` 中註冊，編輯器 (VS Code) 才能精準提供路徑自動補齊。
* 第三方套件缺少 Type Definitions：若引入某些舊套件報錯 `Could not find a declaration file...`，可先透過 `npm install -D @types/套件名稱` 安裝，或在 `src/vite-env.d.ts `建立 `declare module '套件名稱';` 繞過檢查。

## 參考資料
* [Vite 官方文件 - Features (TypeScript)](https://vite.dev/guide/features#typescript)
* [React TypeScript Cheatsheet (官方推薦社群文件)](https://react-typescript-cheatsheet.netlify.app/)

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>