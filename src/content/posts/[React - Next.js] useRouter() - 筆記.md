---
title: "[React - Next.js] useRouter() - 筆記"
pubDatetime: 2026-05-25T11:17:36.424Z
tags: ["React.js","Node.js","Next.js","React Hook"]
description: " Table of contents useRouter() 是什麼？ useRouter 是 Next.js 提供的 ..."
---

## Table of contents

## useRouter() 是什麼？
useRouter 是 Next.js 提供的 React hook，用來：
* 取得路由資訊（像是目前網址、query params 等）
* 程式化地「導引頁面」：跳轉或取代目前頁面

### 匯入方式
```ts
import { useRouter } from 'next/navigation'; 
```

## 常見功能
| 功能         | 用法                     | 說明                             |
| ---------- | ---------------------- | ------------------------------ |
| 導頁（加進歷史紀錄） | `router.push('/game')` | 使用者按返回鍵會回到上一頁                  |
| 導頁（取代歷史紀錄） | `router.replace('/')`  | 使用者按返回鍵 **不會** 回到被 replace 的頁面 |
| 重新整理當前頁面   | `router.refresh()`     | 常見於資料有更新後重新載入                  |

## 設計一段防呆機制
:bulb: 情境：遊戲必須按Start Game按鈕才能進入`/game`頁面，如果使用者自行在瀏覽器網址列輸入`/game`，就導回首頁。按Start Game按鈕才會fetch API、才會有資料回傳，直接輸入`/game`沒有資料，因此可以用以下方式判斷是否透過按鈕進入遊戲頁面：
```ts
useEffect(() => {
  if (emojisdata.length === 0) {
    router.replace('/');
  }
}, [emojisdata, router]);
```
這段是防止使用者直接輸入網址 `/game` 卻沒經過 Start Game 的按鈕。
* 如果 `emojisdata` 是空的（代表沒有資料，可能是沒按按鈕或刷新頁面）：
* 立刻用 `router.replace('/')` 導回首頁（不能玩遊戲）

## replace 和 push 差在哪？
| 方法        | 行為                       | 適合場景                |
| --------- | ------------------------ | ------------------- |
| `push`    | 加入歷史紀錄（可以回上一頁）           | 一般導頁                |
| `replace` | 取代目前頁面（按返回鍵不會回到 `/game`） | **導錯頁時導回去**、登入後跳轉首頁 |