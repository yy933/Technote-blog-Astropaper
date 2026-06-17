---
title: "[Database] SQL JOIN - 筆記"
pubDatetime: 2026-06-02T08:40:24.032Z
tags: ["database","SQL"]
description: "Table of contents :memo: 簡介 在設計關聯式資料庫（RDBMS）時，為了避免資料重複性過高、造..."
hackmd_id: "Sy14Jzheze"
---

## Table of contents

## :memo: 簡介  
在設計關聯式資料庫（RDBMS）時，為了避免資料重複性過高、造成維護困難，我們通常會遵循 **正規化（Normalization）** 的原則，將資料拆分到不同的資料表中儲存。例如：將「汽車的基本車籍」與「車子的銷售紀錄」分別存在兩張獨立的表。

然而，當我們需要產生一份完整的商業報表（如：看哪一款車賣了多少錢）時，就必須將這些被拆散的表格重新串聯起來。這時候，**`JOIN`** 就是 SQL 中不可或缺的核心功能。透過 `JOIN`，我們能建立表格之間的橋樑，將片段的資訊拼湊成完整的數據。


## :memo: 情境  
本篇筆記將使用「汽車銷售系統」作為範例，介紹四種最常見的 `JOIN` 應用。

我們手邊有以下兩張關聯式資料表，它們透過 `cars_id` 與 `id` 相互對應：

### 1. 左表：`sold_cars`（銷售紀錄表）  
記錄已經賣掉的車子與成交價。*（注意：第三筆 `cars_id = 999` 為無效的車牌代號，在車籍資料中並不存在）*

| id | cars_id | sold_price |  
| :--- | :--- | :--- |  
| 1 | 101 | 70 萬 |  
| 2 | 102 | 90 萬 |  
| 3 | **999** | 50 萬 |

### 2. 右表：`cars`（車籍資料表）  
所有在庫車輛的明細。*（注意：第三筆 `id = 103` 的法拉利目前尚未售出，因此銷售紀錄表裡沒有它）*

| id | brand | model | price |  
| :--- | :--- | :--- | :--- |  
| 101 | Toyota | Altis | 75 萬 |  
| 102 | Ford | Focus | 95 萬 |  
| **103** | Ferrari| Roma | 1200 萬|


## :memo: 四種不同的 JOIN 實際範例  
在開始之前，我們要先釐清「左表」與「右表」的定義：
* 寫在 `JOIN` 關鍵字 **左邊** 的叫 **左表** （本範例中為 `sold_cars`）。
* 寫在 `JOIN` 關鍵字 **右邊** 的叫 **右表** （本範例中為 `cars`）。

---
### 1. INNER JOIN 
:bell: **核心思路**：只有兩張表 **皆同時存在、並且互相匹配** 的資料才會被撈出來。任何一邊對應不到，就會被直接剔除。

#### 範例 SQL：

```sql
SELECT C.brand, C.model, SC.sold_price 
FROM sold_cars SC
INNER JOIN cars C ON SC.cars_id = C.id;
```

#### 💡 產出結果：

| brand | model | sold_price |  
| :--- | :--- | :--- |  
| Toyota | Altis | 70 萬 |  
| Ford | Focus | 90 萬 |

* **解析**：因為 999 找不到車籍、103 找不到銷售紀錄，兩者皆不符合「同時存在」的條件，所以最終結果只會有完美的 2 筆匹配資料。

### 2. LEFT JOIN 
:bell: **核心思路**：以左表（sold_cars）為主角。左表的資料不論有沒有對應成功，全部都要留下來；右表若是對應不到，就自動補上 `NULL`。

#### 範例 SQL：

```sql
SELECT C.brand, C.model, SC.sold_price 
FROM sold_cars SC
LEFT JOIN cars C ON SC.cars_id = C.id;
```

#### 💡 產出結果：

| brand | model | sold_price |  
| :--- | :--- | :--- |  
| Toyota | Altis | 70 萬 |  
| Ford | Focus | 90 萬 |  
| NULL | NULL | 50 萬 |


* **解析**：第三筆銷售紀錄（`cars_id = 999`）因為是左表的資料，所以必須保留。但因為右表（cars）找不到它的車籍，所以 `brand` 和 `model` 就被貼心地補上了 `NULL`。

### 3. RIGHT JOIN 
:bell: **核心思路**：與 `LEFT JOIN` 相反，以右表（cars）為主角。右表的資料全部都要留下來；左表若是對應不到銷售紀錄，就補上 `NULL`。

#### 範例 SQL：

```sql
SELECT C.brand, C.model, SC.sold_price 
FROM sold_cars SC
RIGHT JOIN cars C ON SC.cars_id = C.id;
```

#### 💡 產出結果：

| brand | model | sold_price |  
| :--- | :--- | :--- |  
| Toyota | Altis | 70 萬 |  
| Ford | Focus | 90 萬 |  
| Ferrari | Roma | NULL |


* **解析**：這次右表的所有車子都被強制保留了。在庫的法拉利（Roma）因為還沒賣出去，在左表找不到對應的 `sold_price`，因此銷售金額顯示為 `NULL`。

### 4. FULL JOIN / FULL OUTER JOIN
:bell: **核心思路**：兩張表的所有資料通通都留下來。不管是左邊找不到右邊，還是右邊找不到左邊，只要對應不到的地方一律補 `NULL`。

#### 範例 SQL：

```sql
SELECT C.brand, C.model, SC.sold_price 
FROM sold_cars SC
FULL JOIN cars C ON SC.cars_id = C.id;
```

(註：部分資料庫如 MySQL 不直接支援 `FULL JOIN`，通常會使用 `LEFT JOIN` 與 `RIGHT JOIN` 進行 `UNION` 來模擬)

#### 💡 產出結果：

| brand | model | sold_price |  
| :--- | :--- | :--- |  
| Toyota | Altis | 70 萬 |  
| Ford | Focus | 90 萬 |  
| NULL | NULL | 50 萬 |  
| Ferrari | Roma | NULL |

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

在實際的資料庫運作中（不論是真正的 `FULL JOIN` 還是用 `LEFT JOIN UNION RIGHT JOIN` 模擬），資料的排序通常會依照主鍵（`ID`）或資料表讀取的先後順序。

既然前面提到了 `UNION`，在實務產出時，通常會是 `LEFT JOIN` 的結果在上方，緊接著合併 `RIGHT JOIN` 剩餘的資料。

</blockquote>


* **解析**：包含了錯填的 999 銷售紀錄，也包含了未售出的法拉利車籍，所有可能出現的狀況一次呈現。

## 小結  
綜合以上的實務應用：
* 想看完美匹配的精準報表 --> 用 `INNER JOIN`
* 想看完整銷售紀錄（包含異常數據） --> 用 `LEFT JOIN`
* 想看完整庫存與銷售狀況 --> 用 `RIGHT JOIN` 或 `FULL JOIN`

在業界實務開發中，**通常會優先使用 LEFT JOIN，因為人類「從左到右」的閱讀與思考習慣最符合它的邏輯。** 如果需要使用 RIGHT JOIN 的效果，通常只需要把 SQL 語句中的 A、B 表對調順序，再繼續用 LEFT JOIN 即可！







<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>