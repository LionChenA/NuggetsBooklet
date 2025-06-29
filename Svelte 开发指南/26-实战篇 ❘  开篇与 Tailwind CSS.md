> 推荐学习指数：⭐️️⭐️️⭐️️

## 1\. 前言

实战篇要写一个什么样的项目呢？这个问题困扰了我许久。

从纯前端角度来看，很多大型的应用不过是调用了一堆 API 进行 “增删改查”，固然有一些看起来有些技术难度的项目，多是因为对相关技术并不熟悉、细节众多内容繁杂工程量巨大、或是底层不够完善造成坑很多而已，比如富文本编辑器、低代码编辑器、炫酷的 3D 特效、playground、音视频处理、类似于在线 PS 这样的图片设计工具等等。

如果要写一个完善的这类项目，都够开一个单独的小册了。如果简单一些，则是用一些现成的封装实现一些 Demo，给与读者实现思路，用作入门。当然这是可以的，只不过学到后应用的范围会有些小，限制于特定的技术领域。

而回到大部分的技术项目，又不过是增删改查，如果是写增删改查，写前一两个增删改查可能会有很有收获，但越往后写，项目看起来是在不断完善，但收获可能越来越少，感觉更像是在堆代码……也就是说学习的边际收益在递减。这样一想，写 TodoList 反而是最好的实战项目……因为它简单却又能帮助读者快速掌握项目开发的基础。

不过在实际项目开发中，并不会像练手篇中只使用 Svelte 实现，为了更快速的开发，实际开发中，往往会搭配众多技术选型一起使用。我根据自己的开发经验总结了主流搭配 Svelte 使用的技术选型：

1. 样式：Tailwind
2. 表单验证：Superforms 和 Zod
3. 组件库：Daisy UI、Skeleton UI、Shadcn Svelte、Flowbite Svelte 等等
4. Auth：Clerk、Supabase
5. ORM：Prisma、Drizzle
6. 数据库：都行，MySQL、PostgreSQL、MongoDB 都主流的数据库，使用线上数据库也不错，以前可白嫖 Planetscale 的 MySQL，收费后，大家开始转为 TiDB

实战篇我们会先对这些技术选型进行单独的介绍，最后综合使用这些技术选项实现一个全栈项目。

本篇我们先介绍 Tailwind CSS。SvelteKit 配合 Tailwind CSS 是一个非常常用的技术选型。

## 2\. Tailwind CSS

Tailwind CSS 是一个用于构建 Web 项目的 CSS 框架，它提供了一系列预定义的原子 CSS 类，可以帮助开发人员快速构建 Web 界面，并且可以通过自定义主题和扩展来满足不同的需求。

目前 Tailwind CSS 在 GitHub 有 80k Stars、Npm 周下载量 733W，已经成为前端主流的 CSS 框架。

Tailwind CSS 看起来很简单，就是将 CSS 的简写放到 HTML 类名中，但是 `ring`、`truncate`这些类名是啥意思？响应式该如何处理？暗黑模式如何支持？如何自定义一个类名的样式？使用时有哪些注意事项？常搭配哪些库使用？这些进阶的问题又要花费一些时间去了解。

不过**本篇我们并不会具体介绍 Tailwind CSS 的使用，但我会根据自己的学习经验告诉大家如何系统学习 Tailwind CSS。**

### 2.1. 安装环境

首先入门 Tailwind CSS 是很简单的，你只需要装好环境即可。Next.js 官方脚手架默认创建的时候就会选择是否支持 Tailwind CSS。Svelte 5 正式发布后，创建项目的时候也会让你选择是否使用 Tailiwnd CSS。

