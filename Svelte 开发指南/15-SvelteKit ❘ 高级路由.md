> 推荐学习指数：⭐️⭐⭐️，必学内容️

## 1\. 前言

本篇我们继续介绍路由。

## 2\. 动态路由

有的时候，你并不能提前知道路由的地址，就比如根据 URL 中的 id 参数展示该 id 对应的文章内容，文章那么多，我们不可能一一定义路由，这个时候就需要用到动态路由。

### 2.1. \[folderName\]

**使用动态路由，你需要将文件夹的名字用方括号括住，比如 `[id]`、`[slug]`。这个路由的具体值会传给 `+page.svelte`、`+page.js`等文件。**

新建 `src/routes/posts/[id]/+page.svelte`，代码如下：

```xml
<script>
  import { page } from '$app/stores';
  let { data } = $props();
</script>

{ $page.params.id } Post Content

<hr />

<h2>{ data.post.title }</h2>

<hr />

{ data.post.body }
```

如果 `+page.svelte`需要获取动态路由参数，可以从 `$app/stores`中获取。

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

如果 `+page(.server).js`需要获取动态路由参数，可以从 `load`函数的 `params` 参数中获取。

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50ac40076fe349e4a7b32a81175926cc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2766&h=874&s=161019&e=png&b=fefefe)

### 2.2. \[...folderName\] 剩余参数

**在命名文件夹的时候，如果你在方括号内添加省略号，比如 `[...folderName]`，表示捕获匹配的所有路由片段。**

新建 `src/routes/blog/[...id]/+page.svelte`，代码如下：

```xml
<script>
  import { page } from '$app/stores';
</script>

{JSON.stringify($page.params, null, 2)}
```

|路由地址|params 值|
|---|---|
|`/blog`|`{ "id": "" }`|
|`/blog/a`|`{ "id": "a" }`|
|`/blog/a/b`|`{ "id": "a/b" }`|
|`/blog/a/b/c`|`{ "id": "a/b/c" }`|
|...||

与 Next.js 不同，`[...folderName]`也可用于路由中间，比如 `src/routes/a/[...rest]/z/+page.svelte`可以匹配 `/a/z`（此时值为空），也可匹配 `/a/b/z` 和 `/a/b/c/z` 等值。

`[...folderName]` 这种剩余参数用法可用于渲染自定义的 404 页面。比如这样一个目录结构：

```xml
src/routes/
├ blog/
│ ├ javascript/
│ ├ css/
│ ├ html/
│ └ +error.svelte
└ +error.svelte
```

当访问 `/blog/javascript/react` 的时候，并不会渲染 `blog/+error.svelte`，因为没有匹配的路由。如果你想自定义该错误页面，就需要使用 \[...xxx\] 匹配到这些路由：

```diff
  src/routes/
  ├ blog/
+ │ ├ [...notfound]
+ │ │  └ +page.svelte
  │ ├ javascript/
  │ ├ css/
  │ ├ html/
  │ └ +error.svelte
  └ +error.svelte
```

`/src/routes/blog/[...notfound]/+page.svelte`的代码为：

```xml
import { error } from '@sveltejs/kit';

export function load(event) {
  error(404, 'Not Found');
}
```

### 2.3. \[\[folderName\]\] 可选参数

**如果将文件夹的名字用双方括号括住，比如 `[[folderName]]`，表示该路由段可选。**

像 `[lang]/home`可以匹配 `/en/home`、`/zh/home`，但不能匹配 `/home`。而 `[[lang]]/home`，可以匹配 `/en/home`、`/zh/home`，也可以匹配 `/home`。

> 注意：可选参数不能跟在剩余参数后面 (`[...rest]/[[optional]]`这样是不行的)，因为参数会贪婪匹配，可选参数将始终为空

## 3\. 匹配器

匹配器可以让路由进行更精确的匹配。比如 `src/routes/posts/[id]`，我只希望匹配 id 从 1 到 50 的整数（因为我们用的模拟数据接口就只支持 1 到 100，为了做区分，我们只匹配 1 到 50），此时就可以借助匹配器。

匹配器的用法比较特殊。首先在 `src/params`文件夹下建一个导出 `match` 函数的文件。

新建 `src/params/blog.js`，代码如下：

```javascript
export function match(param) {
  return Number.isInteger(+param) && param > 0 && param < 51;
}
```

该函数会接受动态路由值作为参数，通过返回 true/false 决定是否成功匹配。如果不匹配，则会继续进行其他匹配，如果都不匹配，则是 404。

