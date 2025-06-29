> 推荐学习指数：⭐️️，了解即可

## 1\. 前言

Hooks 是应用级别的函数，可以响应特定事件，从而对框架行为进行更精确的控制。

> 注：Hooks 在不同框架下有不同的含义。在 SvelteKit 下，hooks 的作用类似于中间件。

有 3 个 hooks 文件，皆是可选（可以有这个文件，也可以没有这个文件）：

1. `src/hooks.server.js`：Server hooks，用于监听服务端事件
2. `src/hooks.client.js`：Client hooks，用于监听客户端事件
3. `src/hooks.js`：Universal hooks，在服务端和客户端运行的 hooks

每个 hooks 文件可以定义的 hook 如下：

|**hooks.server.js**|**hooks.client.js**|**hooks.js**|
|---|---|---|
|1\. `handle`2\. `handleFetch`3\. `handleError`|1\. `handleError`|1\. `reroute`|

通过定义 hook 函数，可以全局控制应用的服务端请求、数据获取、错误处理等。这些模块的代码会在应用程序启动的时候运行，因此也可用于初始化数据库客户端等。

接下来我们开始详细介绍这些 hooks。

## 2\. Server hooks

`src/hooks.server.js`可以添加这两个 hook：

### 2.1. handle

每当 SveteKit 服务端收到请求（无论请求是在运行时还是预渲染过程中），该函数就会运行并决定响应。

新建 `src/hooks.server.ts`，代码如下：

```javascript
export async function handle({ event, resolve }) {
  return resolve(event);
}
```

此时不会有任何行为，因为这就是默认行为。让我们具体看下 handle 函数。

#### 2.1.1. 基础使用

handle 函数接收一个`input` 对象，该对象有 2 个属性 —— `event` 和 `resolve`。 其中 `event` 是[RequestEvent](https://kit.svelte.dev/docs/types#public-types-requestevent "https://kit.svelte.dev/docs/types#public-types-requestevent") 类型，存储着本次请求的相关信息。`resolve` 是函数，该函数会渲染路由并创建 Response。

我们看个例子：

```javascript
// src/hooks.server.js
export async function handle({ event, resolve }) {
  if (event.url.pathname.startsWith("/custom")) {
    return new Response("custom response");
  }

  const response = await resolve(event);
  return response;
}
```

此时访问 `/custom`开头的路由（哪怕没有定义），都会返回：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c13bcf47567143e9bc31766da3f6909c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1572&h=340&s=33724&e=png&b=1b1b1b)

#### 2.1.2. 自定义数据

`handle` 返回的数据会传递给 `+server.js`和 server load 函数（`+[page|layout].server.js`），如果要添加自定义数据，可以填充到 `event.locals` 对象（这个对象就是专门用来自定义数据的）：

```javascript
// src/hooks.server.js
export async function handle({ event, resolve }) {
  // event.locals.user = await getUserInformation(event.cookies.get('sessionid'));
  event.locals.user = "YaYu";

  const response = await resolve(event);
  response.headers.set("x-custom-header", "potato");

  return response;
}
```

上面这段代码是一个身份验证的简单例子，通过 cookies 中的 sessionid 获取用户信息，然后将数据放到 `event.locals` 中，server load 函数可以获取到该自定义信息。

新建 `src/routes/+layout.server.js`，代码如下：

```javascript
export async function load({ locals }) {
  return { user: locals.user };
}
```

此时页面 `+page.svelte`就可以通过 `data`或者 `$page.data`获取：

```xml
<script>
  let { data } = $props();
  import { page } from '$app/stores';
</script>

{#if data.user}
  Welcome {data.user}!
{/if}

{#if $page.data.user}
  Welcome {$page.data.user}!
{/if}
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9dce6861c07e4cce8b240134aaf6ce66~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1588&h=284&s=35704&e=png&b=fefefe)

注意：`handle` 会在 action 调用之前运行，并且不会在 load 函数之前重新运行。所以如果使用 handle 基于 cookie 修改了 event.locals，记得在设置或删除 cookie 的 action 中更新 event.locals：

```javascript
// src/hooks.server.js
export async function handle({ event, resolve }) {
  event.locals.user = await getUser(event.cookies.get("sessionid"));
  return resolve(event);
}

// src/routes/account/+page.server.js
export function load(event) {
  return {
    user: event.locals.user,
  };
}

