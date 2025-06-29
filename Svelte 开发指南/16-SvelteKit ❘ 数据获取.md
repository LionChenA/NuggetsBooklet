> 推荐学习指数：⭐️⭐⭐️，必学内容️

## 1\. 前言

在之前的路由篇中，我们讲到 `+page(.server).js`和`+layout(.server).js`都可以导出一个名为 `load`的函数用于获取渲染页面或布局所需的数据。我们可以从 `load` 函数的第一个参数中解构获得 `params`、`fetch` 等值。除此之外，还能够获取哪些值？`load` 函数在不同环境运行时又有什么差别？本篇我们就来详细解析一下 load 函数。

## 2\. load

`load` 函数分为 2 类：

1. **universal load：**`+page.js` 和 `+layout.js` 导出的在服务端和浏览器都可以运行的通用 load 函数
2. **server load：**`+page.server.js` 和 +`layout.server.js` 导出的仅在服务器端运行的服务端 load 函数

### 2.1. 示例 Demo

两者在使用上很类似，但有一些重要的差别。我们可以写一个简单的例子理解两者的不同：

```json
routes
└─ posts
   ├─ +layout.js
   ├─ +layout.server.js
   ├─ +page.js
   ├─ +page.server.js
   └─ +page.svelte
```

`layout.js`代码如下：

```javascript
export async function load({ data }) {
  return {
    ...data,
    layout: true
  };
}
```

`layout.server.js`代码如下：

```javascript
export async function load() {
  return {
    'layout.server': true
  };
}
```

`+page.js` 代码如下：

```javascript
export async function load({ data }) {
  return {
    ...data,
    page: true
  };
}
```

`+page.server.js` 代码如下：

```javascript
export async function load() {
  return {
    'page.server': true
  };
}
```

`+page.svelte` 代码如下：

```xml
<script>
  let { data } = $props();
</script>

<div>{JSON.stringify(data, null, 2)}</div>

```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0990ff1ec1ad4947af0ec38409d3d2d5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1890&h=358&s=48150&e=png&b=fefefe)

注意：像 `+page.js`和 `+page.server.js`很少会同时用到，这里为了举例才这样写。**当两者同时使用的时候，**`+page.server.js`的返回值并不能直接传给 `+page.svelte`，而是会将返回值以 `data` 属性传给 `+page.js`的 `load` 函数，所以上面的代码中，我们会获取 data 属性，放到返回值中再传给 `+page.svelte`。`layout.js` 和 `layout.server.js`同理。

### 2.2. 运行时机不同

server load 始终在服务端运行。universal load 在默认情况下，首次访问的时候在服务端运行（SSR），然后在客户端水合期间再次运行（CSR），后续导航发生的 universal load 都在客户端运行。但你可以通过页面选项修改此行为，比如设置 `const ssr = false`，此时禁用服务端渲染，universal load 将始终在客户端运行。

如果同一个路由包含 universal load 和 server load，server load 将首先运行。

load 函数始终在运行时调用，除非设置了 `const prerender = true`，此时会在构建时调用。

### 2.3. 输入不同

我们打印下 server load 和 universal load 的第一个参数：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4358f056d6bc4dc7a68f034ae0e08aa5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1956&h=1486&s=343147&e=png&b=fefefe)

可以看出有些属性相同，又有些属性不同。

比如 server load 和 universal load 函数都可以访问请求相关的属性（`params`、`route` 和 `url`）和各种函数（`fetch`、`setHeaders`、`parent`、`depends` 和 `untrack` ）

