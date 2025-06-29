> 推荐学习指数：⭐️️⭐️️⭐️️

## 1\. 前言

Zod 几乎是前端开发的必学内容了，因为大部分的全栈项目都会有数据校验的场景，像 Next.js [官方文档](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#server-side-validation-and-error-handling "https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#server-side-validation-and-error-handling")推荐的正是 [Zod](https://zod.dev/ "https://zod.dev/")。

目前 Zod GitHub 31.5k Star，Npm 周均下载量 784W，几乎是前端做数据校验的第一选择。

本篇带大家快速上手 Zod。

## 2\. Zod 介绍

### 2.1. 基础介绍

**Zod 是一个 TypeScript 优先（TypeScript-first）的模式声明（schema declaration）和验证库（validation library）**。

第一次听到这个介绍可能会“不明觉厉”，但其实很简单，举个简单的例子：

```javascript
import { z } from "zod";

// 模式声明
const schema = z.string();

// 数据校验
schema.parse("tuna"); // => "tuna"
schema.parse(12); // => throws ZodError
```

这就是一个基本的模式声明和数据验证的例子。那什么是 TypeScript 优先呢？

简单来说，就是和 TypeScript 搭配使用，效果更佳。Zod 的目的在于消除重复的类型声明。使用 Zod，你只需声明一次验证器（validator），Zod 就会自动推断出静态 TypeScript 类型。细看 Zod 的 API，你会发现 Zod 与 TypeScript 的类型系统几乎是一对一的映射。

```typescript
import { z } from "zod";

// 模式声明
const User = z.object({
  username: z.string(),
});

// 数据校验
User.parse({ username: "Ludwig" });

// 提取推断类型
type User = z.infer<typeof User>;
// { username: string }
```

注意：但这并不是说使用 Zod 就一定要使用 TypeScript，Zod 也可用于纯 JavaScript。

### 2.2. 运行时校验

那你可能就好奇了，不都是数据校验，我都有 TypeScript 了，用 Zod 干嘛？

简单来说，TypeScript 是静态类型检查，但 Zod 不仅能在编译时提供类型检查，还能在运行时进行数据校验。这样就可以从源头上防止数据不合法而导致的错误，提高应用的稳定性。

举个例子，我们调用接口，获取返回的数据并进行处理：

```javascript
export async function GET() {
  const res = await fetch("/api/product");
  const data = await res.json();

  const showPrice = data.price.toFixed(2);
  return Response.json({ showPrice });
}
```

在这段代码中，data 肯定会被推断为 any，因为 data 是运行时返回的数据，TypeScript 并不知道：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/995c4ad41b7a456f93a3ab60702ee5d4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1578&h=430&s=107232&e=png&b=1e1e1e)

我们当然可以补全类型声明：

```typescript
type Product = {
  price: number;
};

export async function GET() {
  const res = await fetch("/api/product");
  const data = (await res.json()) as Product;

  const showPrice = data.price.toFixed(2);
  return Response.json({ showPrice });
}
```

现在 price 字段声明了数字类型，如果我们使用了字符串的方法, TypeScript 就会报错。

但问题在于，即便我们不使用，但接口的返回数据类型突然改了呢？比如本来是 Number 类型，后端改为了 String 类型？因为 String 类型没有 toFixed 方法，那这段代码运行的时候就会报错。

为了防止运行时产生问题，我们还需要做判断，比如：

```typescript
type Product = {
  price: number;
};

export async function GET() {
  const res = await fetch("/api/product");
  const data = (await res.json()) as Product;

  if (data && data.price && typeof data.price == "number") {
    const showPrice = data.price.toFixed(2);
    return Response.json({ showPrice });
  } else {
    return Response.json({ success: false });
  }
}
```

如果涉及的字段众多，每个字段都写一段校验，代码很快就会变得臃肿难以维护。

而 Zod 正好可以解决这一问题，使用 Zod 后，代码改为：

```javascript
import { z } from "zod";

const schema = z.object({
  price: z.number(),
});

export async function GET() {
  const res = await fetch("/api/product");
  const data = await res.json();

  const parsedData = schema.safeParse(data);

  if (parsedData.success) {
    const showPrice = parsedData.data.price.toFixed(2);
    return Response.json({ showPrice });
  } else {
    return Response.json({ success: false });
  }
}
```

整体代码更加简洁优雅，而且你也不需要再写类型声明：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/100aa0e4055142aa92fc9aa1d83ff7dd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1824&h=588&s=153334&e=png&b=1f1f1f)

先对接口返回的数据进行校验，通过后再进行后续操作，从源头上防止数据不合法而导致的错误，提高应用的稳定性，而且还能帮助 TypeScript 进行推断，使用起来非常方便。

### 2.3. 如何学习 Zod

那具体如何学习 Zod 呢？就我个人看法，学习 Zod 的最好方法就是看 Zod 的官方文档：

