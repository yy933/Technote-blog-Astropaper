---
title: "[Database] 資料庫的核心防禦機制：Transaction、Commit 與 Rollback - 筆記"
pubDatetime: 2026-06-09T07:31:34.867Z
tags: ["database","Node.js","Redis","MongoDB","NoSQL","SQL","SQLite","Backend"]
description: "Table of contents :memo: 前言 在後端開發中，確保資料的安全與完整性是首要任務。想像一下，如果..."
hackmd_id: "rywGnVBZGe"
---

## Table of contents

## :memo: 前言
在後端開發中，確保資料的**安全**與**完整性**是首要任務。想像一下，如果使用者在電商網站下單，系統已經扣了使用者的錢，卻在更新商品庫存時當機，這將會是一場災難。

為了防止這種「半殘」的異常狀態，資料庫提供了最強大的安全防護罩：**Transaction（交易）** 機制。

## :memo: 核心概念

資料庫的交易機制主要由以下三個關鍵指令組成，它們遵循 **ACID** 中的 **原子性 (Atomicity)** —— 「**要嘛全部成功，要嘛全部失敗 (All or Nothing)**」。

| 指令 | 中文名稱 | 實際角色 | 核心功能 |
| :--- | :--- | :--- | :--- |
| **`BEGIN TRANSACTION`** | 開始交易 | **安全防護罩** | 開啟一個臨時的暫存區，接下來的所有寫入動作都不會立刻影響真正的資料。 |
| **`COMMIT`** | 確認提交 | **按下儲存鍵** | 當所有步驟都正確無誤時發出。資料庫此時才會將暫存區的修改**正式且永久**地寫入硬碟。 |
| **`ROLLBACK`** | 回滾復原 | **萬能上一步 (Ctrl+Z)** | 當過程中發生任何錯誤（異常、斷電、邏輯失敗）時發出。立刻**作廢**暫存區內所有操作，恢復到交易開始前的狀態。 |


## :memo: SQL 實際範例：購物車結帳


### 情境說明
一個完整的結帳流程包含三個步驟：**建立訂單 ➡️ 扣減庫存 ➡️ 扣除會員餘額**。這三個步驟彼此環環相扣，必須被打包在同一個 Transaction 中。


### 完整程式碼範例 (Node.js + SQLite)

以下展示如何在實務上利用 `try...catch...finally` 結構來完美實作 Transaction 控制：

```javascript
import * as sqlite3 from "sqlite3";
import { open } from "sqlite";
import path from "node:path";

async function checkout() {
  const db = await open({
    filename: path.join(process.cwd(), "database.db"),
    driver: sqlite3.Database,
  });

  try {
    // 1. 啟動交易防護罩 Transaction
    await db.exec("BEGIN TRANSACTION");

    // 【步驟 A】建立訂單
    await db.run("INSERT INTO orders (user_id, total) VALUES (1, 1000)");
    
    // 【步驟 B】扣除商品庫存（假設庫存不足，此處會拋出 Error）
    await db.run("UPDATE products SET stock = stock - 1 WHERE id = 99");

    // 【步驟 C】扣除使用者帳戶餘額
    await db.run("UPDATE users SET balance = balance - 1000 WHERE id = 1");

    // 2. 順利走完 A, B, C 三個步驟，確認沒問題！
    // 正式將所有變更寫入硬碟
    await db.exec("COMMIT");
    console.log("🎉 結帳成功！資料已安全存檔。");

  } catch (error) {
    // 3. 【核心防禦】只要步驟 A, B, C 任何一步噴出錯誤
    // 立刻啟動防禦機制：全部回滾！
    await db.exec("ROLLBACK");
    console.error("❌ 結帳失敗！資料庫已安全恢復原狀。錯誤訊息：", error.message);

  } finally {
    // 4. 無論成功或失敗，最後一定要關閉連線，釋放資源
    await db.close();
    console.log("🔌 資料庫連線已關閉。");
  }
}

checkout();
```

