---
title: "[Next.js] [Authentication] 使用NextAuth進行驗證- 筆記"
pubDatetime: 2026-05-26T03:01:46.947Z
tags: ["Authentication","React.js","Next.js","NextAuth","OAuth"]
description: " Table of contents 在[上一篇文章](https://hackmd.io/A4memsR6Tcy5KP..."
---

## Table of contents

在[上一篇文章](https://hackmd.io/A4memsR6Tcy5KPAwlz053g)中，我們設計了 Prisma schema 並完成了 signup / login API，建立了最基本的登入流程。但目前的流程只會比對帳號密碼，**沒有 cookie / session / JWT 的機制**，也就是說：

* 無法在多頁面之間維持登入狀態
* 使用者關閉分頁或重新整理後，需要重新登入
* 安全性不足

因此，這一篇就要介紹 NextAuth，幫我們整合 cookie、session 與 JWT，建立完整且安全的驗證流程。

## 什麼是NextAuth?
NextAuth.js 是一個完整的開源 Next.js 驗證解決方案，專門設計來支援 Next.js 與 Serverless 架構。
它提供簡單易用的 API，可以快速整合各種登入方式，包括 OAuth（Google, GitHub, Facebook…等）、Email/Password（自訂憑證登入） 以及 JWT 與 Database session等。
👉 簡單來說，**NextAuth 把登入系統常見的「session 管理、token 發放、第三方登入」都封裝好了**，開發者只要專注在業務邏輯，不必從零實作驗證流程。

## 為什麼使用NextAuth做驗證?
雖然我們可以自己實作 JWT 或 session middleware，但這樣有幾個問題：

* 安全性風險：自己處理 token 加密/驗證，容易有漏洞
* 重複造輪子：session 管理、token refresh、第三方登入… 都要自己寫
* 維護成本高：未來若要擴充（例如加 Google 登入），得大改程式碼

### 使用 NextAuth 的好處：

* 官方推薦：Next.js 官方文件中建議使用 NextAuth 作為驗證方案
* 開箱即用：只要簡單設定 provider 和 adapter，就能啟用完整的登入系統
* 高度彈性：支援自訂登入邏輯（像上一篇我們做的 email/password）
* 自動處理 session / JWT：幫你處理 token 存取、cookie 設定
* 快速擴充：日後要支援 Google / GitHub / Facebook 登入非常簡單


## 實作
### 建立NextAuth API
首先建立`src/app/api/auth/[...nextauth]/route.ts`檔案：
> 在 Next.js 的 App Router 結構裡，`route.ts` 是固定用來定義 API 的檔案，所在的資料夾路徑會對應到實際的 API path。
> 例如：
> ```bash
> src/app/api/auth/[...nextauth]/route.ts
> ```
> 對應API path:
> ```
> /api/auth/[...nextauth]
> ```
> 其中 `[...nextauth]` 是 Next.js 的 catch-all 動態路由，會攔截 `/api/auth/*` 下的請求，例如 `/api/auth/signin`、`/api/auth/callback/google`，它不會直接以 `[...nextauth]` 出現在 URL，但實際上會展開成多個 NextAuth 所需的 API endpoint。

```ts
// src/app/api/auth/[...nextauth]/route.ts
// NextAuth 主程式 & 型別
import NextAuth, { NextAuthOptions } from 'next-auth'

// 內建 Provider：帳號密碼登入
import CredentialsProvider from 'next-auth/providers/credentials'
// 內建 Provider：Google OAuth 登入
import GoogleProvider from 'next-auth/providers/google'

// Prisma adapter，讓 NextAuth 可以存取 PostgreSQL 資料庫
import { PrismaAdapter } from '@next-auth/prisma-adapter'
import { prisma } from '@/lib/prisma'

// bcryptjs：用來驗證密碼
import { compare } from 'bcryptjs'

// Session 型別（jwt / database）
import type { SessionStrategy } from 'next-auth'

// NextAuth 設定檔
export const authOptions: NextAuthOptions = {
  // 讓 NextAuth 使用 Prisma 來存取使用者資料
  adapter: PrismaAdapter(prisma),

  // session 採用 JWT
  session: { 
      strategy: 'jwt' as SessionStrategy,
      maxAge: 30 * 24 * 60 * 60, // 30 days},

  // 登入方式 (Providers)
  providers: [
    // 1. 自訂帳號密碼登入
    CredentialsProvider({
      name: 'Credentials', // 登入方式名稱（顯示用）
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      // 使用者登入驗證邏輯
      async authorize(credentials) {
        // 如果沒有填 email 或 password → 直接拒絕
        if (!credentials?.email || !credentials?.password) return null

        // 查詢 DB 看 email 是否存在
        const user = await prisma.user.findUnique({
          where: { email: credentials.email },
        })

        // 如果使用者不存在，或沒有密碼欄位 → 拒絕
        if (!user || !user.password) return null

        // 驗證密碼（比對輸入的 password 與資料庫中的 hash password）
        const isValid = await compare(credentials.password, user.password)
        if (!isValid) return null

        // 成功 → 回傳 user 物件（必要欄位：id / email）
        // 這裡回傳的欄位會影響 token 與 session 的內容，
        // 如果需要 role 或其他欄位，要在 authorize callback 或後續 jwt callback 補上。
        return { id: user.id, email: user.email, name: user.name }
      },
    }),

    // 2. Google OAuth 登入
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,         // Google OAuth Client ID
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!, // Google OAuth Secret
    }),
  ],

  // 自訂登入頁面 (預設 /api/auth/signin)
  pages: { signIn: '/auth/login' },

  // 用來簽發 JWT 的加密金鑰
  secret: process.env.NEXTAUTH_SECRET,
  debug: process.env.NODE_ENV === 'development',

  // callback：控制 token / session 內容
  callbacks: {
    // jwt callback：每次產生/更新 JWT 時會觸發
    async jwt({ token, user }) {
      if (user) {
        // 登入成功時，將 user 資訊存入 token
        token.id = user.id
        token.email = user.email
        token.name = user.name
      }
      return token
    },

    // session callback：決定要把哪些資訊回傳給前端
    async session({ session, token }) {
      if (token && session.user) {
        // 將 token 資訊傳遞給 session，方便前端存取
        session.user.id = token.id as string
        session.user.email = token.email as string
        session.user.name = token.name as string
      }
      return session
    },
  },
}

// 建立 NextAuth handler
// Next.js App Router 要 export GET / POST handler
const handler = NextAuth(authOptions)
export { handler as GET, handler as POST }
```
### 說明
#### 1. 可以**自訂 NextAuth 的設定（NextAuthOptions）**
包含 providers、callbacks、pages、session strategy、adapter 等。詳細設定請參考[官方文件](https://next-auth.js.org/configuration/options)

#### 2. Prisma Adapter 作用是**把 NextAuth 與資料庫（例如 PostgreSQL）串接**
NextAuth 會透過 adapter 在資料庫中讀寫user、account、session、verification token 等資料表。實務上會用 `@next-auth/prisma-adapter` 搭配已設定好的 `prisma` client。參考[NextAuth官方文件](https://authjs.dev/getting-started/adapters/prisma?_gl=1*455gwo*_gcl_au*MjA1NDMxNjgyNS4xNzU3MzkyMDQ5LjEyNTU1NTA5OTEuMTc1NzM5MjIxMC4xNzU3MzkyMjEw)。


#### 3. NextAuth 支援不同的 session strategy，常見有兩種:

* **jwt（token-based）**：NextAuth 會產生一個簽名（signed，或依設定也可加密）的 token（JWT），並把它放在 cookie（通常是 httpOnly、secure）中。
這個 cookie 存在客戶端，但無法被一般的 JS 存取（httpOnly），伺服器會解讀/驗證 token 以取得使用者資訊。**JWT 策略不需要在資料庫中建立 session table（除非刻意把資料另外存到 DB）。**
* **database（DB session）**：**NextAuth 會在 Session table 中建立 session 紀錄，並在 client 端的 cookie 中放一個 sessionToken（通常是隨機 token）。**
每次請求時，NextAuth 會用 cookie 的 sessionToken 去 DB 查出對應的 session。
這種 session 資料存在伺服器端（DB）的方式，**比較容易做 session invalidation、登出所有裝置等操作。**
* NextAuth 預設採用 JWT 策略，但如果設定了 Adapter（例如 PrismaAdapter），且沒有指定 session.strategy，就會自動改用 database session。
* **小結**：想把 session 保存在資料庫就選 database；想維持無狀態、以 cookie+JWT 為主就選 jwt strategy。 


#### 4. 為何要在 jwt / session 裡加入 user.id ?

NextAuth 預設的 token/session 內容會根據 provider 與 adapter 而異，實務上常常需要把 user.id 明確放入 token（在 jwt callback）並再把它複製到 session（在 session callback），原因是前端常需要 user id 來呼叫 API 或顯示資料。也就是：

* **jwt callback**：當使用者登入（或 token 更新）時，可以把想保留在 token 的欄位（例如 id）寫入 token。
* **session callback**：每次回傳 session 給前端時，可以從 token 裡把 id 等欄位加入 `session.user`。

#### 5. 驗證流程: 

使用者在前端輸入帳號/密碼 → 前端呼叫 NextAuth 的 Credentials（或 custom API）→ NextAuth 的 authorize（或自訂邏輯）去資料庫查使用者 → 使用 bcrypt 比對密碼 → 若驗證成功，NextAuth 會：

* 在 jwt callback 中產生/更新 token（可以把 `user.id` 加入 token），token 由 `NEXTAUTH_SECRET` 簽名（或依設定加密）；
* NextAuth 把 token 放到 httpOnly cookie 回傳給 client（若使用 jwt 策略）；若使用 DB session，則回傳一個 sessionToken cookie 並在 DB 建立 session row；
* 當前端呼叫 `getSession()` 或 `useSession()` 時，NextAuth 會在 session callback 將 token（或 DB 查到的資料）格式化成 session，並把 session 回傳給前端（此時 `session.user.id` 可供使用）。

#### 其他補充
1. `NEXTAUTH_SECRET` 很重要：用於簽名（sign）JWT 與產生 CSRF token，務必要固定且安全，需在 production 設定為強隨機值。
2. Cookie 安全性：在 production 要確保 cookie 設為 `secure`、使用 https，並適當設定 `sameSite`。
3. 如果使用 Provider（Google 等）與 Prisma Adapter，當第三方登入時，NextAuth 會自動在 Account / User 等 table 建/查紀錄；Credentials provider 則是開發者負責驗證邏輯（如上方 authorize 裡做的事）。

## 簡單解釋NextAuth做了什麼
過去如果要自己實作登入驗證，通常需要：
* 管理 JWT token、session、以及 OAuth 流程
* 處理安全性相關問題（例如 CSRF 攻擊防護、session 安全性等)

而 NextAuth 幫我們把這些繁瑣的工作整合起來：
* 內建 credentials 登入 與 多種第三方登入 provider（Google、GitHub、Facebook...）
* 自動處理 session 管理 與 CSRF 防護
* 透過簡單設定即可擴充其他登入方式

換句話說，NextAuth 幫開發者把「驗證」這件事模組化了，讓我們專心在應用程式的核心功能，而不是重複造輪子。

## 在前端送出請求
設置完後端驗證API後，就可以試著在前端呼叫API。修改一下login page:
```tsx
// src/app/[locale]/auth/login/page.tsx
'use client'
import { useState } from 'react'
import { useRouter } from 'next/navigation'
import { useLocale } from '@/context/locale'
import { getData } from '@/data'
import { withLocale } from '@/lib/utils'
import { AuthForm } from '@/components/auth/auth-form'
import { loginSchema, LoginFormValues } from '@/schemas/authSchema'
import { useAuthForm } from '@/hooks/useAuthForm'
// NextAuth 提供的 signIn 方法，用來呼叫後端 /api/auth/* 登入 API
import { signIn } from 'next-auth/react'


export default function LoginPage() {
const router = useRouter()
const { locale } = useLocale()
const [error, setError] = useState('')

const form = useAuthForm<LoginFormValues>(loginSchema)
const { authData } = getData(locale)

const data = {
  ...authData.login,
  fields: authData.login.fields.map((field) => ({
    ...field,
    extra: field.extra
      ? { ...field.extra, url: withLocale(locale, field.extra.url) }
      : undefined,
  })),
  footer: authData.login.footer
    ? {
        ...authData.login.footer,
        linkUrl: withLocale(locale, authData.login.footer.linkUrl),
      }
    : undefined,
}

// 當使用者送出登入表單時
const onSubmit = async (values: LoginFormValues) => {
  setError('')

  try {
    const dashboardUrl = withLocale(locale, '/user/dashboard')
      // 呼叫 NextAuth 的 signIn('credentials')
      // - provider 名稱必須和 route.ts 裡的 CredentialsProvider 對應
      // - redirect:false 不由 NextAuth 自動redirect，由router.push()轉導
    const result = await signIn('credentials', {
      email: values.email,
      password: values.password,
      callbackUrl: dashboardUrl,
      redirect: false,
    })
     // 如果驗證失敗（authorize 回傳 null），NextAuth 會在 result.error 帶回錯誤
     // signIn 回傳的 result 包含 ok / error / status，
     // 通常建議用 result?.error 來判斷是否登入失敗。
    if (result?.error) {
      setError('Login failed. Please check your credentials.')
    } else {
        router.push(dashboardUrl) // 成功後再導向
      }
  } catch (err) {
    setError('An unexpected error occurred')
    console.error('Login error:', err)
  }
}

return (
  <AuthForm<LoginFormValues>
    data={data}
    registerField={form.register}
    errors={form.formState.errors}
    onSubmit={form.handleSubmit(onSubmit)}
    isSubmitting={form.formState.isSubmitting}
  >
    {error && <p className="text-sm text-red-500">{error}</p>}

    ...
  </AuthForm>
)
}
```
也可以把驗證邏輯存成一個hook:
```tsx
//src/hooks/useAuth.ts
import { useSession, signIn, signOut } from 'next-auth/react'

export function useAuth() {
   const { data: session, status } = useSession()

  return {
    session,
    isLoading: status === 'loading',
    login: async (email: string, password: string, callbackUrl?: string) => {
      const result = await signIn('credentials', {
        email,
        password,
        callbackUrl: callbackUrl ?? '/user/dashboard',
        redirect: false,
      })
      return result
    },
    logout: () => signOut(),
  }
}
```
然後在Login page使用`useAuth` hook:
```tsx
const onSubmit = async (values: LoginFormValues) => {
    setError('')
    try {
      const dashboardUrl = withLocale(locale, '/user/dashboard')
      const result = await login(values.email, values.password, dashboardUrl)
      
      if (result?.error) {
        setError('Login failed. Please check your credentials.')
      } else {
        router.push(dashboardUrl) // 成功後這邊再導向
      }
    } catch (err) {
      setError('An unexpected error occurred')
      console.error('Login error:', err)
    }
  }
```
到這裡，NextAuth的登入驗證已經大功告成了！[前一篇文](https://hackmd.io/A4memsR6Tcy5KPAwlz053g)中的login API也可以刪除了，不管是jwt、session、cookie或是安全性，NextAuth都幫我們攢便便(tshuân piān-piān)了，是不是很方便呢？

## 總結：完整驗證流程
為避免本篇文過長，完整驗證流程就在[另一篇文章](https://)詳細拆解與說明。
這裡簡單放一張流程圖:
### 自訂credentials(email/password)登入的流程
```
使用者登入流程（NextAuth + Credentials）

[前端表單提交]
   |
   v
signIn('credentials', { email, password })
   |
   v
POST /api/auth/callback/credentials
   |
   v
[NextAuth server - route.ts]
   └─> CredentialsProvider.authorize(credentials)
         |
         |-- 查 DB: prisma.user.findUnique({ email })
         |-- bcrypt.compare(password, user.password)
         |
         ├─ 驗證失敗 → return null → signIn 回傳 error
         └─ 驗證成功 → return user { id, email, name }
   |
   v
[NextAuth callbacks]
   ├─ jwt({ token, user })
   │    → 將 user.id 寫入 token
   │    → 簽名成 JWT
   │
   └─ session({ session, token })
        → 將 token.id 注入 session.user.id
   |
   v
[Set-Cookie]
   └─ JWT 存入 httpOnly cookie (e.g. next-auth.session-token)
   |
   v
[前端收到結果]
   ├─ redirect: false → result = { ok, error? }
   └─ redirect: true  → 自動跳轉 callbackUrl
   |
   v
[之後需要 session 時]
   └─ getSession() / useSession()
         |
         └─ 讀 cookie → 驗簽 JWT → 執行 session callback
              → 回傳 { user: { id, email, name }, expires }
```


因為文章已經太長，[下一篇文](https://hackmd.io/WM2Mmp4qSKWZ_FEYBxG1tA)再來說明整合Google登入的步驟。