SvelteKit 可以参考 Tailwind CSS 提供的 [SvelteKit 使用的教程](https://tailwindcss.com/docs/guides/sveltekit "https://tailwindcss.com/docs/guides/sveltekit")手动添加或者运行 `npx svelte-add`自动添加。Tailwind CSS 官网也提供了各种[安装方式](https://tailwindcss.com/docs/installation "https://tailwindcss.com/docs/installation")。

#### 2.1.1. 手动安装

首先运行：

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

此时会创建 `tailwind.config.js` 和 `postcss.config.js` 文件。

然后修改 `tailwind.config.js`，添加代码如下：

```diff
export default {
+  content: ['./src/**/*.{html,js,svelte,ts}'],
  theme: {
    extend: {}
  },
  plugins: []
};
```

创建 `src/app.css`（如果已经创建了就直接修改代码），添加代码如下：

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

创建 `src/routes/+layout.svelte`（如果已经创建了就直接修改代码），目的是引入 `app.css`，添加代码如下：

```diff
<script>
  import "../app.css";
</script>

<slot />
```

再然后重新运行 `npm run dev`，修改 `src/routes/+page.svelte`测试一下效果：

```xml
<h1 class="text-3xl font-bold underline">
  Hello world!
</h1>

<style lang="postcss">
  :global(html) {
    background-color: theme(colors.gray.100);
  }
</style>
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bc18c89041849d09c98770a594318f7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1358&h=366&s=36222&e=png&b=f3f4f6)

#### 2.1.2. 自动安装

如果你没有在创建项目的时候选择 `tailwindcss`，你也可以运行：

```xml
npx svelte-add
```

选择 Tailwind CSS 就可以自动安装使用：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c27dc82587324f53b2563f8db2bf3f5f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2460&h=1232&s=168822&e=png&b=1e1e1e)

第二个选项是判断是否选择 [typography plugin](https://github.com/tailwindlabs/tailwindcss-typography "https://github.com/tailwindlabs/tailwindcss-typography")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc0404704f6f4f9b94a0047b7b4d912d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1994&h=498&s=65549&e=png&b=1e1e1e)

`typography plugin` 是 Tailwind CSS 的官方插件，可以在官方文档的侧边导航栏底部找到介绍：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/984f1d23ce6044b497e4e0bc0c5b946d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2974&h=956&s=366142&e=png&b=101729)

简单来说，使用默认 Tailwind CSS 的时候，所有的样式都被重置。如果你要设置一篇文章的样式时，所有元素的样式都需要自己重写，这就很麻烦了。Tailwind CSS 直接提供了 typography 插件，为元素添加上漂亮的排版默认值，你只需要在最外层添加一个 `prose`类名，元素就会自动带上合适的样式：

```html
<article class="prose lg:prose-xl">
  <h1>Garlic bread with cheese: What the science tells us</h1>
  <p>
    For years parents have espoused the health benefits of eating garlic bread
    with cheese to their children, with the food earning such an iconic status
    in our culture that kids will often dress up as warm, cheesy loaf for
    Halloween.
  </p>
  <p>
    But a recent study shows that the celebrated appetizer may be linked to a
    series of rabies cases springing up around the country.
  </p>
  <!-- ... -->
</article>
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0db566285ce148a89bc6115237f4e5db~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1970&h=944&s=228509&e=png&b=ffffff)

