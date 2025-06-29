> 推荐学习指数：⭐️⭐⭐️，必学内容️

## 1\. 前言

本篇我们介绍 SvelteKit 的重要组成部分 —— 路由。所谓路由，指的是根据 URL 匹配对应处理程序的过程。

在开始之前，先分享一个小技巧，因为我发现有的同学新建文件是一个文件夹一个文件夹建立的……

在 VSCode 中，其实可以一次性建立目录和文件：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b88ef31084fb404f81a35b8e2bec2c18~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1040&h=187&s=112823&e=gif&f=57&b=252525)

这本小册讲到需要修改代码的时候，我会写出明确的操作和具体目录，比如：

```bash
新建 src/routes/posts/+page.js，代码如下：

修改 src/routes/posts/+page.js，代码如下：
```

当是“新建”的时候，你可以拷贝地址，然后在项目根目录新建文件，输入地址，VSCode 就会自动新建文件。

当是“修改”的时候，你可以拷贝地址，然后 Command + P，输入地址，VSCode 应该会正确打开文件。

> 注：小册《Next.js 开发指南》也是这样的

## 2\. 文件即路由

跟 Next.js 一样，Svelte 采用的是基于文件系统的路由。也就是说，**应用的路由地址由代码库中的目录决定**，比如：

|文件地址|对应路由地址|
|---|---|
|`src/routes/+page.svelte`|`/`|
|`src/routes/about/+page.svelte`|`/about`|
|`src/routes/sverdle/how-to-play/+page.svelte`|`/sverdle/how-to-play`|

Svelte 约定了多个路由相关的文件，它们的文件名都会添加 `+`前缀用于识别，比如 `+page.svelte`。本篇我们会一一介绍。

## 3\. +page

### 3.1. +page.svelte

`+page.svelte`用于定义路由对应的页面。**默认情况下，页面会在初始请求的时候采用服务端渲染，然后在后续导航的时候使用客户端渲染。**

这句话是什么意思呢？我们写个简单的 Demo 就知道了。

新建 `src/routes/authors/+page.svelte`，代码如下：

```xml
<a href="/authors">Auther List</a>
<a href="/posts">Post List</a>

<h1>Hello Auther List!</h1>
```

新建 `src/routes/posts/+page.svelte`，代码如下：

```xml
<a href="/authors">Auther List</a>
<a href="/posts">Post List</a>

<h1>Hello Post List!</h1>
```

