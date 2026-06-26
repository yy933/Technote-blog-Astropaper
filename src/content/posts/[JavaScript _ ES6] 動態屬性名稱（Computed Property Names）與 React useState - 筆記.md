---
title: "[JavaScript / ES6] 動態屬性名稱（Computed Property Names）與 React useState - 筆記"
pubDatetime: 2026-06-26T05:08:05.497Z
tags: ["JavaScript","React.js","React Hook"]
description: "Table of contents :memo: 簡介 在 React 開發中，處理表單欄位（Form Inputs）..."
---

## Table of contents

## :memo: 簡介  
在 React 開發中，處理表單欄位（Form Inputs）是極為常見的需求。傳統上，如果一個表單有數個不同的輸入框，我們可能需要為每個輸入框獨立撰寫一套事件處理函式（Event Handler）。當表單欄位增加時，程式碼就會變得非常冗長且難以維護。

為了提升程式碼的**重用性（Reusability）**並**精簡邏輯**，我們可以結合 JavaScript 的核心特性 **「動態屬性名稱（Computed Property Names）」** 與 React 的 **`useState` Hook**。這樣一來，無論表單有多少欄位，都只需要用「單一一個函式」就能完美搞定所有輸入值的更新。


## :memo: 情境  
假設我們正在開發一個「迷因產生器（Meme Generator）」，頁面上有兩個輸入框：  
1. **Top Text**（上排文字輸入框，HTML `name="topText"`）  
2. **Bottom Text**（下排文字輸入框，HTML `name="bottomText"`）

我們希望使用者在任一輸入框打字時，畫面上對應的位置能即時更新文字。

如果不用動態屬性名稱，我們就必須寫 `handleTopTextChange` 和 `handleBottomTextChange` 兩個幾乎一模一樣的函式。但透過 `[name]: value` 的語法，我們可以將它們合併成一個萬用的 `handleChange`。

## :memo: 核心觀念解析

在實作之前，必須先釐清兩個關鍵的 React 與 JavaScript 底層觀念：

### 1. `useState` 的本質到底是什麼？
* **`useState` 本身是一個「函式」（Function）**。
* **所謂的 Hook 都是函式**：在 React 中，只要是 `use` 開頭的 Hook，本質上都是 JavaScript 函式，用來讓函式組件可以「鉤進（Hook into）」React 的核心狀態或生命週期。
* **它的回傳值是「陣列」**：呼叫 `useState(初始值)` 後，它會吐回一個剛好有兩個元素的陣列。
  * 索引 `0`：當前的狀態值（State）
  * 索引 `1`：用來更新狀態的更新函式（Setter Function）
* **陣列解構賦值**：我們使用 `const [meme, setMeme] = useState(...)` 的語法，就是利用 JavaScript 的陣列解構，把這兩個元素拆包出來並自由命名。

### 2. 為什麼更新物件狀態時，Key 要加中括號 `[name]`？  
這裡的 `[name]` 與陣列無關，它是 JavaScript 在 ES6 推出的 **「動態屬性名稱（Computed Property Names）」**。

* **不加中括號**：JavaScript 會死板地將它代入為字串 `"name"`。
* **加上中括號**：告訴 JavaScript **「請把 `name` 當成變數來讀取它的值」**。當 `name` 變數的值是 `"topText"` 時，`[name]` 就會自動解讀成 `"topText"`；當變數是 `"bottomText"` 時，就會自動解讀成 `"bottomText"`。


## :memo: 實作分享

### 1. **定義複雜狀態（Object State）**
   在 React 元件中，將多個關聯的欄位包在同一個物件狀態中：
   
```javascript
   const [meme, setMeme] = useState({
       topText: "One does not simply",
       bottomText: "Walk into Mordor",
       imageUrl: "[http://i.imgflip.com/1bij.jpg](http://i.imgflip.com/1bij.jpg)"
   })
   ```
   
### 2. 撰寫萬用萬能的事件處理函式  
利用解構賦值拿到 `input` 觸發事件的 `name` 與 `value`，並使用 `[name]` 動態更新：

```javascript
function handleChange(event) {
    // 從發出事件的 input 標籤中，取出它的 name 屬性與當前輸入的 value
    const { value, name } = event.currentTarget

    // 更新狀態：傳入一個回呼函式確保拿到最即時的舊狀態 (prevMeme)
    setMeme(prevMeme => ({
        ...prevMeme,        // 1. 先用展開運算子複製舊物件的所有內容
        [name]: value       // 2. 根據變數 name 的值，動態覆蓋對應的屬性！
    }))
}
```

### 3. 完整元件程式碼範例

```javascript
import { useState } from "react"

export default function Main() {
    const [meme, setMeme] = useState({
        topText: "One does not simply",
        bottomText: "Walk into Mordor",
        imageUrl: "[http://i.imgflip.com/1bij.jpg](http://i.imgflip.com/1bij.jpg)"
    })

    function handleChange(event) {
        const { value, name } = event.currentTarget
        setMeme(prevMeme => ({
            ...prevMeme,
            [name]: value
        }))
    }

    return (
        <main>
            <div className="form">
                <label>Top Text
                    <input
                        type="text"
                        placeholder="One does not simply"
                        name="topText"       // 👈 對應狀態的 key
                        onChange={handleChange}
                        value={meme.topText}
                    />
                </label>

                <label>Bottom Text
                    <input
                        type="text"
                        placeholder="Walk into Mordor"
                        name="bottomText"    // 👈 對應狀態的 key
                        onChange={handleChange}
                        value={meme.bottomText}
                    />
                </label>
                <button>Get a new meme image 🖼</button>
            </div>
            <div className="meme">
                <img src={meme.imageUrl} alt="Meme base" />
                <span className="top">{meme.topText}</span>
                <span className="bottom">{meme.bottomText}</span>
            </div>
        </main>
    )
}
```

## 小結  
透過 `useState` 搭配物件、HTML 的 `name` 屬性以及 JavaScript 的 `[name]` 動態屬性語法，我們成功將原本可能需要好幾個處理函式的表單精簡成了一個。這不僅符合 DRY（Don't Repeat Yourself）的乾淨程式碼原則，也是 React 處理複雜表單、受控組件（Controlled Components）時最推薦的標準起手式！

## 參考資料
* [MDN Web Docs - Object initializer (Computed property names)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer)
* [React Rules of Hooks 官方文件](https://react.dev/reference/rules/rules-of-hooks)
* [React Docs - Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure)


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>