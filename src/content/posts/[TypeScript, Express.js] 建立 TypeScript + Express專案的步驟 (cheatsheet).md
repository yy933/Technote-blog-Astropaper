---
title: "[TypeScript, Express.js] 建立 TypeScript + Express專案的步驟 (cheatsheet)"
pubDatetime: 2026-08-06T08:46:11.992Z
tags: ["TypeScript","cheatsheet","JavaScript","Project","Node.js","Express.js","Backend"]
description: "Table of contents 1.初始化專案資料夾 建立專案目錄並生成 package.json 在終端機執行以..."
hackmd_id: "H1HbU6b8Ml"
---

## Table of contents

## 1.初始化專案資料夾  
建立專案目錄並生成 `package.json`  
在終端機執行以下指令建立專案目錄並初始化 Node.js 環境：

```bash
mkdir express-ts-demo
cd express-ts-demo
npm init -y
```

## 2.安裝 Express 與 TypeScript 開發Dependencies  
包含核心套件、型別定義與開發輔助工具。  
安裝 Express 核心套件與環境變數管理工具，以及 TypeScript 相關的開發Dependencies（DevDependencies）：

```bash
# 安裝運行時需要的套件
npm install express dotenv

# 安裝 TypeScript、型別定義檔與開發監控工具
npm install -D typescript @types/node @types/express tsx @tsconfig/node20
```

## 3.產生並配置 `tsconfig.json`  
設定 TypeScript 編譯選項與輸出路徑。執行以下指令生成 TypeScript 設定檔：

```bash
npx tsc --init
```

開啟生成的 `tsconfig.json`，確認或修改以下核心設定：

```json
{
  "extends": "@tsconfig/node20/tsconfig.json",
  "compilerOptions": {
    /* File Layout */
    "rootDir": "./src",
    "outDir": "./dist",

    /* Environment Settings */
    "module": "nodenext",
    "target": "esnext",

    /* Other Outputs */
    "sourceMap": true,
    "declaration": true,
    "declarationMap": true,

    /* Stricter Typechecking Options */
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,

    /* Recommended Options */
    "strict": true,
    "verbatimModuleSyntax": true,
    "isolatedModules": true,
    "noUncheckedSideEffectImports": true,
    "moduleDetection": "force",
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}

```

## 4.建立專案結構與入口檔案  
撰寫第一個包含型別標註的 Express 伺服器。  
建立 `src` 資料夾，並在其下新增 `index.ts` 檔案：

```typescript
import express from 'express';
import type { Express, Request, Response } from 'express';
import dotenv from 'dotenv';

dotenv.config();

const app: Express = express();
const port = process.env.PORT || 3000;

app.use(express.json());

app.get('/', (req: Request, res: Response) => {
  res.send('Express + TypeScript Server running!');
});

app.listen(port, () => {
  console.log(`Server is running at http://localhost:${port}`);
});
```

## 5.設定 package.json 執行腳本  
加入開發模式與編譯建置指令。  
開啟 `package.json`，在 `"scripts"` 區塊新增 `dev`、`build` 與 `start` 指令：

```json
"scripts": {
  "dev": "tsx watch src/index.ts",
  "build": "tsc",
  "start": "node dist/index.js"
}
```

## 6.啟動與測試專案  
驗證熱重載與編譯輸出。
- 開發環境測試（支援存檔自動重啟）：

```bash
npm run dev
```

- 生產環境編譯與執行：

```bash
# 將 TS 編譯為 JS 到 dist/ 資料夾
npm run build

# 執行編譯後的 JS 檔案
npm start
```