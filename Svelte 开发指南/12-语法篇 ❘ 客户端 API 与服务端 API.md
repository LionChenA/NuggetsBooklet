> 本篇推荐学习指数：⭐️️

## 1\. 前言

通常 Svelte 项目我们会使用官方脚手架 SvelteKit 进行开发，所以不会用到这些内容，本篇简单看一下了解即可。

## 2\. 客户端 API

当我们导入一个 `.svelte` 模块的时候，导入的其实是一个组件类：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fe08f5d3b5b42eeb636cee76fdc1f3a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2162&h=924&s=206434&e=png&b=1c1e21)

我们该如何使用这个组件类呢？这就是 Svelte 客户端 API 要介绍的内容。

如果我们使用的是 SvelteKit，基本不需要使用客户端 API。但如果是借助其他构建工具从头构建 Svelte 项目脚手架，则有可能会用到，比如使用 vite 创建一个 Svelte 的项目：

```html
npm create vite@latest
```

我们选择 Svelte 框架和 JavaScript 语法：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20fe55a3e1024ca38c9d62e6307395ff~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1704&h=220&s=54090&e=png&b=1e1e1e)

> 注：Svelte 项目并不建议使用 Vite 创建，因为这个模板的很多东西比如路由都需要自己从头实现，最好还是使用 SvelteKit，这里是为了方便客户端 API 的演示

创建的项目目录结构如下（做了部分精简）：

```xml
vite-svelte
├─ src
│  ├─ lib
│  │  └─ Counter.svelte
│  ├─ App.svelte
│  ├─ app.css
│  └─ main.js
├─ index.html
├─ jsconfig.json
├─ package.json
├─ svelte.config.js
└─ vite.config.js
```

这个项目是如何运行起来的呢？

首先 `vite.config.js` 引入了 `svelte` 插件，负责编译 `.svelte` 模块：

```javascript
import { defineConfig } from "vite";
import { svelte } from "@sveltejs/vite-plugin-svelte";

export default defineConfig({
  plugins: [svelte()],
});
```

然后查看`main.js` 的代码：

```javascript
import { mount } from 'svelte'
import './app.css'
import App from './App.svelte'

const app = mount(App, {
  target: document.getElementById('app'),
})

export default app
```

在 `main.js` 中引入主要的 `App.svelte` 模块，调用 Svelte 提供的 `mount` 函数，通过 `target` 属性指定挂载的位置，然后导出该组件实例。

在 `index.html`中引入打包后的 `main.js`：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + Svelte</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

简而言之：vite 负责打包编译 svelte 模块，main.js 引入 svelte 模块并进行实例化，index.html 引入 main.js

当然我们并不是要讲解如何从头构建一个 Svelte 项目，而是借这个例子说明 **Svelte 客户端 API 要解决的问题是如何在正常的 JavaScript 文件中使用 Svelte 模块**。

### 2.1. mount：创建组件

如果创建并挂载一个组件呢？在 Svelte 4 中 ，是通过 new 的方式

```javascript
import App from './App.svelte'; 
const app = new App({ target: document.body });
```

在 Svelte 5 中，统一改成了函数的形式：

```javascript
import { mount } from 'svelte'; 
import App from './App.svelte'; 
const app = mount(App, { target: document.querySelector('#app'), props: { some: 'property' } });
```

除了 target，还有以下选项：

|选项|默认|描述|
|:---:|:---:|---|
|`target`|`none`|必须，组件要挂载的位置|
|`anchor`|`null`|该元素必须是 target 的子级，组件会挂载在该元素之前|
|`props`|`{}`|传递给组件的属性|
|`context`|`new Map()`|传递给组件的上下文|
|`intro`|`false`|当为 true 的时候，会在组件初始化的时候播放过渡动画|

与 React 不同，当进行挂载的时候，target 的现有子级会保留在原处。

为了体会这些选项的作用，我们可以写一个简单的例子。

