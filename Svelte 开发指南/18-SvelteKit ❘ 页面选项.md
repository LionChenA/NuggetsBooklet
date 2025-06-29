> 推荐学习指数：⭐️️，了解即可

## 1\. 前言

路由篇讲到 `+page(.server).js`和 `+layout(.server).js`可以导出一些页面选项（page options）控制页面的行为。这些选项大多是为了让开发者根据实际情况自定义渲染方式，从而实现页面性能的最佳化。

本篇就为大家详细讲解这些选项。

## 2\. prerender

`prerender` 选项用于控制页面是否进行预渲染。

所谓预渲染，指的是在构建的时候就将路由渲染为 HTML。优势在于减少服务端开销，劣势在于预渲染的内容只能通过构建和部署新版本的应用程序来更新。

```javascript
// +page(.server).js/+layout(.server).js/+server.js
export const prerender = true | false | auto;
```

如何判断一个页面是否适合预渲染呢？

最基本的判断是 **“是否所有用户都必须从服务端获取相同的内容”。**

但这并不是说页面只能是纯 HTML 和 CSS 才能预渲染，页面依然可以有 JavaScript，可以有事件监听，可以在 onMount 时访问页面参数、获取个性化数据等等。

比如一篇文章，如果只有文章内容，哪个用户访问都还是这个内容，所以可以预渲染成 HTML。但如果还有点赞、评论等数据，两个用户访问的时候数据可能并不相同，所以只能动态渲染。

此外要注意，带 actions 的页面不能预渲染，因为相同的路由还要有一个服务端用于处理 POST 请求，就不是纯静态了。

### 2.1. +page(.server).js

新建 `src/routes/posts/+page.js`，代码如下：

```javascript
export async function load({ fetch }) {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  const posts = await response.json();

  return {
    posts,
  };
}

export const prerender = true;
```

新建 `src/routes/posts/+page.svelte`，代码如下：

```xml
<script>
  let { data } = $props();
</script>

<h1>Posts list</h1>
<ul>
  {#each data.posts as { id, title }}
    <li>{title}</li>
  {/each}
</ul>
```

运行 `npm run build`，查看 `.svelte-kit/output/prerendered/pages`文件夹，可以看到预渲染生成的 HTML 文件：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66665d28e1184636bc592b90838afa78~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=4604&h=640&s=446078&e=png&b=1f1f1f)

运行 `npm run preview`预览效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e8ee1457c404d138d8416fbf0ab5a9f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2510&h=938&s=373283&e=png&b=2a2a2a)

### 2.2. +server.js

`prerender` 也可用于 `+server.js`，新建 `src/routes/api/posts/+server.js`，代码如下：

```javascript
import { json } from '@sveltejs/kit';

export async function GET() {
  const res = await fetch('https://jsonplaceholder.typicode.com/posts');
  const data = await res.json();

  return json(data);
}

export const prerender = true;
```

运行 `npm run build` ，查看 `.svelte-kit/output/prerendered/pages`文件夹：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f96458e1ea2d4915a4c2e55357975de3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2182&h=830&s=279925&e=png&b=1f1f1f)

运行 `npm run preview`预览效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ee3abceb8af47f68676ff3707471828~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2520&h=964&s=452479&e=png&b=292929)

这里要注意一个路由冲突问题：

`routes/api/posts/+server.js`对应创建的文件是 `.svelte-kit/output/prerendered/pages/api/posts`

`routes/api/posts/about/+server.js`对应创建的文件是 `.svelte-kit/output/prerendered/pages/api/posts/about`

如果两个文件同时存在，就会造成冲突。因为 Linux 系统同一个目录下，文件和目录不能同名。也就是说，如果 `api`目录下已经有一个 posts 文件存在，则不能建立一个名为 posts 的文件夹，反之也是一样，已经有了一个名为 posts 的文件夹，不能再创建一个名为 posts 的文件。

所以如果你同时写了 `api/posts/+server.js`和 `api/posts/about/+server.js`，预渲染时就会报错。为了避免这个问题，你可以写成 `api/posts.json/+server.js`和 `api/posts/about.json/+server.js`。

页面并没有这个问题，因为 SvelteKit 自动将预渲染的页面写成了 `pages/posts.html`而不是 `pages/post`。

### 2.3. auto

prerender 选项也可以设置成 `"auto"`：