浏览器打开 [http://localhost:5173/authors](http://localhost:5173/authors "http://localhost:5173/authors")，效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff8e1e6e71164d32911d0188de6d1be4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=708&h=380&s=63072&e=gif&f=37&b=fefefe)

效果描述：首次打开 [http://localhost:5173/authors](http://localhost:5173/authors "http://localhost:5173/authors") 的时候，`/authors`页面采用服务端渲染，查看 `/authors` HTML 文件即可看出：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c9c628b490d487fa77d79d518027c92~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3152&h=800&s=263153&e=png&b=2c2c2c)

当点击跳转其他路由地址的时候，你会发现路由地址虽然发生了改变，但页面并没有发生刷新行为，而是直接渲染了对应路由的内容。这就是客户端导航。通过 JavaScript 拦截默认的跳转行为，然后加载对应路由的内容进行渲染，因为在客户端完成渲染，所以是 CSR。整个应用的表现类似于 SPA。

> 注意：不同于 Next.js 使用特定的 `<Link>`组件，SvelteKit 可以直接使用 `<a>`元素在路由间导航

### 3.2. +page.js

通常，页面需要先加载一些数据再进行渲染。为此我们可以创建一个导出 `load`函数的 `+page.js`。

新建 `src/routes/posts/+page.js`，代码如下：

```javascript
export async function load({ fetch }) {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts');
  const posts = await response.json();
  console.log(posts.length)
  return {
    posts
  };
}
```

此函数会和 `+page.svelte` 一起运行，也就是说，它也会在初始请求的时候在服务端运行，然后在后续导航的时候在客户端运行。

不过这样说并不完全准确，实际上，此函数在初始请求的时候也会在客户端运行一次。不信我们补全一下 Demo。

修改 `src/routes/posts/+page.svelte`，完整代码如下：

```xml
<script>
  let { data } = $props();
</script>

<a href="/authors">Auther List</a>
<a href="/posts">Post List</a>

<h1>Posts list</h1>
<ul>
  {#each data.posts as {id, title}}
    <li><a href={`/posts/${id}`}>{title}</li>
  {/each}
</ul>
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfd15cfe1c4e4103a717a99d8b2f5d11~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2230&h=660&s=143473&e=png&b=fefefe)

你会发现，`+page.js`中的 `console.log(posts.length)`在初始请求的时候，会在浏览器和命令行各自打印一次。但如果你查看页面请求，你会发现并没有 `https://jsonplaceholder.typicode.com/posts` 这个 HTTP 请求。

如果从其他页面导航至 `/posts`，则只有浏览器会打印一次，页面中会有 `https://jsonplaceholder.typicode.com/posts` 这个请求。

这是为什么呢？

原因在于我们使用了 Svelte 提供的 `fetch` 函数，而非原生的 `fetch`：

```javascript
export async function load({ fetch }) {
  fetch(...)
}
```

Svelte 提供的 `fetch` 函数与原生 `fetch` 一致，但会有一些额外的功能，就比如在服务端渲染的时候，会将获取的数据内联到 HTML 中，然后在水合的时候，从 HTML 读取响应数据，从而防止重复的网络请求。这也就是为什么初始请求中 `console.log` 执行了两次，但客户端并没有发起网络请求。

开发的时候，应该尽可能使用 `load` 函数提供的 `fetch`，否则浏览器会有 warning 提示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9e5aec4b372405883ce3f98a1ce76aa~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1652&h=130&s=59111&e=png&b=403c29)

除了导出 `load` 函数，`+page.js`也可以导出一些用于配置页面行为的变量：

```javascript
// 配置页面是否使用 SSR 渲染，默认为 true，如果设置为 false，页面在初始请求的时候使用客户端渲染
export const ssr = true | false;

// 配置页面是否使用 CSR 渲染。默认为 true，如果设置为 false，表示不需要客户端渲染，则只走服务端渲染，适用于只有 HTML 和 CSS 的静态页面
export const csr = true | false;

// 除此之外还有一些其他的配置项，《SvelteKit | 页面选项》篇会介绍
```

### 3.3. +page.server.js

有的时候，`load` 函数只能运行在服务端，比如需要从数据库中查找数据或者用到一些涉及安全的私钥。此时可以将 `+page.js` 重命名为 `+page.server.js`，函数将只在服务端运行：

```javascript
// +page.js => +page.server.js
export async function load({ fetch }) {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  const posts = await response.json();
  console.log(posts.length);
  return {
    posts,
  };
}
```

`load` 函数将只在服务端运行。在初始请求的时候，load 返回的数据将内联在 HTML 中，然后水合的时候从 HTML 中获取。在客户端导航的时候，SvelteKit 将从服务端加载这些数据，为了能够让客户端成功获取这些数据，数据必须是可序列化的，SvelteKit 背后使用 [devalue](https://github.com/rich-harris/devalue "https://github.com/rich-harris/devalue") 这个库实现数据序列化。

> 注：devalue 类似于 JSON.stringify，但是能处理如循环引用等特殊情况，相当于加强版的 JSON.stringify

`+page.server.js` 也可以导出页面配置选项如 `prerender`、`ssr`、`csr` 等。

## 4\. +error.svelte

如果 `load` 函数发生错误，SvelteKit 将渲染默认错误页面。我们可以在 `load`函数中手动抛出一个错误模拟效果：

```diff
export async function load({ fetch }) {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts');
  const posts = await response.json();
+ throw new Error('custom error')
  console.log(posts.length);
  return {
    posts
  };
}
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/602ad73e6c704239950e735e3b200f7d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1504&h=468&s=39921&e=png&b=ffffff)

你可以添加一个 `+error.svelte`为路由自定义错误页面。

新建 `src/routes/posts/+error.svelte`，代码如下：

```xml
<script>
  import { page } from '$app/stores';
</script>

<h1>{$page.status}: {$page.error.message}</h1>

<style>
  h1 {
    color: red
  }
</style>
```

此时浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f55002481664db488c95437c027346d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1324&h=410&s=40746&e=png&b=ffffff)

当出现错误时，SvelteKit 会向上查找最近的 `+error.svelte`。假设 `src/routes/posts/+page.server.js`出错，它的查找顺序是：

1. `src/routes/posts/+error.svelte`
2. `src/routes/+error.svelte`

如果都没有，则渲染默认的错误界面。

> 注意：下节会讲到 `+layout(.server).js`，因为布局会包裹页面，所以如果是布局中的 load 函数出错，最近的 +error.svelte 不是相同目录下的，而是父级目录下的。
>
> 如果错误从根布局 `src/routes/+layout(.server).js`抛出，则会渲染静态的 fallback 错误页面，样式同默认错误页面，但可以通过创建 `src/error.html`自定义该页面

## 5\. +layout

### 5.1. +layout.svelte

客户端导航的时候，原有的 `+page.svelte` 组件将被销毁，取而代之使用新的 `+page.svelte`，但有的时候，有些元素应该保持状态不变，比如导航栏、页脚等。此时就可以将这些元素放到布局中。

新建 `src/routes/+layout.svelte`，代码如下：

```xml
<script>
  let { children } = $props();
</script>

<a href="/authors">Auther List</a>
<a href="/posts">Post List</a>

{@render children()}
```

其中 `{@render children()}`表示页面和子页面内容。此时就可以删除 `svelte-app/src/routes/authors/+page.svelte`和 `svelte-app/src/routes/posts/+page.svelte`中的导航链接。

浏览器效果不变：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ab4eacad5fc4c568a2d1d968860bd08~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1342&h=518&s=50335&e=png&b=ffffff)

**布局支持嵌套。默认情况下，每个布局都会继承其上方的布局。** 比如这样一个文件目录：

```xml
routes
├─ auth
│  ├─ signin
│  │  └─ +layout.svelte
│  │  └─ +page.svelte
│  └─ +layout.svelte
├─ posts
│  ├─ +page.server.js
│  └─ +page.svelte
├─ +layout.svelte
└─ +page.svelte
```

其中 `routes/auth/signin/+page.svelte` 会继承`routes/auth/signin/+layout.svelte`、`routes/auth/+layout.svelte` 和 `routes/+layout.svelte`的内容。

当同一目录有 `+page.svelte`、`+layout.svelte`、`+error.svelte` 时，三者的层级关系为：

```xml
<Layout>
  <Error>
    <Page />
  </Error>
</Layout>
```

### 5.2. +layout.js

就像 `+page.svelte` 可以从 `+page.js` 获取数据一样，`+layout.svelte` 也可以从 `+layout.js` 中的 `load` 函数获取数据。

新建 `src/routes/+layout.js`，代码如下：

```javascript
export function load() {
  return {
    navs: [
      { slug: "authors", title: "Auther List" },
      { slug: "posts", title: "Post List" },
    ],
  };
}
```

修改 `src/routes/+layout.svelte`，完整代码如下：

```xml
<script>
  let { data, children } = $props();
</script>

<ul>
  {#each data.navs as {slug, title}}
    <li><a href={`/${slug}`}>{title}</a></li>
  {/each}
</ul>

{@render children()}
```

此时浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b2730a7907d479fbd8e3bdc6d32b85c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=708&h=380&s=60547&e=gif&f=26&b=fefefe)

需要注意的是，布局 `load` 函数返回的数据，所有子页面也都可以使用。修改 `src/routes/authors/+page.svelte`，代码如下：

```xml
<script>
  let { data } = $props();
  console.log(data.navs)
</script>

<h1>Hello Auther List!</h1>
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74073dd2818f4fb3848042d62c60ec17~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2364&h=606&s=133657&e=png&b=2c2c2c)

你肯定会想到冲突的情况，比如布局 load 函数返回 `{ a: 1, b: 2 }`，页面 load 函数返回 `{ b: 3, c: 4 }`，最后的结果会是 `{ a: 1, b: 3, c: 4 }`。

`+layout.js` 也可以导出页面配置项如 `prerender`、`ssr`、`csr`，它们会作为子页面配置项的默认值。

### 5.3. +layout.server.js

如果只能在服务器上运行 `load` 函数，将 `+layout.js`重命名为 `+layout.server.js`。

与 `+layout.js` 一样，`+layout.server.js` 可以导出页面配置项如 `prerender`、`ssr`、`csr`。

## 6\. +server.js

`+server.js` 用于创建 API，也就是我们常说的 “Web 接口”，有时也被称为 “API 路由（API route）”、“端点（endpoint）”等，对应 Next.js 中的路由处理程序。

### 6.1. 基础使用

`+server.js` 导出对应 HTTP 请求方法的函数即可创建对应的请求类型，我们以创建 GET 请求为例。新建 `src/routes/api/post/+server.js`，代码如下：

```javascript
export async function GET({ url }) {
  const id = url.searchParams.get("id");
  return new Response(id);
}
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7a444f78cd849c685a66832d6152412~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1162&h=214&s=24122&e=png&b=202020)

函数接收一个 [RequestEvent](https://kit.svelte.dev/docs/types#public-types-requestevent "https://kit.svelte.dev/docs/types#public-types-requestevent")（从中可以解构获取 url、fetch 等）对象作为参数。该函数需要返回一个 Response 对象。

> 记住这个 RequestEvent 类型，+server.js 的 API 路由、form actions 的 +page.server.js 和 +page.server.js 和 +layout.server.js 的 load 函数传入的都是这个类型

让我们看个更复杂的例子，修改 `src/routes/api/post/+server.js`，代码如下：

```javascript
import { error, json } from "@sveltejs/kit";

export async function GET({ url, fetch }) {
  const id = url.searchParams.get("id");

  if (!id) {
    error(400, "没有 id 参数");
  }

  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${id}`
  );
  const post = await response.json();
  return json(post);
}
```

为了方便，`@sveltejs/kit`提供了 `error`、`redirect`、`json` 等方法。浏览器效果如下：

当没有 `id` 参数时：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74018d5f75404e9fa4cb34653afeae6e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1690&h=576&s=53152&e=png&b=242424)

当有 `id`参数时：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf4f658029044e3cb7d75cb2765894a0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2076&h=658&s=130249&e=png&b=1a1a1a)

### 6.2. 错误处理

如果 `+server.js`抛出错误，无论是通过 error 函数抛出，还是意外错误，响应将会是错误的 JSON 形式或者是错误页面。具体取决于获取的形式。比如你通过 fetch 获取则返回：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a7019441ac24bb3a25ddba646352453~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1460&h=296&s=64405&e=png&b=2e2e2e)

