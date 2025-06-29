> 推荐学习指数：⭐️️，了解即可

## 1\. 前言

本篇我们讲解 SveteKit 如何配置路径别名和环境变量，以及如何保证文件只运行在服务端。

## 2\. 路径别名

SvelteKit 内置了 `$lib`路径别名，指向 `src/lib`目录，这样当你导入 `src/lib`下的文件时，不需要 `../../../`这样找到相对地址，而是可以从 `$lib/...`导入。

除了 `$lib`，还有一个 `$lib/server`，它指向 `src/lib/server`文件夹，之所以特殊，是因为这个目录下的所有文件，SveteKit 会阻止导入到客户端模块中。

如果你想要配置自己的路径别名，可以在 `svelte.config.js`中配置：

```javascript
import adapter from '@sveltejs/adapter-auto';

const config = {
    kit: {
        adapter: adapter(),
        alias: {
            $components: 'src/components'
        }
    }
};

export default config;
```

此时新建 `src/components/Button.svelte`，代码如下：

```html
<button>Click</button>
```

`src/routes/+page.svelte` 就可以使用路径别名导入：

```html
<script>
  import Button from "$components/Button.svelte";
</script>

<button />
```

## 3\. 环境变量

通常大部分应用都会用到环境变量。环境变量可以让你：

1. 根据不同的环境使用不同的值
2. 防止敏感信息暴露到浏览器中

环境变量可以添加到 `.env`文件（项目根目录），还可以使用 `.env.local`或 `.env.[mode]`，表示不同的模式，具体参考 [Vite 环境变量文档](https://cn.vitejs.dev/guide/env-and-mode "https://cn.vitejs.dev/guide/env-and-mode")。

写入的环境变量可以从 `$env/static/private`、`$env/dynamic/private`、`$env/static/public`、`$env/static/public`导入。简单来说，分为 2 类：

1. static 和 dynamic：static 表示值在构建的时候是已知的，可以静态替换，做些优化，但更新需要重新构建部署。dynamic 表示值是动态的，引用的时候才知道具体的值
2. public 和 private：private 表示只能用于服务端，public 表示服务端客户端都可用

我们接下来一一讲解。

### 3.1. $env/static/private

新建 `.env`，代码如下：

```bash
DB_PASSWORD = "edr83k1eou0"
```

此时可以在 `+page.server.js`中通过 `$env/static/private`获取。

新建 `src/routes/+page.server.js`，代码如下：

```javascript
import { DB_PASSWORD } from '$env/static/private';

export function load() {
    console.log(DB_PASSWORD);
    return {
        data: []
    };
}
```

打开 [http://localhost:5173/](http://localhost:5173/ "http://localhost:5173/")，此时会看到命令行中打印了环境变量的值：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9234466e7db420eb27c6ac7a23173e8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1394&h=192&s=30350&e=png&b=1f1f1f)

但如果从 `+page.svelte`中导入，新建 `src/routes/+page.svelte`，代码如下：

```html
<script>
  import { DB_PASSWORD } from "$env/static/private";
</script>

<h1>Hello World!</h1>
```

此时命令行会出现报错：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e610cdd3922451abd8f4fa78a373c2e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2424&h=312&s=129888&e=png&b=1f1f1f)

所以 `$env/static/private`只能导入到服务端模块：

1. `+page.server.js`
2. `+layout.server.js`
3. `+server.js`
4. 以 `.server.js`结尾的模块
5. `src/lib/server`下的模块

导入到客户端代码则会出现报错，这就防止了敏感数据意外发送到浏览器中。

### 3.2. $env/dynamic/private

修改 `src/routes/+page.server.js`，代码如下：

```javascript
import { env } from '$env/dynamic/private';

export function load() {
    console.log(env.DB_PASSWORD);
    return {
        data: []
    };
}
```

使用动态环境变量，需要先导入 `env`对象，然后从 `env` 对象获取具体的环境变量。

### 3.3. $env/static/public

如果环境变量要暴露给浏览器，环境变量需要携带 `PUBLIC_`前缀和私有环境变量做区分。

修改 `svelte-app/.env`，代码如下：

```bash
DB_PASSWORD = "edr83k1eou0"

PUBLIC_THEME_FOREGROUND="blue"
```

修改 `svelte-app/src/routes/+page.svelte`，代码如下：

```xml
<script>
    import { PUBLIC_THEME_FOREGROUND } from '$env/static/public';
</script>

<h1 style="color: {PUBLIC_THEME_FOREGROUND} ">Hello World!</h1>
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85d0d1a31bb84c459754dad61818d589~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1298&h=272&s=25236&e=png&b=ffffff)

### 3.4. $env/dynamic/public

修改 `svelte-app/src/routes/+page.svelte`，代码如下：

```xml
<script>
  import { env } from "$env/dynamic/public";
</script>

<h1 style="color: {env.PUBLIC_THEME_FOREGROUND}">Hello World!</h1>
```

浏览器效果同上。

## 4\. Server-only

如何保证自己的模块只在服务端运行呢？

有两种方式：

1. 文件名添加 `.server`，比如 `secrets.server.js`
2. 将文件放到 `$lib/server`中，比如 `$lib/server/secrets.js`

此时无论是直接还是间接导入这些模块，都会导致错误。官方文档里举的例子：

```javascript
// $lib/server/secrets.js
export const atlantisCoordinates = [/* redacted */];

// src/routes/utils.js
export { atlantisCoordinates } from '$lib/server/secrets.js';

export const add = (a, b) => a + b;

// src/routes/+page.svelte
<script>
    import { add } from './utils.js';
</script>
```

虽然 `+page.svelte`只使用了 `add` 而没有使用 `atlantisCoordinates`，但敏感信息依然可能会以 JavaScript 的形式出现，所以也被认为是不安全的，此时会有错误提示：

```html
Cannot import $lib/server/secrets.js into public-facing code:
- src/routes/+page.svelte
	- src/routes/utils.js
		- $lib/server/secrets.js
```