```json
export const prerender = "auto";
```

"auto" 是什么意思呢？它跟 true 的区别是什么呢？这在初学的时候可能让人有点难以理解。其实很简单，正如官方文档中讲的：

> for example, with a route like /blog/\[slug\] where you want to prerender your most recent/popular content but server-render the long tail

假设 `/blog/[slug]`这个路由下有 1000 篇文章，根据二八定律，20% 的文章具有 80% 的流量，剩下 80% 的文章流量星星点点构成长尾。

我们没有必要每次都重新构建预渲染 1000 篇文章，我们只需要用 `entries` 指定最有流量的文章预渲染即可，那剩下的文章怎么办呢？如果设置 `prerender = true`，访问这些文章会出现 404 错误，因为没有预渲染。但如果设置 `prerender=auto`，虽然没有预渲染，但会保留服务端渲染的代码，此时依然可以正常渲染（不过加载速度肯定会慢于预渲染）。

简单来说，借助 `prerender = "auto"` 和 `entries` ，将主要的文章预渲染保证性能，剩下的文章走服务端渲染保证能正常访问。

### 2.4. 自动生成

关于预渲染有一点要讲解的是，当为动态路由设置 `prerender = true | "auto"`时，SvelteKit 其实会自动生成预渲染的页面。这是什么意思呢？我们举个简单的例子，文件目录如下：

```json
routes
└─ posts
   ├─ [id]
   │  ├─ +page.js
   │  └─ +page.svelte
   ├─ +page.js
   └─ +page.svelte
```

新建 `src/routes/posts/+page.js`，代码如下：

```javascript
export async function load() {
  const posts = [
    {
      userId: 1,
      id: 1,
      title: 'sunt aut facere repellat provident occaecati excepturi optio reprehenderit',
      body: 'quia et suscipit
suscipit recusandae consequuntur expedita et cum
reprehenderit molestiae ut ut quas totam
nostrum rerum est autem sunt rem eveniet architecto'
    },
    {
      userId: 1,
      id: 2,
      title: 'qui est esse',
      body: 'est rerum tempore vitae
sequi sint nihil reprehenderit dolor beatae ea dolores neque
fugiat blanditiis voluptate porro vel nihil molestiae ut reiciendis
qui aperiam non debitis possimus qui neque nisi nulla'
    },
    {
      userId: 1,
      id: 3,
      title: 'ea molestias quasi exercitationem repellat qui ipsa sit aut',
      body: 'et iusto sed quo iure
voluptatem occaecati omnis eligendi aut ad
voluptatem doloribus vel accusantium quis pariatur
molestiae porro eius odio et labore et velit aut'
    }
  ];

  return {
    posts
  };
}

export const prerender = true;
```

新建 `src/routes/posts/+page.svelte`，代码如下：

```xml
<script>
  let { data } = $props();
</script>

<h1>Posts list</h1>
<ul>
  {#each data.posts as { id, title }}
    <li><a href={`/posts/${id}`}>{title}</a></li>
  {/each}
</ul>

```

新建 `src/routes/posts/[id]/+page.js`，代码如下：

```javascript
import { error } from '@sveltejs/kit';

export async function load({ params }) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${params.id}`);
  const post = await response.json();
  if (post.title) {
    return {
      post
    };
  }

  error(404, 'Not found');
}

export const prerender = 'true';
```

新建 `src/routes/posts/[id]/+page.svelte`，代码如下：

```xml
<script>
  import { page } from '$app/stores';
  let { data } = $props();
</script>

{$page.params.id} Post Content

<hr />

<h2>{data.post.title}</h2>

<hr />

{data.post.body}
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/944646fbcf4943c89ae0471ab017a26b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=979&h=396&s=55942&e=gif&f=12&b=fefefe)

此时运行 `npm run build`，SvelteKit 会预渲染出哪些 HTML 文件呢？答案是：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2dc8878364014a11a9e6a636a55f7dcd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2690&h=636&s=255445&e=png&b=1f1f1f)

问题在于，为什么会生成 `posts/1.html`、`posts/2.html`、`posts/3.html` 呢？明明在动态路由中，我们并没有指定要预渲染的路由。