server load 运行在服务端，所以还有服务端相关的属性，比如 `clientAddress`、`cookies` 、`locals` 、 和 `request`，这些继承自服务端 [RequestEvent](https://kit.svelte.dev/docs/types#public-types-requestevent "https://kit.svelte.dev/docs/types#public-types-requestevent") 对象。

universal load 有一个 data 属性，当 server load 和 universal load 一起使用的时候，server load 的返回值会以 data 属性传给 universal load。

### 2.4. 输出不同

universal load 可以返回包含任何值的对象。而 server load 必须返回可被 devalue 序列化的数据，因为它要将服务端返回的数据传给客户端。

## 3\. 属性

现在让我们来具体介绍下这些属性。

### 3.1. ur

url 是 [URL](https://developer.mozilla.org/en-US/docs/Web/API/URL "https://developer.mozilla.org/en-US/docs/Web/API/URL") 实例，可以获取 `origin`、`hostname`、`pathname`等属性：

```json
 url: URL {
    href: 'http://localhost:5173/posts',
    origin: 'http://localhost:5173',
    protocol: 'http:',
    username: '',
    password: '',
    host: 'localhost:5173',
    hostname: 'localhost',
    port: '5173',
    pathname: '/posts',
    search: '',
    searchParams: URLSearchParams {},
    hash: ''
  },
```

但是要注意：`url.hash` 在 load 中无法访问，因为 URL 中的 hash 值只是客户端的一种状态，当向服务器端发出请求时，hash 部分并不会被发送，所以无法获取该值。

### 3.2. route

route 是包含当前路由目录的对象。比如 `src/routes/a/[b]/[...c]/+page.js`：

```javascript
export function load({ route }) {
  console.log(route.id); // '/a/[b]/[...c]'
}
```

### 3.3. params

params 是指路由动态参数，还是 `src/routes/a/[b]/[...c]/+page.js`：

```javascript
// 访问 http://localhost:5173/a/x/y/z

export function load({ params }) {
  console.log(params); // { b: 'x', c: 'y/z' }
}
```

params 是根据 url.pathname 和 route.id 进行判断的：

```javascript
条件一：url.pathname: /a/x/y/z

条件二：route.id: /a/[b]/[...c]

算出：params: { b: 'x', c: 'y/z' }
```

### 3.4. fetch()

fetch 函数与[原生 fetch Web API](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch "https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch") 一致，但会有一些额外的功能：

1. 可用于在服务端上发起凭证请求，因为继承了页面请求的 `cookie` 和 `authorization` 标头。
2. 它可以在服务端上发起相对地址请求
3. 在服务端运行时，内部请求（例如 `+server.js` 路由请求）将直接进入处理程序函数，不会产生 HTTP 调用开销
4. 在服务端渲染期间，响应将被捕获并内联到渲染的 HTML 中。在水合期间，将从 HTML 中读取响应，从而保证数据一致性并防止额外的网络请求

```javascript
// src/routes/items/[id]/+page.js
export async function load({ fetch, params }) {
  const res = await fetch(`/api/items/${params.id}`);
  const item = await res.json();

  return { item };
}
```

### 3.5. cookies()

server load 可以通过 cookies 获取和设置 cookie

```javascript
// src/routes/+layout.server.js
import * as db from "$lib/server/database";

export async function load({ cookies }) {
  const sessionid = cookies.get("sessionid");

  return {
    user: await db.getUser(sessionid),
  };
}
```

具体 Cookies 的获取和设置方法参考 [SvelteKit docs](https://kit.svelte.dev/docs/types#public-types-cookies "https://kit.svelte.dev/docs/types#public-types-cookies")。

当在 load 函数中使用 fetch 时，只有当目标主机和 SvelteKit 应用相同时或者是更具体的子域时，fetch 请求才会携带 cookies，比如应用域名是 `my.domain.com`，请求 `my.domain.com`、`sub.my.domain.com`都会收到 cookies，但请求 `api.domain.com`、`domain.com`不会收到 cookies。

### 3.6. setHeaders()

setHeaders 用于设置标头，当在服务端运行时，此函数可以为响应设置标头。在浏览器中运行时，setHeaders 不起作用。

```javascript
export async function load({ fetch, setHeaders }) {
  const url = `https://cms.example.com/products.json`;
  const response = await fetch(url);

  setHeaders({
    age: response.headers.get("age"),
    "cache-control": response.headers.get("cache-control"),
  });

  return response.json();
}
```

注意：

1. 不能多次设置同一标头
2. 不能通过设置 set-cookie 添加 Cookies，应该使用上节的 cookies.set

### 3.7. parent()

有的时候，load 函数需要从父 load 函数获取数据，此时可以通过 `await parent()`：

```javascript
routes
├─ abc
│  ├─ +layout.js
│  ├─ +page.js
│  └─ +page.svelte
└─ +layout.js
```

```javascript
// src/routes/+layout.js
export function load() {
  return { a: 1 };
}

// src/routes/abc/+layout.js
export async function load({ parent }) {
  const { a } = await parent();
  return { b: a + 1 };
}

// src/routes/abc/+page.js
export async function load({ parent }) {
  const { a, b } = await parent();
  return { c: a + b };
}

// src/routes/abc/+page.svelte
<script>
  export let data;
</script>

<p>{data.a} + {data.b} = {data.c}</p>
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0c065dc45114f11b861b73f3f64cdaf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1068&h=320&s=23809&e=png&b=fefefe)

在`+page.server.js` 和 `+layout.server.js` 中，`parent()`会返回父 `+layout.server.js`的数据。

在 `+page.js`或 `+layout.js`中，`parent()`会返回父 `+layout.js`的数据。缺失的 `+layout.js`会被视为 `({ data }) => data` 函数，所以它会转发父 `+layout.server.js` 中的数据。也就是说，将 src/routes/+layout.js 重命名为 src/routes/+layout.server.js 也是可以生效的。

使用 await parent 注意不要造成阻塞：

```javascript
export async function load({ params, parent }) {
  // 如果数据不需要依赖 parent() 返回的数据，放在前面调用，因为所有的 load 函数是平行运行的
  const data = await getData(params);
  const parentData = await parent();

  return {
    ...data
    meta: { ...parentData.meta, ...data.meta }
  };
}
```

## 4\. load 使用技巧

### 4.1. 流式加载

当使用 server load 的时候，可以返回一个 Promise，数据会流式传输给浏览器，适用于一些非必要的缓慢数据。

新建 `src/routes/blog/[slug]/+page.server.js`，代码如下：

```javascript
function sleep(time) {
  return new Promise((resolve) => setTimeout(resolve, time));
}

async function loadComments() {
  await sleep(4000);
  return [
    { content: "This is a test comment!" },
    { content: "This is a test comment!" },
  ];
}

async function loadPost(id) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${id}`
  );
  const post = await response.json();
  return post;
}