export const actions = {
  logout: async (event) => {
    event.cookies.delete("sessionid", { path: "/" });
    event.locals.user = null;
  },
};
```

#### 2.1.3. resolve 函数

`resolve` 还支持第二个可选参数，可以对响应的渲染方式进行更多控制，该参数是一个对象：

* `transformPageChunk`：其语法如下：

```bash
transformPageChunk(opts: { html: string, done: boolean }): MaybePromise<string | undefined>
```

用于自定义 HTML 转换。如果 done 为 true，表示是最终的块，之所以分成 chunk，是考虑到流式渲染。

我们举个例子，修改 `src/app.html`，代码如下：

```html
<!DOCTYPE html>
<html lang="%lang%">
  // ...
</html>
```

我们给 HTML 添加了 lang 属性，希望将 `%lang%`替换为 cookies 中设置的 locale 值：

```javascript
// src/hooks.server.js
export async function handle({ event, resolve }) {
  let page = '';
  // 从 event.cookies 中获取 locale，假设为 zh
  const locale = 'zh'

  return resolve(event, {
    transformPageChunk: ({ html, done }) => {
      page += html;
      if (done) {
        return html.replace('%lang%', locale)
      }
    }
  });
}
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6269053c88a4ea68afc6b274166e757~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2228&h=698&s=159410&e=png&b=2a2a2a)

* `filterSerializedResponseHeaders`：其语法如下：

```bash
filterSerializedResponseHeaders(name: string, value: string): boolean
```

用于决定当 `load` 函数使用 `fetch` 加载资源时，序列化响应中应包含哪些标头。默认不包含任何标头信息。

```bash
export async function handle({ event, resolve }) {
  const response = await resolve(event, {
    filterSerializedResponseHeaders: (name) => name.startsWith('x-'),
  });

  return response;
}
```

* `preload`：其语法如下：

```bash
preload(input: { type: 'js' | 'css' | 'font' | 'asset', path: string }): boolean
```

用于确定将哪些文件添加到 `<head>`标签中以进行预加载。在构建代码块时，该方法会与构建时发现的文件一起运行。比如 `+page.svelte`中 `import './styles.css'`，那么在访问该页面时，preload 将与该 css 文件的解析地址一起调用。

注意开发模式下不会调用 preload，它取决于构建时进行的分析。默认情况下，js 和 css 文件会被预加载。

```javascript
export async function handle({ event, resolve }) {
  const response = await resolve(event, {
    preload: ({ type, path }) => type === "js" || path.includes("/important/"),
  });

  return response;
}
```

#### 2.1.4. 定义多个函数

实际项目开发中，handle 的处理可能会很复杂，有的处理国际化，有的处理身份验证，当代码混淆在一起的时候，很快就会积重难返，此时可以定义多个 handle 函数将逻辑分离，然后借助 `sequence`工具函数帮助执行。

一个简单的例子如下：

```javascript
import { sequence } from "@sveltejs/kit/hooks";

async function first({ event, resolve }) {
  console.log("first pre-processing");
  event.locals.user = "YaYu";
  const response = await resolve(event);
  console.log("first post-processing");
  return response;
}

async function second({ event, resolve }) {
  console.log("second pre-processing");
  const response = await resolve(event);
  response.headers.set("x-custom-header", "potato");
  console.log("second post-processing");
  return response;
}

export const handle = sequence(first, second);
```

打印顺序如中间件：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/150b39a49ac4458f87116956416853ec~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1560&h=224&s=37617&e=png&b=1e1e1e)

`handle` 选项的行为如下：

1. `transformPageChunk` 以相反的顺序合并应用
2. `preload` 按正向顺序应用，只会使用第一个选项（意思是只能设置一次）
3. `filterSerializedResponseHeaders` 同 `preload`

让我们再看个例子：

```javascript
import { sequence } from "@sveltejs/kit/hooks";

/// type: import('@sveltejs/kit').Handle
async function first({ event, resolve }) {
  console.log("first pre-processing");
  const result = await resolve(event, {
    transformPageChunk: ({ html }) => {
      // transforms are applied in reverse order
      console.log("first transform");
      return html;
    },
    preload: () => {
      // this one wins as it's the first defined in the chain
      console.log("first preload");
    },
  });
  console.log("first post-processing");
  return result;
}

/// type: import('@sveltejs/kit').Handle
async function second({ event, resolve }) {
  console.log("second pre-processing");
  const result = await resolve(event, {
    transformPageChunk: ({ html }) => {
      console.log("second transform");
      return html;
    },
    preload: () => {
      console.log("second preload");
    },
    filterSerializedResponseHeaders: () => {
      // this one wins as it's the first defined in the chain
      console.log("second filterSerializedResponseHeaders");
    },
  });
  console.log("second post-processing");
  return result;
}

export const handle = sequence(first, second);
```