这是因为 SvelteKit 预渲染器会爬取预渲染的页面，并扫描其中的 `<a>`元素指向的页面，如果这些页面也设置了预渲染，就会预渲染这些页面。所以之所以会生成 `1-3.html`，是因为 `posts.html` 有这些页面的链接。查看上图中的右侧部分，发现确实有这些链接。

你也可以取消 `posts/+page.js`的预渲染或者删除掉 `posts/+page.svelte`中的链接继续验证这一规律。

## 3\. entries

对于一个动态路由 `posts/[id]`，SvelteKit 其实并不知道有哪些需要预渲染的路由，所以只能通过爬取链接确定。但开发者可以通过 `entries`选项明确指定有哪些路由。

修改 `src/routes/posts/[id]/+page.js`，代码如下：

```javascript
export function entries() {
  return [{ id: '1' }, { id: '2' }, { id: '3' }, { id: '4' }];
}

export const prerender = true;
```

此时 SvelteKit 会预渲染 `posts/1`到 `posts/4`这 4 个路由：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f26d70c9e77c489d8b739a1fb232cc67~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2818&h=988&s=211270&e=png&b=1e1e1e)

`entries`也可以是一个 `async` 函数，用于从接口或数据库获取列表数据。

`entries`会和 SvelteKit 自动爬取链接一起决定预渲染哪些页面，比如 SvelteKit 爬取到了 `posts/1`、`post/2`，而 `entries` 指定 `posts/1`、`posts/3`，最终会预渲染 `posts/1`、`post/2`、`posts/3`。

## 4\. ssr

通常情况下，SvelteKit 会先在服务端渲染页面，然后将 HTML 发送给客户端，在客户端进行水合。如果设置 `ssr=false`，SvelteKit 会渲染一个“空”页面。适用于页面无法在服务端渲染的情况比如使用了浏览器的 API 如 document 等。当然，大部分情况下并不推荐这样做。因为 SSR 对 SEO 更好，且性能更好一点。

```javascript
// +page.js
export const ssr = false;
// 如果 `ssr` 和 `csr` 都是 `false`, 啥也不渲染！
```

如果在根布局中添加 `export const ssr = false`，整个应用就是仅在客户端渲染，相当于变成了 SPA 应用。

## 5\. csr

有些页面并不需要 JavaScript，比如一些静态“关于”页面，此时可以禁用 CSR：

```javascript
// +page.js
export const csr = false;
// 如果 `ssr` 和 `csr` 都是 `false`, 啥也不渲染！
```

禁用 CSR 意味着不会向客户端发送 JavaScript：

1. 网页只靠 HTML 和 CSS 工作
2. 所有 Svelte 组件内的 `<script>` 标签都会被删除
3. `<form>` 无法渐进式增强
4. 跳转时页面会刷新
5. 不能热更新（HMR）

如果只在开发期间启用 CSR（利用 HMR），可以这样写：

```javascript
// +page.js
import { dev } from '$app/environment';

export const csr = dev;
```

## 6\. trailingSlash

`trailingSlash` 选项用于配置页面尾部斜杠。默认情况下，SvelteKit 会删除 URL 中的尾部斜杠，比如访问 `/about/`会重定向到 `/about`。`trailingSlash` 就是用于更改此行为（通常建议不要改）：

```javascript
// +page(.server).js/+layout(.server).js/+server.js
export const trailingSlash = 'always' | 'never'（默认） | 'ignore'
```

该选项也会影响预渲染，举个例子，如果 trailingSlash 是 `always`，`/about` 会生成 `about/index.html`，否则会创建 `/about.html`。

## 7\. config

借助适配器，SvelteKit 能够在各种平台上运行，每一个平台可能都需要一些特定的配置来调整部署，比如 Vercel 可以让你选择将应用的部分部署在 edge 上，或将其他部分部署在 serverless 环境。

此时就需要 config 选项，它具体有哪些字段，取决于使用的适配器：

```javascript
export const config = {
  runtime: 'edge'
};
```

注意 config 对象的合并只在顶层，并不是深合并。比如这是布局的配置：

```javascript
export const config = {
  runtime: 'edge',
  regions: 'all',
  foo: {
    bar: true
  }
}
```

这是页面的配置：

```javascript
export const config = {
  regions: ['us1', 'us2'],
  foo: {
    baz: true
  }
}
```

最终的配置是：

```json
{
  "runtime": "edge",
  "regions": ["us1", "us2"],
  "foo": { "baz": true }
}
```