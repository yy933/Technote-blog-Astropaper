---
title: "[Database] SQL：UNION - 筆記"
pubDatetime: 2026-06-04T04:46:20.853Z
tags: ["database","SQL"]
description: "Table of contents :memo: UNION 是什麼？ 在 SQL 的世界裡，當我們想要把多個查詢結果..."
hackmd_id: "BJVDA_CgMx"
---

## Table of contents

## :memo: UNION 是什麼？  
在 SQL 的世界裡，當我們想要把多個查詢結果整合在一起時，除了常見的 `JOIN` 之外，另一個核心強者就是 **`UNION`（聯集）**。

* **核心功能**：用來合併兩個或多個 `SELECT` 語句的結果集。
* **合併方向**：屬於 **「縱向合併」**，也就是把不同的查詢結果 **由上而下地堆疊** 在一起，讓資料列（Rows）變多。

###  UNION 的使用時機與三大限制  
當我們有**結構極度相似、但因分類或歷史原因分開儲存**的資料表（例如：國產車表與進口車表、今年訂單表與歷史訂單表），需要將它們整理成單一清單呈現時。

使用 `UNION` 時，多個 `SELECT` 語句必須符合以下**限制**，否則資料庫會直接報錯：  
1. 每個 `SELECT` 語句中的**欄位數量必須相同**。  
2. 每個 `SELECT` 語句中對應欄位的**資料型態（Data Type）必須相容**。  
3. 每個 `SELECT` 語句中的**欄位順序必須一致**。


## :memo: UNION 基本語法與範例

假設我們有兩張汽車資料表，分別是 `domestic_cars`（國產車）與 `imported_cars`（進口車），兩者都擁有 `brand` 與 `model` 欄位。

### 1. `UNION` — 自動去除重複（預設）
* **用途**：合併結果，且**自動過濾掉重複的資料**（背後會自動執行 `DISTINCT` 運算）。

```sql
SELECT brand, model FROM domestic_cars
UNION
SELECT brand, model FROM imported_cars;
```

### 2. `UNION ALL` — 保留所有重複資料
* **用途**：單純將兩份資料上下拼起來，不過濾掉重複的資料。
* **特性**：執行速度比 `UNION` 快非常多！ 因為資料庫不需要耗費記憶體與效能去逐行檢查、比對資料有沒有重複。實務上如果確定資料不會重複，或本來就需要重複資料，強烈建議優先使用 `UNION ALL`。

```sql
SELECT brand, model FROM domestic_cars
UNION ALL
SELECT brand, model FROM imported_cars;
```


## :memo: UNION 和 JOIN 的核心差別

`UNION` 和 `JOIN` 的本質差別在於「延伸的方向」：

* `JOIN` 是「左右拼接（橫向）」：把兩張表不同的欄位黏在一起，**通常需要透過 `ON` 的關聯欄位（外鍵）來對齊。**
* `UNION` 是「上下堆疊（縱向）」：**把格式一模一樣的資料往下延伸，不需要外鍵關係**。

| 特性 | JOIN  | UNION  |  
| :--- | :--- | :--- |  
| **合併方向** | **橫向合併（左右拼接）** | **縱向合併（上下堆疊）** |  
| **資料改變** | **增加欄位（Columns 變多）** | **增加資料列（Rows 變多）** |  
| **合併條件** | 兩張表通常需要有**關聯的外鍵**（如 `ON A.id = B.id`）。 | 兩張表的**欄位數量、順序與資料型態必須完全一致**。 |

### 範例情境對比
#### 使用 JOIN 的情境（欄位擴展）  
「我想知道汽車（`cars` 表）的車款，以及它在哪一家經銷商（`dealerships` 表）賣出。」

```sql
SELECT c.model, d.city
FROM cars c
INNER JOIN dealerships d ON c.dealership_id = d.id;
-- 結果：往右延伸出了 city 欄位
```

#### 使用 UNION 的情境（資料列擴展）  
「我想把台北店的汽車名單，和高雄店的汽車名單，合併成一份大清單。」

```sql
SELECT model FROM cars WHERE city = 'Taipei'
UNION ALL
SELECT model FROM cars WHERE city = 'Kaohsiung';
-- 結果：往下一口氣吐出兩店合併的長長資料列
```

## :memo: 進階應用：用 UNION 模擬 FULL JOIN

在 SQL 標準中，`FULL JOIN` 可以列出左右兩邊所有的資料（沒匹配到的補 `NULL`）。然而，MySQL 資料庫原生是不支援 `FULL JOIN` 語法的！

解決方案：我們可以利用 `UNION` 的去除重複特性，將 `LEFT JOIN` 與 `RIGHT JOIN` 的結果集上下拼接。這樣既能拿到左表全部、也能拿到右表全部，且重疊的部分會被自動過濾，完美模擬出 `FULL JOIN`！

### 模擬 FULL JOIN 的標準語法

```sql
-- 1. 先抓出左表所有的資料 (包含右表為 NULL 的情況)
SELECT c.model, d.city
FROM cars c
LEFT JOIN dealerships d ON c.dealership_id = d.id

UNION -- 2. 利用 UNION 自動去重的特性進行縱向合併

-- 3. 再抓出右表所有的資料 (包含左表為 NULL 的情況)
SELECT c.model, d.city
FROM cars c
RIGHT JOIN dealerships d ON c.dealership_id = d.id;
```

## 小結
* 想要擴充資訊，把「不同維度的欄位（左右）」拼起來 -->  `JOIN`
* 想要合併清單，把「格式相同的資料（上下）」疊起來 -->  `UNION`
* 在使用 `UNION` 時，永遠在心中衡量一下是否需要去重。如果不需要，改用 `UNION ALL` 能幫 API 與資料庫省下驚人的運算時間！


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>