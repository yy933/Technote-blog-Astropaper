---
title: "[Database] SQL 基礎：DDL(Data Definition Language) 與 DML(Data Manipulation Language) - 筆記"
pubDatetime: 2026-06-02T10:52:15.768Z
tags: ["database","SQL","cheatsheet"]
description: "Table of contents :memo: 簡介 DDL(Data Definition Language) 與..."
hackmd_id: "SJZqxE3gze"
---

## Table of contents

## :memo: 簡介  
DDL(Data Definition Language) 與 DML(Data Manipulation Language) 是 SQL（結構化查詢語言）家族中，依據「操作對象的不同」所劃分的兩大核心子集。

簡單用一個「蓋房子」的比喻來區分它們：

* DDL：負責「蓋房子、改格局」。決定這棟房子（資料庫）有幾層樓、每個房間（資料表）有多大、有哪些隔間（欄位）。**用來定義結構（如 `CREATE TABLE`, `DROP TABLE`）。**
* DML：負責「搬家具、住進去」。房間格局都固定了，它只負責在房間裡擺放家具、更換沙發、清除垃圾（資料列的操作）。**表格已經建立好，它只負責在裡面塞資料、改資料。**


## :memo: DDL (Data Definition Language) 

### 1. 定義與用途
* 定義：專門用來定義、修改、刪除資料庫結構（Schema）與資料表骨架的語言。
* 操作對象：資料庫（Database）、資料表（Table）、索引（Index）、欄位結構（Column）。
* 特性：執行後會直接改變資料庫的結構，通常無法輕易透過 ROLLBACK（復原）回到上一秒的狀態。

### 2. 常見語法與範例
#### CREATE — 建立資料庫或資料表結構
* 用途：在資料庫中從零開始打造一個全新的資料表。
* 範例：建立一張汽車資料表

```sql
CREATE TABLE cars (
    id SERIAL PRIMARY KEY,
    brand VARCHAR(50) NOT NULL,
    price INTEGER
);
```

#### ALTER — 修改既有的結構
* 用途：變更資料表的欄位、資料型態、或增刪限制條件。
* 範例：在 cars 表中加一個「顏色」欄位

```sql
ALTER TABLE cars
ADD COLUMN color VARCHAR(30);
```

#### TRUNCATE — 清空資料表
* 用途：保留資料表的結構，但一口氣斬斷、清空裡面所有的資料列。速度比 DELETE 快非常多。
* 範例：清空所有汽車資料但保留表格骨架

```sql
TRUNCATE TABLE cars;
```

#### DROP — 刪除整個結構
* 用途：將整張資料表或整個資料庫連根拔除，結構與裡面的所有資料會一起消失。
* 範例：直接刪掉整張汽車資料表

```sql
DROP TABLE cars;
```

## :memo: DML (Data Manipulation Language)

### 1. 定義與用途
* 定義：專門用來對資料表內部的具體數據（Data Rows）進行增、刪、查、改的語言。
* 操作對象：資料列（Rows）、內容物。
* 特性：也就是後端開發常說的 CRUD。它只會動到格子裡的數值，絕對不會影響資料表原有的結構和欄位。

### 2. 常見語法與範例

#### SELECT — 查詢 / 撈取資料 (Read)
* 用途：從資料表中找出符合條件的資料，並呈現出來。
* 範例：撈出所有 Toyota 的車子

```sql
SELECT * FROM cars 
WHERE brand = 'Toyota';
```

#### INSERT INTO — 新增資料 (Create)
* 用途：將一筆或多筆全新的資料列塞入指定的資料表中。
* 範例：新增一台藍色、價格 95 萬的 Ford 汽車資料

```sql
INSERT INTO cars (brand, price, color) 
VALUES ('Ford', 95, 'Blue');
```

#### UPDATE — 修改資料 (Update)
* 用途：更新或修改特定欄位裡的數值。通常會搭配 WHERE，否則會把整張表的數值全部改掉。
* 範例：把 Ford 車子的價格修改為 98 萬

```sql
UPDATE cars 
SET price = 98 
WHERE brand = 'Ford';
```

#### DELETE — 刪除特定的資料 (Delete)
* 用途：刪除符合條件的特定幾筆資料。
* 範例：刪除價格低於 50 萬的骨董車資料

```sql
DELETE FROM cars 
WHERE price < 50;
```

## 比較DDL和DML

| 特性 | DDL (資料定義語言) | DML (資料操作語言) |  
| :--- | :--- | :--- |  
| **主要操作對象** | **骨架** (Database / Table / Columns) | **血肉** (Data Rows / Values) |  
| **經典關鍵字** | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |  
| **對結構的影響** | 改變整個表格的結構。 | 不改變結構，僅欄位內的資料改變。 |  
| **執行時機** | 專案初始化、資料庫版本控制轉移 (Migration)。 | 網站日常運作、使用者與資料庫的互動。 |

## 小結
* 想要動到「直的欄位」、改限制條件、創表格 --> DDL。
* 想要動到「橫的資料」、撈報表、改特定數值 --> DML。

在實務開發中，如果使用的是 ORM（如 Prisma），日常程式碼（如 `db.user.create()`）大部分都在產生 DML；而當需要更新資料庫結構、執行 `prisma migrate dev` 時，ORM 就會自動產生並執行 DDL 語法喔！




<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>