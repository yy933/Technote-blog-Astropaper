---
title: "SQL：sqlite3的常用方法- 筆記"
pubDatetime: 2026-06-08T08:01:10.561Z
tags: ["database","Node.js","SQL","SQLite","Backend","cheatsheet"]
description: "Table of contents :memo: 認識 Node.js 中的 SQLite3 在 Node.js 中使..."
hackmd_id: "S1E2a14bzg"
---

## Table of contents

## :memo: 認識 Node.js 中的 SQLite3
在 Node.js 中使用 SQLite3 嵌入式資料庫時，我們通常會透過 sqlite3 或更現代化的 sqlite (Promise 基礎) 套件來操作資料庫。

不論使用哪種套件，與資料庫互動的核心方法都圍繞在特定的幾個 API。這些方法看似大同小異，但針對寫入、單筆查詢、多筆查詢等不同情境，選錯方法可能會導致效能低落或拿不到預期的回傳資料。


## :memo: 常用方法說明
### 1. `db.exec()`
* 功能：執行一條或多條 SQL 語句。它不支援參數綁定（Parameter Binding），純粹將傳入的字串當作 SQL 執行。
* 使用情境：**通常用於資料庫初始化**（例如：建立多個資料表、載入初始預設資料）。
* 舉例：

```javascript
const initSql = `
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL
  );
  CREATE TABLE IF NOT EXISTS posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL
  );
`;

db.exec(initSql, (err) => {
  if (err) console.error("初始化失敗:", err.message);
  else console.log("資料表建立成功！");
});
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

⚠️ 注意事項：

* 無回傳值：它不會回傳任何資料列（Rows）或受影響的行數。
* 不支援安全綁定：不能使用 `?` 帶入變數，容易引發 SQL 注入（SQL Injection）風險。絕對不要用它來處理使用者輸入的變數。

</blockquote>

### 2. `db.run()`
* 功能：執行一條 SQL 語句，支援參數綁定。
* 使用情境：用於資料變動的操作（非查詢類），例如：`INSERT`、`UPDATE`、`DELETE`。
* 舉例：

```javascript
const sql = `INSERT INTO users (name) VALUES (?)`;
const userName = "Alice";

// 使用 this 可以拿到執行後的狀態
db.run(sql, [userName], function(err) {
  if (err) return console.error(err.message);

  // 💡 關鍵屬性
  console.log(`成功插入資料！新產生的 ID 為: ${this.lastID}`);
  console.log(`受影響的資料筆數: ${this.changes}`);
});
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

⚠️ 注意事項：
* 無資料回傳：它不會回傳查詢結果的資料內容（如果對它下 `SELECT`，回傳值會是 `undefined`）。
* Context (this) 的陷阱：如果想要獲取 `this.lastID`（最後插入的自動遞增 ID）或 `this.changes`（更新/刪除的筆數），回呼函式絕對不能使用箭頭函式 (`() => {}`)，否則會綁定不到正確的 `this`。

</blockquote>

### 3. `db.get()`
* 功能：執行 SQL 查詢，並**只回傳符合條件的第一筆資料**。
* 使用情境：用於查**詢單一物件**，例如：依據特定 ID 尋找使用者、確認某個帳號是否存在。
* 舉例：

```javascript
const sql = `SELECT * FROM users WHERE id = ?`;
const userId = 1;

db.get(sql, [userId], (err, row) => {
  if (err) return console.error(err.message);

  // row 是一個單一的 JavaScript 物件，若找不到則為 undefined
  if (row) {
    console.log(`找到使用者：${row.name}`);
  } else {
    console.log("找不到該使用者");
  }
});
```

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

> `db.get()` 的參數是什麼？為什麼 userId 要加中括號 `[]`？

我們直接來看 `db.get(sql, params, callback)` 的標準結構：
* 第一個參數 (sql)：你要執行的 SQL 指令字串（內含 `?` 預留位置）。
* 第二個參數 (params)：這是一個陣列，裡面放著你要依序填入 SQL 中 `?` 的變數值。
* 第三個參數 (callback)：當資料庫處理完、撈到資料後，會自動執行的回呼函式（含有 err 和 row）。

>為什麼 `userId` 要加 `[]`？

在寫 SQL 時，為了防止 SQL 注入攻擊 (SQL Injection)，我們絕對不能用字串拼接的方式把變數塞進去（例如：`WHERE id = ${userId}` ❌ 這樣很危險）。

正確的做法是在 SQL 裡面留一個問號 `?`（預留位置，Placeholders），然後透過一個陣列，把真正的變數依序傳進去讓 SQLite 幫你安全地過濾並填入。

因為可能有複數個問號，例如：
`SELECT * FROM users WHERE status = ? AND age > ?`

這時候你的第二個參數就要按順序放入對應的值：
`['active', 18]`

實際寫出來像這樣:

```javascript
const sql = `SELECT * FROM users WHERE status = ? AND age > ?`;
const userStatus = 'active';
const userAge = 18;

// 執行 db.get()，回傳的 row 是一筆資料（物件）
db.get(sql, [userStatus, userAge], (err, row) => {
  if (err) return console.error(err.message);

  if (row) {
    console.log(`成功撈出符合條件的第一筆使用者！`);
    console.log(row); // 印出來會是：{ id: 3, name: 'Alice', status: 'active', age: 25 }
    console.log(`使用者的名字是：${row.name}`); 
  } else {
    console.log("資料庫中完全沒有符合條件的使用者。");
  }
});
```

