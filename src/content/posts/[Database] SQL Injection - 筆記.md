---
title: "[Database] SQL Injection - 筆記"
pubDatetime: 2026-06-03T04:57:55.212Z
tags: ["database","SQL","web attacks","web security"]
description: "Table of contents :memo: 簡介 SQL Injection（SQL 注入，簡稱 SQLi） 是..."
hackmd_id: "B18byE6eGx"
---

## Table of contents

## :memo: 簡介
**SQL Injection（SQL 注入，簡稱 SQLi）** 是網頁開發中最古老、最知名，同時也最危險的安全漏洞之一。

* **核心原理**：當網頁後端程式在處理使用者輸入的資料時，**沒有做好過濾與檢查**，就直接將輸入內容拼接到原生的 SQL 指令字串中。 **導致資料庫將攻擊者刻意輸入的「惡意程式碼」誤認為是「系統指令」而執行。**


## :memo: 三大常見的 SQL Injection 形式與範例  
為了方便說明，我們假設後端有一段**非常危險、使用字串拼接**的登入驗證程式碼：
```sql
-- 錯誤示範：極度危險的字串拼接寫法
SELECT * FROM users WHERE username = '輸入的帳號' AND password = '輸入的密碼';
```

### 1. 繞過身份驗證（Authentication Bypass）
* 核心思路：攻擊者不需要知道任何人的密碼，就能直接以特定（例如管理員）身分登入。
* 攻擊手法：
  - 帳號輸入框：`'admin' OR '1'='1'`
  - 密碼輸入框：隨便填（如 `123`）

* 資料庫實際執行的 SQL：

```sql
SELECT * FROM users WHERE username = 'admin' OR '1'='1' AND password = '123';
```

* 解析：因為` '1'='1'` 在邏輯判斷中永遠是 `TRUE`。在 `OR` 的作用下，後半段的密碼檢查直接被資料庫忽略了。資料庫會判定這行指令成立，並直接回傳 `admin` 的帳號資料，攻擊者順利繞過密碼鎖登入。

### 2. 聯集注入（UNION-based SQLi）
* 核心思路：當網頁會將 SQL 查詢結果直接顯示在畫面上時（如：商品搜尋頁、部落格文章頁），攻擊者利用 `UNION `關鍵字，強行把「另一張敏感資料表」的內容合併進去一起顯示。
* 攻擊手法：
  - 正常網址：`https://example.com/books?id=101`
  - 攻擊網址：`https://example.com/books?id=101 UNION SELECT username, password FROM users`

* 資料庫實際執行的 SQL：

```sql
SELECT title, price FROM books WHERE id = 101 
UNION 
SELECT username, password FROM users;
```

### 3. 盲注（Blind SQL Injection）
* 核心思路：最狡猾的手段。網頁不會顯示任何錯誤訊息，也不會把資料庫內容噴在畫面上。網頁只會依據查詢成功與否，顯示「有這筆資料（True）」或「找不到資料（False）」。
* 常見分類：
  - 布林盲注（Boolean-based）：攻擊者會像玩猜數字一樣，不斷拷問資料庫。例如輸入：`101 AND (SELECT SUBSTRING(password, 1, 1) FROM users WHERE username='admin') = 'a'`。如果網頁正常顯示，代表密碼第一個字是 `a`；若是顯示找不到，就繼續猜 `b`、`c`...。
  - 時間盲注（Time-based）：如果網頁連 True/False 都不明顯，攻擊者會下包含 `SLEEP(5)` 的指令。如果網頁轉圈圈轉了 5 秒才載入完成，代表猜對了。攻擊者會利用自動化腳本，花時間把資料一字一字「擠」出來。

## :memo: 如何有效防範 SQL Injection？  
防範 SQLi 的至高防護原則：**「絕對不要相信使用者的任何輸入，永遠不要直接拼接 SQL 字串。」**

### 1. 使用參數化查詢（Parameterized Queries）(最重要!)  
這是根治 SQL Injection 最完美的防禦方法。它會先把 SQL 的「骨架命令」送給資料庫編譯，此時指令的邏輯已經固定。隨後使用者輸入的內容，只會被當成單純的「參數數值」填入，絕對不可能動搖到 SQL 的結構。

❌ 錯誤寫法（字串拼接）：

```javascript
const query = `SELECT * FROM users WHERE id = ${req.body.id}`; 
```

💡 正確寫法（參數化預編譯）：

```javascript
const query = 'SELECT * FROM users WHERE id = ?';
db.execute(query, [req.body.id]); 
```

(註：此時即便使用者輸入 `101 OR 1=1`，資料庫也只會去尋找 `id` 欄位剛好叫做 `"101 OR 1=1"` 的使用者，不會執行` OR` 的邏輯)

### 2. 使用現代的 ORM 框架  
如果在開發時使用的是**現代的 ORM（如 Prisma、Sequelize、Mongoose 等），這些框架在底層處理 CRUD（如 `prisma.user.findUnique()`）時，預設就已經全部做好了參數化查詢**。

⚠️ 注意例外：即便使用了 ORM，也千萬不要去調用類似 `prisma.$queryRaw( SELECT * FROM ... ${input} )` 這種允許你傳入原生拼接字串的方法，否則漏洞依然存在。

### 3. 嚴格的輸入驗證與型態檢查（Input Validation）  
如果某個欄位明明是數字（例如商品 ID、經銷商 ID），**在後端收到請求的第一時間就應該強迫轉型或驗證，確保不合法的惡意 SQL 字串在進入資料庫查詢前就被攔截**：

```javascript
const dealershipId = parseInt(req.body.dealership_id, 10);
if (isNaN(dealershipId)) {
    return res.status(400).send('格式錯誤');
}
```

### 4. 最小權限原則（Principle of Least Privilege）
**在正式上線（Production）環境中，網頁程式連線資料庫的帳號，絕對不要使用 root 或 sa（超級管理員）。**
**應該針對該專案建立一個專用帳號，並且只賦予它操作特定資料表的 `SELECT`, `INSERT`, `UPDATE` 權限。** 這樣即使網站不幸被攻破，駭客也無法利用 SQLi 去刪除其他系統資料表或關閉整個資料庫。

## 總結防禦清單

| 防護層級 | 具體手段 | 效果說明 |  
| :--- | :--- | :--- |  
| **第一道防線 (核心)** | 參數化查詢 (Prepared Statements) / 現代 ORM | **完全阻斷**惡意字串被當成指令執行的可能性。 |  
| **第二道防線 (前端/路由)** | 嚴格型態檢查 (`parseInt`)、輸入驗證 | 在惡意資料接觸到資料庫之前就將其**彈回並拒絕**。 |  
| **第三道防線 (系統層)** | 最小權限原則 (不使用 root 連線) | 即使網站不幸被攻破，也能**將災害控制在最小範圍**。 |

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>