## NoSQL 的防禦機制
現代主流的 NoSQL 資料庫也有類似機制，但本質與思維大不相同。

早期的 NoSQL（如 MongoDB、Redis）為了追求極致的效能與橫向擴充，捨棄了複雜的 Transaction。但隨著企業需求演進，現代 NoSQL 已紛紛補強此功能。

### MongoDB (文件型 NoSQL)
MongoDB 在 4.0 版本後已完全支援多文件（Multi-document）的 ACID 交易，其操作邏輯與 SQL 極其相似（有 Start、Commit、Abort）：

```javascript
const session = client.startSession();
try {
  session.startTransaction(); // 開始交易
  
  await db.collection('orders').insertOne({ user_id: 1, total: 1000 }, { session });
  await db.collection('products').updateOne({ _id: 99 }, { $inc: { stock: -1 } }, { session });
  
  await session.commitTransaction(); // 確認提交
} catch (error) {
  await session.abortTransaction(); // 發生錯誤，回滾復原 (等同 Rollback)
} finally {
  await session.endSession();
}
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

雖然 MongoDB 支援Transaction，但實務上很少使用。
因為 MongoDB 允許將訂單與商品明細直接嵌套成同一個 JSON 文件。
**MongoDB 保證「對單一文件的修改天生具備原子性」**，因此透過良好的資料結構設計，往往就能避開昂貴的 Transaction 消耗。

</blockquote>

### Redis (記憶體快取資料庫)
Redis 的交易機制使用 `MULTI`（開始排隊）與 `EXEC`（批次執行）。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

⚠️ 致命限制：**Redis 不支援 Rollback！** 如果在 `EXEC` 執行一連串指令時，中途某個指令出錯，Redis 會繼續執行完後續指令，且前面已經成功的指令無法被撤銷。
**這是為了追求極致速度所做的架構妥協。**

</blockquote>

### Firebase Firestore (雲端 NoSQL)
採用 **樂觀鎖 (Optimistic Concurrency Control) 機制**。

* 它在寫入時不會鎖定資料庫，而是會在送出時檢查「這段時間內資料有沒有被別人動過」。
* 若被動過，則不進行 Rollback，而是自動重新嘗試（Retry） 整個流程，直到成功或超時為止。

## SQL vs NoSQL 防禦系統比較

| 特性 | SQL 資料庫 (如 SQLite, Postgres) | NoSQL 資料庫 (如 MongoDB) |
| :--- | :--- | :--- |
| **交易支援** | 天生自帶，屬於底層架構的核心。 | 後天補強，現代版本才支援。 |
| **效能影響** | 開啟 Transaction 會鎖定資料，高併發時可能稍微變慢。 | 跨文件 Transaction 效能消耗極大，官方不建議頻繁使用。 |
| **設計哲學** | 依賴資料庫的 Transaction 機制來確保跨表安全。 | 依賴良好的 Schema 設計（嵌套結構），盡量在單一文件內搞定，避開交易。 |

## 關鍵思維與開發細節
### 為什麼在寫入複數資料時，一定要用 `for...of` 迴圈而非 `.map()`？

在非同步環境中，`array.map()` 沒辦法等待內部的 `await`。它會同時發出多個獨立非同步請求（回傳滿滿的 Promise 陣列），導致外部的 `try...catch` 無法精準捕捉到某一筆寫入失敗的錯誤，進而讓 `ROLLBACK` 機制完全失效。

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

⚠️Tips：
在 Transaction 內部處理迴圈寫入時，務必使用 `for...of`，確保資料是「一筆接一筆」依序且同步地執行，才能完美觸發安全回滾。

</blockquote>


## 小結
不論是 SQL 還是 NoSQL，Transaction 的精髓可以用一句話概括：

**「在沒有按下 COMMIT 之前，所有的承諾都是暫時的；只要一有不對勁，ROLLBACK 是你最強大的後盾。」**

掌握這套機制與跨資料庫的思維差異，後端系統才真正具備了應對現實世界各種突發狀況的Robustness。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>