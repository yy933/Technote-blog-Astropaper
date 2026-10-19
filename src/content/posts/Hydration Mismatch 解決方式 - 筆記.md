---
title: "Hydration Mismatch 解決方式 - 筆記"
pubDatetime: 2026-10-19T09:01:39.608Z
tags: ["JavaScript","Frontend","Concepts","Next.js","Issue","Hydration","React.js","DOM"]
description: "Tags: JavaScript Frontend Concepts React.js Issue Next.js H..."
hackmd_id: "H1vKdIXhMl"
---

###### Tags: `JavaScript` `Frontend` `Concepts` `React.js` `Issue` `Next.js` `Hydration` `DOM`


## Table of contents

## 問題 （Issue）

套用next-themes來建立ThemeProvider打造身淺色模式切換，但是Next.js跳出警告：

```bash
## Error Type
Console Error

## Error Message
A tree hydrated but some attributes of the server rendered HTML didn't match the client properties. This won't be patched up. This can happen if a SSR-ed Client Component used:

A server/client branch `if (typeof window !== 'undefined')`.
Variable input such as `Date.now()` or `Math.random()` which changes each time it's called.
Date formatting in a user's locale which doesn't match the server.
External changing data without sending a snapshot of it along with the HTML.
Invalid HTML tag nesting.

It can also happen if the client has a browser extension installed which messes with the HTML before React loaded.

https://react.dev/link/hydration-mismatch
```



## 為什麼會出現這個警告？  
這個 Hydration 錯誤是 Next.js 開發中使用 `next-themes` 時最常見的經典情況：伺服器端（SSR）與客戶端（Client）在載入網頁時，對 `<html>` 標籤注入的屬性不一致。

從錯誤日誌中可以看到：
* 伺服器端渲染出的 `<html>` 沒有帶 `style={{color-scheme: "light"}}`。
* 但客戶端因為載入了 `next-themes` 或是瀏覽器擴充功能（例如 Dark Reader 等），在畫面掛載的瞬間動態補上了主題相關的屬性（如 `style` 或主題 `class`），導致 React 認為前後不匹配而噴出警告。


## 如何解決

在 `<html>` 標籤上加上 `suppressHydrationWarning` 屬性。

這個屬性會明確告訴 React：「我知道這個元素（通常是 `<html>` 或 `<body>`）會因為主題切換外掛或腳本在客戶端被動態修改屬性，請忽略這部分的差異比對，不要跳出 Hydration 警告。」這是官方與 `next-themes` 推薦的標準作法。

```typescript
// app/layout.tsx
import type { Metadata } from 'next';
import { Geist, Geist_Mono } from 'next/font/google';
import './globals.css';
import { Navbar } from '@/components/Navbar';
import { ThemeProvider } from 'next-themes'; // 確保有正確引入 ThemeProvider

const geistSans = Geist({
  variable: '--font-geist-sans',
  subsets: ['latin'],
});

const geistMono = Geist_Mono({
  variable: '--font-geist-mono',
  subsets: ['latin'],
});

export const metadata: Metadata = {
  title: 'Ask My Notes',
  description: 'AI Knowledge Assistant with RAG & Supabase',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    /* 👈 關鍵：加上 suppressHydrationWarning 屬性 */
    <html lang="en" className={`${geistSans.variable} ${geistMono.variable} h-full antialiased`} suppressHydrationWarning>
      <body className="flex min-h-full flex-col">
        <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
          <Navbar />
          <main className="flex-1 flex flex-col">{children}</main>
        </ThemeProvider>
      </body>
    </html>
  );
}
```

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

### 為什麼加上這個就好了？  
`next-themes` 在頁面剛載入時，必須透過客戶端的 JavaScript 去讀取使用者的 `localStorage` 或系統偏好（預防畫面閃爍），並在 `<html>` 上即時注入樣式或 `Class`。  
因為這個動作發生在瀏覽器端，伺服器在編譯 HTML 時無法預知結果。加上 `suppressHydrationWarning` 就能完美包容這個正常的客戶端行為，主控台的 Hydration 錯誤就會瞬間消失！

</blockquote>


## 觀念解釋：什麼是 Hydration？  
在 Next.js 這類全端框架中：

* 伺服器端（SSR）：當使用者請求頁面時，伺服器會先把 React 元件跑過一遍，生成一堆靜態的 HTML 字串（例如 `<html lang="en">`），然後傳給瀏覽器。這時候網頁沒有 JavaScript 互動能力，只是一張「靜態圖片」。

* 客戶端（Client）：瀏覽器下載 HTML 顯示後，會接著下載 React 的 JavaScript 檔案。此時 React 會進行 Hydration —它會掃描現有的 DOM 樹，並把事件監聽器（Event Listeners）「黏」上去，讓網頁變成有互動能力的活網頁。

## 為什麼會產生 Hydration Mismatch（不匹配警告）？  
在Hydration的過程中，React 會進行一個關鍵的動作：**比對（Reconciliation）**。

- React 會拿伺服器傳來的 HTML 結構，跟客戶端 React 第一次準備接管時的 DOM 狀態進行屬性與標籤的逐一比對。
- 正常的狀況：兩者應該要 100% 吻合。
- 出問題的狀況（以 next-themes 為例）：
  - 伺服器傳下來的 HTML 是乾淨的：`<html lang="en">`。
  - 但為了防止畫面閃爍（FOUC, Flash of Unstyled Content），`next-themes` 會在 `<head>` 注入一段極快的同步 JavaScript 小腳本。這個腳本在瀏覽器解析 HTML 的瞬間（甚至在 React 啟動之前），就讀取了使用者的偏好，並立刻修改了 DOM：變成 `<html lang="en" class="dark" style="color-scheme: dark">`。
  - 當 React 幾毫秒後醒來準備接管時，它傻眼了：「奇怪？我伺服器紀錄的 HTML 明明沒有 `class="dark"` 和 `style`，為什麼現在的 DOM 有？」
  - 於是，React 為了安全起見，**認為「伺服器端與客戶端的狀態不一致」，直接在控制台噴出 Hydration Mismatch 警告**。

## suppressHydrationWarning 到底做了什麼？  
當你在標籤上加上 `suppressHydrationWarning={true}`（在 JSX 中簡寫為 `suppressHydrationWarning`）時，其實是向 React 發出一個明確的指令：

>「我知道這個元素的屬性（Attributes，例如 class、style）在伺服器端渲染完成後、到 React 啟動Hydration之前，可能會被外部的腳本或瀏覽器外掛動態修改。**請你在比對時，直接『放過』這個元素的屬性檢查，不要報錯，也不要試圖去覆蓋它。**」

### 特點
- 局部放行，而非全面關閉：它只針對該元素本身及其直接屬性關閉比對，不會影響整個應用的其他元件安全檢查。
- 信任客戶端狀態：它告訴 React：「如果伺服器和客戶端的屬性有落差，請直接以客戶端當下的狀態為準，不需要報警。」

## 為什麼這樣做是安全的？  
有些人會擔心：「關閉警告會不會隱藏其他嚴重的 Bug？」  
答案是：不會，只要用在對的地方。

- 不該用的地方：如果你是因為文字內容（Text Content）不一致（例如伺服器印出 Current time: 10:00，客戶端印出 10:01）而想加這個屬性，那是危險的，因為這代表你的業務邏輯或時間戳記有根本上的瑕疵。
- 適合用的地方：像 **「主題切換（Theme Mode）」、「瀏覽器外掛（如 Dark Reader、翻譯外掛）動態注入屬性」這種純粹跟視覺外觀、客戶端偏好相關的動態屬性，它是唯一且最標準的官方解法**。