export async function load({ params }) {
  return {
    comments: loadComments(params.slug),
    post: await loadPost(params.slug),
  };
}
```

新建 `src/routes/blog/[slug]/+page.svelte`，代码如下：

```javascript
<script>
  export let data;
</script>

<h2>{data.post.title}</h2>
<div>{@html data.post.body}</div>

{#await data.comments}
  Loading comments...
{:then comments}
  {#each comments as comment}
    <p>{comment.content}</p>
  {/each}
{:catch error}
  <p>error loading comments: {error.message}</p>
{/await}

```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f5df214e9b74526b822eff2cf9c0150~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=708&h=380&s=26737&e=gif&f=13&b=fefefe)

使用流式数据传输时要谨慎，如果你没有妥善处理 Promise reject 的情况，比如：

```javascript
async function loadComments() {
  throw new Error("error");
  await sleep(4000);
  return [
    { content: "This is a test comment!" },
    { content: "This is a test comment!" },
  ];
}
```

此时会直接导致应用崩溃。

所以写的时候最好添加一个 noop-catch 函数：

```javascript
export async function load({ params }) {
  return {
    comments: loadComments(params.slug).catch(() => {}),
    post: await loadPost(params.slug),
  };
}
```

这样即便抛出错误，也不会影响应用其他部分。

### 4.2. 重新运行

SvelteKit 会追踪每个 `load` 函数的依赖，以避免导航时没有必要的重新运行。比如这样一个例子：

```javascript
// src/routes/blog/[slug]/+page.server.js
import * as db from "$lib/server/database";

export async function load({ params }) {
  return {
    post: await db.getPost(params.slug),
  };
}

// src/routes/blog/[slug]/+layout.server.js
import * as db from "$lib/server/database";

export async function load() {
  return {
    posts: await db.getPostSummaries(),
  };
}
```

在 `blog/xxx`之间导航的时候，`+page.server.js` 会重新运行，但 `+layout.server.js` 不会重新运行，因为 params.slug 发生了变化。

如果你想要取消追踪某些属性：

```javascript
export async function load({ untrack, url }) {
  // 取消追踪 url.pathname 当路径改变的时候不会触发重新运行
  if (untrack(() => url.pathname === "/")) {
    return { message: "Welcome!" };
  }
}
```

你也可以手动失效让 load 函数重新运行，需要借助 invalidate(url) 或 invalidateAll() 函数。先声明 load 函数依赖的 url，然后再手动调用 invalidate(url)，手动使其重新运行。

声明依赖的 url 有两种方式，一种是 `fetch(url)`，一种是 `depends(url)`，举个例子：

```javascript
// src/routes/random-number/+page.js
export async function load({ fetch, depends }) {
  // 当调用 fetch 的时候会自动声明依赖的 url，调用 `invalidate('https://api.example.com/random-number')` 可使其重新运行
  const response = await fetch('https://api.example.com/random-number');

  // 手动声明依赖的 url，该字符需要以 [a-z]: 开头，调用 `invalidate('app:random')` 可使其重新运行
  depends('app:random');

  return {
    number: await response.json()
  };
}

// src/routes/random-number/+page.svelte
<script>
  import { invalidate, invalidateAll } from '$app/navigation';

  export let data;

  function rerunLoadFunction() {
    // 这些都可以让 load 函数重新运行
    invalidate('app:random');
    invalidate('https://api.example.com/random-number');
    // 这个用法比较特殊，传入一个函数，接受 url 作为参数，判断需要重新运行的路由地址
    invalidate(url => url.href.includes('random-number'));
    invalidateAll();
  }
</script>

<p>random number: {data.number}</p>
<button on:click={rerunLoadFunction}>Update random number</button>
```

所以总结一下 load 函数重新运行的规则：

1. 引用了 `params` 中的属性，当其值发生更改时
2. 引用了 `url`的属性，当其值发生更改时
3. 调用 `url.searchParams.get(...)`、 `url.searchParams.getAll(...)` 或`url.searchParams.has(...)`，并且相关参数会发生变化
4. 调用了 `await parent()` 当父 load 函数重新运行
5. 通过 fech、depends 声明了对特定 URL 的依赖，调用 invalidate(url) 时
6. 调用 invalidateAll()，所有激活的 load 函数都要重新运行

注意：重新运行 load 函数会更新对应 +layout.svelte 或 +page.svelte 的 data 属性，但不会导致组件重新创建，因为内部的状态还会继续保留。

## 5\. @sveltejs/kit 辅助函数

### 5.1. error

```javascript
import { error } from "@sveltejs/kit";

export function load({ locals }) {
  if (!locals.user) {
    error(401, "not logged in");
  }

  if (!locals.user.isAdmin) {
    error(403, "not an admin");
  }
}
```

如果在 load 函数抛出错误，会渲染最近的 `+error.svelte`。

### 5.2. redirect

```javascript
import { redirect } from "@sveltejs/kit";

export function load({ locals }) {
  if (!locals.user) {
    redirect(307, "/login");
  }
}
```

redirect 用于重定向，需要指定重定向的位置以及 3xx 状态代码。

注意不要在 `try {...}` 块中使用 `redirect()`，因为重定向会抛出错误，导致立即触发 catch 语句。

## 6\. $page.data

`+page.svelte` 和 `+layout.svelte` 可以通过 data 属性获取父级们的所有数据。有的时候，父布局可能需要访问页面或者子布局中的数据。例子，根布局希望访问 +page.js 或 +page.server.js load 函数返回的 title 属性，此时可以通过 $page.data 完成：

```xml
// src/routes/+layout.svelte
<script>
  import { page } from '$app/stores';
</script>

<svelte:head>
  <title>{$page.data.title}</title>
</svelte:head>
```