修改 `index.html`，代码如下：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + Svelte</title>
  </head>
  <body>
    <div id="app">
      <div id="header">Header</div>
      <div id="footer">
        <button id="destroy">destroy</button>
      </div>
    </div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

修改 `src/main.js`，代码如下：

```javascript
import { mount } from "svelte";
import "./app.css";
import App from "./App.svelte";

const app = mount(App, {
  target: document.getElementById("app"),
  anchor: document.getElementById("footer"),
  props: {
    counter: 10,
  },
  context: new Map([["theme", "dark"]]),
  intro: true,
});

export default app;
```

修改 `src/App.svelte`，代码如下：

```xml
<script>
  import { getContext } from "svelte";
  import { fade } from "svelte/transition";
  export let counter;

  const theme = getContext("theme");
</script>

当前主题：{theme}

<div transition:fade|global={{ delay: 250, duration: 1000 }}>
  <button
    on:click={() => {
      counter += 1;
    }}>+</button
  >
  {counter}
  <button
    on:click={() => {
      counter -= 1;
    }}>-</button
  >
</div>
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07834e5f85d144dfbc13277ec532b1f8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=895&h=396&s=67898&e=gif&f=28&b=232323)

这个例子演示了除 hydrate 之外所有属性的用法。因为设置了 anchor，所以组件挂载在 Footer 之前。因为设置了 context，所以能在组件中通过 getContext 获取主题的值。因为设置了 props，所以组件能够获取该值并设置 计数器的初始值。因为设置了 intro，所以组件初始化的时候也有过渡动画。

### 2.2. unmount：卸载组件

从 DOM 中删除组件，并触发 onDestroy 监听器。

修改 `main.js`，添加如下代码：

```javascript
import { mount, unmount } from "svelte";

document.getElementById("destroy").addEventListener("click", () => {
  unmount(app);
});
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/833f1aa47c3942f5ade1841636f9fedb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=895&h=396&s=111198&e=gif&f=35&b=262626)

## 3\. 服务端 API

Svelte 服务端 API 也有 2 个：render 和 hydrate

### 3.1. render：渲染组件

render 的用法如下：

```html
import { render } from 'svelte/server'; import App from './App.svelte'; const result = render(App, { props: { some: 'property' } }); result.body; // HTML for somewhere in this
<body>
  tag result.head; // HTML for somewhere in this
  <head>
    tag
  </head>
</body>
```

服务端组件的主要工作就是渲染对应的 HTML 和 CSS。它会返回一个带有 head、html 和 css 属性的对象。

比如 Svelte 模块的代码为：

```xml
<script>
  export let counter = 10;
</script>

<svelte:head>
  <title>Hello Counter!</title>
  <meta name="description" content="This is a Counter Example" />
</svelte:head>

<div>
  <button on:click={()=> { counter += 1}}>+</button>
  {counter}
  <button on:click={()=> { counter -= 1}}>-</button>
</div>

<style>
  button {
    color: red
  }
</style>
```

它会返回这样一个对象：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3fc1278d230494bb8520101be47983e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3122&h=492&s=166955&e=png&b=1e1e1e)

render 方法也支持参数，目前有 2 个，一个是 props 一个是 options。options 中目前只有 context。示例用法如下：

```xml
const { head, html, css } = App.render(
  // props
  { answer: 42 },
  // options
  {
    context: new Map([['context-key', 'context-value']])
  }
);
```

### 3.2. hydrate：水合组件

hydrate 的用法如下：

```html
import { hydrate } from 'svelte'; 
import App from './App.svelte'; 
const app = hydrate(App, { target: document.querySelector('#app'), props: { some: 'property' } });
```

所谓水合就是为 DOM 元素添加事件的过程。hydrate 通常搭配服务端渲染一起使用，Svelte 会对现有 DOM 进行水合，而非重新创建 DOM 元素。

