---
title: "Prisma整合Next.js：建立使用者註冊與登入基本流程 - cheatsheet"
pubDatetime: 2025-09-09T23:35:28.000Z
tags: ["Next.js","React.js","database","authentication","Docker","PostgreSQL","Prisma"]
description: " Table of contents 在[上一篇](https://hackmd.io/fX8YKgdmQsmpNt2F..."
---

## Table of contents


在[上一篇](https://hackmd.io/fX8YKgdmQsmpNt2FWGRVVQ)中我們利用 Docker 建立了本地 PostgreSQL 並成功連線，這一篇將示範如何使用 Prisma 來操作資料庫，並建立基本的 Email 註冊 / 登入流程。

## 什麼是Prisma

Prisma 是一個 **TypeScript/JavaScript 的資料庫工具集**，主要功能包含：

* **Prisma Client**：自動生成型別安全（type-safe）的資料庫 client，用 JS/TS 直接操作資料庫。
* **Prisma Migrate**：用來做 schema 定義與 migration，能夠版本化資料庫結構。
* **Prisma Studio**：一個簡單的 GUI，可以直接檢視/編輯資料。
輸入`npx prisma studio`開啟瀏覽器介面快速檢查資料，很方便。

Prisma 在[官方文件](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma)中被稱為 Next-generation Node.js and TypeScript ORM。
不過要注意，**Prisma 跟傳統 ORM（像 Sequelize、TypeORM）有點差別**：

* 傳統 ORM：強調物件導向（Object ↔ Table 映射），例如你定義一個 User class，就會直接對應資料庫的 users table。
* Prisma：**強調 schema-first**，你會先寫 schema.prisma 來定義資料表，Prisma 再幫你生成型別安全的 client。

👉 所以**有些人會說 Prisma「不是傳統意義上的 ORM」**，但 Prisma 自己文件中仍然定位為 ORM，只是做法更現代化。

| 傳統 ORM                   | Prisma                           |
| ------------------------ | -------------------------------- |
| 物件導向 (Object ↔ Table 映射) | schema-first，依據 schema 產生 client |
| 手動寫一堆 model class        | 自動生成 TypeScript 型別               |
| 常見：Sequelize, TypeORM    | Prisma                           |


## 為什麼 Prisma 特別適合 Next.js + NextAuth.js

### NextAuth 官方推薦

* NextAuth[官方文件Adapters](https://authjs.dev/getting-started/adapters/prisma)提到，Prisma 是最常見、最完整的 Adapter 實作。
* 透過 Prisma，可以快速在 Postgres / MySQL / SQLite / MongoDB 等資料庫中建立 NextAuth 所需的 User、Session、Account 等 table。

### 型別安全 (Type Safety)

* Prisma 會依據 schema 自動產生 TypeScript 型別，減少 runtime 出錯的可能。
* 例如 `prisma.user.findUnique({ where: { email } })`，如果 email 在 schema 沒有定義，就會直接在編譯時報錯。

### 開發效率高

* 只要修改 `schema.prisma`，然後執行 `npx prisma migrate dev`，就能同步更新資料庫結構並生成 client。
* 不需要自己手寫 SQL，大幅加快 CRUD 開發速度。

### 和 Next.js 整合佳

* Prisma client 可以安全地在 server-side 環境（API routes, server components）使用。
* 再搭配 NextAuth，可以直接存取與驗證使用者資料，不需要自己設計 session table。

## 設定Prisma
1. 先在專案中安裝必要的套件:
```bash
npm install next-auth @auth/prisma-adapter prisma @prisma/client bcrypt
```

2. 在根目錄建立`prisma/schema.prisma`：
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id            String   @id @default(cuid())
  name          String?
  email         String?  @unique
  emailVerified DateTime?
  image         String?
  password      String?

  accounts      Account[]
  sessions      Session[]
}

model Account {
  id                String   @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?
  user              User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}
```
3. 建立 `.env`：
```bash
DATABASE_URL="postgresql://admin:admin@localhost:5432/nextauthdb?schema=public"
NEXTAUTH_SECRET="some-random-secret"
GOOGLE_CLIENT_ID="Your Google client id"
GOOGLE_CLIENT_SECRET="Your Google client secret"
```
4. Prisma migration：
```bash
npx prisma generate
npx prisma migrate dev --name init
```
* 建立 `prisma/migrations/` 資料夾
* 自動把 `schema.prisma` 中的 `User, Account, Session, VerificationToken` 建立到 PostgreSQL
* 產生 Prisma Client，可以在 Next.js API Route 中直接使用
* 檢查 pgAdmin → 應該會看到 User 等表格。

## 管理Prisma Client
<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
Prisma Client的功能:
* **型別安全 (Type-Safe)**  
  會根據 `schema.prisma` 自動產生 TypeScript 型別，避免寫錯欄位或傳錯資料型別。
* **自動補全 (Autocomplete)**  
  在 VSCode 中使用 `prisma.user.findUnique` 時，會自動跳出可用的欄位選項。
* **支援關聯查詢 (Relations)**  
  可以透過 `include` 或 `select` 一次撈出關聯資料，例如：  
  ```ts
  const userWithAccounts = await prisma.user.findUnique({
    where: { email },
    include: { accounts: true },
  })
* 交易 (Transactions)
需要一次性執行多個操作時，可以用 transaction 保證一致性：
```ts
await prisma.$transaction([
  prisma.user.create(...),
  prisma.session.create(...),
])
```
</div>


在 `src/lib/` 底下新增一個 `prisma.ts`:
```ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = global as unknown as { prisma: PrismaClient }

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: ['query'], // 開發時可看到 SQL log方便 debug，之後可移除
  })

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```
> ⚠️ 注意：在 Next.js 開發環境（Hot Reload）中，Prisma Client 容易重複初始化，所以建議像上面一樣用 `globalForPrisma` 來避免多次 `new PrismaClient`。

## 建立 Signup API

新增檔案 `/src/app/api/auth/signup/route.ts`：
```ts
import { NextResponse } from 'next/server'
import { hash } from 'bcrypt'
import { prisma } from '@/lib/prisma'

export async function POST(req: Request) {
  try {
    // get data from request
    const { email, password, name } = await req.json()

    // check if email and password are provided
    if (!email || !password) {
      return NextResponse.json({ error: 'Missing email or password' }, { status: 400 })
    }

    // check if user already exists
    const existingUser = await prisma.user.findUnique({ where: { email } })
    if (existingUser) {
      return NextResponse.json({ error: 'User already exists' }, { status: 400 })
    }

    // Hash password
    const hashedPassword = await hash(password, 10)

   // create user
    const user = await prisma.user.create({
      data: {
        email,
        name,
        password: hashedPassword,
      },
    })
    
    // return user
    return NextResponse.json({ user }, { status: 201 })
  } catch (err) {
    console.error(err)
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 })
  }
}
```

## 建立 Login API
新增檔案 `/src/app/api/auth/login/route.ts`：
```ts
import { NextResponse } from 'next/server'
import { compare } from 'bcrypt'
import { prisma } from '@/lib/prisma'

export async function POST(req: Request) {
  try {
    // get data from request
    const { email, password } = await req.json()
    // check if the user exists
    const user = await prisma.user.findUnique({ where: { email } })
    if (!user || !user.password) {
      return NextResponse.json(
        { error: 'Invalid credentials' },
        { status: 401 },
      )
    }
    
    // check if the password is correct
    const isValid = await compare(password, user.password)
    if (!isValid) {
      return NextResponse.json(
        { error: 'Invalid credentials' },
        { status: 401 },
      )
    }

    // TODO: integrate with NextAuth session
    return NextResponse.json(
      { message: 'Login success', user },
      { status: 200 },
    )
  } catch (err) {
    console.error(err)
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 },
    )
  }
}
```
> ⚠️ 真正登入流程需要建立 session / JWT / cookie。下一篇會透過 NextAuth Adapter 自動完成，這裡只做基礎驗證。

## 在前端呼叫API
有了後端API route、schema也定義好、migration也完成了，接下來試著在前端呼叫API看看能否成功寫入資料。

在前端Signup page加submit handler並傳入元件中:(也可以把submit handler封裝放在hook/資料夾管理)
```tsx
'use client'

import { AuthForm } from '@/components/auth/auth-form'
import { useLocale } from '@/context/locale'
import { getData } from '@/data'
import { withLocale } from '@/lib/utils'
import { FormEvent } from 'react'

export default function SignupPage() {
  const { locale } = useLocale()
  const { authData } = getData(locale)
  const data = {
    ...authData.signup,
    footer: authData.signup.footer
      ? {
          ...authData.signup.footer,
          linkUrl: withLocale(locale, authData.signup.footer.linkUrl),
        }
      : undefined
  }

  const handleSubmit = async (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault()
    const formData = new FormData(event.currentTarget)
    const email = formData.get('email') as string
    const password = formData.get('password') as string
    const name = formData.get('name') as string

    const response = await fetch('/api/auth/signup', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password, name }),
    })

    if (response.ok) {
      console.log('User created:', await response.json())
    } else {
      console.error('Signup failed:', await response.json())
    }
  }

  return (
    <AuthForm
      data={data} onSubmit={handleSubmit}
    >
      ...
    </AuthForm>
  )
}
```

## 測試是否成功
1. 打開 `/auth/signup` → 填入 email & password。
1. 送出 → API `/api/auth/signup` 會建立一筆 `User`。
1. 到 pgAdmin → 選 DB → `User` table → `View/Edit Data` 就能看到新增的紀錄。

Prisma client log 設了 `log: ['query']`，也可以在 Next.js console 看到 SQL。
也可以用sql command查詢:
```sql
SELECT * FROM public."User";
```
如下圖，資料成功寫入:
![螢幕擷取畫面 2025-09-10 150456](https://hackmd.io/_uploads/SyfWzi09lx.png)

4. 在`/auth/login`測試看看能否用剛剛signup的帳號密碼登入(成功轉導`/user/dashboard`)
![螢幕擷取畫面 2025-09-10 151110](https://hackmd.io/_uploads/BJmdmoC9ll.png)
這樣就成功了!

Email註冊登入基本流程就完成了，但目前還沒有真正建立 session / cookie，只是確認帳號密碼正確。下一篇會整合 NextAuth 來完成真正的驗證流程。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>