如果是浏览器直接访问：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1373b7351f78467da15c9f44c95da8ca~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1566&h=508&s=48879&e=png&b=252525)

SvelteKit 本质是使用 `Accept` header 进行的判断。

注意：当 `+server.js`出现错误的时候，`+error.svelte` 并不会被渲染，只会触发 fallback 错误页面，可以通过创建 `src/error.html`文件进行自定义：

```xml
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>%sveltekit.error.message%</title>
  </head>
  <body>
    <h1>My custom error page</h1>
    <p>Status: %sveltekit.status%</p>
    <p>Message: %sveltekit.error.message%</p>
  </body>
</html>
```

### 6.3. 前后端交互

让我们再看个前后端交互的例子。

新建 `src/routes/add/+page.svelte`，代码如下：

```xml
<script>
  let a = 0;
  let b = 0;
  let total = 0;

  async function add() {
    const response = await fetch('/api/add', {
      method: 'POST',
      body: JSON.stringify({ a, b }),
      headers: {
        'content-type': 'application/json'
      }
    });

    total = await response.json();
  }
</script>

<input type="number" bind:value={a}> +
<input type="number" bind:value={b}> =
{total}

<button on:click={add}>Calculate</button>
```

新建 `src/routes/api/add/+server.js`，代码如下：

