---
title: "[Issues] Prisma migration卻找不到.env中的DATABASE_URL? - 筆記"
pubDatetime: 2025-09-18T20:58:02.000Z
modDatetime: 2026-05-25T10:04:23.268Z
tags: ["database","Prisma","PostgreSQL","Issue"]
description: "Table of contents 問題簡述 更改Prisma schema後，要進行資料庫migration，執行了..."
---

## Table of contents

## 問題簡述
更改Prisma schema後，要進行資料庫migration，執行了以下指令：
```bash
npx prisma migrate dev --name update_user_model
```
卻出現以下錯誤：
```bash
Loaded Prisma config from prisma.config.ts.

Prisma config detected, skipping environment variable loading.
Prisma schema loaded from prisma\schema.prisma
Error: Prisma schema validation - (get-config wasm)
Error code: P1012
error: Environment variable not found: DATABASE_URL.
  -->  prisma\schema.prisma:3
   |
 2 |   provider = "postgresql"
 3 |   url      = env("DATABASE_URL")
   |

Validation Error Count: 1
[Context: getConfig]

Prisma CLI Version : 6.15.0
```
找不到環境變數中的DATABASE_URL，無法正確連線資料庫，因此無法migrate。

## 問題排除
### 1. 首先先確定`.env`中有沒有`DATABASE_URL`，以及格式是否正確
```
DATABASE_URL="postgresql://username:password@localhost:5432/database_name"
NEXTAUTH_SECRET="xxxxx"
GOOGLE_CLIENT_ID="xxxxx"
GOOGLE_CLIENT_SECRET="xxxxx"
```
### 2. 檢查資料庫是否存在，以及是否連線
測試prisma是否連接到資料庫:
```
npx prisma db pull
```
如果正確連接，會顯示如以下訊息: (這裡db name是nextauthdb)
```
Loaded Prisma config from prisma.config.ts.

Prisma config detected, skipping environment variable loading.
Prisma schema loaded from prisma\schema.prisma
Datasource "db": PostgreSQL database "nextauthdb", schema "public" at "localhost:5432"
```
檢查container是否正在跑:`docker ps` 
```
CONTAINER ID   IMAGE         COMMAND                   CREATED      STATUS         PORTS                                         NAMES
a6482392b756   postgres:15   "docker-entrypoint.s…"   9 days ago   Up 5 seconds   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp   nextauth-postgres
```
看起來也沒問題。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

* 如果還沒建立資料庫，可以在docker建一個:
參考[利用Docker建立本地PostgreSQL - cheatsheet](https://hackmd.io/fX8YKgdmQsmpNt2FWGRVVQ)
* 或是在本地建一個:
連接到 PostgreSQL：`psql -U postgres`
創建資料庫和用戶：
 ```sql
CREATE DATABASE nextauthdb;
CREATE USER username WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE nextauthdb TO username;
```

</blockquote>

### 3. 測試資料庫連線
寫一個簡單的文件測試資料庫是否正確連線:
```js
// test-db.js
const { PrismaClient } = require('@prisma/client')

async function testConnection() {
  const prisma = new PrismaClient()
  
  try {
    await prisma.$connect()
    console.log('Database connection successful!')
  } catch (error) {
    console.error('Database connection failed:', error.message)
  } finally {
    await prisma.$disconnect()
  }
}

testConnection()
```
執行`node test-db.js`，如果成功連線會顯示 `Database connection successful!`

---

以上都沒問題了，再執行一次
```bash
npx prisma migrate dev --name update_user_model
```
卻還是出現同樣的錯誤訊息:
```bash
Loaded Prisma config from prisma.config.ts.

Prisma config detected, skipping environment variable loading.
Prisma schema loaded from prisma\schema.prisma
Error: Prisma schema validation - (get-config wasm)
Error code: P1012
error: Environment variable not found: DATABASE_URL.
  -->  prisma\schema.prisma:3
   |
 2 |   provider = "postgresql"
 3 |   url      = env("DATABASE_URL")
   |

Validation Error Count: 1
[Context: getConfig]

Prisma CLI Version : 6.15.0
```
仔細讀一下這個錯誤訊息:
```
Loaded Prisma config from prisma.config.ts.
Prisma config detected, skipping environment variable loading.
```
**Prisma 找到了 `prisma.config.ts` 文件，並且跳過了環境變數載入，所以讀不到 `.env` 文件中的 `DATABASE_URL`。**

目前的prisma.config.ts:
```ts
import { defineConfig } from 'prisma/config'

export default defineConfig({
  migrations: {
    seed: 'tsx prisma/seed.ts',
  },
})
```
* Prisma v5+ 引入了 prisma.config.ts 的概念
* 當存在該文件時，Prisma CLI 會優先使用它，並且預設不載入 `.env` 文件

## 可能的解決方法
1. 刪除prisma.config.ts
如果不需要特殊的 Prisma 配置，直接刪除 prisma.config.ts 文件：

```
# 在 Windows 中
del prisma.config.ts
```
2. 修改prisma.config.ts
如果要保留prisma.config.ts，需要在其中正確配置環境變數：
```
// prisma.config.ts
import { defineConfig } from 'prisma'
import dotenv from 'dotenv'

// 載入環境變數
dotenv.config({ path: '.env' })
// 如果使用 .env.local，改為：
// dotenv.config({ path: '.env.local' })

export default defineConfig({
  // 其他配置...
  schema: 'prisma/schema.prisma', //指定schema為prisma/schema.prisma文件
})
```
3. 直接指定 schema 文件，繞過配置文件：
`npx prisma migrate dev --name update_user_model --schema prisma/schema.prisma`

再執行一次migration，會看到類似這樣的訊息:
```
Prisma config detected, skipping environment variable loading.
Prisma schema loaded from prisma\schema.prisma
Datasource "db": PostgreSQL database "nextauthdb" at "localhost:5432"

Applying migration `20250918113839_update_user_model`

The following migration(s) have been created and applied from new schema changes:

prisma\migrations/
  └─ 20250918113839_update_user_model/
    └─ migration.sql

Your database is now in sync with your schema.

✔ Generated Prisma Client (v6.15.0) to .\node_modules\@prisma\client in 
 130ms
```
看一下migration紀錄:
```sql
-- AlterTable
ALTER TABLE "public"."User" ALTER COLUMN "name" DROP NOT NULL;
```
看一下prisma studio，也有正確更新:
![螢幕擷取畫面 2025-09-19 130218](/images/ryf1XP9ole.png)

這樣就成功migrate了!
<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>