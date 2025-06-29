> 推荐学习指数：⭐️️⭐️️⭐️️

## 1\. 前言

Superforms 是一个 SvelteKit 表单库，提供了服务端和客户端表单验证的全面解决方案，支持 Arktype、Joi、JSON Schema、Superstruct、TypeBox、Valibot、VineJS、Yup、Zod 等多种验证库。

它是 [Sveltehack 2023](https://hack.sveltesociety.dev/winners "https://hack.sveltesociety.dev/winners") 年 Best Library 的冠军，目前 GitHub 2.1k Star，是 SvelteKit 项目做表单校验的常用选择。

根据 [GitHub](https://github.com/ciscoheat/sveltekit-superforms?tab=readme-ov-file#feature-list "https://github.com/ciscoheat/sveltekit-superforms?tab=readme-ov-file#feature-list") 上的介绍，它具有以下这些特性：

1. 自由选择最喜欢的验证库进行数据验证
2. 无缝合并 `PageData` 和 `ActionData`，只需专注于表单数据
3. 自动居中并聚焦无效的表单字段
4. 自动转换 `FormData`为正确的类型
5. 从 schema 生成默认表达值
6. 支持同页面多表单处理
7. 即可在服务端运行，也可以和 SPA 意使用
8. 实时客户端验证
9. 轻松创建 Loading
10. 大量的定制选项
11. 默认不需要 JavaScript，支持渐进式增强
12. 自带 SuperDebug 组件
13. ……

简而言之，功能强大，那就让我们开始使用吧！

## 2\. 安装 Superforms

安装：

```bash
npm i sveltekit-superforms
```

数据校验库选择自己喜欢的就行，我们以 Zod 为例：

```bash
npm i zod
```

## 3\. 基础使用方式

### 3.1. 初始化表单

首先是初始化表单，修改 `+page.server.ts`，代码如下：

```typescript
import type { PageServerLoad } from "./$types.js";
import { superValidate } from "sveltekit-superforms";
import { zod } from "sveltekit-superforms/adapters";
import { UserSchema } from "$lib/types";

export const load: PageServerLoad = async () => {
  return { form: await superValidate(zod(UserSchema)) };
};
```

> 注：这里只是为了讲解如何使用而写的演示代码，下节会写具体的例子

Superform 的服务端 API 叫 `superValidate`。这步是为了设置表单的初始值，Superforms 会根据 Schema 自动生成默认值，比如你在 schema 中：

```typescript
import { z } from "zod";

export const UserSchema = z.object({
  username: z.string().min(2, "用户名最少 2 位").default("hello world"),
});
```

与表单元素绑定后，它就会作为表单的初始值：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59bc5bca5a37403d9273f2407e36808d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1608&h=644&s=47462&e=png&b=ffffff)

因为 load 函数运行在服务端，所以也可以从服务端（比如数据库）获取数据，然后设置初始值，此时需要将数据作为第一个参数，第二个参数作为适配器：

```typescript
export const load: PageServerLoad = async () => {
  return {
    form: await superValidate(
      {
        username: '123',
        password: '123123'
      },
      zod(UserSchema)
    )
  };
};
```

与表单元素绑定后，它就会作为表单的初始值：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8762c688d2da4b8eb436967a0c844e66~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1604&h=656&s=45181&e=png&b=ffffff)

### 3.2. 绑定表单元素

然后是绑定表单元素，修改 `+page.svlete`：

```xml
<script lang="ts">
  import { superForm } from 'sveltekit-superforms';
  const { data } = $props();

  // Client API:
  const { form } = superForm(data.form);
</script>

<form method="POST">
  <label for="name">Name</label>
  <input type="text" name="name" bind:value={$form.name} />

  <label for="email">E-mail</label>
  <input type="email" name="email" bind:value={$form.email} />

  <div><button>Submit</button></div>
</form>
```