可以在 [live demo](https://play.tailwindcss.com/uj1vGACRJA?layout=preview "https://play.tailwindcss.com/uj1vGACRJA?layout=preview") 看到更加完整的样式展示。

安装 Tailwind CSS 后，使用很简单，更多考验的是你的 CSS 知识：

```html
<div class="flex justify-center items-center">center</div>
```

### 2.2. 多查多学

如果不知道样式对应的类名，打开 [Tailwind CSS 官网](https://tailwindcss.com/ "https://tailwindcss.com/")，点击搜索框，直接搜索即可：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41f076fd7e5646788ae43d1bc35c951a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2956&h=1440&s=345056&e=png&b=1c2336)

点击链接就会跳转到对应的文档，查看需要的类名即可：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fdc04fa0adbe4eb590c9e6a999d94088~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2918&h=1078&s=641674&e=png&b=11182b)

此时已经能够处理绝大部分的样式场景了，除了这些简写类名，其实还有很多工具类名，比如 [container](https://tailwindcss.com/docs/container "https://tailwindcss.com/docs/container")（设置容器宽度）、[size](https://tailwindcss.com/docs/size "https://tailwindcss.com/docs/size")（设置高度和宽度）、[divide](https://tailwindcss.com/docs/divide-width "https://tailwindcss.com/docs/divide-width")（设置元素之间的 border）、[space](https://tailwindcss.com/docs/space "https://tailwindcss.com/docs/space")（设置元素之间的距离）、[line-clamp](https://tailwindcss.com/docs/line-clamp "https://tailwindcss.com/docs/line-clamp")（设置文字行数，超出的部分显示 `...`）、[truncate](https://tailwindcss.com/docs/text-overflow#truncate "https://tailwindcss.com/docs/text-overflow#truncate")（文字显示一行）、[ring](https://tailwindcss.com/docs/ring-width "https://tailwindcss.com/docs/ring-width")（使用 box-shadow 实现元素轮廓）、[animate](https://tailwindcss.com/docs/animation "https://tailwindcss.com/docs/animation")（一些常用的动画效果）、[sr-only](https://tailwindcss.com/docs/screen-readers#screen-reader-only-elements "https://tailwindcss.com/docs/screen-readers#screen-reader-only-elements")（只显示在屏幕阅读器）等等

这些类名的学习我并没有看到专门的整理，虽然都可以在官方文档里查到，但文档实在是太多了，从头看到尾确实有些累。我的建议是在日常开发中，比如使用 [Tailwind CSS Components](https://tailwindui.com/components "https://tailwindui.com/components") 的时候多加注意用到的样式，遇到不熟悉的就顺便学习一下，从实践中掌握。

### 2.3. 核心概念

Tailwind CSS 还有一些核心的概念，官方文档中进行了详细的介绍：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/daac117cbf434d8cbf6982af8476c625~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2780&h=864&s=373381&e=png&b=11192c)

主要是介绍了元素的各种状态样式如何写，如何写响应式、如何写暗黑模式、如何自定义样式等等，都很重要，日常开发会用到上，内容至少要过一遍。

如果你需要自定义样式，那就需要看这块内容：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/209475f51bab4f06b557c96b8e91348b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2622&h=974&s=412823&e=png&b=11192c)

这块内容基本都是讲如何自定义样式，如果有自定义的需求，就看这里的配置方式。

### 2.4. 多用组件库

实际项目开发中，如果不需要严格控制样式，自由度较高，那最好不要自己从头写样式，而是多借助现有的组件库。Tailwind CSS 提供了 [Tailwind CSS Components](https://tailwindui.com/components "https://tailwindui.com/components")，这种是预设的 UI 组件，只有样式，可直接复制修改，但是免费的只开放了一些，不过网上也有一些类似的免费 UI 组件，比如 [Float UI](https://floatui.com/components/banners "https://floatui.com/components/banners")。

除此之外，还有很多 Tailwind CSS 库，这些有更多的封装，比如 [Shadcn/ui](https://ui.shadcn.com/docs "https://ui.shadcn.com/docs")、[Daisyui](https://daisyui.com/docs/install/ "https://daisyui.com/docs/install/")、[Flowbite](https://flowbite.com/ "https://flowbite.com/") 等等（搜“Tailwind CSS 组件库”，你会发现很多）。其中最火的是 Shadcn UI，它是 2023 年 JavaScript 领域 GitHub Stars 增长最多的开源项目，2023 年 1 月创建的项目，一年便涨了 39.5K Star，足以看出其火爆程度。

### 2.5. 搭配工具库

这篇文章一定要看看：[《Next.js 项目写 Tailwind CSS 基本都会遇到的两个问题》](https://juejin.cn/post/7387611028988002314 "https://juejin.cn/post/7387611028988002314")，内容同样适用于 Svelte。

它介绍了使用 Tailwind CSS 的两大问题，以及解决这些问题会用到的 3 个库：tailwind-merge、clsx、cva，最后补充了一些配合写 Tailwind CSS 的插件和网站。

基本的学习内容就这些，你要是还不放心，这还有一个 4 小时的 Youtube [Tailwind CSS 教程](https://www.youtube.com/watch?v=KFtM4tuCTZs&list=PL8HkCX2C5h0VM7zNTaGgRDEb0Mp18ttqp&index=1 "https://www.youtube.com/watch?v=KFtM4tuCTZs&list=PL8HkCX2C5h0VM7zNTaGgRDEb0Mp18ttqp&index=1")可以看一下。

## 3\. 总结

虽然这篇文章关于 Tailwind CSS 的介绍看似有些少，主要是因为将学习内容放到了链接中，其实展开后还是有很多内容需要学习的。不过 Tailwind CSS 入门还是很容易的，哪怕不系统性学习也可以上手使用。遇到具体的问题时再去查找解决方案也是不错的学习方式。