打印顺序是：

```bash
first pre-processing
first preload
second pre-processing
second filterSerializedResponseHeaders
second transform
first transform
second post-processing
first post-processing
```

按照上面的规则很容易就推得此结果。

### 2.2. handleFetch

通过该函数，可以修改（或替换）服务端（或预渲染）`load` 或 `action` 函数内部发起的 `fetch` 请求。默认行为类似于：

```javascript
// src/hooks.server.js
export async function handleFetch({ event, request, fetch }) {
  return await fetch(request);
}
```

比如我们将请求统一改为 `https` 请求：

```javascript
// src/hooks.server.js
export async function handleFetch({ request, fetch }) {
  if (request.url.startsWith("http")) {
    const url = request.url.replace("http", "https");
    request = new Request(url, request);
  }

  return fetch(request);
}
```

比如当用户导航至自己应用的页面，与其使用公共互联网访问，不如直接访问本机 API：

```javascript
export async function handleFetch({ request, fetch }) {
  if (request.url.startsWith("https://api.yourapp.com/")) {
    // 克隆原生请求，但是修改地址
    request = new Request(
      request.url.replace("https://api.yourapp.com/", "http://localhost:9999/"),
      request
    );
  }

  return fetch(request);
}
```

注意上面这段代码，需要请求运行在服务端的时候，比如 `+page.server.js`

#### 2.2.1. Credentials

对于同源请求，SvelteKit 的 `fetch` 将转发 `cookie` 和 `authorization` 标头，除非 `credentials` 选项设置为 `"omit"`。

对于跨源请求，如果请求 URL 属于应用程序的子域，则会包含 cookie，比如应用是 `my-domain.com` 上，而 API 位于 `api.my-domain.com` 上，则会在请求中包含 cookie。

如果应用程序和 API 位于同级子域上（例如 `www.my-domain.com` 和 `api.my-domain.com`），那么属于共同父域（例如 `my-domain.com` ）的 cookie 将不会被包含在内，因为 SvelteKit 无法知道 cookie 属于哪个域。在这种情况下，需要使用 handleFetch 手动包含 cookie：

```javascript
export async function handleFetch({ event, request, fetch }) {
  if (request.url.startsWith("https://api.my-domain.com/")) {
    request.headers.set("cookie", event.request.headers.get("cookie"));
  }

  return fetch(request);
}
```

## 3\. Server hooks 和 Client hooks

可在 `src/hooks.server.js` 和 `src/hooks.client.js` 中添加以下内容：

### 3.1. handleError

如果在加载或渲染过程中出现意外错误，将调用该函数，并传递 `error`、`event`、`status`状态码和 `message`。开发者可以：

* 记录错误日志
* 生成一个可安全显示的自定义错误，省略堆栈等敏感信息

其中，`error`是错误信息，`event`是请求信息。如果是开发者代码引发的错误，一般状态码 `status`为 500，消息 `message` 为 `"Internal Error"` 。这个错误信息初学的时候经常遇到：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4dca6e8c6b2048c3b50134e8382ec0c0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1142&h=466&s=36491&e=png&b=ffffff)

handleError 可以搭配错误监控系统如 Sentry 一起工作：

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

```javascript
// src/hooks.client.js
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

注意：预期之内的错误并不会调用该函数，比如使用 `@sveltejs/kit`的 `error`函数引发的错误。

## 4\. Universal hooks

以下内容添加到 `src/hooks.js`中，通用 hooks 会在服务端和客户端上运行：

### 4.1. reroute

此函数会在 `handle`之前运行，用于定义 URL 如何转换为路由，功能类似于 Next.js 的 rewrite。比如将 `/about`通过 `reroute` 重写为 `/company`后，将使用 `/company`对应的路由文件进行处理。

该函数返回路径名（默认是 `url.pathname`），用于选择路由及其参数。

比如有一个 `src/routes/[[lang]]/about/+page.svelte`页面，你希望 `/en/about`、`/de/ueber-uns` 、`/fr/a-propos`都指向该页面，那就可以使用 `reroute` 来实现：

```javascript
// src/hooks.js

const translated = {
  "/en/about": "/en/about",
  "/de/ueber-uns": "/de/about",
  "/fr/a-propos": "/fr/about",
};

export function reroute({ url }) {
  if (url.pathname in translated) {
    return translated[url.pathname];
  }
}
```

注意：

1. `lang`参数将从返回的路径名中派生
2. `reroute` 不会更改浏览器地址栏中的内容（跟重定向不同）或 `event.url` 的值