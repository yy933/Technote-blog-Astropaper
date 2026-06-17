---
title: "[CSS] [Grid] auto-fit vs auto-fill - 筆記"
pubDatetime: 2025-05-05T19:56:54.000Z
modDatetime: 2026-05-25T10:04:23.638Z
tags: ["CSS","Interview Preparation"]
description: "Table of contents 語法 都是搭配 repeat() 使用： gridtemplatecolumns:..."
hackmd_id: "S1X-qLj0ye"
---

## Table of contents




## 語法
都是搭配 `repeat()` 使用：
`grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));`



## 比較

| 特性       | `auto-fit`                            | `auto-fill`                           |
|------------|----------------------------------------|----------------------------------------|
| 空格處理   | 🔄 自動壓縮、**隱形收合空格**          | ⬜️ 空格**會保留，占版面**             |
| 視覺效果   | 卡片會自動撐滿整行                     | 即使空了，也會保持「看似可以擺放」的空格 |
| 常見用途   | ✅ 自適應、可收合卡片 → _RWD排版_  | ✅ 固定格線、占位用 → _表格、表單排版_  |

## 實際範例差異


```css
/* 自動壓縮空格，不夠就合併（最常用） */
grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));


/* 保留空格，常用於要等資料補上時 */
grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
```
`grid-template-columns: repeat(auto-fit, minmax(200px, 1fr))`:
這個語法對瀏覽器的要求是，如果空間有多，但多不到 200px，不足以多畫一格的話，現有的格子會以 max 值所設定的 1fr 來計算最終寬度；一旦多的空間達到 200px 時，就再多畫一格，此時，每個欄位皆為 200px。 

直接看圖的範例：
* 寫死的狀況：直接定義要三個欄位，每格寬度 100px。
![ExportedContentImage_00](/images/SktdpLi0Jx.png)

* `auto-fill`： 想辦法讓 container 列上放滿越多欄位越好，因此當 container 列上還有多餘空間再放一格寬度為 100px 的欄位時，會自動產生空欄位。
![ExportedContentImage_01](/images/BkAqTUsCJx.png)

* `auto-fit`：與 `auto-fill` 類似，但自動產生的空欄位，也就是沒有內容的部分，會被收起來 (寬度變成 0px) 。
![ExportedContentImage_02 (1)](/images/HJR3aIjRJe.png)

## 搭配 `minmax()`

### auto-fill 搭配 minmax()
```css
.container {
  grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
}
```
在 300px 寬的小螢幕中，畫面會是剛好三格:
![ExportedContentImage_05 (1)](/images/BJqRA8sAkg.png)
假定畫面為 360px，三個格子的寬度會改以 1fr 來計算，每個格子的寬度最終會是 360 / 3=120px:
![ExportedContentImage_06 (2)](/images/SkQekDoRJe.png)
畫面繼續被拉寬，來到 400px，會自動產生一個寬度 100px 的新格子，但因為沒有內容，是一個空格:
![ExportedContentImage_07 (1)](/images/BkDZ1Po0Jx.png)


### auto-fit 搭配 minmax()
```css
.container {
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
}
```
在 300px 寬的小螢幕中，畫面也是剛好三格：
![ExportedContentImage_09](/images/HJfByvoRJg.png)
假定畫面為 360px，三個格子的寬度會改以 1fr 來計算--每個格子的寬度最終會是360 / 3=120px。和 auto-fill 相同:
![ExportedContentImage_10](/images/r1xPyDi0Jx.png)
當畫面繼續被拉寬，每多出 100px 時雖然一樣會自動產生新格子，但凡是沒有內容的格子都會被收起來，寬度變為 0px。也就是說，現有的三個欄位在計算寬度所使用的 1fr 會因為剩餘空間變大而增加：
![ExportedContentImage_11](/images/By-t1wsAyl.png)

## 簡單記憶法
* `auto-fit`：像彈性書櫃，書少時會自動把格子壓縮掉，其他書自動填滿整行。簡單來說，**就是請瀏覽器計算這一列可以擺幾格，盡量擺好擺滿**。
* `auto-fill`：像固定格子的書架，就算沒書也會留空白格在那裡。

## 小建議
✅ 想要「每列滿版卡片，數量自動調整」→ 用 `auto-fit`
✅ 想要「格子固定排版，但有些可能沒資料」→ 用 `auto-fill`



<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>