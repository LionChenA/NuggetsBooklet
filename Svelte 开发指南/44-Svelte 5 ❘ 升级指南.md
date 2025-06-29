> 推荐学习指数：⭐️️，了解即可

## 1\. 前言

本篇讲解 Svelte 4 到 Svelte 5 的 API 变化，方便大家快速升级到 Svelte 5。

## 2\. 状态声明

`$state`替代之前的 `let`声明。

`$derived`替代之前的 `$: x = ...`。

`$effect`替代 onMount 和 `$: { ... }`。

Svelte 4：

```xml
<script>
  let count = 0;
  $: double = count * 2;
  $: {
    if (count > 10) {
      alert('Too high!');
    }
  }
</script>

<button on:click={() => count++}>
  {count} / {double}
</button>
```

Svelte 5：

```xml
<script>
  let count = $state(0);
  let double = $derived(count * 2);
  $effect(() => {
    if (count > 10) {
      alert('Too high!');
    }
  });
</script>

<button onclick={() => count++}>
  {count} / {double}
</button>
```

## 3\. 组件 props

$props 可替代之前的 `export let`、`export { x as y }`、`$props`和`$restProps`：

```xml
<script>
  let { count = 0, catch: theCatch, ...rest } = $props();
</script>
```

## 4\. 生命周期

`$effect.pre` 替代之前的 `beforeUpdate`

## 5\. 事件处理程序

Svelte 4：

```xml
<button on:click={() => count++}>
  clicks: {count}
</button>
```

Svelte 5：

```xml
<button onclick={() => count++}>
  clicks: {count}
</button>
```

事件处理程序变成了普通的属性，所以不再需要转发事件，正常传递、正常使用回调函数即可。

```xml
<script>
  let { onclick, onkeydown, ...attributes } = $props();
</script>

<button
  {...attributes}
  {onclick}
  {onkeydown}
>a button</button>
```

## 6\. 插槽

Snippets 将完全替代之前的插槽。

Svelte 4：

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
</div>
```

Svelte 5：

```xml
<!-- App.svelte -->
<script>
  import Article from './Article.svelte';
</script>

<Article>
  {#snippet header()}
    header
  {/snippet}
  <small>body</small>
</Article>

<!-- Article.svelte -->
<script>
  let { header, children } = $props();
</script>

<div class="article">
  <header>
    {@render header()}
  </header>
  {@render children()}
</div>
```

这些更新比如 Snippets 和事件处理程序，虽然 Svelte 5 已经不用，但在 Svelte 5 中依然可以正常工作。下面的内容是要被废弃的。

## 7\. Svelte 5 废弃

### 7.1. 删除 beforeUpdate 和 afterUpdate

在 Svelte 5 中，改为使用 `$effect.pre` 和 `$effect`。尽管作用并不完全相同，但调用更加可控，我们可以确保只有在我们关心的值发生变化时才进行调用。

### 7.2. 删除 createEventDispatcher

Svelte 4：

```xml
<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  function greet() {
    dispatch('greet')
  }
</script>

<button
  on:click={greet}
>greet</button>
```

Svelte 5：

```xml
<script>
  let { greet } = $props();
</script>

<button
  onclick={greet}
>greet</button>
```

### 7.3. immutable 编译器选项被弃用

符文模式下，所有的状态都是 immutable。

## 8\. 恭喜你！

看到这里，恭喜你完成了第二阶段 —— Svelte 5 新增内容的学习：

1. 第一阶段：Svelte 4 🎉
2. 第二阶段：Svelte 5 🎉
3. 第三阶段：SvelteKit
4. 第四阶段：项目实战
5. 第五阶段：Svelte 原理

这个时候你应该对 Svelte 语法已经有了一个全面的了解。

接下来我们会开始讲解 Svelte 的官方脚手架 SvelteKit，如果你有其他全栈框架比如 Next.js 的学习经验，这块内容相信你会感到很相似，学习进度很快。同理，如果你没有接触过其他框架，SvelteKit 的学习经验也会帮助你使用其他全栈脚手架。

那么就让我们进入第三阶段 —— SvelteKit 的学习中吧！