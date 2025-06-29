> 推荐学习指数：⭐️⭐⭐️，必学内容️

## 1\. 前言

类似于 Next.js 有自己的官方脚手架 create-next-app，Svelte 也有自己的官方脚手架名为 [SvelteKit](https://kit.svelte.dev/ "https://kit.svelte.dev/")，提供了各种开箱即用的功能，比如路由、服务端渲染、数据获取、集成 TypeScript、预渲染、单页应用、生产优化、实时刷新等等。根据一项投票统计：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c656993c6a2e47e8b50685a2e959fe8c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2388&h=1314&s=108132&e=webp&b=16202c)

可以看出，SvelteKit 是非常受欢迎的脚手架。本篇开始就为大家详细介绍 SvelteKit 中的功能。

## 2\. 项目初始化

运行：

```bash
npx sv create your-svelte-app-name
```

首先会进入 app template 选择界面：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad59dacbfe9f4b31aafc14a455cd4af2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1668&h=354&s=64542&e=png&b=1e1e1e) 其中：

1. **SvelteKit minimal**：一个空项目，适合于实际开发中使用
2. **SvelteKit demo**：一个包含 Demo 的项目，适合于初学 Svelte 的同学
3. **Svelte library**：如果你要开发 Svelte 组件库等项目，选择此模板

初学时可选择 **SvelteKit demo**，等熟练后使用 **SvelteKit minimal**。

然后是类型检查选择界面：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc6bce04f2c04b149c8d7c01c000d7ba~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2862&h=270&s=56486&e=png&b=1e1e1e)

看个人喜好，是选择 TypeScript 还是 JSDoc 还是不使用类型检查。

再然后是技术选型选择：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ed09a2edc4843559998670ecc7570ad~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2202&h=590&s=89797&e=png&b=1e1e1e)

其中：

1. **prettier**：用于代码格式化
2. **esLint**：用于代码校验
3. **vitest**：用于单元测试
4. **playwright**：用于浏览器测试
5. **tailwindcss**：Tailwind CSS
6. **drizzle**：ORM
7. **lucia**：用户管理和会话验证
8. **mdsvex**：Svelte 组件的 Markdown 预处理器
9. **paraglide**：i18n 处理
10. **storybook**： 用于 UI 开发、组件测试和编写组件文档

按下空格键表示选择，再按一次表示取消选择。按下字母键 `a` 表示全选，再按一次取消全选。

建议至少选择 prettier、esLint、tailwindcss（如果选择了 tailwindcss，还会提示你使用哪些插件，按使用情况选择即可）

最后是包管理器选择：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95a3387366d74293be750a38fbcf9ae0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1768&h=328&s=43874&e=png&b=1e1e1e)

项目创建成功！SvelteKit 会提示接下来需要的操作：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ef1ea630faf43a98e4e383ab14b8ae4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1948&h=532&s=85786&e=png&b=1e1e1e)

按照这些命令依此执行即可。

如果安装的时候遇到这种报错：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/938840662b454ff1bc00b09d47ed289e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2708&h=352&s=155131&e=png&b=1d1d1d)

这是因为 Node 版本不符合要求，使用 nvm 切换到合适的版本即可：

```bash
# 使用最新的 LTS 版本，目前会切换到 node v20.11.0 (npm v10.2.4)，符合安装要求
nvm use --lts
```

浏览器效果如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/977c670781574baf87e7848e15774e5b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2728&h=1356&s=1348496&e=png&b=e9ecf2)

## 3\. 编辑器插件安装

因为 VSCode 默认无法识别 `.svelte`语法，所以如果使用 VSCode，一定要安装官方插件 [Svelte for VS Code](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode "https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode")，提供语法高亮、智能补全等功能。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3c4a9e55c654d219fbc2abd03562790~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2330&h=504&s=160406&e=png&b=1f1f1f)

## 4\. 文件目录

生成的文件目录如下：

```md
svelte-app
├─ src
│  ├─ lib
│  │  └─ [your lib files]
│  ├─ routes
│  │  └─ [your routes]
│  ├─ app.css
│  └─ app.html
├─ static
│  └─ [your static assets]
├─ README.md
├─ eslint.config.js
├─ package.json
├─ svelte.config.js
├─ tsconfig.json
└─ vite.config.ts
```

其中：

1. `src/lib`：存放一些工具库
2. `src/routes`：路由文件
3. `src/app.css`：全局样式
4. `src/app.html`：HTML 模板
5. `static`：静态目录
6. `eslint.config.js`：ESLint 配置文件
7. `svelte.config.js`：Svelte 配置文件
8. `tsconfig.json`：TypeScript 配置文件
9. `vite.config.ts`：Vite 配置文件

查看目录，你还会发现一个隐藏的 `.svelte-kit`文件，类似于 Next.js 的 `.next`文件，由 Svelte 自动生成，不要去修改其中的代码。有的时候，感觉代码有问题，可以删除此文件重新运行试试。

## 5\. 相关命令

使用该脚手架的命令可以查看 `package.json`：

```json
{
  "scripts": {
    "dev": "vite dev",
    "build": "vite build",
    "preview": "vite preview",
    "check": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json",
    "check:watch": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json --watch",
    "lint": "prettier --check . && eslint .",
    "format": "prettier --write ."
  }
}

```

* `npm run dev`：开发模式
* `npm run build`：构建生产代码
* `npm run preview`：预览生产代码
* `npm run check`：检查代码，使用 Svelte 自己的 [svelte-check](https://www.npmjs.com/package/svelte-check "https://www.npmjs.com/package/svelte-check")，检查 JavaScript、TypeScript 编译错误、A11y 提示、未使用的 CSS 等
* `npm run lint`：检查代码，使用 Prettier 和 ESLint
* `npm run format`：格式化代码，修复代码风格等

值得一提的是 `npm run check`，有的时候你会发现 VSCode 提示了一些莫名其妙的错误，比如导入 `$lib`下的 Svelte 组件，但提示没有找到对应的类型定义文件，虽然代码可以正常运行……

不要怀疑自己，我也不知道谁的错，但反正不是我的错！错误的不是我，而是这个世界！

如果是在普通的 TypeScript 中，我们可以通过 `Command + Shift + P`唤起命令框：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c695bfce59a48e9848653235a843bf9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2078&h=362&s=76784&e=png&b=1f3a58)

然后重启 TS 服务器。但在 Svelte 组件中，这有的时候并不管用。

所以你需要先通过运行 `npm run check`，检查是否真的有错误。

如果没有这个错误，但是重启 TS Server 或者重启 ESLint Server 都没用的话，试试重启整个编辑器。 `Command + Shift + P`唤起命令框：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/209c05de4f8a47fe9b2fe83adfd2d541~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2058&h=292&s=58214&e=png&b=1f3a58)

选择重新加载窗口。

算是开发 Svelte 项目时踩过的一个小小坑吧。