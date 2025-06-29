> 推荐学习指数：⭐⭐⭐

## 1\. 前言

本篇我们介绍 Svelte 5 新增的 Snippets 功能，它可以替代 Svelte 4 中原本的插槽功能。

此外，我们还会介绍 Svelte 中的 Special elements。为什么叫 Specail elements 呢？因为它们的写法类似于普通的 elements，但实现的功能却比较特殊，比如可以修改 HTML `<head>` 的 `<svelte:head>`、为 HTML `<body>` 添加事件的`<svelte:head>` 等等。

这些也是 Svlete 项目开发中常用的内容。就让我们一起看看如何使用吧！

## 2\. Snippets

Snippets 用于在组件内部创建可复用的代码块。

### 2.1. 使用方法

假设原本的代码是：

```xml
{#each images as image}
  {#if image.href}
    <a href={image.href}>
      <figure>
        <img
          src={image.src}
          alt={image.caption}
          width={image.width}
          height={image.height}
        />
        <figcaption>{image.caption}</figcaption>
      </figure>
    </a>
  {:else}
    <figure>
      <img
        src={image.src}
        alt={image.caption}
        width={image.width}
        height={image.height}
      />
      <figcaption>{image.caption}</figcaption>
    </figure>
  {/if}
{/each}
```

先通过 `#snippet` 声明可复用的代码片段：

```xml
{#snippet figure({ src, caption, width, height })}
  <figure>
    <img alt={caption} {src} {width} {height} />
    <figcaption>{caption}</figcaption>
  </figure>
{/snippet}
```

再通过 `@render` 标签指定渲染的位置和依赖值：

```xml
{#each images as image}
  {#if image.href}
    <a href={image.href}>
      {@render figure(image)}
    </a>
  {:else}
    {@render figure(image)}
  {/if}
{/each}
```

[浏览器效果如下：](https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE5VTYW-bMBD9KyeiKYlEY4jWfSAk2n5H6QcXDmwVbMs2SzuL_z6DTRqp2rQJ2Ycfd_ced2eXtLxHkxRPLhF0wKRIfiiVpIl9V_PB_MTeoj8bOep6RkpTa67spRKV7dECH2iHBs7wNCOVdcFU1ui6gC2zVpmCEMVrMw4HxaSVhnzLMnLMsm26Ol95Y1kBHr9BDHnHbAHHO6ymynIpfF7LuAncwKgBCj0Xrx_5mMb2jh3f6KB6PNRy2AaXKf1fuY__KPfxj3KlQGikL5aQdpUxm-dTJUryUVdRsvwSqEviX2fIbYzgSvmCt7wbNe4ceMUpRIoUFkkpBBkw7ZfMZXC-BLKSDx3Q3p5djJrA-SR-X4K9DdHT6u-jo-flFlKSO3ThIDcSR6LIKUhGWrN1QGhs16LLbXgbjoe5U1PkozCfzu7uy2WtpfuuUTSo1_9ffPZrJKGLoyuwNxjBv0Q4wmdSR2aFi9jS2Pc-FIrlEKeilcI-GP4LfVtxOM1gyO1XSLp6vtD6tdNyFE0BV8YtngKuaNNw0RWQx_jKDlR33M9E5h-PQhZxfxEt6gIaLdWDYbSR191RvcFXv_LMb7p7obssXZ5Dvt_f9HgzdzZKibOZZ9mXmHkdTTpaefqsd4OIay4_hksd_I0fZMNbjk1SWD3i9Dz9BpdEPu8sBAAA "https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE5VTYW-bMBD9KyeiKYlEY4jWfSAk2n5H6QcXDmwVbMs2SzuL_z6DTRqp2rQJ2Ycfd_ced2eXtLxHkxRPLhF0wKRIfiiVpIl9V_PB_MTeoj8bOep6RkpTa67spRKV7dECH2iHBs7wNCOVdcFU1ui6gC2zVpmCEMVrMw4HxaSVhnzLMnLMsm26Ol95Y1kBHr9BDHnHbAHHO6ymynIpfF7LuAncwKgBCj0Xrx_5mMb2jh3f6KB6PNRy2AaXKf1fuY__KPfxj3KlQGikL5aQdpUxm-dTJUryUVdRsvwSqEviX2fIbYzgSvmCt7wbNe4ceMUpRIoUFkkpBBkw7ZfMZXC-BLKSDx3Q3p5djJrA-SR-X4K9DdHT6u-jo-flFlKSO3ThIDcSR6LIKUhGWrN1QGhs16LLbXgbjoe5U1PkozCfzu7uy2WtpfuuUTSo1_9ffPZrJKGLoyuwNxjBv0Q4wmdSR2aFi9jS2Pc-FIrlEKeilcI-GP4LfVtxOM1gyO1XSLp6vtD6tdNyFE0BV8YtngKuaNNw0RWQx_jKDlR33M9E5h-PQhZxfxEt6gIaLdWDYbSR191RvcFXv_LMb7p7obssXZ5Dvt_f9HgzdzZKibOZZ9mXmHkdTTpaefqsd4OIay4_hksd_I0fZMNbjk1SWD3i9Dz9BpdEPu8sBAAA")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79ae7e3d2992443385394b27ab4c9b10~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3316&h=1216&s=2297425&e=png&b=272727)

