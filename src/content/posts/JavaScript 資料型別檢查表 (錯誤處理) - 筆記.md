---
title: "JavaScript 資料型別檢查表 (錯誤處理) - 筆記"
pubDatetime: 2025-04-09T20:20:30.000Z
modDatetime: 2026-05-25T10:04:23.781Z
tags: ["cheatsheet","JavaScript","Interview Preparation","Data type"]
description: "Table of contents 給 JS 開發者的 「資料型別檢查表」，適合在寫 if 判斷時或做錯誤處理(err..."
hackmd_id: "S1Z8b6NRyl"
---

## Table of contents


 給 JS 開發者的 「資料型別檢查表」，適合在寫 if 判斷時或做錯誤處理(error handling) 時使用，寫出比較穩健的資料驗證邏輯，來**防止應用程式未來遇到非預期型別的資料時出錯**。

## JavaScript 資料型別檢查表

| ✅ 目標 | ✅ 安全寫法 | 🔍 說明 |  
| --- | --- | --- |  
| 是否為陣列 | `Array.isArray(value)` | `typeof []` 是 object，不可靠* |  
| 陣列是否有資料 | `Array.isArray(arr) && arr.length > 0` | 避免 `arr` 是 `null/undefined` |  
| 是否為物件（非陣列） | `typeof obj === 'object' && obj !== null && !Array.isArray(obj)` | 過濾掉 `null` 和陣列 |  
| 是否為字串 | `typeof str === 'string'` | 字串型別確認 |  
| 字串是否有值（非空） | `typeof str === 'string' && str.trim() !== ''` | 空白字串也不算有值 |  
| 是否為數字（不是 NaN） | `typeof num === 'number' && !isNaN(num)` | NaN 也是 number，所以要排除 |  
| 是否為布林值 | `typeof val === 'boolean'` | `true`/`false` 檢查 |  
| 是否為函式 | `typeof fn === 'function'`

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

*在 JavaScript 中，某些內建的型別檢查方式（例如 typeof）會有誤導或不準確的情況，所以在做資料判斷時需要「防呆」。例如：  
`typeof null  //  回傳 'object' `　  
所以不能用　`typeof something === 'object'`　判斷是否為物件。

應該要這樣寫：  
`typeof value === 'object' && value !== null`

常見的 JavaScript 型別防呆對照表：

| 類型 | ❌ 錯誤寫法 | ✅ 建議寫法 | 🔍 為什麼 |  
|------|--------------|-------------------------------|-----------------------------|  
| 陣列 | `typeof arr === 'object'` | `Array.isArray(arr)` | 陣列其實是 object，typeof 不準確 |  
| 數字 | `typeof val === 'number'` | `typeof val === 'number' && !isNaN(val)` | `NaN` 也是 number，需要額外排除 |  
| 空值判斷 | `if (val)` | 視需求寫更明確的邏輯判斷 | `0`、`''`、`false` 也會變成 falsy，不一定代表錯誤 |

</blockquote>

## JavaScript 判斷「有值」的常見寫法對照表

|  需求 |  寫法 |  
| --- | --- |  
| 有值才繼續往下執行（Early Return） | `if (!data) return;` |  
| 有值才顯示某欄位（Template） | `{{ data?.name }}` |  
| 有值才執行函式 | `fn && fn()` 或 `typeof fn === 'function' && fn()` |  
| 有值就用，沒值用預設 | `const x = val ?? 'default'` ** |

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

### ** Nullish Coalescing Operator（空值合併運算子）  
`??`是 JavaScript 裡的 Nullish Coalescing Operator（空值合併運算子），簡單說就是：
**當左邊是 null 或 undefined 的時候，才會使用右邊的預設值。**

`const name = userInput ?? 'Default';`
* 如果 `userInput` 是` null` 或 `undefined`，就使用 `'Default'`
* 如果 `userInput` 是 `''`（空字串）、`0`、`false` 這些有「值」但是假值（falsy）的東西，它還是會使用原本的 `userInput`，不會跑預設值！
### 和 `||` 的差別是什麼？  
`const name = userInput || 'Default';`

這樣寫的話，如果 `userInput` 是：  
`null` ✅  
`undefined` ✅

`''`（空字串）❌ ← 也會被當作沒值  
`0` ❌  
`false` ❌

就會被換掉。

</blockquote>





<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>