`superForm` 函数用于在客户端创建表单，`bind:value`用于创建表单数据和输入字段的双向绑定。注意表单元素要有 `name` 属性，这是提交的表单数据的键值。

### 3.3. Debugging

SuperForm 提供了 `<SuperDebug />`组件用于查看表单数据：

```xml
<script lang="ts">
  import SuperDebug from 'sveltekit-superforms';
</script>

<SuperDebug data={$form} />
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f74396a0e954ba1965139537bd9d7ca~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1920&h=1034&s=94990&e=png&b=ffffff)

当修改表单字段时，组件中的数据也会自动更新。

### 3.4. 提交数据

最后让我们看看服务端该如何处理。修改 `+page.server.ts`，代码如下：

```typescript
import { fail } from "@sveltejs/kit";
import { superValidate, message } from "sveltekit-superforms";
import { zod } from "sveltekit-superforms/adapters";

export const actions = {
  default: async ({ request }) => {
    const form = await superValidate(request, zod(UserSchema));
    console.log(form);

    if (!form.valid) {
      return fail(400, { form });
    }

    return message(form, "Form posted successfully!");
  },
};
```

这是错误时 `form` 的数据结构：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6fe5c97939d044a594e248078f3b5b8c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2080&h=492&s=100551&e=png&b=1e1e1e)

这是成功时 `form` 的数据结构：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b09e615124e3460799a742f19c204393~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1932&h=354&s=57353&e=png&b=1e1e1e)

### 3.5. 错误展示

最后让我们看看如何展示错误。修改 `+page.svelte`，代码如下：

```typescript
<script lang="ts">
  const { form, errors, constraints, message } = superForm(data.form);
</script>

