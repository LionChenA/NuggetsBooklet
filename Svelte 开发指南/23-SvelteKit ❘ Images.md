> 推荐学习指数：⭐️️，了解即可

## 1\. 前言

根据 [Web Almanac](https://almanac.httparchive.org/en/2022/media "https://almanac.httparchive.org/en/2022/media") 中的介绍，图片大小占典型网站页面大小的很大一部分。根据统计，2021 年 6 月网站的总大小中位数是 2019 KB（移动端），其中 881 KB 是图片。这比 HTML（30 KB），CSS（72 KB），JavaScript（461 KB）和字体（97 KB）的总和还要多。

在绝大多数页面上（70% 移动设备，80% 桌面），最有影响的就是图片。Largest Contentful Paint（最大内容绘制，简写：[LCP](https://web.dev/articles/lcp?hl=zh-cn "https://web.dev/articles/lcp?hl=zh-cn")） 是一种 Web 性能指标，可以标识首屏中最大的内容元素。大部分时候，该元素都有图片。

所以图片对应用性能的影响很大，本篇就来讲讲在 Svelte 项目中如何使用图片。

## 2\. Vite 内置处理

SvelteKit 使用 Vite 作为构建工具，而 Vite 内置对[静态资源的处理](https://cn.vitejs.dev/guide/assets "https://cn.vitejs.dev/guide/assets")：

```javascript
import imgUrl from './img.png'
```

因为 Vite 的自动处理，所以导出的并不是图片元素本身，而是图片地址。`imgUrl` 在开发时会是 `/img.png`，在生产构建后会是 `/assets/img.2d8efhg.png`，Vite 会自动为引入的资源生成 hash 文件名，体积较小的资源还会内联为 base64 data URL 等。这些是 Vite 的自带优化，而开发者只需正常使用即可：

```tsx
<script>
  import logo from '$lib/assets/logo.png';
</script>

<img alt="The project logo" src={logo} />
```

## 3\. @sveltejs/enhanced-img

`@sveltejs/enhanced-img` 是基于 Vite 的一个构建插件，提供图片处理功能，可处理 `avif`、`webp`等文件格式，可自动设置图片的 `width`和 `height`防止布局偏移，为不同设备创建多种尺寸的图片，删除 EXIF 数据保护隐私。可用于任何基于 Vite 的项目，包括但不限于 SvelteKit 项目。

不过作为构建插件，`@sveltejs/enhanced-img`只能在构建过程中优化位于本机的文件，其他来源的图片建议用 CDN 进行优化，也可借助 [@unpic/svelte](https://unpic.pics/img/svelte/ "https://unpic.pics/img/svelte/") 这样的图片优化库。

### 3.1. 基础使用

安装：

```bash
npm install --save-dev @sveltejs/enhanced-img
```

修改 `vite.config.js`，代码如下：

```diff
import { sveltekit } from '@sveltejs/kit/vite';
+import { enhancedImages } from '@sveltejs/enhanced-img';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [
+   enhancedImages(),
    sveltekit()
  ]
});
```

> 注意：由于图片转换的成本，第一次构建的时候会有些慢，后面因为有缓存，速度会快很多

现在我们就可以在 `.svelte`中使用：

```html
<enhanced:img src="./path/to/your/image.jpg" alt="An alt text" />
```

让我们写个 Demo 看下具体输出的结果：

```bash
routes
└─ img
   ├─ +page.svelte
   └─ wukong.webp
```

`src/routes/img/+page.svelte`的代码如下：

```bash
<enhanced:img src="./wukong.webp" alt="wukong" class="wukong" />
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eda9b2be1ea14eb08bd70b5bbdb18334~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2038&h=928&s=728753&e=png&b=2b2b2b)

在构建时，`<enhanced:img>`将被替换为 `<picture>`，并且设置了多种图片类型和尺寸。开发的时候只用提供最高分辨率的图片，插件会自动为各种设备类型生成较小的版本。

### 3.2. 动态导入多张

开发者可以手动导入图片然后传递给 `<enhanced:img>`：

```bash
<script>
  import WuKong from './wukong.webp?enhanced';
</script>

<enhanced:img src={WuKong} alt="wukong" />
```

注意使用这种方式的时候，地址要加个 `?enhanced`，否则会导入错误。

效果跟之前基本一致：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/234977edc1c14a079a96231c6c5b48f1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2002&h=900&s=721387&e=png&b=2b2b2b)

当然这些写并没有什么明显的好处，但借助 [Vite's import.meta.glob](https://cn.vitejs.dev/guide/features#glob-import "https://cn.vitejs.dev/guide/features#glob-import") 可以导入多张图片：

```xml
<script>
  const imageModules = import.meta.glob('./*.{avif,gif,heif,jpeg,jpg,png,tiff,webp,svg}', {
    eager: true,
    query: {
      enhanced: true
    }
  });
</script>

{#each Object.entries(imageModules) as [_path, module]}
  <enhanced:img src={module.default} alt="wukong img" class="mb-2" />
{/each}
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11ae7b0d22fc4732b5a1cfe8b304f2ae~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2034&h=1152&s=1481311&e=png&b=2a2a2a)

> 注意：width 和 height 属性并不需要添加，Vite 会自动从源图片中计算得出，以防止布局偏移。但这并不影响开发者设置图片样式：

```xml
<script>
  import WuKong from './wukong.webp?enhanced';
</script>

<enhanced:img src={WuKong} alt="wukong" class="wukong" />

<style>
  .wukong {
    width: 200px;
    height: auto;
  }
</style>
```

### 3.3. 图片转换

开发者也可以给图片添加一些转换比如模糊、旋转、调整明亮度等操作，用法也很简单：

```xml
<enhanced:img src="./wukong.webp?blur=15" alt="wukong" class="wukong" />
```

只需要在图片地址后添加参数即可。浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7614dbd8289b4ad68db862b28ca02ab5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2032&h=934&s=545674&e=png&b=2c2c2c)

这个转化是对图片本身的转换，不是通过添加 CSS 实现。完整指令参考 [GitHub imagetools](https://github.com/JonasKruckenberg/imagetools/blob/main/docs/directives.md "https://github.com/JonasKruckenberg/imagetools/blob/main/docs/directives.md")。

### 3.4. srcset and sizes

我们先来讲讲 `srcset` 和 `sizes` 本身的用法。

HTML 5.1 新增加了 `<img>` 元素的 `srcset`、`sizes` 属性，用于设置响应式图片。

当我们需要不同的设备展示不同的图片的时候，就需要用到 `srcset`。这里具体又分为两种情况，一种是图片是相同的尺寸，但是不同的分辨率对应不同的图片，即高分辨率下对应高倍图。一种是相同的图片内容，但依据设备显示的更大或者更小。这分别对应着 `srcset` 的两种语法。

先说第一种，根据分辨率不同展示不同的图片，使用示例如下：

```javascript
<img
  srcset="elva-fairy-320w.jpg, elva-fairy-480w.jpg 1.5x, elva-fairy-640w.jpg 2x"
  src="elva-fairy-640w.jpg"
  alt="Elva dressed as a fairy"
/>
```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a316177c7f664b9ab520736e62d054d2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.png#?w=480&h=425&s=130728&e=png&a=1&b=f4f1f0)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/377c7051a0054b2991821b23049d491c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.png#?w=480&h=425&s=115279&e=png&a=1&b=f4f1f0)

srcset 是由逗号分隔的一个或多个字符串组成，每段字符串由以下组成：

1. 指向图片的 URL
2. 一个空格（可选）
3. 一个像素密度描述符（一个正浮点数，后面紧跟 x 符号）

如果我们给图片应用一个 CSS 样式：

```css
img {
  width: 320px;
}
```

它的宽度在屏幕上就是 320 像素（CSS 像素）。浏览器会计算出正在显示的显示器的分辨率，然后显示 srcset 引用的最适合的图片。如果是普通的分辨率，一个设备像素表示一个 CSS 像素，那就会加载 `elva-fairy-320w.jpg`，它的大小是 39KB，如果设备是高像素，用两个或者更多的设备像素表示一个 CSS 像素，那就会加载 `elva-fairy-640w.jpg`，它的大小是 93KB。

再说第二种情况，相同的图片内容，但依据设备显示的更大或者更小。使用示例代码如下：

```javascript
<img
  srcset="elva-fairy-small.jpg 480w, elva-fairy-large.jpg 800w"
  src="elva-fairy-large.jpg"
  alt="Elva dressed as a fairy"
/>
```

srcset 的语法与刚才略有不同，它定义了浏览器可选择的图片设置以及每个图片的大小。分析它的语法，依然是由逗号分隔的一个或多个字符串组成，每段字符串由以下部分组成：

1. 指向图片的 URL
2. 一个空格
3. 图片的固有宽度（以像素为单位）。注意，这里使用宽度描述符 w，而非 px。但一个 w 对应 1 个像素。图片的固有宽度是指它的真实大小。

此时我们就告诉了浏览器，这张图片有两种备选图片可以显示，一张是 `elva-fairy-small.jpg`，这张图片的宽度是 480px，一张是 `elva-fairy-large.jpg`，它的宽度是 800px。

那浏览器怎么知道用哪张图片呢？比如当前设备视口宽度是 640px，是选择 480px 的图片还是选择 800px 的图片呢？

为了帮助浏览器判断，你就需要写 sizes 属性。sizes 属性就是一组媒体查询条件，告诉浏览器，什么样的条件使用什么样的图片。一个使用示例如下：

```javascript
<img
  srcset="elva-fairy-small.jpg 480w, elva-fairy-large.jpg 800w"
  sizes="(max-width: 600px) 480px,
         800px"
  src="elva-fairy-large.jpg"
  alt="Elva dressed as a fairy"
/>
```

sizes 也是由逗号分隔的一个或多个字符串组成，每段字符串由以下组成：

1. 一个媒体条件，例子中为 `(max-width:600px)`，它表示当视口的宽度小于等于 600px 时
2. 一个空格
3. 当媒体条件为真时，图片将填充的宽度，这个宽度可以是固定值，比如这个例子中的 480px，也可以是一个相对于视口的宽度（如 50vw），但不能是百分比。如果没有媒体条件，表示是默认生效。当浏览器成功匹配第一个媒体条件的时候，剩下所有的条件都会被忽略。所以顺序很重要。

有了这些属性后，浏览器会：

1. 检查设备宽度
2. 检查 sizes 列表中哪个媒体条件是第一个为真
3. 查看给予该媒体查询的槽大小
4. 加载 srcset 列表中引用的最接近所选的槽大小的图片

比如在这个例子中，如果浏览器的视口是 480px，那么 sizes 中的第一个条件 (max-width: 600px) 就为真，所以选择 480px 大小，因为 480px 与固有宽度 480w 最接近，所以加载 elva-fairy-small.jpg。通过这种方式就可以实现移动端加载小图片，从而加快移动端的加载速度。

了解了 `srcset` 和 `sizes` 属性，让我们回到 `<enhanced:img>`的使用上。

如果使用了比较大的图片，则应指定 `sizes`以便在小设备上请求小图片。比如我们加载一个高清的图片：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0581c63dae1342cea7d9b60e66206329~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2726&h=1374&s=4706053&e=png&b=212829)

此图片的大小为 2160 \* 1080 像素，大小为 423 KB。

修改 `src/routes/img/+page.svelte`，代码如下：

```xml
<enhanced:img src="./wukong.jpg" alt="wukong" sizes="min(2160px, 100vw)" class="wukong" />
```

在指定 sizes 后，`<enhanced:img>` 将为较小的设备生成小图片并填充 `srcset` 属性。浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0bf138dc0964ba6b39f43a8a080ce61~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3026&h=900&s=1112563&e=png&b=2a2a2a)

你会发现，不同尺寸从 `540`、`768`、`1080`、`1366`、`1536`、`1920`、`2160` 都生成了一张图片。那么浏览器是如何判断使用哪张尺寸的图片呢？

我们举个例子，大家就知道了。

首先是 `sizes="min(2160px, 100vw)"`，这个语法看起来有些奇怪，其实意思是在 2160px 下，以视口宽度为准。比如视口宽度是 `1080px`，`sizes=min(2160px, 100vw)` 的结果是 `sizes=1080px`，然后就以 `1080px` 为结果到 `srcset` 中查找合适的图片尺寸。

现在让我们把设备切到 iphone 12 Pro：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc2aa22044af4a218b6f2b318a343ba9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3126&h=898&s=803674&e=png&b=2f2f2f)

设备是 390 x 844，DPR 为 3，所以视口宽度是 390 x 3 = 1170。以 `1170` 为结果去 srcset 中查找适合的尺寸，最为接近的是 `1080w`，所以用的图是对应 1080w 的那张图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94539545095445a6a02c390b72e22ee4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3112&h=942&s=740129&e=png&b=303030)

