> 推荐学习指数：⭐️️，了解即可

## 1\. 前言

在 Next.js 中，客户端导航使用 `<Link>` 组件实现，但 SvelteKit 直接使用 `<a>`标签实现。猛一看，使用 `<a>`更为自然，但细细一想，这样就修改了原生 `<a>`标签的默认行为。如果一个链接我不希望点击后被 JS 拦截，而是正常刷新页面该怎么是实现呢？

本篇讲解的链接选项就是为了解决这一问题。只要在 `<a>`标签上添加 `data-sveltekit-*`属性就可以自定义链接的行为。

当然这些属性不止可用于 `<a>`标签，也可以用于 `<a>` 的父元素，也适用于 `<form method="GET">`。

## 2\. data-sveltekit-preload-data

当用户将鼠标悬停在链接上，或者触发了 `touchstart`或 `mousedown`事件，此时用户很有可能会触发 `click`事件，SvelteKit 就会开始预加载数据，这会为应用多处几百毫秒的加载时间，让应用表现更加自然。

`data-sveltekit-preload-data`属性就是控制这种行为，有两个值：

1. `"hover"`：鼠标悬停的时候就开始预加载。如果是移动设备，则从 `touchstart` 开始
2. `"tap"`：从 `touchstart` 或 `mousedown` 事件开始，适用于预加载会导致误报的情况，因为预加载会调用目标页面的 load 函数

默认项目模板的 `src/app.html`中的 `<body>`元素就应用了 `data-sveltekit-preload-data="hover"`：

```xml
<body data-sveltekit-preload-data="hover">
  <div style="display: contents">%sveltekit.body%</div>
</body>
```

这意味着每个链接都会在鼠标悬停时预加载。

### 2.1. 拦截路由

也可以调用 `$app/navigation`的 `preloadData`实现预加载，这种实现方式类似于 Next.js 的拦截路由效果。让我们写个例子，目录结构如下：

```diff
routes
├─ posts
│  ├─ [id]
│  │  ├─ +page.js
│  │  └─ +page.svelte
│  ├─ +page.js
│  └─ +page.svelte
```

新建 `src/routes/posts/+page.js`，代码如下：

```javascript
export async function load() {
  const posts = [
    {
      userId: 1,
      id: 1,
      title:
        "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
      body: "quia et suscipit
suscipit recusandae consequuntur expedita et cum
reprehenderit molestiae ut ut quas totam
nostrum rerum est autem sunt rem eveniet architecto",
    },
    {
      userId: 1,
      id: 2,
      title: "qui est esse",
      body: "est rerum tempore vitae
sequi sint nihil reprehenderit dolor beatae ea dolores neque
fugiat blanditiis voluptate porro vel nihil molestiae ut reiciendis
qui aperiam non debitis possimus qui neque nisi nulla",
    },
    {
      userId: 1,
      id: 3,
      title: "ea molestias quasi exercitationem repellat qui ipsa sit aut",
      body: "et iusto sed quo iure
voluptatem occaecati omnis eligendi aut ad
voluptatem doloribus vel accusantium quis pariatur
molestiae porro eius odio et labore et velit aut",
    },
  ];

  return {
    posts,
  };
}
```

新建 `src/routes/posts/+page.svelte`，代码如下：

```xml
<script>
  let { data } = $props();
  import { preloadData, pushState, goto } from '$app/navigation';
  import { page } from '$app/stores';
</script>

<h1>Posts list</h1>

<ul>
  {#each data.posts as { id, title }}
    <li class="list-disc list-inside underline text-blue-500">
      <a
        href={`/posts/${id}`}
        onclick={async (e) => {
          e.preventDefault();
          const { href } = e.currentTarget;
          const result = await preloadData(href);
          if (result.type === 'loaded' && result.status === 200) {
            console.log(result.data);
            pushState(href, { selected: result.data });
          } else {
            goto(href);
          }
        }}>{title}</a
      >
    </li>
  {/each}
</ul>

{#if $page.state.selected}
  <div
    class="flex h-screen w-full place-content-center items-center fixed left-0 bottom-0 py-5 bg-gray-500 bg-opacity-50"
  >
    <div class="bg-white mx-10 rounded-md p-5">
      <h2>{$page.state.selected.post.title}</h2>
      <hr />
      {$page.state.selected.post.body}
    </div>
    <button class="absolute right-5 top-0 text-lg" onclick={() => history.back()}>x</button>
  </div>
{/if}
```

