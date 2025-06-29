> 推荐学习指数：⭐️️⭐️️⭐️️，必学内容

## 1\. 前言

不同于 Next.js 的构建部署，SveleKit 为适应不同平台的部署，引入了“适配器”的概念。所以 SvelteKit 应用的构建（`npm run build`）分为 2 个阶段：

1. 第一阶段：通过 Vite 构建一个优化的生产版本
2. 第二阶段：适配器根据目标环境对生产构建进行调整

SvelteKit 提供了一些官方适配器可以直接使用，可以应对大部分环境。本篇就来讲解这些适配器的使用方式。

## 2\. 构建命令

我们先说说常用的 2 个构建部署命令 —— `npm run build`和 `npm run preview`

### 2.1. `npm run build`

当运行 `npm run build`的时候，SvelteKit 将加载 `+page/layout(.server).js`以及它们导入的所有文件，以便在构建过程中进行分析。如果要对代码运行阶段进行判断，可以使用 `$app/environment`中的 `building` 字段：

```diff
+import { building } from '$app/environment';
import { setupMyDatabase } from '$lib/server/database';

+if (!building) {
  setupMyDatabase();
+}

export function load() {
  // ...
}
```

### 2.2. `npm run preview`

构建完成后，可以使用 `npm run preivew`本地预览生产构建。但注意这是在 Node 环境中运行，针对适配器的特定调整并不能在预览中看到

## 3\. 适配器

在部署 SvelteKit 应用程序之前，需要根据部署目标选择合适的适配器。适配器本质是一个小型插件，它将构建代码作为输入，输出用于生产的代码。

常用的官方适配器如下：

