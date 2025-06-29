> 推荐学习指数：⭐️️⭐️️

## 1\. 前言

登录与注册，一个说简单也简单，说复杂也复杂的功能。

往简单的说，注册的时候搞两个输入框，将数据写入数据库，登录的时候校验一下数据是否正确即可。

往复杂的说，除了登录注册，还有账号注销、忘记密码、密码重置、邮箱验证、更新个人资料、更新密码、删除账号等功能，此外，数据的安全性如何保证？Magic Links、Multi-Factor Auth (MFA)、社交媒体登录 (Google, Facebook, Twitter, GitHub, Apple, and more) 是否要支持？是不是还要做个后台统计用户登录数据？想一想都是工作量。

所以虽然可以自己从头做，但有现成的还是用现成的吧。

所幸关于登录注册的技术方案并不多，推荐 3 个主流的技术选择：

1. [**Clerk**](https://clerk.com/ "https://clerk.com/")

> The most comprehensive User Management Platform

Clerk 提供了一个开发人员友好的身份验证和用户管理解决方案，帮助开发者轻松构建和管理用户身份验证、用户账户和权限管理功能。它提供了安全的身份验证、社交登录集成、角色和权限管理等功能。

2. [**Supabase**](https://supabase.com/ "https://supabase.com/")

> Supabase is an open source Firebase alternative.

> Start your project with a Postgres database, Authentication, instant APIs, Edge Functions, Realtime subscriptions, Storage, and Vector embeddings.

简单来说，Supabase 是 Firebase 的开源替代品，属于 BaaS（后端即服务）产品。所谓 BaaS，开发者只需要开发和维护前端代码，由 BaaS 服务商提供了开发应用所需要的后端服务。Supabase 使用 Postgres 数据库，提供了身份验证、即时 APIs，边缘函数、实时订阅、存储等后端服务。所以用户身份验证只是 Supabase 提供的众多功能之一。

3. [Auth.js](https://authjs.dev/ "https://authjs.dev/")

前身是 NextAuth.js，后来拓展为支持更多框架的通用 Web 身份验证解决方案，并更名为 Auth.js。Auth.js 不是平台，是一个开源库，可以帮助我们快速实现登录注册以及三方授权登录功能。

总的来说，如果要快速接入登录注册功能，最好还是使用平台，也就是 Clerk 和 Supabase，平台都提供了免费版，但免费版毕竟有一些限制，比如 Clerk 会限制 10000 月活跃用户等，Supabse 会限制 50000 月活跃用户，数据库大小为 500 MB 等等。对于个人项目来说基本是够用的。

如果要比较 Clerk 和 Supabase 的话，Clerk 更专注于身份验证和用户管理，对应功能更加丰富。Supabase 实现的功能更多，身份验证只是其中之一。

我会更推荐使用 Clerk，因为登录注册、授权登录、账号设置等都需要 UI 界面，虽然 Clerk 和 Supabase 都提供了 UI，但 Supabase 的 Auth UI 目前正处于废弃状态，已经不再维护。这也可以理解，毕竟 Supabase 是 BaaS 产品，自然要专注于提供后端服务。

所以从首次使用的开发成本而言，我认为 Clerk < Supabase < Auth.js。

## 2\. Clerk

### 2.1. Clerk 注册并创建应用

打开 [clerk.com/](https://clerk.com/ "https://clerk.com/") 注册一个账号，首次登录后会进入创建应用界面：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29fea7a6e3a046a2b3a1f86caa464180~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3414&h=2088&s=525729&e=png&b=ffffff)

输入应用的名字，左边选择支持的登录方式，右边为预览登录界面 UI，样式会根据左边的选择有所不同。

创建应用后，会跳转到 Get started 页面，指导用户如何接入 Clerk：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1afcba6a4cb64af198f979ac6b098969~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3878&h=2090&s=687809&e=png&b=ffffff)

不过你会发现并没有 Svelte，还好社区有 [Clerk SvelteKit](https://github.com/markjaquith/clerk-sveltekit "https://github.com/markjaquith/clerk-sveltekit")，并提供了演示 [Demo](https://clerk-sveltekit.markjaquith.com/ "https://clerk-sveltekit.markjaquith.com/")。

### 2.2. 项目初始化

运行：

```bash
npm create svelte@latest svelte-clerk
```

选择 **Skeleton project、TypeScript、Svelte 5（4 当然也是可以的，5 会有安装时的兼容问题，不过很容易解决）：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/347ed7f2473547d59e3cfc9140847e71~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2642&h=854&s=157406&e=png&b=1e1e1e)

按照命令行中的提示提交代码：

```bash
cd svelte-clerk

npm install

git init && git add -A && git commit -m "Initial commit"
```

### 2.3. 接入 Clerk

安装依赖项：

```bash
npm i clerk-sveltekit
```

新建 `.env`，代码如下：

```bash
PUBLIC_CLERK_PUBLISHABLE_KEY=pk_xxxxxxxxxxxxxxxxxxxxxxxxxxxx
CLERK_SECRET_KEY=sk_xxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

查看这两个值最简单的方法是查看项目创建后的 Next.js 接入文档，第二步就有写：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33195f75d4a348178ccccd0b11be91f2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2784&h=488&s=166542&e=png&b=ffffff)

拷贝代码，然后把 `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` 更名为 `PUBLIC_CLERK_PUBLISHABLE_KEY`即可。

新建 `src/hooks.server.ts`，代码如下：

```typescript
import type { Handle } from "@sveltejs/kit";
import { sequence } from "@sveltejs/kit/hooks";
import { handleClerk } from "clerk-sveltekit/server";
import { CLERK_SECRET_KEY } from "$env/static/private";

export const handle: Handle = sequence(
  handleClerk(CLERK_SECRET_KEY, {
    debug: true,
    protectedPaths: ["/admin"],
    signInUrl: "/sign-in",
  })
);
```

在这段代码中，我们声明了保护路由和登录地址。注意像 `/admin` 也会保护 `/admin/xxx`这样的路由。

新建 `src/hooks.client.ts`，代码如下：

```typescript
import type { HandleClientError } from "@sveltejs/kit";
// To use Clerk components:
import { initializeClerkClient } from "clerk-sveltekit/client";
// Or for headless mode:
// import { initializeClerkClient } from 'clerk-sveltekit/headless'
import { PUBLIC_CLERK_PUBLISHABLE_KEY } from "$env/static/public";

initializeClerkClient(PUBLIC_CLERK_PUBLISHABLE_KEY, {
  afterSignInUrl: "/admin/",
  afterSignUpUrl: "/admin/",
  signInUrl: "/sign-in",
  signUpUrl: "/sign-up",
});

export const handleError: HandleClientError = async ({ error, event }) => {
  console.error(error, event);
};
```

在这段代码中，我们声明了登录和注册地址以及登录和注册后的跳转地址。

基本配置完毕。现在我们开始构建登录界面。

新建 `src/routes/+layout.svelte`，代码如下：

```typescript
<script lang="ts">
  import UserButton from 'clerk-sveltekit/client/UserButton.svelte';
  import SignedIn from 'clerk-sveltekit/client/SignedIn.svelte';
  import SignedOut from 'clerk-sveltekit/client/SignedOut.svelte';
</script>

<SignedIn>
  <UserButton showName afterSignOutUrl="/" />
</SignedIn>

<SignedOut>
  <a href="/sign-in">Sign in</a> <span>|</span> <a href="/sign-up">Sign up</a>
  <!-- You could also use <SignInButton mode="modal" /> and <SignUpButton mode="modal" /> here -->
</SignedOut>

<slot />
```

在这段代码中，`<SignedIn>`表示登录后显示的内容，`<SignedOut>`表示未登录时现实的内容。我们放在 `+layout.svelte`中用于模拟顶部导航栏。

新建 `src/routes/sign-in/+page.svelte`，这是登录界面，代码如下：

```typescript
<script lang="ts">
  import SignIn from 'clerk-sveltekit/client/SignIn.svelte';
</script>

<div style="display: flex;justify-content:center">
  <SignIn redirectUrl="/admin" />
</div>
```

新建 `svelte-clerk/src/routes/sign-up/+page.svelte`，这是注册界面，代码如下：

```typescript
<script lang="ts">
  import SignUp from 'clerk-sveltekit/client/SignUp.svelte';
</script>

<div>
  <SignUp redirectUrl="/admin" />
</div>
```

新建 `src/routes/admin/+page.svelte`，代码如下：

```typescript
<script>
  import SignedIn from 'clerk-sveltekit/client/SignedIn.svelte';
</script>

<SignedIn let:user>
  Welcome {JSON.stringify(user?.fullName)}!
</SignedIn>
```

这是登录后跳转的界面。

此时基本配置完成，我们看下浏览器的效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/496beb945da648feb5e81590f1c235d2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1332&h=834&s=429662&e=gif&f=110&b=fefefe)

### 2.4. 中文汉化

虽然登录注册功能解决了，但你会发现，登录以及管理界面都是英文，我们该如何改为中文呢？

安装：

```javascript
npm i @clerk/localizations --legacy-peer-deps
```

修改 `src/hooks.client.ts`，代码如下：

```diff
import type { HandleClientError } from '@sveltejs/kit';
// To use Clerk components:
import { initializeClerkClient } from 'clerk-sveltekit/client';
// Or for headless mode:
// import { initializeClerkClient } from 'clerk-sveltekit/headless'
import { PUBLIC_CLERK_PUBLISHABLE_KEY } from '$env/static/public';
+import { zhCN } from '@clerk/localizations';

initializeClerkClient(PUBLIC_CLERK_PUBLISHABLE_KEY, {
+	localization: zhCN,
  afterSignInUrl: '/admin/',
  afterSignUpUrl: '/admin/',
  signInUrl: '/sign-in',
  signUpUrl: '/sign-up'
});

export const handleError: HandleClientError = async ({ error, event }) => {
  console.error(error, event);
};
```

此时界面改为中文：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/647c48494fa643e293843c116eb0ce91~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1332&h=834&s=589793&e=gif&f=107&b=fefefe)

### 2.5. 获取用户信息

现在我们该如何获取用户信息呢？

如果是用于展示，上节的例子中已经展示了一种方法，那就是使用 `<SignedIn let:user />`组件：

```diff
<SignedIn let:user>
  Welcome {JSON.stringify(user?.fullName)}!
</SignedIn>
```

SvelteKit Clerk 也提供了其他一些组件方便使用，具体查看[官方文档](https://github.com/markjaquith/clerk-sveltekit/tree/ff8e381705c49c59750206b409aa00f14127a0f1?tab=readme-ov-file#components "https://github.com/markjaquith/clerk-sveltekit/tree/ff8e381705c49c59750206b409aa00f14127a0f1?tab=readme-ov-file#components")。

如果需要在服务端获取，对于服务端受保护的路由将自动将 Clerk 用户对象注入到 `locals.session` 中，也就是说，可以在 load 函数、form actions 中获取该信息。

新建 `src/routes/admin/+page.server.ts`，代码如下：

```diff
export async function load({ locals }) {
  console.log(locals.session);
  return {
    user: locals.session
  };
}
```

查看打印的结果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3ca32e6c0b1480988f76ce536ee6fd0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1598&h=596&s=135441&e=png&b=1e1e1e)

此时可以获取 userId 信息，如果要获取更完整的信息，可以通过 Clerk 提供的 [API](https://clerk.com/docs/references/backend/overview "https://clerk.com/docs/references/backend/overview") 获取。

修改 `src/routes/admin/+page.server.ts`，代码如下：

```diff
import { createClerkClient } from '@clerk/clerk-sdk-node';
import { CLERK_SECRET_KEY } from '$env/static/private';

const clerkClient = createClerkClient({ secretKey: CLERK_SECRET_KEY });

export async function load({ locals }) {
  const userId = locals.session.userId;
  const { firstName, lastName } = await clerkClient.users.getUser(locals.session.userId);

  return {
    user: {
      userId,
      userName: firstName + ' ' + lastName
    }
  };
}
```

修改 `src/routes/admin/+page.svelte`，代码如下：

```diff
<script>
  let { data } = $props();
  import SignedIn from 'clerk-sveltekit/client/SignedIn.svelte';
</script>

{#if data.user}
  Welcome {JSON.stringify(data.user.userName)}!
{/if}

<br />

<SignedIn let:user>
  Welcome {JSON.stringify(user?.fullName)}!
</SignedIn>
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84649b0be10b4e6faaeafac5ca5924ff~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1482&h=402&s=70833&e=png&b=fefefe)

### 2.6. 后台界面

Clerk 提供了后台界面，可以查看登录用户数据以及进行登录相关的设置：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccf81d7e0cf04060a35499cab56c093d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=4154&h=1884&s=591147&e=png&b=fefefe)

## 3\. 最后

目前 SvelteKit 接入 Clerk 还是要靠社区的贡献，目前看到有两个不错的库：

1. [github.com/markjaquith…](https://github.com/markjaquith/clerk-sveltekit "https://github.com/markjaquith/clerk-sveltekit") 这是本篇用到的
2. [github.com/wobsoriano/…](https://github.com/wobsoriano/svelte-clerk "https://github.com/wobsoriano/svelte-clerk") 这是 Svelte 5 的，需要用 Svelte 5

使用 Clerk 主要是看中 UI 界面比较省事，一步到位。如果界面需要从零构建，使用 Supabase 也是一个不错的选择。