然后修改文件夹的名称：

```diff
-src/routes/posts/[id]
+src/routes/posts/[id=blog]
```

此时访问 [http://localhost:5173/posts/52](http://localhost:5173/posts/52 "http://localhost:5173/posts/52")，浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3218c50448af4eb1a0397d53ea318303~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1266&h=648&s=52369&e=png&b=ffffff)

> 注意：`src/params`的每个模块都对应一个匹配器。匹配器既运行在服务端也运行在客户端。

## 4\. 优先级

当多个路由都匹配一个路径的时候，就需要进行优先级判断。比如：

```diff
src/routes/[...catchall]/+page.svelte
src/routes/[[a=x]]/+page.svelte
src/routes/[b]/+page.svelte
src/routes/foo-[c]/+page.svelte
src/routes/foo-abc/+page.svelte
```

这些路由都可以匹配 `/foo-abc`，那访问 `/foo-abc`的时候，会使用哪个文件进行处理呢？

优先级规则：

1. 更具体的路由优先级更高（比如普通路由比动态路由更具体）
2. 带匹配器的路由比不带匹配器的路由优先级更高
3. `[[optional]]` 和 `[...rest]`以最低优先级进行处理
4. 如果前面都一样那就按字母顺序解决

所以让我们再回看下这个例子，它的优先级排序应该是：

```diff
# 1. 最具体
src/routes/foo-abc/+page.svelte
# 2.
src/routes/foo-[c]/+page.svelte
# 3. 带匹配器
src/routes/[[a=x]]/+page.svelte
# 4.
src/routes/[b]/+page.svelte
# 5.
src/routes/[...catchall]/+page.svelte
```

## 5\. 路由组

**将文件夹用括号括住如`(auth)`就可以声明一个路由组，** 比如这样一个目录：

```diff
src/routes/
│ (app)/
│ ├ dashboard/
│ ├ item/
│ └ +layout.svelte
│ (marketing)/
│ ├ about/
│ ├ testimonials/
│ └ +layout.svelte
├ admin/
└ +layout.svelte
```

首先路由组并不会影响路由地址，比如 `src/routes/(app)/dashboard`对应的路由地址依然是 `/dashboard`。

然后使用路由组可以让不同路由组使用不同的布局结构。比如 `(app)/dashboard`和`(app)/item`使用 `(app)/+layout.svelte`布局，而 `(marketing)/about`和`(marketing)/testimonials`使用 `(marketing)/+layout.svelte`布局。而 admin 只使用 `+layout.svelte` 布局。

### 5.1. 脱离布局

通常，布局会进行继承，比如这样一个目录结构：

```diff
src/routes/
├ (app)/
│ ├ item/
│ │ ├ [id]/
│ │ │ ├ embed/
│ │ │ │ └ +page.svelte
│ │ │ └ +layout.svelte
│ │ └ +layout.svelte
│ └ +layout.svelte
└ +layout.svelte
```

`(app)/item/[id]/embed/+page.svelte`会继承根布局、`(app)` 布局、`item` 布局、`[id]` 布局。

我们可以通过在文件名中添加 `@`指定继承的布局。比如：

* `+page@[id].svelte`： 继承 `src/routes/(app)/item/[id]/+layout.svelte`，跟当前一致
* `+page@item.svelte`： 继承 `src/routes/(app)/item/+layout.svelte`
* `+page@(app).svelte`： 继承 `src/routes/(app)/+layout.svelte`
* `+page@.svelte`： 继承 `src/routes/+layout.svelte`

指定继承的布局后，比如 `+page@(app).svelte` 会继承根布局、(app) 布局，但不会继承 `item` 布局、`[id]` 布局的内容。

布局也可以指定继承的布局：

```diff
src/routes/
├ (app)/
│ ├ item/
│ │ ├ [id]/
│ │ │ ├ embed/
│ │ │ │ └ +page.svelte  // 使用 (app)/item/[id]/+layout.svelte
│ │ │ ├ +layout.svelte  // 继承 from (app)/item/+layout@.svelte
│ │ │ └ +page.svelte    // 使用 (app)/item/+layout@.svelte
│ │ └ +layout@.svelte   // 继承根布局, 会跳过 (app)/+layout.svelte
│ └ +layout.svelte
└ +layout.svelte
```

## 最后

路由是每一个全栈框架的基础知识，可以在开发中慢慢体会。