{#if $message}<h3>{$message}</h3>{/if}

<form method="POST">
  <label for="name">Name</label>
  <input
    type="text"
    name="name"
    aria-invalid={$errors.name ? 'true' : undefined}
    bind:value={$form.name}
    {...$constraints.name}
  />
  {#if $errors.name}<span>{$errors.name}</span>{/if}

  <label for="email">E-mail</label>
  <input
    type="email"
    name="email"
    aria-invalid={$errors.email ? 'true' : undefined}
    bind:value={$form.email}
    {...$constraints.email}
  />
  {#if $errors.email}<span>{$errors.email}</span>{/if}

  <div><button>Submit</button></div>
</form>
```

其中 `errors`用于获取 `fail()` 返回的错误信息，`message`用于获取 `message()`返回的消息，`$constraints`用于提供浏览器验证，后面还会讲到。

现在我们就应该有了一个可以正常工作的表单。

### 3.6. 实例演示

现在让我们重写一下上篇 Zod 中的账号注册例子。

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

新建 `src/routes/signup2/+page.svelte`，代码如下：

```typescript
<script lang="ts">
  import { page } from '$app/stores';
  import { superForm } from 'sveltekit-superforms';
  import SuperDebug from 'sveltekit-superforms';

  const { data } = $props();
  const { form, errors, message, enhance } = superForm(data.form);
</script>

<div class="flex min-h-full flex-1 flex-col justify-center gap-4 p-6">
  <SuperDebug data={$form} />
  <form method="POST" use:enhance>
    <div class="mb-2">
      <label for="username" class="mb-2 block text-sm font-medium leading-6 text-gray-900">
        用户名
      </label>
      <input
        type="text"
        name="username"
        class="mb-2 block w-full rounded-md border-0 px-2 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
        aria-invalid={$errors.username ? 'true' : undefined}
        bind:value={$form.username}
      />
      {#if $errors.username}<span class="text-sm text-red-500">{$errors.username}</span>{/if}
    </div>

    <div class="mb-4">
      <div class="flex items-center justify-between">
        <label for="password" class="block text-sm font-medium leading-6 text-gray-900">
          密码
        </label>
      </div>
      <div class="mt-2">
        <input
          type="password"
          name="password"
          class="mb-2 block w-full rounded-md border-0 px-2 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
          aria-invalid={$errors.password ? 'true' : undefined}
          bind:value={$form.password}
        />
        {#if $errors.password}<span class="text-sm text-red-500">{$errors.password}</span>{/if}
      </div>
    </div>

    <button
      type="submit"
      class="mb-4 flex w-full justify-center rounded-md bg-indigo-600 px-3 py-1.5 text-sm font-semibold leading-6 text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
    >
      注册
    </button>
    {#if $message}
      <div class="status" class:error={$page.status >= 400} class:success={$page.status == 200}>
        {$message}
      </div>
    {/if}
  </form>
</div>
```

新建 `src/routes/signup2/+page.server.ts`，代码如下：

```typescript
import type { Actions, PageServerLoad } from "./$types.js";
import { fail } from "@sveltejs/kit";
import { superValidate, message } from "sveltekit-superforms";
import { zod } from "sveltekit-superforms/adapters";
import { UserSchema } from "$lib/types";

export const load: PageServerLoad = async () => {
  return {
    form: await superValidate(zod(UserSchema)),
  };
};

export const actions: Actions = {
  default: async ({ request }) => {
    const form = await superValidate(request, zod(UserSchema));
    console.log(form);

    if (!form.valid) return fail(400, { form });

    return message(form, "账号注册成功!");
  },
};
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0eff5a90e27f4c57baae648452b2f612~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1187&h=592&s=169161&e=gif&f=55&b=2b2b2b)

效果描述：每次点击注册按钮的时候，数据都会提交给服务端进行验证。客户端根据响应展示不同的结果。

## 4\. 进阶使用方式

### 4.1. 客户端验证

上节我们实现的效果是每次提交都会在服务端进行验证，为了避免服务端资源浪费，如何开启客户端验证呢？

在 Superforms 中有两个客户端验证方式：

1. 内置浏览器验证：也就是浏览器自带的验证，好处在于甚至不需要启用 JavaScript，坏处在于无法进行自定义验证
2. 使用校验 schema：通常与服务端校验 schema 一致，需要 JavaScript 和 use:enhance

现在让我们具体看下如何使用。

#### 4.1.1. 内置浏览器校验

使用内置浏览器校验，只需要在输入字段上添加 `$constraints`Store：

```typescript
<script lang="ts">
  export let data;
  const { form, constraints } = superForm(data.form);
</script>

<input
  name="email"
  type="email"
  bind:value={$form.email}
  {...$constraints.email} />
```

`$constraints` 是一个对象，由 schema 生成而来。

比如 schema 为：

```typescript
export const UserSchema = z.object({
  username: z.string().min(2, "用户名最少 2 位"),
});
```

`Superforms` 会对应为绑定的元素生成这些属性：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f090ee980b0846fb977348d0b57018a1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1250&h=170&s=72550&e=png&b=2a2a2a)

schema 为：

```typescript
export const UserSchema = z.object({
  password: z
    .string()
    .regex(new RegExp("^(?=.*?[A-Z])(?=.*?[a-z])(?=.*?[0-9]).{8,}$"), {
      message: "密码至少 8 位，包含一个大写字母，一个小写字母和一个数字",
    }),
});
```

`Superforms` 会对应为绑定的元素生成这些属性：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2239f536463449ed9084a5b199727801~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1392&h=160&s=79810&e=png&b=282828)

也就是说，Superforms 借助浏览器提供的原生校验 API 进行校验，根据 schema 生成元素属性。

这样做的好处在于甚至不需要 JavaScript 也可以进行校验。坏处在于无法控制错误信息的位置、样式，也无法进行一些自定义验证。

如果你要自定义这些信息，就需要借助第二种客户端校验方式 —— 使用校验 schema。

#### 4.1.2. 使用校验 schema

使用校验 schema，以 Zod 为例：

```typescript
import { zodClient } from "sveltekit-superforms/adapters";
import { UserSchema } from "$lib/types";

const { form, errors, message, enhance, constraints } = superForm(data.form, {
  validators: zodClient(UserSchema),
});
```

此时就会为绑定的元素添加上客户端校验。

> 注意：这仅适用于与服务端使用相同的 schema 时。如果使用不同的模式，则需要完整的适配器，比如用 zod 替代 zodClient

#### 4.1.3. 实例演示

现在让我们修改上节的例子。修改 `src/routes/signup2/+page.svelte`，代码如下：

```xml
<script lang="ts">
  import { page } from '$app/stores';
  import { superForm } from 'sveltekit-superforms';
  import SuperDebug from 'sveltekit-superforms';
  import { zodClient } from 'sveltekit-superforms/adapters';
  import { UserSchema } from '$lib/types';

  const { data } = $props();
  const { form, errors, message, enhance, constraints } = superForm(data.form, {
    validators: zodClient(UserSchema)
  });
</script>

<div class="flex min-h-full flex-1 flex-col justify-center gap-4 p-6">
  <SuperDebug data={$form} />
  <form method="POST" use:enhance>
    <div class="mb-2">
      <label for="username" class="mb-2 block text-sm font-medium leading-6 text-gray-900">
        用户名
      </label>
      <input
        type="text"
        name="username"
        autocomplete="off"
        class="mb-2 block w-full rounded-md border-0 px-2 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
        aria-invalid={$errors.username ? 'true' : undefined}
        bind:value={$form.username}
        {...$constraints.username}
      />
      {#if $errors.username}<span class="text-sm text-red-500">{$errors.username}</span>{/if}
    </div>

    <div class="mb-4">
      <div class="flex items-center justify-between">
        <label for="password" class="block text-sm font-medium leading-6 text-gray-900">
          密码
        </label>
      </div>
      <div class="mt-2">
        <input
          type="password"
          name="password"
          autocomplete="off"
          class="mb-2 block w-full rounded-md border-0 px-2 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
          aria-invalid={$errors.password ? 'true' : undefined}
          bind:value={$form.password}
          {...$constraints.password}
        />
        {#if $errors.password}<span class="text-sm text-red-500">{$errors.password}</span>{/if}
      </div>
    </div>

    <button
      type="submit"
      class="mb-4 flex w-full justify-center rounded-md bg-indigo-600 px-3 py-1.5 text-sm font-semibold leading-6 text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
    >
      注册
    </button>
    {#if $message}
      <div class="status" class:error={$page.status >= 400} class:success={$page.status == 200}>
        {$message}
      </div>
    {/if}
  </form>
</div>
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c14a000e8c2140dbbc92fdb8bd977808~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1187&h=592&s=273739&e=gif&f=85&b=2b2b2b)

这是一个拥有两种客户端校验的例子。一开始数据为空，直接提交的时候，触发的是浏览器内置校验。当输入字段失焦后，触发的是 schema 校验。

**默认的校验方式是“尽早奖励、延迟验证”，这是一个经过研究的校验输入数据的方法，旨在提高用户满意度：**

---

1. 如果在有错误的字段上输入数据，在 `input`的时候校验
2. 否则，在 `blur`的时候校验

意思是：当用户第一次输入数据的时候，在 blur 的时候校验数据，如果数据发生错误，用户再输入的时候，在输入的时候就进行校验，而非等到失焦的时候。**这就是“尽早奖励、延迟验证”。**

现在的这个例子就是这个校验逻辑。

### 4.2. 加载状态

加载状态用于在服务端响应延迟时提供反馈。默认设置如下：

```javascript
const { form, enhance, submitting, delayed, timeout } = superForm(data.form, {
  delayMs?: 500
  timeoutMs?: 8000
})
```

Superforms 认为在短等待时间内，除了显示结果外，不需要任何反馈，意思是如果接口返回的快，那就不要 loading 效果了。所以才有一个 delayMs 设置，默认是 500ms，也就是提交后，如果超过了 500ms，还没有返回结果，那就最好显示加载状态。使用如下：

```xml
<script lang="ts">
  const { form, errors, enhance, delayed } = superForm(data.form);
  import spinner from '$lib/assets/spinner.svg';
</script>

<form method="POST" use:enhance>
  <button>Submit</button>
  {#if $delayed}<img src={spinner} />{/if}
</form>
```

#### 4.2.1. 实例演示

修改 `src/routes/signup2/+page.svelte`，完整代码如下：

```xml
<script lang="ts">
  import { page } from '$app/stores';
  import { superForm } from 'sveltekit-superforms';
  import SuperDebug from 'sveltekit-superforms';
  import { zodClient } from 'sveltekit-superforms/adapters';
  import { UserSchema } from '$lib/types';

  const { data } = $props();
  const { form, errors, message, enhance, constraints, delayed } = superForm(data.form, {
    validators: zodClient(UserSchema)
  });
</script>

<div class="flex min-h-full flex-1 flex-col justify-center gap-4 p-6">
  <SuperDebug data={$form} />
  <form method="POST" use:enhance>
    <div class="mb-2">
      <label for="username" class="mb-2 block text-sm font-medium leading-6 text-gray-900">
        用户名
      </label>
      <input
        type="text"
        name="username"
        autocomplete="off"
        class="mb-2 block w-full rounded-md border-0 px-2 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
        aria-invalid={$errors.username ? 'true' : undefined}
        bind:value={$form.username}
        {...$constraints.username}
      />
      {#if $errors.username}<span class="text-sm text-red-500">{$errors.username}</span>{/if}
    </div>

    <div class="mb-4">
      <div class="flex items-center justify-between">
        <label for="password" class="block text-sm font-medium leading-6 text-gray-900">
          密码
        </label>
      </div>
      <div class="mt-2">
        <input
          type="password"
          name="password"
          autocomplete="off"
          class="mb-2 block w-full rounded-md border-0 px-2 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
          aria-invalid={$errors.password ? 'true' : undefined}
          bind:value={$form.password}
          {...$constraints.password}
        />
        {#if $errors.password}<span class="text-sm text-red-500">{$errors.password}</span>{/if}
      </div>
    </div>

    <button
      type="submit"
      class="mb-4 flex w-full justify-center rounded-md bg-indigo-600 px-3 py-1.5 text-sm font-semibold leading-6 text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
    >
      注册{#if $delayed}中……{/if}
    </button>

    {#if $message}
      <div class="status" class:error={$page.status >= 400} class:success={$page.status == 200}>
        {$message}
      </div>
    {/if}
  </form>
</div>
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98dd9498df384f34b2ff89aab052f3ac~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=781&h=569&s=66581&e=gif&f=40&b=fefefe)

此为接口响应时间较长时的效果。可通过 `sleep` 函数模拟，修改 `src/routes/signup2/+page.server.ts`，代码如下：

```typescript
import type { Actions, PageServerLoad } from "./$types.js";
import { fail } from "@sveltejs/kit";
import { superValidate, message } from "sveltekit-superforms";
import { zod } from "sveltekit-superforms/adapters";
import { UserSchema } from "$lib/types";

const sleep = (ms: number) => new Promise((resolve) => setTimeout(resolve, ms));

export const load: PageServerLoad = async () => {
  return {
    form: await superValidate(zod(UserSchema)),
  };
};

export const actions: Actions = {
  default: async ({ request }) => {
    await sleep(3000);
    const form = await superValidate(request, zod(UserSchema));
    console.log(form);

    if (!form.valid) return fail(400, { form });

    return message(form, "账号注册成功!");
  },
};
```

> 注意：默认会阻止表单多次提交

## 5\. 总结

Superforms 功能强大，还有很多其他功能比如 Event、嵌套数据等内容没有讲解。具体查看[官方文档](https://superforms.rocks/get-started "https://superforms.rocks/get-started")。SvelteKit 项目做表单验证，Superforms + Zod 是一个常见的组合拳。