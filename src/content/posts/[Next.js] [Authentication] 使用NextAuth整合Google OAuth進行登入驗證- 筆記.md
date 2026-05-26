---
title: "[Next.js] [Authentication] 使用NextAuth整合Google OAuth進行登入驗證- 筆記"
pubDatetime: 2025-09-18T19:38:04.000Z
tags: ["React.js","Next.js","Authentication","Prisma","PostgreSQL","JWT","token","session","cookie","NextAuth","OAuth"]
description: " Table of contents 在[上一篇文章](https://hackmd.io/Msx1EFz6Q8Ob5C..."
---

## Table of contents

在[上一篇文章](https://hackmd.io/Msx1EFz6Q8Ob5C6NaH3jDA)中，我們利用NextAuth幫助處理JWT、session等驗證流程進行登入，這一篇就來說明如何整合NextAuth進行第三方登入驗證(以Google為例)

## 在NextAuth API中加入GoogleProvider
```tsx
import NextAuth, { NextAuthOptions } from 'next-auth'
import CredentialsProvider from 'next-auth/providers/credentials'
import GoogleProvider from 'next-auth/providers/google'
import { PrismaAdapter } from '@next-auth/prisma-adapter'
import { prisma } from '@/lib/prisma'
import { compare } from 'bcryptjs'
import type { SessionStrategy } from 'next-auth'

export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(prisma),
  session: { strategy: 'jwt' as SessionStrategy },
  providers: [
    CredentialsProvider({
      ...
    }),
    // 加入GoogleProvider
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
  pages: { signIn: '/auth/login' },
  secret: process.env.NEXTAUTH_SECRET,
  callbacks: {
    // Google 登入
    async signIn({ user, account, profile }) {
      if (account?.provider === 'google') {
        // 確保 name 一定有值
        if (!user.name) {
          user.name = profile?.name || user.email?.split('@')[0] || 'User'
          await prisma.user.update({
            where: { id: user.id },
            data: { name: user.name },
          })
        }
      }
      return true
    },
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id
      }
      return token
    },
    async session({ session, token }) {
      if (token) {
        session.user.id = token.id as string
      }
      return session
    },
  },
}

const handler = NextAuth(authOptions)
export { handler as GET, handler as POST }
```
說明:
1. 提供nextauth GoogleProvider，NextAuth就會將使用者導向至 Google OAuth consent screen，獲得使用者同意授權後，依據Google回傳的資料(scope: profile, email)，再由GoogleProvider 驗證授權碼 → 換取 access_token & 使用者資訊(id, email, name, image等)
2. callback中可以寫入希望資料存取的邏輯，例如這裡:
* **name欄位的問題**：Google provider 預設會提供 `profile.name`，如果沒有公開姓名，就用`profile.email.split('@')[0]` 當 fallback。
* **email 已存在時要合併**:在 Prisma schema 已經設定 `User.email @unique`，所以不會建立重複的 `User`，只會建立新的 `Account` 記錄。
→ 意思是：如果同一個 email 先用 credentials 註冊，之後 Google 登入時會自動連到同一筆 `User`。
3. 取得Google API client id&secret可參考[官方說明](https://developers.google.com/identity/gsi/web/guides/get-google-api-clientid?hl=zh-tw#get_your_google_api_client_id)。NextAuth 預設的 Google redirect URI (Google 把授權碼送回來的 API endpoint) 是：`http://localhost:3000/api/auth/callback/google`


## 前端呼叫API
之前已經把登入驗證相關的邏輯寫在`useAuth` hook中，這裡只要補上：
```tsx
import { useSession, signIn, signOut } from 'next-auth/react'

export function useAuth() {
   const { data: session, status } = useSession()

  return {
    session,
    isLoading: status === 'loading',
    login: (email: string, password: string) =>
      signIn('credentials', { email, password, redirect: false }),
    // Google login
    loginWithGoogle: (callbackUrl?: string) =>
      signIn('google', {
        callbackUrl: callbackUrl ?? '/user/dashboard',
      }),
    logout: () => signOut(),
  }
}
```

然後放進login page:
```tsx
'use client'
import { useState } from 'react'
import { useRouter } from 'next/navigation'
import { useLocale } from '@/context/locale'
import { getData } from '@/data'
import { withLocale } from '@/lib/utils'
import { AuthForm } from '@/components/auth/auth-form'
import { loginSchema, LoginFormValues } from '@/schemas/authSchema'
import { useAuthForm } from '@/hooks/useAuthForm'
import { useAuth } from '@/hooks/useAuth'


export default function LoginPage() {
const router = useRouter()
const { locale } = useLocale()
const [error, setError] = useState('')
// 加入loginWithGoogle
const { login, session, isLoading, loginWithGoogle } = useAuth()

const form = useAuthForm<LoginFormValues>(loginSchema)
const { authData } = getData(locale)

const data = {
  ...
}

const onSubmit = async (values: LoginFormValues) => {
  ...
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

    <Button
      type="button"
      variant="outline"
      className="w-full"
      // 點擊按鈕觸發Google登入，參數callbackUrl是成功登入後轉導的url
      onClick={() => loginWithGoogle(withLocale(locale, '/user/dashboard'))}
    >
      ...
  </AuthForm>
)
}
```
到這裡Google登入就完成啦!

## Google登入流程
```
用戶點擊 loginWithGoogle →
NextAuth 走 GoogleProvider OAuth →
Google 回傳 profile (email, name, image) →
PrismaAdapter:
  - 查 User.email 是否存在
  - 若存在 → 建立 Account 並關聯到該 User
  - 若不存在 → 建立新的 User 與 Account
最後回傳 session
```



## 總結：完整驗證流程
### Google OAuth
```
Google OAuth 登入流程（NextAuth + PrismaAdapter）

[前端觸發登入]
   |
   v
signIn('google')
   |
   v
[NextAuth - GoogleProvider]
   └─ redirect 使用者到 Google OAuth consent screen
   |
   v
[Google Login Page]
   └─ 使用者同意授權 (scope: profile, email)
   |
   v
Google → redirect back → /api/auth/callback/google
   |
   v
[NextAuth server - route.ts]
   └─ GoogleProvider 驗證授權碼 → 換取 access_token & 使用者資訊
         (id, email, name, image)
   |
   v
[PrismaAdapter]
   ├─ findUserByEmail → 若存在 → 直接回傳
   └─ 不存在 → 建立新使用者紀錄 (User table)
   |
   v
[NextAuth callbacks]
   ├─ jwt({ token, user })
   │    → 把 user.id 存進 JWT
   │
   └─ session({ session, token })
        → 把 token.id 注入 session.user.id
   |
   v
[Set-Cookie]
   └─ JWT 存入 httpOnly cookie (next-auth.session-token)
   |
   v
[前端收到結果]
   └─ redirect → 預設回到首頁 (或指定 callbackUrl)
   |
   v
[之後需要 session 時]
   └─ getSession() / useSession()
         |
         └─ 驗簽 JWT → 執行 session callback
              → 回傳 { user: { id, email, name, image }, expires }
```

### Credentials 登入 & Google OAuth 登入的 redirect / callback 流程:
```
使用者點選「登入」 
       │
       ├──> 選擇「Credentials 登入」
       │        │
       │        ├── 前端送出 email + password 給 
       │        │   /api/auth/callback/credentials
       │        │
       │        ├── NextAuth 呼叫 authorize()
       │        │       │
       │        │       ├── 用 Prisma 查詢 User 資料庫
       │        │       ├── bcrypt.compare() 檢查密碼
       │        │       └── 驗證成功 → 回傳 user
       │        │
       │        └── NextAuth 建立 JWT (或 Session) 
       │                 │
       │                 └── 設定 cookie → 導向 callbackUrl
       │
       └──> 選擇「Google 登入」
                │
                ├── 使用者被導向 Google OAuth 同意畫面
                │
                ├── Google 完成驗證後
                │    redirect 到
                │    /api/auth/callback/google
                │
                ├── NextAuth 收到授權碼 → 向 Google 換取 token
                │
                ├── PrismaAdapter → upsert User:
                │       - 如果 email 沒出現過 → 建立新 User
                │       - 如果 email 已存在 (Credentials 註冊過)
                │           → 把 Google 帳號 (Account) 綁定到同一筆 User
                │       - name 若沒填，會取 email 前綴當暫時 name
                │
                ├── NextAuth 建立 JWT (或 Session)
                │
                └── 設定 cookie → 導向 callbackUrl
```
### 核心差異：
* Credentials：自己檢查密碼 → 回傳 user → NextAuth 做 session。
* Google OAuth：Google 驗證 → NextAuth 拿到授權碼 → PrismaAdapter upsert 使用者 → session。

這樣的話，兩種登入方式最後都會走到 NextAuth 的 session/cookie 建立流程，所以前端在 `useSession()` 看到的 user 格式是一致的。

這一篇利用nextauth整合credentials&Google OAuth登入驗證的[文章](https://karthickragavendran.medium.com/setup-guide-for-nextauth-with-google-and-credentials-providers-in-next-js-13-8f5f13414c1e)寫得滿清楚，也可以參考一下。