1. 英文：[zod.dev/](https://zod.dev/ "https://zod.dev/")
2. 中文：[zod.dev/README\_ZH](https://zod.dev/README_ZH "https://zod.dev/README_ZH")

内容并不算多， 20 分钟就可以看个大概。首要目的是了解 Zod 有哪些功能，具体要用的时候边查文档边学习。

如果文档看不下去，这是一个 30 分钟学习 Zod 的 [Youtube 视频](https://www.youtube.com/watch?v=L6BE-U3oy80&ab_channel=WebDevSimplified "https://www.youtube.com/watch?v=L6BE-U3oy80&ab_channel=WebDevSimplified")。

## 3\. Svelte + Zod

### 3.1. 项目初始化

运行：

```bash
npm create svelte@latest svelte-todolist
```

选择 **Skeleton project、TypeScript、Svelte 5：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/347ed7f2473547d59e3cfc9140847e71~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2642&h=854&s=157406&e=png&b=1e1e1e)

按照命令行中的提示提交代码：

```bash
cd svelte-todolist

npm install

git init && git add -A && git commit -m "Initial commit"
```

运行：

```bash
npx svelte-add
```

选择 Tailwind CSS：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39ec64bbbc694829bb223212c975a4e0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1760&h=790&s=114693&e=png&b=1e1e1e)

查看此时的 Git 提交记录，可以清晰的看到添加的内容：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/467e78623def4b4abe6d1f61a28bb80a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1936&h=498&s=193572&e=png&b=f6f2ee)

开启本地开发模式：

```bash
npm run dev -- --open
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4952bc70f80c41ae9047718970d65275~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1464&h=388&s=48430&e=png&b=fefefe)

### 3.2. 使用 Zod

安装 Zod：

```html
npm install zod
```

我们以注册页面为例为大家进行讲解：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/113905b6d6e74e81937bd0db7aa06df8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1650&h=906&s=62678&e=png&b=ffffff)

新建 `src/lib/types.ts`，代码如下：

```typescript
import { z } from "zod";

export const UserSchema = z.object({
  username: z.string().min(2, "用户名最少 2 位"),
  password: z
    .string()
    .regex(new RegExp("^(?=.*?[A-Z])(?=.*?[a-z])(?=.*?[0-9]).{8,}$"), {
      message: "密码至少 8 位，包含一个大写字母，一个小写字母和一个数字",
    }),
});
```

这里我们声明了登录时提交数据的 Schema，之所以将其单独抽离，是因为数据校验往往是前后端都需要的。前端做校验是因为要避免浪费服务端资源，后端做校验是因为不能信任来自前端的数据。

新建 `src/routes/signup/+page.svelte`，代码如下：

```xml
<script lang="ts">
  import type { ZodIssue } from 'zod';
  import ZodIssues from '$lib/components/ZodIssues.svelte';
  import { enhance } from '$app/forms';
  import { UserSchema } from '$lib/types';

  let issues: ZodIssue[] = $state([]);
</script>

<div class="flex min-h-full flex-1 flex-col justify-center px-6 py-12 lg:px-8">
  <form
    method="POST"
    use:enhance={({ formElement, formData, action, cancel, submitter }) => {
      const result = UserSchema.safeParse(Object.fromEntries(formData));
      if (!result.success) {
        issues = result.error.issues;
        cancel();
      }
      return async ({ result, update }) => {
        if (result.type == 'failure') {
          issues = result.data!.issues as ZodIssue[];
        }
        await update();
      };
    }}
  >
    <div class="mb-2">
      <label for="username" class="mb-2 block text-sm font-medium leading-6 text-gray-900">
        用户名
      </label>
      <input
        id="username"
        name="username"
        type="text"
        required
        class="block w-full rounded-md border-0 px-2 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
      />
    </div>

    <div class="mb-4">
      <div class="flex items-center justify-between">
        <label for="password" class="block text-sm font-medium leading-6 text-gray-900">
          密码
        </label>
      </div>
      <div class="mt-2">
        <input
          id="password"
          name="password"
          type="password"
          required
          autoComplete="current-password"
          class="block w-full rounded-md border-0 px-2 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
        />
      </div>
    </div>

    <button
      type="submit"
      class="mb-4 flex w-full justify-center rounded-md bg-indigo-600 px-3 py-1.5 text-sm font-semibold leading-6 text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
    >
      注册
    </button>
  </form>
  <ZodIssues {issues} />
</div>
```

这是实现数据校验的核心代码，主要是使用 `use:enhance`实现提交前的数据验证，如果无法通过 Zod 校验，就取消提交数据。如果通过，再根据返回结果展示数据。

新建 `src/routes/signup/+page.server.ts`，代码如下：

```xml
import { fail, redirect } from '@sveltejs/kit';
import { UserSchema } from '$lib/types';

export const actions = {
  default: async ({ request }) => {
    const formData = Object.fromEntries(await request.formData());

    const safeParse = UserSchema.safeParse(formData);

    if (!safeParse.success) {
      return fail(400, { issues: safeParse.error.issues });
    }

    redirect(303, '/');
  }
};
```

这段代码很简单，主要是用 Zod 做数据校验，如果失败，就返回错误信息。如果成功，就跳转到 `/`。

新建 `src/lib/components/ZodIssues.svelte`，代码如下：

```xml
<script lang="ts">
  import type { ZodIssue } from 'zod';

  let { issues }: { issues: ZodIssue[] } = $props();
</script>

<ul>
  {#each issues as { message, path }}
    <li>{path[0]} - {message}</li>
  {/each}
</ul>
```

这是为了方便展示 Zod 返回的错误。

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e1d9ab637444cf3ae4144fac1beb7da~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=817&h=538&s=72022&e=gif&f=43&b=fefefe)

## 4\. 最后

本篇我们介绍了 Zod 的背景和用途，并实现了一个 Svelte + Zod 的全栈例子。常见的全栈框架如 Next.js、SvelteKit 都常使用 Zod。但在实际项目开发中，我们往往并不只用 Zod，比如 Next.js 会使用 Shadcn UI + React Hook Form + Zod 的组合拳，在 Svelte 中，常会用 SuperForm + Zod，这正是下篇我们要介绍的内容。