```javascript
import { json } from "@sveltejs/kit";

export async function POST({ request }) {
  const { a, b } = await request.json();
  return json(a + b);
}
```

访问 [http://localhost:5173/add](http://localhost:5173/add "http://localhost:5173/add")，浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3b7c406c80848c282b25fe4c45f6871~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1455&h=415&s=103034&e=gif&f=47&b=2c2c2c)

### 6.4. fallback HTTP 方法

对于未定义的 HTTP 请求方法，比如你只定义了 GET 请求，但有人以 POST 方法请求该路由，此时会报错：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63e1f7007c054d3fb4a293f462dea158~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2316&h=614&s=115465&e=png&b=242424)

如果你想自定义这个错误，你可以导出一个名为 `fallback`的函数。修改 `src/routes/api/post/+server.js`，添加如下代码：

```javascript
import { error, json, text } from "@sveltejs/kit";

export async function fallback({ request }) {
  return text(`I caught your ${request.method} request!`);
}
```

此时效果为：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6d77230fca54289be585dd8b22510ac~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2324&h=596&s=111752&e=png&b=232323)

### 6.5. 优先级

与 Next.js 不同，`+server.js` 可以与 `+page.svelte` 放在同一目录下，所以同一路由既可以是页面也可以是 API 接口。为了判断是页面还是接口，SvelteKit 的规则如下：

1. `PUT`、`PATCH`、`DELETE`、`OPTIONS` 请求方法由 `+server.js` 处理
2. `GET`、`POST`、`HEAD` 请求方法根据 `accept` 标头是否以 `text/html`优先

## 7\. 总结

SvelteKit 采用基于文件系统的路由，有 4 大类路由约定文件：

1. `+page`：负责渲染页面
2. `+error`：负责渲染错误界面
3. `+layout`：负责处理布局
4. `+server.js`：负责处理 API 接口

其中，`+layout` 和 `+error` 文件可用于子目录及其当前所在目录。