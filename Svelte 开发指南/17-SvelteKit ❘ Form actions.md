> 推荐学习指数：⭐️⭐⭐️，必学内容️

## 1\. 前言

`+page.server.js`可以导出 *actions*，所谓 actions，本质是一个异步函数。开发者使用 `<form>` 元素将数据以 POST 请求的方式发送给服务端，服务端使用 actions 进行处理。

为了展现使用 Form actions 的优势，我们先看下传统的实现方式。

### 1.1. 传统实现方式

新建 `/src/routes/signin/+page.svelte`，代码如下：

```xml
<script>
  let username = $state('');
  let password = $state('');
  let result = $state({});

  async function submit() {
    const response = await fetch('/api/add', {
      method: 'POST',
      body: JSON.stringify({ username, password }),
      headers: {
        'content-type': 'application/json'
      }
    });

    result = await response.json();
  }
</script>

<label class="block text-sm text-gray-900 mb-2">
  UserName
  <input
    type="text"
    bind:value={username}
    class="block w-full rounded-md border-0 p-1.5 mt-2 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400"
  />
</label>

<label class="block text-sm text-gray-900 mb-2">
  Password
  <input
    type="password"
    bind:value={password}
    class="block w-full rounded-md border-0 p-1.5 mt-2 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400"
  />
</label>

<button
  onclick={submit}
  class="flex w-full justify-center rounded-md bg-indigo-600 px-3 py-1.5 text-sm text-white shadow-sm"
  >提交</button
>

{#if result?.success}
  <p class="text-sm text-gray-900 mt-2">登录成功!</p>
{/if}
```

新建 `src/routes/api/add/+server.js`，代码如下：

```javascript
import { json } from "@sveltejs/kit";

export async function POST({ request }) {
  const data = await request.json();
  return json({
    success: true,
    data,
  });
}
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b5f56518a436494a8cb56381de55f75b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1138&h=404&s=130263&e=gif&f=53&b=2c2c2c)

先通过 `+server.js`声明一个 API 接口，然后点击提交按钮，获取表单数据，调用接口，根据接口响应进行处理。这就是最常见的前后端交互方式。

### 1.2. Actions 实现方式

现在我们使用 Form actions 来实现一遍。

修改 `src/routes/signin/+page.svelte`，代码如下：

```xml
<script>
  import { enhance } from '$app/forms';
  const { form } = $props();
</script>

<form method="POST" use:enhance>
  <label class="block text-sm text-gray-900 mb-2">
    UserName
    <input
      type="text"
      name="username"
      class="block w-full rounded-md border-0 p-1.5 mt-2 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400"
    />
  </label>

  <label class="block text-sm text-gray-900 mb-2">
    Password
    <input
      type="password"
      name="password"
      class="block w-full rounded-md border-0 p-1.5 mt-2 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400"
    />
  </label>

  <button
    type="submit"
    class="flex w-full justify-center rounded-md bg-indigo-600 px-3 py-1.5 text-sm text-white shadow-sm"
    >提交</button
  >
</form>

