---
title: App Router では NextAuth.js の authOptions は外部ファイルに切り出して export しよう
tags:
  - tips
  - Next.js
  - next-auth
  - nextauth.js
  - AppRouter
private: false
updated_at: '2023-12-05T18:23:47+09:00'
id: becacbdc8b1cebe81ceb
organization_url_name: null
slide: false
ignorePublish: false
---
# 結論

Next.js で App Router を利用している場合、 [NextAuth.js](https://next-auth.js.org/) の config である `authOptions` は `/api/auth/[...nextauth]/route.ts` に直接記述するのではなく、外部ファイルへ切り出してから`import`しましょう。

- NG

```ts:/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth'
import type { NextAuthOptions } from 'next-auth'

export const authOptions: NextAuthOptions = {
  // your configs
}

export default NextAuth(authOptions);
```

- OK

```ts:/lib/auth.ts
import type { NextAuthOptions } from "next-auth"

export const authOptions = {
  providers: [], // rest of your config
} satisfies NextAuthOptions

export const { handlers, auth, signIn, signOut } = NextAuth(authOptions)
```


```ts:/api/auth/[...nextauth]/route.ts
import { handler } from "@/lib/auth"
export { handler as GET, handler as POST }
```

# 理由
App Router の Route Handler では`/api/.../route.ts`内で`export`できる関数が特定の HTTP Methods に限定されており、`const authOptions: NextAuthOptions` のようなオブジェクトを`export`すると`Type error`となるため。

> **Supported HTTP Methods**
>The following HTTP methods are supported: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, and `OPTIONS`. If an unsupported method is called, Next.js will return a `405 Method Not Allowed` response.

https://nextjs.org/docs/app/building-your-application/routing/route-handlers#supported-http-methods

実際に`next build`すると、以下のようなエラーが投げられます。
```shell
src/app/api/auth/[...nextauth]/route.ts
Type error: Route "src/app/api/auth/[...nextauth]/route.ts" does not match the required types of a Next.js Route.
  "authOptions" is not a valid Route export field.
```

# 関連情報
NextAuth.js 公式のサンプルでも同様の実装がなされています。

https://github.com/nextauthjs/next-auth-example/blob/main/auth.ts

また、これまで`authOptions`を`[...nextauth]/route.ts`に記述していたのは単なる慣例である、と NextAuth.js のメインメンテナから言及もされています。

> The place for `authOptions` in `[...nextauth]/route.ts` was just a convention. It came from before when the restriction for Next.js was introduced to only allow certain exports from Route Handler files.
> *`authOptions`を`[...nextauth]/route.ts`に置くのは、単なる慣例でした。これは以前、Next.jsにRouteハンドラファイルからの特定のエクスポートのみを許可する制限が導入されたときに由来しています。*（[DeepL](https://www.deepl.com/translator#en/ja/The%20place%20for%20authOptions%20in%20%5B...nextauth%5D%5C%2Froute.ts%20was%20just%20a%20convention.%20It%20came%20from%20before%20when%20the%20restriction%20for%20Next.js%20was%20introduced%20to%20only%20allow%20certain%20exports%20from%20Route%20Handler%20files.) による翻訳）

https://github.com/vercel/next.js/issues/56832#issuecomment-1763206667

## おまけのtips

同ファイル内で`getServerSession`関数に`authOptions`を渡したヘルパー関数を定義して`export`すれば各ファイルで`authOptions`を`import`する必要がなくなるよ、というtipsが公式から出されています。ご参考までに。

```ts:auth.ts
import type { GetServerSidePropsContext, NextApiRequest, NextApiResponse } from "next"
import type { NextAuthOptions } from "next-auth"
import { getServerSession } from "next-auth"

// You'll need to import and pass this
// to `NextAuth` in `app/api/auth/[...nextauth]/route.ts`
export const config = {
  providers: [], // rest of your config
} satisfies NextAuthOptions

// Use it in server contexts
export function auth(...args: [GetServerSidePropsContext["req"], GetServerSidePropsContext["res"]] | [NextApiRequest, NextApiResponse] | []) {
  return getServerSession(...args, config)
}
```

https://next-auth.js.org/configuration/nextjs#getserversession


# 参考

https://next-auth.js.org/configuration/nextjs

https://stackoverflow.com/questions/76298505/my-next-js-app-isnt-building-and-returing-a-type-error-how-do-i-fix

https://github.com/vercel/next.js/issues/56832