* [@sveltejs/adapter-cloudflare](https://kit.svelte.dev/docs/adapter-cloudflare "https://kit.svelte.dev/docs/adapter-cloudflare")：用于 Cloudflare 页面
* [@sveltejs/adapter-cloudflare-workers](https://kit.svelte.dev/docs/adapter-cloudflare-workers "https://kit.svelte.dev/docs/adapter-cloudflare-workers")：用于 Cloudflare Workers
* [@sveltejs/adapter-netlify](https://kit.svelte.dev/docs/adapter-netlify "https://kit.svelte.dev/docs/adapter-netlify")：用于 Netlify
* [@sveltejs/adapter-node](https://kit.svelte.dev/docs/adapter-node "https://kit.svelte.dev/docs/adapter-node")：用于 Node Server
* [@sveltejs/adapter-static](https://kit.svelte.dev/docs/adapter-static "https://kit.svelte.dev/docs/adapter-static")：用于静态网站（SSG）
* [@sveltejs/adapter-vercel](https://kit.svelte.dev/docs/adapter-vercel "https://kit.svelte.dev/docs/adapter-vercel")：用于 Vercel

社区也提供了一些适配器，[这是列表地址](https://sveltesociety.dev/packages?category=sveltekit-adapters "https://sveltesociety.dev/packages?category=sveltekit-adapters")。

使用适配器的方式也很简单，安装适配器后，在 `svelte.config.js` 中指定：

```javascript
import adapter from 'svelte-adapter-foo';

const config = {
  kit: {
    adapter: adapter({
      // 具体有哪些选项跟适配器有关
    })
  }
};

export default config;
```

## 4\. adapter-auto

当使用 `npm create svelte@latest` 创建新的 SvelteKit 项目时，默认使用的适配器就是 `adapter-auto`。该适配器会自动安装，并在部署时根据环境使用正确的适配器：

* [@sveltejs/adapter-cloudflare](https://kit.svelte.dev/docs/adapter-cloudflare "https://kit.svelte.dev/docs/adapter-cloudflare")：用于 Cloudflare Pages
* [@sveltejs/adapter-netlify](https://kit.svelte.dev/docs/adapter-netlify "https://kit.svelte.dev/docs/adapter-netlify")：用于 Netlify
* [@sveltejs/adapter-vercel](https://kit.svelte.dev/docs/adapter-vercel "https://kit.svelte.dev/docs/adapter-vercel")：用于 Vercel
* [svelte-adapter-azure-swa](https://github.com/geoffrich/svelte-adapter-azure-swa "https://github.com/geoffrich/svelte-adapter-azure-swa")：用于 [Azure 静态 Web 应用程序](https://docs.microsoft.com/en-us/azure/static-web-apps/ "https://docs.microsoft.com/en-us/azure/static-web-apps/")
* [svelte-kit-sst](https://github.com/sst/sst/tree/master/packages/svelte-kit-sst "https://github.com/sst/sst/tree/master/packages/svelte-kit-sst")：用于[通过 SST 构建部署在 AWS](https://docs.sst.dev/start/svelte "https://docs.sst.dev/start/svelte")
* [@sveltejs/adapter-node](https://kit.svelte.dev/docs/adapter-node "https://kit.svelte.dev/docs/adapter-node")：用于 [Google Cloud Run](https://cloud.google.com/run "https://cloud.google.com/run")

也就是说，如果要部署到这些平台，不需要再特别指定适配器，直接部署使用即可。

## 5\. adapter-node

`adapter-node` 适配器用于生成独立的 Node Server，可用于我们自己购买的服务器上。

### 5.1. 使用方式

安装适配器：

```bash
npm i -D @sveltejs/adapter-node
```

修改 `svelte.config.js`：

```javascript
import adapter from '@sveltejs/adapter-node';

export default {
  kit: {
    adapter: adapter()
  }
};
```

此时运行 `npm run build`默认将 `build`目录作为输出目录：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fbfb1d6d9964d63aef25d4d877f1520~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1642&h=752&s=81887&e=png&b=252526)

但不同于 Next.js 的 standalone 模式，这个输出目录还需要依赖项目的 `package.json`和 `node_modules`才能运行。让我们运行以下命令开启 Node 应用：

```bash
node build/index.js
```

简写为：

```bash
node build
```

命令行效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2131e7e4ba914023a0e2d53395e1290d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1472&h=170&s=28861&e=png&b=1e1e1e)

所以如果要部署到自己的服务器，你需要做的就是将代码推到服务器上，然后运行构建，开启 Node 应用。

关于如何将代码推送到服务器，可以参考 [《一篇从购买服务器到部署博客代码的详细教程》](https://juejin.cn/post/7049692191110725645 "https://juejin.cn/post/7049692191110725645")。

如果需要 `node_modules` 只有生产依赖项，那可以运行 `npm ci --omit dev`。`npm ci` 是 npm 的清理安装项命令，`ci`是 `clean install`的缩写，运行之后 `node_modules`将只剩下生产环境依赖项：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25a445fe654c4ee7a36392a25f775154~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1500&h=1010&s=307213&e=png&b=201d26)

### 5.2. 环境变量

在 `dev` 和 `preview` 中，SvelteKit 将从 `.env` 文件（或 `.env.local` ，或 `.env.[mode]` ，由 Vite 决定）读取环境变量。

在生产环境中，`.env`文件不会自动加载，所以需要安装 `dotenv`：

```bash
npm install dotenv
```

运行命令改为：

```diff
-node build
+node -r dotenv/config build
```

如果是 Node.js v20.6+，可以使用 `--env-file`替代：

```diff
-node build
+node --env-file=.env build
```

### 5.3. 端口、主机

默认使用 `3000`端口和 `0.0.0.0`。可以通过 `PORT`和`HOST`环境变量自定义：

```diff
HOST=127.0.0.1 PORT=4000 node build
```

## 6\. adapter-static

`adapter-static`用于将应用生成静态网站（SSG），这会将整个网站预渲染为静态文件集合。

### 6.1. 使用方法

安装适配器：

```bash
npm i -D @sveltejs/adapter-static
```

修改 `svelte.config.js`，代码如下：

```javascript
import adapter from '@sveltejs/adapter-static';

export default {
  kit: {
    adapter: adapter({
      // 这些是默认选项
      pages: 'build',
      assets: 'build',
      fallback: undefined,
      precompress: false,
      strict: true
    })
  }
};
```

在根布局添加 `prerender`选项：

```javascript
// src/routes/+layout.js
export const prerender = true;
```

此时运行 `npm run build`，此时 `build`目录会生成静态的 HTML 文件

如果你想转为 SPA，也可以通过该适配器，参考 [Single-page apps](https://kit.svelte.dev/docs/single-page-apps "https://kit.svelte.dev/docs/single-page-apps")

## 7\. 其他

除了这些适配器：

* [adapter-auto](https://kit.svelte.dev/docs/adapter-auto "https://kit.svelte.dev/docs/adapter-auto")
* [adapter-node](https://kit.svelte.dev/docs/adapter-node "https://kit.svelte.dev/docs/adapter-node")
* [adapter-static](https://kit.svelte.dev/docs/adapter-static "https://kit.svelte.dev/docs/adapter-static")

还有：

* [@sveltejs/adapter-cloudflare](https://kit.svelte.dev/docs/adapter-cloudflare "https://kit.svelte.dev/docs/adapter-cloudflare")：用于 Cloudflare 页面
* [@sveltejs/adapter-cloudflare-workers](https://kit.svelte.dev/docs/adapter-cloudflare-workers "https://kit.svelte.dev/docs/adapter-cloudflare-workers")：用于 Cloudflare Workers
* [@sveltejs/adapter-netlify](https://kit.svelte.dev/docs/adapter-netlify "https://kit.svelte.dev/docs/adapter-netlify")：用于 Netlify
* [@sveltejs/adapter-vercel](https://kit.svelte.dev/docs/adapter-vercel "https://kit.svelte.dev/docs/adapter-vercel")：用于 Vercel

点击对应的链接查看文档说明即可。

## 8\. 恭喜你！

看到这里，恭喜你完成了第二阶段 —— SvelteKit 的使用：

1. 第一阶段：Svelte 5 🎉
2. 第二阶段：SvelteKit 🎉
3. 第三阶段：项目实战
4. 第四阶段：Svelte 原理

此时你应该对 SvelteKit 有哪些功能，可以实现哪些效果有了一个大致的了解。

接下来我们进入实战项目，在实战中将这些知识融会贯通。