> 推荐学习指数：⭐️️，了解即可

## 1\. 前言

SvelteKit 中有两种错误：

1. 预期错误（expected errors）
2. 非预期错误（unexpected errors）

默认情况下，这两种错误都表示为简单的 `{ message: string }`对象。本篇就来详细讲解下这两种错误。

## 2\. 预期错误

预期错误是指使用 `@sveltejs/kit`的 `error`函数创建的错误：

```javascript
// src/routes/blog/[slug]/+page.server.js
import { error } from "@sveltejs/kit";
import * as db from "$lib/server/database";

export async function load({ params }) {
  const post = await db.getPost(params.slug);

  if (!post) {
    error(404, {
      message: "Not found",
    });
  }

  return { post };
}
```

`error()`的第一个参数是 HTTP 状态码，第二个参数是可选消息。如果只有 `message` 字段，也可以为了方便简写为：

```javascript
error(404, "Not found");
```

`error` 会抛出一个 SvelteKit 会捕获的异常，并渲染最近的 `+error.svelte`，函数的实参分别可以通过 `$page.status` 和 `{$page.error}` 获取：

```javascript
<script>
  import { page } from '$app/stores';
</script>

<h1>{$page.status}: {$page.error.message}</h1>
```

当然你也可以为错误添加一些额外的属性（此时就不能简写了）：

```javascript
error(404, {
  message: "Not found",
  code: "NOT_FOUND",
});
```

## 3\. 非预期错误

非预期错误就是处理请求时发生的其他异常，由于可能包含敏感信息，所以不会暴露给用户。默认情况下，会输出到控制台（或者在生产环境下，输出到服务端日志），向用户公开的是通用的 500 错误，message 为：

```javascript
{ "message": "Internal Error" }
```

浏览器效果为：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1700808cd5604276a33dd07da5b7370c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1142&h=466&s=7820&e=webp&b=ffffff)

非预期错误可以通过 `handleError` hook 自定义处理方式 ，比如上报错误日志。

```javascript
// src/hooks.server.js
import * as Sentry from "@sentry/sveltekit";

Sentry.init({
  /*...*/
});

export async function handleError({ error, event, status, message }) {
  const errorId = crypto.randomUUID();

  // example integration with https://sentry.io/
  Sentry.captureException(error, {
    extra: { event, errorId, status },
  });

  return {
    message: "Whoops!",
    errorId,
  };
}
```

浏览器效果为：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21529d43570b4f888d1a3a85b56ad631~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1202&h=484&s=37492&e=png&b=ffffff)

handleError 返回的自定义信息可以通过 `$page.error` 获取。

## 4\. Responses

如果是 `handle`hook 或 `+server.js`请求内部出现错误，SveteKit 将根据 `Accept`标头决定返回错误对象的 JSON 形式还是 fallback 错误页面。

添加 `src/error.html` 文件可自定义 fallback 错误页面：

```javascript
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

SvelteKit 会将 `%sveltekit.status%` 和 `%sveltekit.error.message%` 替换为相应的值。

如果 load 函数内部发生错误，SveteKit 将会渲染最近的 `+error.svelte`。

注意如果错误发生在 `+layout(.server).js` 的 load 函数，最近的 `+error.svelte`在布局上级，而非同级。除非如果错误发生在根布局 `+layout.js`或 `+layout.server.js`内部，此时 SvelteKit 会使用 fallback 错误页面。

## 5\. 类型安全

如果使用 TypeScript 并且添加了自定义的错误属性，你可以在 `src/app.d.ts`中进行声明一个 App.Error 接口：

```diff
// src/app.d.ts
declare global {
  namespace App {
    interface Error {
+     code: string;
+     id: string;
    }
  }
}

export {};
```

该接口始终包含 `message: string` 属性。