{#if form?.success}
  <p class="text-sm text-gray-900 mt-2">登录成功!</p>
{/if}
```

新建 `src/routes/signin/+page.server.js`，代码如下：

```javascript
export const actions = {
  default: async ({ request }) => {
    const formData = await request.formData();
    const data = Object.fromEntries(formData);
    return {
      success: true,
      data,
    };
  },
};
```

浏览器效果与上节一致：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/001e68cd5c4b4c68b5a0e6bfff04066f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1138&h=404&s=94755&e=gif&f=37&b=2c2c2c)

不需要提前声明一个接口，也不需要 `await fetch(xxx)` 这样的样板代码，只需要在 `+page.server.js`导出一个异步函数即可。当提交表单的时候，会发送一个 POST 请求到当前页面，由 action 函数进行处理。

使用 `<form action="xxx">` 的一大好处在于即便禁用 JavaScript，表单也是可以正常提交的，效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30572f0b3d8b4534a4f2065b91af5672~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1141&h=418&s=48653&e=gif&f=21&b=292929)

注意禁用和不禁用时的差别：禁用 JavaScript 时，表单提交后页面会刷新，不禁用 JavaScript 时，表单提交后页面并无刷新行为。

现在让我们来看下 action 的具体用法和注意事项。

## 2\. 默认 action

使用 action，最简单的方法是在 `+page.server.js` 声明一个 `default` action：

```javascript
// src/routes/login/+page.server.js
export const actions = {
  default: async (event) => {
    // ...
  },
};
```

此时 action 对应的路由是 `/login`。如果从 `/login` 页面调用此 action，只需添加 `<form>`元素：

```xml
<!-- src/routes/login/+page.svelte -->
<form method="POST">
  <label>
    Email
    <input name="email" type="email">
  </label>
  <label>
    Password
    <input name="password" type="password">
  </label>
  <button>Log in</button>
</form>
```

如果有人点击按钮，浏览器就会将数据以 POST 请求的方式发送给服务端，并运行 `default` action。

> 注意：此时提交表单，页面会刷新，因为表单默认不会渐进式增强。参考本篇的“渐进式增强”章节。

当 `<form>` 的 `action` 属性为空时，数据默认会提交给当前页面，所以在上面的例子中我们不写 `action` 属性也没有关系。但如果从其他页面调用此 `action`，则需要指定 `action` 属性：

```xml
<!-- src/routes/+layout.svelte -->
<form method="POST" action="/login">
  <!-- content -->
</form>
```

在这个例子中，当前布局的路由地址是 `/`，调用 `/login` 路由下的 action，需要通过 `<form>` action 属性指定调用的 action 所在的路由位置

> 注意：actions 将始终使用 POST 请求，因为 GET 请求不应该有副作用

## 3\. 命名 action

一个页面可以拥有多个命名 action：

```javascript
// src/routes/login/+page.server.js
export const actions = {
  login: async (event) => {
    // TODO log the user in
  },
  register: async (event) => {
    // TODO register the user
  },
};
```

要调用命名 action，需要添加一个以 `/`作为前缀字符的查询参数。举两个例子：

```xml
<!-- src/routes/login/+page.svelte -->
<form method="POST" action="?/register">
```

```xml
<!-- src/routes/+layout.svelte -->
<form method="POST" action="/login?/register">
```

选择 `/`作为前缀稍微有些奇怪，第一次接触的时候可能有些不明所以。以 `/login?/register`为例，`/login`是路由地址，`?` 表示后面的都是查询参数，`/register`是查询参数，其中 `/`是 action 约定的前缀字符，`/register`表示请求该路由下的 `register` 命名 action。

`action` 除了用在 `<form>` 元素的 `action` 属性，也可以用在 `formaction` 属性：

```xml
<form method="POST" action="?/login">
  <label>
    Email
    <input name="email" type="email">
  </label>
  <label>
    Password
    <input name="password" type="password">
  </label>
  <button>Log in</button>
  <button formaction="?/register">Register</button>
</form>
```

这是一个很好的例子，借助 `formaction`属性，相同的表单数据可以提交给不同的 action 进行处理。

> 注意：命名 action 和默认 action 不能同时设置，否则会有报错：When using named actions, the default action cannot be used.

## 4\. action 函数

现在让我们具体看下 action 函数的用法。

### 4.1. 输入输出

action 接收一个 [RequestEvent](https://kit.svelte.dev/docs/types#public-types-requestevent "https://kit.svelte.dev/docs/types#public-types-requestevent") 对象作为参数，RequestEvent 对象之前讲过，`+server.js`导出的函数接收的正是 RequestEvent 对象，server load 函数接收的是一个继承自 RequestEvent 的对象。有这些属性：

```javascript
{
  cookies: {
    get: [Function: get],
    getAll: [Function: getAll],
    set: [Function: set],
    delete: [Function: delete],
    serialize: [Function: serialize]
  },
  fetch: [Function (anonymous)],
  getClientAddress: [Function: getClientAddress],
  locals: {},
  params: {},
  platform: undefined,
  request: Request,
  route: { id: '/sign' },
  setHeaders: [Function: setHeaders],
  url: URL,
  isDataRequest: false,
  isSubRequest: false
}
```

action 返回的数据可以在对应的页面通过 `form` 属性或 `$page.form` 获取。我们展示下用法：

```javascript
// src/routes/login/+page.server.js
export async function load({ cookies }) {
  const user = await db.getUserFromSession(cookies.get("sessionid"));
  return { user };
}

export const actions = {
  login: async ({ cookies, request }) => {
    // 通过 request.formData() 获取数据
    const data = await request.formData();
    const email = data.get("email");
    const password = data.get("password");

    const user = await db.getUser(email);
    // 设置 sessionid
    cookies.set("sessionid", await db.createSession(user), { path: "/" });

    return { success: true };
  },
  register: async (event) => {
    // TODO register the user
  },
};
```

```xml
<!-- src/routes/login/+page.svelte -->
<script>
  import { page } from '$app/stores';
  const { form, data } = $props();
</script>

<!-- 第一种方式获取 action 返回数据 -->
{#if form?.success}
  <p class="text-sm text-gray-900 mt-2">登录成功! {data.user.name}</p>
{/if}

<!-- 第二种方式获取 action 返回数据 -->
{#if $page.form?.success}
  <p class="text-sm text-gray-900 mt-2">登录成功! {data.user.name}</p>
{/if}
```

### 4.2. 错误校验

action 常需要做数据校验，当数据无效时，需要给与用户错误提示。此时可以借助`@sveltejs/kit` 提供的`fail`函数，该函数会返回数据和一个 HTTP 状态码（对于验证错误，通常错误码为 400 或 422）。状态码可通过 `$page.status` 获取，数据可通过 `form` 属性获取：

修改 `src/routes/signin/+page.server.js`，完整代码如下：

```javascript
import { fail } from "@sveltejs/kit";

export const actions = {
  login: async ({ request }) => {
    const formData = await request.formData();
    const username = formData.get("username");

    if (!username) {
      return fail(400, { missing: true });
    }

    return {
      success: true,
      data: Object.fromEntries(formData),
    };
  },
};
```

`src/routes/signin/+page.svelte`，添加代码如下：

```xml
<script>
  const { form } = $props();
  import { page } from '$app/stores';
</script>

{#if $page.status == 400}
  {#if form?.missing}
    <p class="text-sm text-red-500 mt-2">字段缺失，请检查！</p>
  {/if}
{/if}
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37a91f3fbfff4b3aad094b115641d92a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2436&h=768&s=243821&e=png&b=2e2e2e)

> 注意：fail 函数返回的数据必须是可序列化的

### 4.3. 重定向

如果需要重定向，使用 `@sveltejs/kit`的 `redirect` 函数，用法跟在 load 函数中相同。

修改 `src/routes/signin/+page.server.js`，代码如下：

```diff
+import { fail, redirect } from '@sveltejs/kit';

export const actions = {
  login: async ({ request, url }) => {
    const formData = await request.formData();
    const username = formData.get('username');

    if (!username) {
      return fail(400, { missing: true });
    }

+   if (url.searchParams.has('redirectTo')) {
+     redirect(303, url.searchParams.get('redirectTo'));
+   }

    return {
      success: true,
      data: Object.fromEntries(formData)
    };
  }
};

```

在这段代码中，我们会获取网址中的 `redirectTo` 查询参数，然后判断完成后是否重定向。但是注意这里的 url 并不是指页面的 url，而是 form action 的 url，所以要想让重定向生效，需要这样写：

```xml
<form method="POST" action="?/login&redirectTo=/new" use:enhance>
  // ...
</form>
```

### 4.4. 数据加载

action 运行后，页面会重新渲染（除非发生重定向或预期之外的错误），action 的返回值会以 `form` prop 的形式传给页面，这意味着页面的 load 函数会在 action 完成后运行。

## 5\. 渐进式增强

如果我们直接使用 `<form>` 元素：

```html
<form method="POST">// ...</form>
```

此时表单提交，页面会刷新，即使没有客户端 JavaScript 也可以正常运行。但是当 JavaScript 可用时，我们可以渐进式增强表单交互，提供更好的用户体验。

### 5.1. use:enhance

最简单的方法是添加 `use:enhance` Action：

```xml
<script>
  import { enhance } from '$app/forms';
</script>

<form method="POST" use:enhance>
```

`use:enhance` 将模拟浏览器原生行为：

1. 更新页面属性 `form`、`$page.form`、`$page.status`
2. 重置 `<form>` 元素
3. 成功响应的时候，调用 `invalidateAll`重新加载数据
4. 针对重定向的响应调用 `$app/navigation`的`goto`方法
5. 发生错误时渲染最近的错误边界
6. 将焦点重置为合适的元素

模拟的好处在于不会重新加载页面，提升性能，且保持页面其他部分状态不变。

`use:enhance` 支持传入一个函数用于自定义提交逻辑。该函数会在表单提交之前执行，如果返回一个回调函数，该回调函数会在 action 返回之后执行：

```jsx
<form
  method="POST"
  use:enhance={({ formElement, formData, action, cancel, submitter }) => {
    // `formElement` 是 `<form>` 元素
    // `formData` 是提交的 `FormData`对象
    // `action` 是 action URL
    // 调用 `cancel()` 会阻止提交
    // `submitter` 是导致表单提交的 `HTMLElement`

    // 可选，返回一个回调函数时将阻止默认行为
    return async ({ result, update }) => {
      // `result` 是 `ActionResult` 对象
      // 如果返回一个回调函数，将阻止默认行为，等调用 update 时会继续执行默认逻辑
    };
  }}
>
```

其中，`use:enhance` 传入的自定义函数是 [SubmitFunction](https://kit.svelte.dev/docs/types#public-types-submitfunction "https://kit.svelte.dev/docs/types#public-types-submitfunction") 类型，可选返回的回调函数中，result 是 [ActionResult](https://kit.svelte.dev/docs/types#public-types-actionresult "https://kit.svelte.dev/docs/types#public-types-actionresult") 类型。

我们可以借助这个函数实现 loading UI 的展示或隐藏等，举个例子：

```xml
<script>
  // ...
  let loading = $state(false);
</script>

<form
  method="POST"
  action="?/login"
  use:enhance={() => {
    loading = true;
    return async ({ update }) => {
      loading = false;
      update();
    };
  }}
>
  // ...
  <button
    class="flex w-full justify-center rounded-md bg-indigo-600 px-3 py-1.5 text-sm text-white shadow-sm disabled:bg-indigo-300"
    disabled={loading}>提交 {loading ? '...' : ''}</button
  >
</form>
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c328ddd7eee643b4b27fc36e629d63b6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1141&h=418&s=423037&e=gif&f=40&b=2c2c2c)

但是要注意，以上仅发生在表单提交给当前页面的时候，也就是 `+page.server.js`和 `+page.svelte`在同一路由。

如果 `<form>` action 属性指向其他路由，比如 `src/routes/signin/+page.svelte`：

```xml
<form
  method="POST"
  action="/sign?/login"
  use:enhance={() => {
    return async ({ result, update }) => {
      // result 会正常返回，但调用 update 不会生效
      update()
    };
  }}
>
```

`/signin`调用 `/sign` 下的 action，此时调用 update() 不会生效，当然直接使用 `use:enhance`也不会生效。所谓不会生效，不是指有报错，而是提交后看起来“没有反应”，也就是不会更新 form 属性、重新加载数据等。

这是因为 `use:enhance`是为了模拟浏览器原生行为，在原生行为中 ，提交到其他地址时，页面会重定向到该地址。为了能够生效，需要调用 `applyAction`：

```xml
<script>
  import { enhance, applyAction } from '$app/forms';
</script>

<form
  method="POST"
  action="/sign?/login"
  use:enhance={() => {
    loading = true;
    return async ({ result, update }) => {
      loading = false;
      await applyAction(result);
    };
  }}
>
  // ...
</form>
```

`applyAction(result)` 的具体行为取决于 `result.type`：

* `success`, `failure`：将 `$page.status` 设置为 `result.status`，并将 `form` 和 `$page.form` 更新为 `result.data`
* `redirect`：调用重定向，本质是运行`goto(result.location, { invalidateAll: true })`
* `error`：传递 result.error 数据渲染最近的错误边界

开发者也可以根据 `result.type` 自定义后续的逻辑：

```diff
<script>
  import { goto } from '$app/navigation';
  import { enhance, applyAction } from '$app/forms';
</script>

<form
  method="POST"
  action="/sign?/login"
  use:enhance={() => {
    loading = true;
    return async ({ result, update }) => {
      loading = false;
+     if (result.type === 'redirect') {
+        goto(result.location);
+     } else {
+        await applyAction(result);
+     }
    };
  }}
>
  // ...
</form>
```

简单总结一下：如果要实现表单的渐进式增强（一个直观的表现是提交时页面无刷新），SvelteKit 提供了 `use:enhance` Action。如果要自定义表单提交逻辑，可以为 `use:enhance`传入一个 SubmitFunction。如果该 SubmitFunction 返回一个函数，该函数会阻止表单处理响应后的默认行为比如更新页面属性、调用 `invalidateAll`重新加载数据、根据响应重定向或渲染错误边界等等。为了能够运行这些默认行为，与 action 同一路由下的页面需要调用 `update` 函数，非同一路由下，运行 `applyAction(result)`函数。

### 5.2. 自定义实现

我们也可以不使用 `use:enhance`，通过在`<form>` 上添加事件监听器的方式自定义实现渐进增强：

```xml
<!-- src/routes/login/+page.svelte -->

<script>
  import { invalidateAll, goto } from '$app/navigation';
  import { applyAction, deserialize } from '$app/forms';

  export let form;

  let error;

  async function handleSubmit(event) {
    const data = new FormData(event.currentTarget);

    const response = await fetch(event.currentTarget.action, {
      method: 'POST',
      body: data
    });

    const result = deserialize(await response.text());

    if (result.type === 'success') {
      // 重新运行所有 `load` 函数
      await invalidateAll();
    }

    applyAction(result);
  }
</script>

<form method="POST" on:submit|preventDefault={handleSubmit}>
  <!-- content -->
</form>
```

## 6\. GET 请求

使用 Form action 必须是 POST 请求，所以要声明 `<form method="POST">`。但有些情况，表单不需要向服务端 POST 数据，比如搜索。对于这些情况，可以使用 `method="GET"` 或者不声明 `method`（此时默认为 GET），此时 SvelteKit 会将表单视为 `<a>`元素：

```xml
<form action="/search">
  <label>
    Search
    <input name="q">
  </label>
</form>
```

此时表单提交后，会跳转到 `/search?q=xxx`。

此外，可以通过在 `<form>` 元素上设置 `data-sveltekit-reload`、 `data-sveltekit-replacestate`、`data-sveltekit-keepfocus`、`data-sveltekit-noscroll` 属性来控制路由行为。这些属性我们会在[《SvleteKit | 链接选项》](https://juejin.cn/book/7407681878888562723/section/7408068605482860584 "https://juejin.cn/book/7407681878888562723/section/7408068605482860584")篇介绍。