新建 `src/routes/posts/[id]/+page.js`，代码如下：

```javascript
import { error } from "@sveltejs/kit";

export async function load({ params }) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${params.id}`
  );
  const post = await response.json();
  if (post.title) {
    return {
      post,
    };
  }

  error(404, "Not found");
}
```

新建 `src/routes/posts/[id]/+page.svelte`，代码如下：

```javascript
<script>
  let { data } = $props();
</script>

Post Content

<hr />

<h2>{data.post.title}</h2>

<hr />

{data.post.body}
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32129a0d2a21432a8242c13ff91b0ab9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=867&h=485&s=223353&e=gif&f=73&b=fefefe)

其实这种交互效果在国内网站并不常见，国外的一些艺术网站比如 [dribbble.com](https://link.juejin.cn/?target=https%3A%2F%2Fdribbble.com%2F "https://link.juejin.cn/?target=https%3A%2F%2Fdribbble.com%2F") 有类似的效果。当点击链接的时候，会在 Modal 中渲染链接的内容，但此时路由已经发生了变化，用户如果直接复制地址分享给其他用户，打开后就会直接进入分享的内容界面，而不是 Modal。这种交互的好处在于不打断已有的浏览体验，让用户继续停留在重要的页面上。

## 3\. data-sveltekit-preload-code

如果不想预加载数据，也可以预加载代码。

`data-sveltekit-preload-code` 有 4 个值：

* `"eager"`：链接立刻预加载
* `"viewport"` ：链接进入视口后预加载
* `"hover"`：类似于 preload-data，但仅预加载代码
* `"tap"`：类似于 preload-data，但仅预加载代码

这是开发环境预加载数据的效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16cda7efb19f4058b3659e0b3eecf66e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3474&h=758&s=343464&e=png&b=2d2d2d)

这是开发环境预加载代码的效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f888f353be840149ebe93a9ba094c96~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3330&h=716&s=294269&e=png&b=2b2b2b)

可以看出：预加载代码不会触发目标路由的 `load` 函数加载。

> 注意：`viewport` 和 `eager` 仅应用于导航后立刻出现在 DOM 中的链接，如果链接稍后才添加（比如在 `{#if ...}`中），则不会被预加载，直到被 `hover` 或 `tap`触发。这是为了避免监控 DOM 变化而造成的性能损失。

> 注意：当 preload data 和 preload code 一起使用的时候，要注意，preload code 是 preoload data 的前提条件，所以只有 preload code 比 preload data 更急切（eager）的时候，该属性才会起作用。举个例子，preload data 是 hover，preload code 是 tap，鼠标悬浮的时候预加载数据，点击的时候预加载代码，此时 preload code 不会起作用，最终表现是鼠标悬浮的时候预加载数据。

## 4\. data-sveltekit-reload

有的时候，你可能并不希望走客户端导航，而是正常的跳转刷新页面，此时可以添加 data-sveltekit-reload 属性：

```javascript
<a data-sveltekit-reload href="/path">
  Path
</a>
```

具有 `rel="external"`属性的链接也会是相同的处理。

## 5\. data-sveltekit-replacestate

`data-sveltekit-replacestate`在导航时不会创建新的条目，而是替换当前条目：

```javascript
<a data-sveltekit-replacestate href="/path">
  Path
</a>
```

## 6\. data-sveltekit-keepfocus

如果希望导航后，焦点不要重置，比如希望搜索表单提交后，用户的注意力依然在文本输入上：

```html
<form data-sveltekit-keepfocus>
  <input type="text" name="query" />
</form>
```

此时表单提交后，表单元素依然保留焦点。

## 7\. data-sveltekit-noscroll

当导航后，页面会滚动到顶点（除非链接包含 `#hash`），如果希望禁用这种行为：

```html
<a href="path" data-sveltekit-noscroll>Path</a>
```

## 8\. 禁用选项

如果要在已启用这些选项的元素上禁用这些选项，可以使用 `false`值：

```html
<div data-sveltekit-preload-data>
  <!-- 链接预加载 -->
  <a href="/a">a</a>
  <a href="/b">b</a>
  <a href="/c">c</a>

  <div data-sveltekit-preload-data="false">
    <!-- 链接不会预加载 -->
    <a href="/d">d</a>
    <a href="/e">e</a>
    <a href="/f">f</a>
  </div>
</div>
```

也可以有条件的应用到元素上：

```html
<div data-sveltekit-preload-data={condition ? 'hover' : false}>
```