所以，即使你今天只有一個變數 `userId`，因為 SQLite3 這個方法的語法規定「綁定變數必須用陣列帶入」，你就必須寫成 `[userId]`。

>那 `db.get()` 到底回傳什麼？

它透過 Callback 回傳的 row 永遠是個單一的 JavaScript 物件` { id: 1, name: 'Alice' }`（或是 `undefined`）。

</blockquote>

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

⚠️ 注意事項：
* 回傳型態：成功時回傳一個物件（Object），欄位名即為物件屬性。
* 效能優勢：即使資料庫裡有1萬筆符合條件的資料，它在撈到第一筆後就會立刻停止掃描，非常節省效能。

</blockquote>

### 4. `db.all()`
* 功能：執行 SQL 查詢，並回傳符合條件的所有資料。
* 使用情境：用於獲取資料列表，例如：撈出所有的產品清單、某個使用者發表過的所有文章。
* 舉例：

```javascript
const sql = `SELECT * FROM users`;

db.all(sql, [], (err, rows) => {
  if (err) return console.error(err.message);

  // rows 永遠是一個陣列（Array）
  console.log(`總共撈出 ${rows.length} 筆資料`);
  rows.forEach(row => console.log(row.name));
});
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

⚠️ 注意事項：
* 回傳型態：成功時回傳一個陣列（Array of Objects）。若完全沒有符合條件的資料，則回傳空陣列 `[]`（注意：不是 `undefined`）。
* 記憶體考量：它會把所有結果一次性全部載入到記憶體中。如果資料量高達數十萬筆，可能會引發記憶體耗盡（Out of Memory）的風險。

</blockquote>

### 5. `db.close()`
* 功能：關閉與資料庫的連線，釋放相關的系統資源。
* 使用情境：當應用程式準備關閉、指令碼執行完畢（如 Migration 腳本）時呼叫。
* 舉例：

```javascript
db.close((err) => {
  if (err) return console.error("關閉資料庫時發生錯誤:", err.message);
  console.log("資料庫連線已安全關閉。");
});
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

⚠️ 注意事項：
* 非同步安全：呼叫 `db.close()` 後，任何排隊中（佇列中）尚未執行完畢的 SQL 語句仍會被安全地執行完畢，之後才會真正關閉連線。關閉後如果再呼叫查詢，會丟出 `SQLITE_MISUSE` 錯誤。

</blockquote>

### 6. `db.each()`
* 功能：**執行查詢，並對符合條件的每一筆資料單獨觸發一次回呼函式。**
* 使用情境：當你需要處理海量資料（例如幾百萬筆），且需要逐行對資料進行加工處理或串流（Streaming）輸出，不想用 `db.all()`把記憶體撐爆時。
* 舉例：

```javascript
const sql = `SELECT id, name FROM users`;

// 參數 1: SQL 語句
// 參數 2: 綁定變數
// 參數 3: 每撈出一筆就執行一次的 callback
// 參數 4: 完整查詢結束後才執行的完成 callback (可選)
db.each(sql, [], 
  (err, row) => {
    if (err) throw err;
    console.log(`逐行處理：修改 ${row.name} 的資料...`);
  }, 
  (err, count) => {
    if (err) throw err;
    console.log(`全數處理完畢！總共處理了 ${count} 筆資料。`);
  }
);
```

### 7. `db.serialize()`
* 功能：**強制將區塊內的 SQL 語句改為順序性同步執行（Serialized Mode）**。
* 使用情境：**Node.js 預設是非同步的，有時你希望「先建表、再塞資料、最後查資料」，如果不做控制，這三個非同步操作會同時亂序執行導致報錯**。
* 舉例：

```javascript
db.serialize(() => {
  // 這裡面的語句會保證 1 跑完才跑 2，2 跑完才跑 3
  db.run("CREATE TABLE IF NOT EXISTS temp (val TEXT)"); // 1
  db.run("INSERT INTO temp VALUES (?)", ["Hello"]);     // 2
  db.get("SELECT val FROM temp", [], (err, row) => {     // 3
    console.log(row.val); 
  });
});
```

## :memo: 比較各方法的差異

| 方法 | 核心功能 | 適用場景 | 是否回傳結果資料？ |
| :--- | :--- | :--- | :--- |
| **`db.exec()`** | 執行多條大批量 SQL | 專案初始化、建立資料表 | ❌ 否 |
| **`db.run()`** | 執行單條異動語句 | `INSERT`, `UPDATE`, `DELETE` | ❌ 否 (僅透過 `this` 拿狀態) |
| **`db.get()`** | 查詢符合的第一筆 | 依 ID 查單一物件、檢查是否存在 | ⭕ 是 (`Object` / `undefined`) |
| **`db.all()`** | 查詢符合的全部 | 取得清單、報表（需注意記憶體） | ⭕ 是 (`Array` / `[]`) |
| **`db.each()`** | 逐行串流查詢 | 處理數百萬筆的海量數據 | ⭕ 是 (逐筆餵給 Callback) |





<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>