**注意：与函数声明一样，snippets 可以具有任意数量的参数，参数可以有默认值。但不能使用剩余参数（rest parameters）。**

### 2.2. 使用范围

Snippets 可以声明在组件的任何位置，且可以引用 Snippets 之外的值，比如在 `<script>` 或 `{#each ...}`中的值：

```xml
<script>
  let { message = `it's great to see you!` } = $props();
</script>

{#snippet hello(name)}
  <p>hello {name}! {message}!</p>
{/snippet}

{@render hello('alice')}
{@render hello('bob')}
```

Snippets 的引用遵循词法规则。所谓词法规则，以 JavaScript 为例：

```javascript
(function () {
  function a() {
    function b() {}
    // ✅
    b();
  }
  // ❌
  b();
})();

// ❌
a();
```

换成 Snippts 也是一样的：

```xml
<div>
  {#snippet x()}
    {#snippet y()}...{/snippet}

    <!-- ✅ -->
    {@render y()}
  {/snippet}

  <!-- ❌ -->
  {@render y()}
</div>

<!-- ❌ -->
{@render x()}
```

简单来说，`#snippet` 相当于函数声明，`@render` 相当于调用。Snippets 只对同级兄弟姐妹以及同级兄弟姐妹的子级们可见。

Snippets 可以相互引用，甚至可以引用自己：

```xml
{#snippet blastoff()}
  <span>🚀</span>
{/snippet}

{#snippet countdown(n)}
  {#if n > 0}
    <span>{n}...</span>
    {@render countdown(n - 1)}
  {:else}
    {@render blastoff()}
  {/if}
{/snippet}

{@render countdown(10)}
```

[浏览器效果如下：](https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE2WPTQqDMBCFrxLiRqH1Zysi7TlqF1YnENBJSGJLCYGeo5tesUeosfYH3c2bee_jjaWMd6BpfrAU6x5oTvdS0g01V-mFPkNnYNRaDKrxGxto5FKCIaeu1kYwFkauwsoUWtZYPh_3W5FMY4U2mb3egL9kIwY0rbhgiO-sDTgjSEqSTvIDs-jiOP7i_MHuFGAL6p9BtiSbOTl0GtzCuihqE87cqtyam6WRGz_vRcsZh5bmRg3gju4Fptq_kzQBAAA= "https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE2WPTQqDMBCFrxLiRqH1Zysi7TlqF1YnENBJSGJLCYGeo5tesUeosfYH3c2bee_jjaWMd6BpfrAU6x5oTvdS0g01V-mFPkNnYNRaDKrxGxto5FKCIaeu1kYwFkauwsoUWtZYPh_3W5FMY4U2mb3egL9kIwY0rbhgiO-sDTgjSEqSTvIDs-jiOP7i_MHuFGAL6p9BtiSbOTl0GtzCuihqE87cqtyam6WRGz_vRcsZh5bmRg3gju4Fptq_kzQBAAA=")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86047cee865c4636bae64faf9d86d26c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1890&h=1084&s=207838&e=png&b=282828)

### 2.3. 取代插槽

在模板中，Snippets 和其他值其实是一样的，所以 Snippets 也可以作为值传给组件：

```xml
<!-- App.svelte -->
<script>
  import Button from './Button.svelte'
</script>

{#snippet blastoff()}
  <span>🚀</span>
{/snippet}

<Button {blastoff} />

<!-- Button.svelte -->
<script>
  let { blastoff } = $props();
</script>

{@render blastoff()}
```

我们看下编译后的内容就明白了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/065a11be91e5456ab19057bc40a668e8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2560&h=1020&s=309848&e=png&b=1e1e1e)

为了方便，**声明在组件内部的 snippets 会隐式作为 props 传给组件**。所以：

```xml
{#snippet blastoff()}
  <span>🚀</span>
{/snippet}

<Button {blastoff} />

<!-- 等同于 -->

<Button>
  {#snippet blastoff()}
    <span>🚀</span>
  {/snippet}
</Button>
```

可以看出，Snippets 用在组件内部的时候，作用类似于 Svelte 4 的命名插槽功能。

Snippets 用在组件内部的时候，也可以不进行 snippets 声明。如果不进行声明，会隐式成为 children 代码段的一部分：

```xml
<!-- App.svelte -->
<script>
  import Button from './Button.svelte'
</script>

<Button>
  <span>🚀</span>
</Button>

<!-- Button.svelte -->
<script>
  let { children } = $props();
</script>

{#if children}
  {@render children()}
{/if}
```

> 注意：组件的 props 避免命名为 children。倒不是不能用，只是可能会发生冲突。

### 2.4. Type

Svelte 提供了 Snippets 类型：

```xml
<script lang="ts">
  import type { Snippet } from 'svelte';

  let { children } : { children: Snippet } = $props();
</script>
```

总结一下：Snippets 通过复用代码片段，可以完全替代 Svelte 4 的插槽和命名插槽等功能。Snippets 功能更加强大、使用更加灵活。在 Svelte 5 中，插槽已被弃用，但依然可以继续工作。

## 3\. 特殊元素

### 3.1. [svelte:component]()（Legacy）

`<svelte:component>`解决的是动态组件的问题，根据条件渲染不同的组件：

```xml
<!-- App.svelte -->
<script>
  import Foo from './Foo.svelte'
  import Bar from './Bar.svelte'

  let count = 0;
</script>

<button on:click={() => {
  count += 1;
}}>
  Clicked {count}
</button>

{#if count > 3}
  <Foo {count} />
{:else}
  <Bar {count} />
{/if}

<!-- Foo.svelte -->
<script>
  export let count
</script>

Foo {count}

<!-- Bar.svelte -->
<script>
  export let count
</script>

Bar {count}
```

使用 `<svelte:component>`，原本的：

```xml
{#if count > 3}
  <Foo {count} />
{:else}
  <Bar {count} />
{/if}
```

可以简化为：

```xml
<svelte:component this={count > 3 ? Foo : Bar} {count} />
```

[浏览器效果如下：](https://svelte.dev/repl/6c470fdfce8643da866afe437aae7922?version=4.2.18 "https://svelte.dev/repl/6c470fdfce8643da866afe437aae7922?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4501c0698d104079b8e0883ef45cf4b9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1308&h=743&s=68802&e=gif&f=29&b=252525)

使用时注意：

1. 当 this 的值为假值（false、null、undefined 等）时，组件不会渲染
2. 当属性值更改的时候，组件会销毁重建

注：在 Svelte 5 中，这个例子可以写成：

```xml
<script>
    import Foo from './Foo.svelte'
    import Bar from './Bar.svelte'

    let count = $state(0);

    let Component = $derived(count > 3 ? Foo : Bar)
</script>

<button onclick={() => {
    count += 1;
}}>
    Clicked {count}
</button>

<Component {count} />
```

[点击查看浏览器效果](https://svelte.dev/playground/2e4ce15d36d44109aa87f1d1338935f0?version=5.1.3 "https://svelte.dev/playground/2e4ce15d36d44109aa87f1d1338935f0?version=5.1.3")。

因此虽然可以继续使用，但在 Svelte 5 中该语法被认为是过时的。

### 3.2. [svelte:element]()

`<svelte:element>`与 `<svelte:component>`类似，只不过解决的是动态元素的问题：

```xml
<script>
  let count = 0;
</script>

<button on:click={() => {
  count += 1;
}}>
  Clicked {count}
</button>

<svelte:element this={count > 3 ? "h1" : "p"}>{count}</svelte:element>
```

[浏览器效果如下：](https://svelte.dev/repl/b38580c52b88469fbab7435190920c08?version=4.2.18 "https://svelte.dev/repl/b38580c52b88469fbab7435190920c08?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97826fd447274c73909a7b4865a94f14~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1368&h=438&s=829552&e=gif&f=30&b=262626)

使用时注意：

1. 当 this 的值为假值（false、null、undefined 等）时，元素及其子元素都不会渲染
2. 不能使用 bind:value 等绑定，因为 Svelte 并不知道你最终渲染的元素是什么，如果是 input 还好，万一只是一个普通的 div 呢，所以唯一支持的绑定是 `bind:this`

### 3.3. [svelte:window]()

`<svelte:window>` 用于在 window 对象上添加事件监听器。

通常我们需要这样写代码：

```xml
<script>
  import { onMount } from 'svelte';

  let innerWidth = window.innerWidth;

  onMount(() => {
    function onResize() {
      innerWidth = window.innerWidth;
    }
    window.addEventListener('resize', onResize);
    return () => window.removeEventListener('resize', onResize);
  });
</script>

<div>
  Width: {innerWidth}
</div>
```

这里一段监听浏览器窗口宽度的代码。[浏览器效果如下：](https://svelte.dev/repl/930a0ce858564a6da1c4dff978b5d6bf?version=4.2.18 "https://svelte.dev/repl/930a0ce858564a6da1c4dff978b5d6bf?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d467a5580d945849c983aa6027d5736~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1261&h=646&s=992454&e=gif&f=24&b=232323)

我们需要在组件挂载的时候添加监听器，在组件卸载的时候移除监听器。使用 `<svelte:window>`后，代码简化为：

```xml
<script>
  import { onMount } from 'svelte';

  let innerWidth = window.innerWidth;

  function onResize() {
    innerWidth = window.innerWidth;
  }
</script>

<svelte:window on:resize={onResize} />

<div>
  Width: {innerWidth}
</div>
```

`<svelte:window>` 会自动进行处理监听器的添加和移除。

Svelte 甚至提供了 `<svelte:window bind:prop={value} />`的方式，可以将代码进一步简化为：

```xml
<script>
  import { onMount } from 'svelte';

  let innerWidth = window.innerWidth;

</script>

<svelte:window bind:innerWidth={innerWidth} />

<div>
  Width: {innerWidth}
</div>
```

具体能够绑定获取哪些属性的值，Tutorial 中也有介绍：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74ff7b443a5648c6b8dfc7b3af6bacc6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2004&h=772&s=91169&e=png&b=1c1e21)

### 3.4. [svelte:body]()

svelte:body 与 svelte:window 类似，用于在 document.body 上添加事件监听器。因为像比如 mouseenter 和 mouseleave 事件在 window 上就不能添加。

### 3.5. [svelte:document]()

svelte:document 与 svelte:window 类似，用于在 document 上添加事件监听器。因为像比如 visibilitychange 不能在 window 上触发。

### 3.6. [svelte:head]()

svelte:head 用于将元素插入到 `document.head`中：

```xml
<svelte:head>
  <title>Hello world!</title>

  <meta name="description" content="This is where the description goes for SEO" />
</svelte:head>
```

你可以在其中正常使用变量：

```xml
<script>
  let value = 'world'
</script>

<input bind:value />

<svelte:head>
  <title>Hello {value}!</title>

  <meta name="description" content="This is where the description goes for SEO" />
  {@html `<style>
    input {
      color: red
    }
  </style>`}
</svelte:head>
```

[浏览器效果如下：](https://svelte.dev/repl/5d329da2f8d54c5696b354459052b8fd?version=4.2.18 "https://svelte.dev/repl/5d329da2f8d54c5696b354459052b8fd?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb235c85815d476281b653bcf79aac4b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3046&h=1152&s=307644&e=png&b=262626)

与 `<svelte:window>` 一样，该元素只能出现在组件的顶层，绝不能位于块或元素内。

### 3.7. [svelte:options]()

[svelte:options]() 提供了组件的编译选项，将决定 Svelte 如何编译组件。完整的选项查看 [Svelte Options](https://svelte.dev/docs/svelte/svelte-options "https://svelte.dev/docs/svelte/svelte-options")。

* `runes={true}`：强制进入符文模式
* `runes={false}`：强制进入 legacy 模式
* `namespace="..."`：组件命名空间，值有 `html`、`svg`、`mathml`，默认是 `html`
* `customElement={...}`：将此组件编译为[自定义元素](https://svelte.dev/docs/svelte/custom-elements "https://svelte.dev/docs/svelte/custom-elements")
* `css="injected"`：组件会将样式内联注入，在服务端渲染时，作为 `<style>` 标签注入 head 中，在客户端渲染时，将通过 JavaScript 渲染

关于 `namespace`，其实就是告诉 Svelte 应该编译成什么。默认是 html，也就是说告诉 Svelte 我写的是 HTML，但也可以告诉 Svelte 我写的是 svg：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5099d6c182264b2d9bc9aa227004dd7a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2354&h=596&s=77167&e=png&b=1e1e1e)

当使用 `<svelte:options namespace="svg" />`时：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5c4df8b426448dc884f2008cc4cb126~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2926&h=554&s=119479&e=png&b=1e1e1e)

Svelte 会改为创建 Svg 元素。

关于 `customElement`，用于将组件编译为自定义的元素：

```xml
<svelte:options customElement="my-custom-element" />
```

这个时候就可以使用：

```xml
document.body.innerHTML = `
  <my-element>
    <p>This is some slotted content</p>
  </my-element>
`;
```

不算是常用的功能，但展示了编译器的强大功能。

### 3.8. [svelte:fragment]()（Legacy）

`<svelte:fragment>` 用于将内容放置在命名插槽中，而无须包装在 DOM 元素中，类似于 React 的 `<></>`：

```xml
<!-- App.svelte -->
<Widget>
  <h1 slot="header">Hello</h1>

  <svelte:fragment slot="footer">
    <p>All rights reserved.</p>

    <p>Copyright (c) 2019 Svelte Industries</p>

  </svelte:fragment>

</Widget>

<!-- Widget.svelte -->
<div>
  <slot name="header">No header was provided</slot>

  <p>Some content between header and footer</p>

  <slot name="footer" />
</div>
```

[浏览器效果如下：](https://svelte.dev/repl/b2152b8d753b4c87824d4132a6387346?version=4.2.19 "https://svelte.dev/repl/b2152b8d753b4c87824d4132a6387346?version=4.2.19")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1469486939704285b92c4f417c00663a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=4132&h=1056&s=338668&e=png&b=282828)

注：在 Svelte 5 中使用 Snippets 即可，因此该语法虽然可以继续使用，但被认为是过时的

### 3.9. [svelte:self]()（Legacy）

`<svelte:self>` 允许组件以递归的方式包含自身，主要是用来处理递归。场景倒很多，就是一不小心容易写成无限循环。就比如渲染城市数据：

```xml
<script>
  export let data = [
    { type: 'city', value: '北京'},
    {
      type: 'province',
      value: '浙江',
      children: [
        { type: 'city', value: '杭州'},
        { type: 'city', value: '宁波'}
      ]
    }
  ];
</script>

  <ul>
    {#each data as item}
      <li>
        {#if item.type === 'province'}
          {item.value}
          <svelte:self data={item.children} />
        {:else if item.type === 'city'}
          <span>{item.value}</span>

        {/if}
      </li>

    {/each}
  </ul>
```

[浏览器效果如下：](https://svelte.dev/repl/2fca66addb9348da8f78ea995415cf78?version=4.2.18 "https://svelte.dev/repl/2fca66addb9348da8f78ea995415cf78?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65fa8eae8648480b8a342cb71247085e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2348&h=888&s=106551&e=png&b=252525)

使用时注意：

1. 不能用在顶层，只能用在 IF、Each 块或者组件插槽中，这都是为了防止无限循环
2. 注意使用 export 才能在递归的时候获取数据

注：在 Svelte 5 中，这个例子可以写成：

```xml
<script>
    import Self from './App.svelte'
  export let data = [
    { type: 'city', value: '北京'},
    {
      type: 'province',
      value: '浙江',
      children: [
        { type: 'city', value: '杭州'},
        { type: 'city', value: '宁波'}
      ]
    }
  ];
</script>

<ul>
    {#each data as item}
    <li>
        {#if item.type === 'province'}
            {item.value}
            <Self data={item.children} />
        {:else if item.type === 'city'}
            <span>{item.value}</span>

        {/if}
    </li>

    {/each}
</ul>
```

因此虽然可以继续使用，但在 Svelte 5 中该语法被认为是过时的。

## 4\. 最后

本篇我们介绍了 Snippets 和特殊元素的用法，插槽在 Svelte 5 中可被 Snippets 替代。特殊元素不算常用，但有时很有必要，所以了解即可，用到的时候再细查用法。