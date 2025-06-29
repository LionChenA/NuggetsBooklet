> 推荐学习指数：⭐️️，了解即可

## 1\. 前言

本篇我们来介绍 Svelte 的重要概念 —— Special elements。为什么叫 Specail elements 呢？因为它们的写法类似于普通的 elements，但实现的功能却比较特殊。包括实现插槽的 `<slot>`、修改 HTML head 的 `<svelte:body>`等等。

这些也是 Svlete 项目开发中常用的内容。就让我们一起看看如何使用吧！

## 2\. 插槽（Slots）

> 注：Svelte 5 提供了 Snippets 可替代插槽的功能，但 Slots 在 Svelte 5 中依然可以使用。这块内容看不看都行，你看了也不亏，Vue、Astro 框架中都是类似的用法。

### 2.1. Slot Basic

所谓 Slot，类似于 Vue 的 Slot 功能，看个最简单的例子：

```xml
<!-- App.svelte -->
<script>
  import Button from './Button.svelte';
</script>

<Button>
  Primary Button
</Button>

<!-- Button.svelte -->
<slot />
```

[浏览器效果如下：](https://svelte.dev/repl/fc25463c8cdf4662bfe96f207feaa306?version=4.2.18 "https://svelte.dev/repl/fc25463c8cdf4662bfe96f207feaa306?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5a0555e6eab47ff8775b020a6a6cecf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1864&h=606&s=100655&e=png&b=262626)

也就是说，子组件可以使用 `<slot>`元素获取父组件的插槽内容。

### 2.2. Slot Fallbacks

Slot 可以设置 fallback 内容，当父组件没有传入插槽内容的时候，slot 会显示该内容：

```xml
<!-- App.svelte -->
<script>
  import Button from './Button.svelte';
</script>

<Button />

<!-- Button.svelte -->
<slot>Primary Button</slot>
```

[浏览器效果如下：](https://svelte.dev/repl/a108aec0eef04058b4ea0e56491c4f4a?version=4.2.18 "https://svelte.dev/repl/a108aec0eef04058b4ea0e56491c4f4a?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cf47872c6ce4921a4c7646ac2d43e67~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1902&h=464&s=81439&e=png&b=262626)

### 2.3. Slot 中的变量

父组件可以在插槽内容中正常使用变量，我们举个例子：

```xml
<!-- App.svelte -->
<script>
  import Button from './Button.svelte';
  let count = 0;

</script>

<button on:click={() => count++}>+</button>
  {count}
<button on:click={() => count--}>-</button>

<br />

<Button>
  <button on:click={() => count++}>+</button>
    {count}
  <button on:click={() => count--}>-</button>
</Button>

<style>
  button {
    width: 50px;
    color: orange
  }
</style>

<!-- Button.svelte -->
<slot>Primary Button</slot>
```

[浏览器效果如下：](https://svelte.dev/repl/4cbef48b4c164817a1231cfb4a618ec5?version=4.2.18 "https://svelte.dev/repl/4cbef48b4c164817a1231cfb4a618ec5?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d76966043e94256adbcfce27006e62d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1759&h=882&s=107655&e=gif&f=34&b=272727)

在这段代码中，`count` 变量写在了父组件插槽内容中，其值的变化受父组件的状态影响，且样式也是 scoped，受父组件的样式影响，而不会受子组件影响。

### 2.4. Named Slots

命名插槽可以让你在插槽中传递多个内容。让我们看个例子：

```xml
<!-- App.svelte -->
<script>
  import Article from './Article.svelte';
</script>

<Article>
  <span slot="header">header</span>

  <small>body</small>

  <span slot="footer">
    <small>footer</small>
  </span>
</Article>

<!-- Article.svelte -->
<div class="article">
  <header>
    <slot name="header" />
  </header>

  <slot />

  <footer>
    <slot name="footer" />
  </footer>
</div>
```

[浏览器效果如下：](https://svelte.dev/repl/4726cc887b6741e39be724c62276246d?version=4.2.18 "https://svelte.dev/repl/4726cc887b6741e39be724c62276246d?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/174aefb8035a4dfc96337fe8845f7548~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1938&h=1014&s=181274&e=png&b=272727)

也就是说：当你在父组件的插槽内容中通过 `slot="xxx"`方式声明一个插槽，你就可以在子组件通过 `<slot name="xxx" />`引用这个插槽内容。

### 2.5. Slot Props

有的时候，你需要将子组件的数据传递给父组件的插槽内容用于渲染。一个常见的例子是遍历数组，举个例子：

```xml
<!-- App.svelte -->
<script>
  import List from './List.svelte';
  const names = ["Aaron", "Baron", "Caesar"]
</script>

<List data={names} let:item>
  <li>{item}</li>
</List>

<!-- List.svelte -->
<script>
  export let data;
</script>

<ul>
  {#each data as item}
    <slot {item} />
  {/each}
</ul>
```

在这个例子中，我们声明了一个 `<List>` 组件，`<List>`组件的插槽用于声明单个 Item 的渲染样式，其中就需要获取数据，才能指定数据渲染的位置。这个时候就需要使用 Slot props。

它的使用方式是首先在子组件指定 `<slot key={value}>`（比如上面代码中的 `<slot item={item} />`），然后就可以在父组件通过 `let`指令将值传递给插槽模板（比如上面代码中的 `<List let:item={item}>`）。

[浏览器效果如下：](https://svelte.dev/repl/73557390b8b24306a46169bc881a0c53?version=4.2.18 "https://svelte.dev/repl/73557390b8b24306a46169bc881a0c53?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfb6a1b3cac342338e035ec8d2a5e34c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2322&h=676&s=141824&e=png&b=262626)

注意：我们通过 `<List let:item>`获取了给插槽模板的 item 变量，如果该变量与父组件的变量冲突，可以使用 `<List let:item={xxx}>`进行重命名：

```xml
<List data={names} let:item={name}>
  <li>{name}</li>
</List>
```

### 2.6. Slot Forwarding

假设现在有 4 个组件，层级关系是 App > Parent > Child > GrandChild，如果想把 App 中声明的插槽模板用在子孙组件 GrandChild 中，作为中间组件的 Parent 和 Child 该怎么传递呢？

其实并不复杂：

```xml
<!-- App.svelte -->
<script>
  import Parent from './Parent.svelte';
</script>

<Parent>
  <span slot="header">header</span>

  <small>body</small>

  <span slot="footer">
    <small>footer</small>
  </span>
</Parent>

<!-- Parent.svelte -->
<script>
  import Child from './Child.svelte';
</script>

<Child>
  <slot name="header" slot="header" />
  <slot />
  <slot name="footer" slot="footer" />
</Child>

<!-- Child.svelte -->
<script>
  import GrandChild from './GrandChild.svelte';
</script>

<GrandChild>
  <slot name="header" slot="header" />
  <slot />
  <slot name="footer" slot="footer" />
</GrandChild>

<!-- GrandChild.svelte -->
<div class="article">
  <header>
    <slot name="header" />
  </header>
  <slot />
  <footer>
    <slot name="footer" />
  </footer>
</div>
```

[浏览器效果如下：](https://svelte.dev/repl/d23f6998bf384afeb6f56b427499be90?version=4.2.18 "https://svelte.dev/repl/d23f6998bf384afeb6f56b427499be90?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6caada93f394ce4a06e132d7816e338~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1013&h=511&s=109476&e=gif&f=35&b=272727)

那么如果我们想要像 Slot Props 中的例子将子孙组件 GrandChild 的数据传递给顶层组件 App 呢？实现起来也不复杂：

```xml
<!-- App.svelte -->
<script>
  import Parent from './Parent.svelte';
  const names = ["Aaron", "Baron", "Caesar"]
</script>

<Parent data={names}>
  <li slot="item" let:item>{item}</li>
</Parent>

<!-- Parent.svelte -->
<script>
  import Child from './Child.svelte';
  export let data;
</script>

<Child data={data}>
  <slot name="item" slot="item" let:item {item} />
</Child>

<!-- Child.svelte -->
<script>
  import GrandChild from './GrandChild.svelte';
  export let data;
</script>

<GrandChild data={data}>
  <slot name="item" slot="item" let:item {item} />
</GrandChild>

<!-- GrandChild.svelte -->
<script>
  export let data;
</script>

<ul>
  {#each data as item}
    <slot name="item" {item} />
  {/each}
</ul>
```

[浏览器效果如下：](https://svelte.dev/repl/43747e12bd79470f80766317e8a56659?version=4.2.18 "https://svelte.dev/repl/43747e12bd79470f80766317e8a56659?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1dc81f71ec242ceb64ea55618af6997~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2222&h=684&s=155587&e=png&b=272727)

### 2.7. $$slots

`$$slots` 是一个特殊变量，只有一个用途，就是判断父级是否传入了特定名称的插槽。以之前的 Named Slots 为例：

```xml
<!-- App.svelte -->
<script>
  import Article from './Article.svelte';
</script>

<Article>
  <!-- 假设没有下面这行-->
  <span slot="header">header</span>

  <small>body</small>

  <span slot="footer">
    <small>footer</small>
  </span>
</Article>

<!-- Article.svelte -->
<div class="article">
  <header>
    <slot name="header" />
  </header>

  <slot />

  <footer>
    <slot name="footer" />
  </footer>
</div>
```

假设父组件的插槽中没有 `<span slot="header">header</span>`这行内容，子组件会渲染一个空的 `<header>` 元素。如果你想要子组件连 `<header>` 元素也不渲染呢？

这个时候就可以用到 `$$slots`：

```xml
<!-- App.svelte -->
<script>
  import Article from './Article.svelte';
</script>

<Article>
  <small>body</small>

  <span slot="footer">
    <small>footer</small>
  </span>
</Article>

<!-- Article.svelte -->
<script>
  console.log($$slots)
</script>
<div class="article">
  {#if $$slots.header}
    <header>
      <slot name="header" />
    </header>
  {/if}

  <slot />

  <footer>
    <slot name="footer" />
  </footer>
</div>
```

[浏览器效果如下：](https://svelte.dev/repl/da4b29ec0d2e4f2d93c3474401667440?version=4.2.18 "https://svelte.dev/repl/da4b29ec0d2e4f2d93c3474401667440?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13fc4f24e8184c4c9ec6dca81602ae87~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3120&h=1490&s=614092&e=png&b=262626)

可以看到，`$$slots` 是一个对象，它的键名是父组件传入的命名插槽的名称。

> 注意：假如父组件传入了空的命名插槽，比如 ，$$slots.header 也会返回 true

## 3\. 特殊元素

### 3.1. <svelte:component>（Legacy）

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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b820d5877c84125a58911e151559347~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1308&h=743&s=68802&e=gif&f=29&b=252525)

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

### 3.2. <svelte:element>

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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7db254760d449e98eb9dd0bbaa981ab~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1368&h=438&s=46414&e=gif&f=30&b=262626)

使用时注意：

1. 当 this 的值为假值（false、null、undefined 等）时，元素及其子元素都不会渲染
2. 不能使用 bind:value 等绑定，因为 Svelte 并不知道你最终渲染的元素是什么，如果是 input 还好，万一只是一个普通的 div 呢

### 3.3. <svelte:self>（Legacy）

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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54280086ed7047079947d165b502f44f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2348&h=888&s=155061&e=png&b=252525)

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

### 3.4. <svelte:window>

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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d37cdafe94d54adda1c6f83760174520~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1261&h=646&s=141001&e=gif&f=24&b=232323)

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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8cc87f9b3434d04b66fbc6ad8729281~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1630&h=968&s=120142&e=png&b=262626)

### 3.5. <svelte:body>

[svelte:body]() 与 [svelte:window]() 类似，用于在 document.body 上添加事件监听器。因为像比如 mouseenter 和 mouseleave 事件在 window 上就不能添加。

### 3.6. <svelte:document>

[svelte:document]() 与 [svelte:window]() 类似，用于在 document 上添加事件监听器。因为像比如 visibilitychange 不能在 window 上触发。

### 3.7. <svelte:head>

[svelte:head]() 用于将元素插入到 `document.head`中：

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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27739ddaaf1c4038a5265a62ef9d7cba~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3046&h=1152&s=420540&e=png&b=262626)

与 `<svelte:window>` 一样，该元素只能出现在组件的顶层，绝不能位于块或元素内。

### 3.8. <svelte:options>

[svelte:options]() 提供了组件的编译选项，将决定 Svelte 如何编译组件。完整的选项查看 [svelte/compiler • Docs • Svelte](https://svelte.dev/docs/svelte-compiler#compile "https://svelte.dev/docs/svelte-compiler#compile")

> 注：在 Svelte 5 中，immutable 编译器选项被弃用，因为 Svelte 5 的符文模式下，所有的状态都是 immutable

比较常见的是 3 个选型，一个是 immutable，默认为 false，当为 true 时，表示自己使用不可变数据，编译器可以执行简单的引用相等检查来确定值是否已更改，这会减少渲染次数，提升组件性能。让我们看个例子：

```bash
<script>
    let numbers = [1, 2, 3, 4];
    function addNumber() {
        numbers.push(numbers.length + 1);
        numbers = numbers;
    }
</script>

<p>{numbers.join(' + ')}</p>

<button on:click={addNumber}>
    Add a number
</button>

<svelte:options immutable={false} />
```

当 immutable 设置为 false 的时候，[浏览器效果如下](https://svelte.dev/repl/bd6cbc4865d84448bd436e51c89c3e7b?version=4.2.18 "https://svelte.dev/repl/bd6cbc4865d84448bd436e51c89c3e7b?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26114514440e45c0ab0ef3534c0daafb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1188&h=630&s=66094&e=gif&f=18&b=272727)

此时正常运行，但如果修改 `immutable={false}`，点击则不会有任何反应。这是因为你已经告诉了编译器你将使用不可变数据，编译器只会比较 numbers 的引用是否发生变化，而此时并未发生变化，所以不会触发响应式。

如果要触发，可以修改成：

```bash
<script>
    let numbers = [1, 2, 3, 4];
    function addNumber() {
        numbers.push(numbers.length + 1);
        numbers = [...numbers];
    }
</script>

<p>{numbers.join(' + ')}</p>

<button on:click={addNumber}>
    Add a number
</button>

<svelte:options immutable={true} />
```

[浏览器效果如下：](https://svelte.dev/repl/fe676259798c4b87b4670a2e84b54980?version=4.2.18 "https://svelte.dev/repl/fe676259798c4b87b4670a2e84b54980?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2987240e9ff04477a3765e15f49e2bcc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1188&h=630&s=67905&e=gif&f=18&b=282828)

第二个是 accessors，默认为 false，当为 true 时，为组件的 props 添加 getter 和 setter。这样做的好处在于可以读取组件实例的 props，正常是无法直接读取的。

```xml
<!-- App.svelte -->
<script>
  import Child from './Child.svelte'

  let component

  function handler() {
  component.name = 'Dasiy'
  }
</script>

<button on:click={handler}>
  切换名字
</button>

<Child bind:this={component} />

<!-- Child.svelte -->
<svelte:options accessors />

<script>
  export let name = 'Kevin'
</script>
<h1>{name}!</h1>
```

[浏览器效果如下：](https://svelte.dev/repl/966ddce1f9664f60b165d9a3ec66ec78?version=4.2.18 "https://svelte.dev/repl/966ddce1f9664f60b165d9a3ec66ec78?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4fd5b8c4ef0408a9915fe676a0437d8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1188&h=630&s=82471&e=gif&f=19&b=232323)

其实效果类似于使用客户端 API，只不过更加简洁：

```xml
function handler() {
  component.$set({
    name: 'Daisy'
  })
}
```

还有一个是 namespace，将使用该组件的命名空间，最常见的是“svg”；使用“foreign”命名空间选择不区分大小写的属性名称和特定于 HTML 的警告

默认是 html，也就是说告诉 Svelte 我写的是 HTML，但也可以告诉 Svelte 我写的是 svg：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67950fde11c143a78f2a8cfd7483252e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2354&h=596&s=113363&e=png&b=1e1e1e)

当使用 `<svelte:options namespace="svg" />`时：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85877a7a00f5419ca92b39310de376c7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2926&h=554&s=166079&e=png&b=1e1e1e)

Svelte 会改为创建 Svg 元素。

还有一个是 customElement，用于将组件编译为自定义的元素：

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

不算是常用的功能，但展示了编译器的强大功能。具体使用查看[文档《自定义元素 API 》](https://svelte.dev/docs/custom-elements-api "https://svelte.dev/docs/custom-elements-api")。

### 3.9. <svelte:fragment>（Legacy）

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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56df0e722a724f34aa1d410369b801c7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=4132&h=1056&s=463473&e=png&b=282828)

注：在 Svelte 5 中使用 Snippets 即可，因此该语法虽然可以继续使用，但被认为是过时的

## 4\. 最后

本篇我们介绍了插槽和特殊元素的用法，插槽在 Svelte 5 中可被 Snippets 替代。特殊元素不算常用，但有时很有必要，所以了解即可，用到的时候再细查用法。