为了直观的感受服务端 API 的作用。我们再用 Vite 创建一个 SSR Svelte 项目。运行：

```bash
npm create vite@latest
```

但是这次不直接选择 `Svelte`，而是选择 `Others`，然后再选择 `create-vite-extra`，最后选择 `ssr-svelte` 创建项目。

创建后的项目目录如下（做了部分精简）：

```xml
vite-svelte-ssr
├─ src
│  ├─ lib
│  │  └─ Counter.svelte
│  ├─ App.svelte
│  ├─ entry-client.js
│  ├─ entry-server.js
├─ index.html
├─ server.js
├─ svelte.config.js
└─ vite.config.js
```

那这个项目又是如何运行起来的呢？

为了方便理解，首先修改 `src/App.svelte`的代码为：

```xml
<script>
  import { getContext } from 'svelte';
  export let counter;
  const theme = getContext('theme');
</script>

<svelte:head>
  <title>Hello Counter!</title>
  <meta name="description" content="This is a Counter Example" />
</svelte:head>

当前主题：{theme}

<div>
  <button on:click={()=> { counter += 1}}>+</button>
  {counter}
  <button on:click={()=> { counter -= 1}}>-</button>
</div>

<style>
  button {
    color: red
  }
</style>
```

然后查看 `src/entry-server.js` 的代码并修改为：

```xml
import { render as _render } from "svelte/server";
import App from "./App.svelte";

export function render() {
  return _render(App, {
    props: { counter: 10 },
    context: new Map([["theme", "dark"]]),
  });
}
```

再然后查看 `server.js`的代码，为了方便理解，代码做了精简：

```xml
app.use("*", async (req, res) => {

  let template = await fs.readFile('./dist/client/index.html', 'utf-8');
  let render = (await import("./dist/server/entry-server.js")).render;

  const rendered = await render();

  const html = template
    .replace(`<!--app-head-->`, rendered.head ?? "")
    .replace(`<!--app-html-->`, rendered.html ?? "");

  res.status(200).set({ "Content-Type": "text/html" }).send(html);
});
```

在这段代码中，我们导入了`entry-server.js`导出的 render 函数，`await render()`返回一个具有 html、head、css 属性的对象，然后通过 `template.replace` 简单粗暴的得到最终渲染的 HTML 内容。

我们可以打印看下 `rendered` 变量的内容：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7509be197b8484fb49b0972afe04d6b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3106&h=592&s=179792&e=png&b=1e1e1e)

再然后查看 `src/entry-client.js`的代码并修改为：

```xml
import "./app.css";
import { hydrate } from "svelte";
import App from "./App.svelte";

hydrate(App, {
  target: document.querySelector("#app"),
  props: {
    counter: 10,
  },
  context: new Map([["theme", "dark"]]),
});
```

注意在这里我们的 hydrate 要设置为 true，这是因为之前我们已经通过服务端渲染出了最终的 DOM，无须再重新创建 DOM，直接进行水合即可。

最后查看 `vite-svelte-ssr/index.html`的代码：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + Svelte</title>
    <!--app-head-->
  </head>
  <body>
    <div id="app"><!--app-html--></div>
    <script type="module" src="/src/entry-client.js"></script>
  </body>
</html>
```

这只是最一开始的 HTML 模板，server.js 会将组件进行服务端渲染，并将该 HTML 替换为最终的 HTML。然后访问该 HTML 的时候，因为引入了 `entry-client.js`，在 `entry-client.js`中，我们对 DOM 进行了水合，因此 Svelte 模块正常运行。

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75262516a0af4047ad6aa48b0d466b14~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=851&h=400&s=33905&e=gif&f=29&b=242424)

## 4\. 最后

看到这里，那么恭喜你完成了第一阶段 —— Svelte 5 的语法学习 🎉 接下来我们进入 Svelte 官方脚手架 SvelteKit 的学习。