此时图片大小为 26.5k，比原图 423k 要小很多。

其他设备尺寸也是同理。

自动生成的最小图片宽度为 540px。如果想要更小的图片或希望自定义宽度，可以使用 `w` 查询参数来实现：

```xml
<enhanced:img
  src="./wukong.jpg?w=1280;640;400"
  alt="wukong"
  sizes="(min-width:1920px) 1280px, (min-width:1080px) 640px, (min-width:768px) 400px"
  class="wukong"
/>
```

此时浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e955370a92d43e0a72d809b927a30b9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2284&h=948&s=703068&e=png&b=2d2d2d)

此时会根据你指定的宽度来生成对应的尺寸。

## 4\. 使用图片的最佳实践

最后让我们总结一下使用图片的最佳实践：

* 选择合适的图片处理方案，比如 Vite 内置处理方案可内联图片，`@sveltejs/enhanced-img`可用于处理大多数本机图片
* 使用 CDN
* 原始图片应该有良好的品质和分辨率，并且应该有 2x 宽度用于 HiDPI 设备
* 对于远大于移动设备宽度（约 400 像素）的图片（例如采用页面设计宽度的主图片），请指定 `sizes` 以便在较小的设备上提供较小的图片
* 对于重要图片，例如最大内容绘制 （LCP） 图片，请设置为 `fetchpriority="high" loading="eager"` 尽早优先加载
* 为图片提供一个容器或样式，使其受到约束，避免影响累积布局偏移 （CLS）。这也是为什么`@sveltejs/enhanced-img` 会自动添加 `width`和 `height`属性
* 始终提供好的 `alt` 文本。如果不这样做，Svelte 编译器将发出警告
* 不要在 `sizes` 中使用 `em` 或 `rem`，并且更改这些值的默认大小