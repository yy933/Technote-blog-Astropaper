---
title: "[React] 到底什麼時候需要 useEffect ? - 筆記"
pubDatetime: 2025-05-18T23:20:10.000Z
modDatetime: 2026-05-25T10:04:23.521Z
tags: ["JavaScript","React.js","React Hook"]
description: "Table of contents <blockquote class=\"my6 p4 bggreen50 dark:..."
---

## Table of contents

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:bulb: 一句話判斷：如果要做的事是 **「跟畫面渲染以外的東西溝通」** ，才需要`useEffect`。  
如果不是為了「做 React 無法自行處理的事情」，就不要急著用 useEffect。

</blockquote>


## 什麼時候該用 useEffect？  
以下這些都是「React 自己無法控制的外部行為」，就需要用 useEffect 來監控條件、執行動作：  
| 情境                          | 說明                               |  
| --------------------------- | -------------------------------- |  
| 播放音效 / 觸發動畫                 | React 無法主動控制 DOM API、Audio API   |  
| 修改非 React 管理的 DOM           | 例如直接用 `document.querySelector` 改東西 |  
| 使用 `setTimeout` / `setInterval` | React 不知道你設了 timer               |  
| 呼叫外部 API                    | 網路請求、資料存取                        |  
| 記錄 log / localStorage       | React 本身不處理這些                    |

## 什麼時候不用 useEffect？  
1. 純 UI 更新不需side effects的情況:  
純粹根據 props 或 state 回傳 JSX，不要用 useEffect。  
2. 想同步計算的變數或值:  
這種情況直接用普通的函式或計算屬性。  
3. 避免在 render 中執行副作用:  
React render 階段應保持純粹、無side effects。

## 常見盲點  
很多開發者會預設想到狀態變化就用 useEffect，但這會讓程式碼：

* 更難讀（分散邏輯）
* 更難除錯（副作用可能造成 race condition）
* 更難優化（重複運算）

:arrow_right: React 設計理念就是：用狀態決定畫面，除非真的需要，否則不要side effect！

## 簡單判斷流程
* 程式碼需要進行外部行為？  
→ 是 → 用 `useEffect`  
→ 否 → 不用

* 需要非同步或等待渲染後才做的工作？  
→ 是 → 用 `useEffect`

* 需要清理side effect？  
→ 是 → 用 `useEffect`，並回傳cleanup function