> 推荐学习指数：⭐️️

## 1\. 前言

本篇我们讲解 Svelte 中的重要概念 —— Actions。虽然本身比较简单，但它极大的增强了 Svelte 的功能。简单来说，就是能玩出花儿……本篇我们会举出丰富的例子帮助大家理解 Actions 的作用。

那就让我们来看看怎么使用吧！

## 2\. Actions 介绍

**Actions 是元素级别的生命周期函数。** 也就是说，当它搭配 `use`指令用在 HTML 元素时，可设置该元素的生命周期函数。

在 Svelte 4 中，其用法如下：

```xml
<script>
  export let bar;

  function foo(node, bar) {
    // node 表示元素的引用，bar 是自定义传入的值
    // 挂载 DOM 的时候执行

    return {
      update(bar) {
        // 当 bar 的值改变的时候执行
      },

      destroy() {
        // 从 DOM 中移除的时候执行
      }
    };
  }
</script>

<div use:foo={bar} />
```

在 Svelte 5 中，原本的写法依然可用，但建议改为使用 `$effect`：

```xml
<script>
  function myaction(node) {
    // node 表示元素的引用，此时节点已挂载到 DOM 中

    $effect(() => {
      // ...
      return () => {
        // 从 DOM 中移除的时候执行
      };
    });
  }
</script>

<div use:foo>...</div>
```

注意：如果你要获取传入值的更新，这种方式是不可以的：

```xml
<script>
  function myaction(node, bar) {
    $effect(() => {
      console.log(bar)
    });
  }
</script>

<div use:foo={bar}>...</div>
```

你应该改为函数的形式：

```xml
<script>
  function myaction(node, fn) {
    $effect(() => {
      console.log(fn())
    });
  }
</script>

<div use:foo={() => (bar)}>...</div>
```

我们先从最简单的使用例子开始说起：

```xml
<script>
  let name = 'world';

  function log(node) {
    console.log(node)
  }
</script>

<h1 use:log>Hello {name}!</h1>
```

我们使用 `use`指令调用了自定义的 log 函数，这个 log 函数在 Svelte 中就被称为“Action”。

该函数会在元素挂载到 DOM 上的时候执行，相当于元素的 `onMount` 函数。函数的第一个参数为 `node`，表示引用的 DOM 节点。

[浏览器效果如下：](https://svelte.dev/playground/0c06a722cc6343479cb42a3c28222a18?version=5.1.12 "https://svelte.dev/playground/0c06a722cc6343479cb42a3c28222a18?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53ea76ea90184f0db0c72ae85814f786~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3128&h=814&s=121445&e=png&b=292929)

我们举个更复杂的例子，包含挂载、更新、销毁：

```xml
<script>
  let name = $state('world');
  let checked = $state(true);

  function log(node, fn) {
    console.log('mounted')
    $effect(() => {
      console.log('setup')
      node.innerHTML = fn()
      return () => {
        console.log('teardown')
      };
    });
  }
</script>

<label>
  <input type="checkbox" bind:checked={checked} />
  show
</label>

{#if checked}
  <figure>
    <input bind:value={name} />
    <h1 use:log={() => (name)}>'loading'</h1>
  </figure>
{/if}
```

[浏览器效果如下：](https://svelte.dev/playground/a912b9f030354ac6b997d549e1770037?version=5.1.12 "https://svelte.dev/playground/a912b9f030354ac6b997d549e1770037?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f53fd9208cbc4832ac2a226a2f2ca98c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1108&h=726&s=85421&e=gif&f=25&b=1d1e22)

当元素挂载的时候的时候，运行 `log()` 函数，打印 `mounted`和 `setup`，当传入值更新的时候，打印 `teardown`和 `setup`，当元素销毁的时候，打印 `teardown`。

Actions 的用法就这么多，Svelte Tutorial 中介绍的也不多，但**实际上 Actions 是一个非常强大的功能**。我们举几个例子，大家可以在实际操作中体会。

## 3\. 用途 1：输入框聚焦

现在我们来实现这样一个效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39beec304cc144e3965ba5aeb1037a01~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=630&h=105&s=96310&e=gif&f=17&b=303030)

效果本身是比较简单的，实现的重点在于**当点击编辑按钮的时候，输入框要聚焦，方便用户输入**。

第一种实现方式如下：

```xml
<script>
  import { tick } from 'svelte';

  let input = $state();
  let name = $state('YaYu');
  let editing = $state(false);

  async function toggleEdit() {
    editing = !editing
    if (editing) {
      await tick()
      input.focus();
    }
  }
</script>

昵称：
{#if editing}
  <label>
    <input type="text" bind:this={input} bind:value={name}>
  </label>

{:else}
  {name}
{/if}

<button onclick={toggleEdit} style="display: inline">
  {editing ? '确定' : '编辑'}
</button>
```

在这段代码中，我们首先使用 `bind:this` 获取 `<input>`引用，然后在点击编辑的时候，调用 `.focus()`方法。

[浏览器效果如下：](https://svelte.dev/playground/232e637001b144bdb04033e18c368bc9?version=5.1.12 "https://svelte.dev/playground/232e637001b144bdb04033e18c368bc9?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f51bbe53c8ce4e6e98b6ab0e0e17e7cb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1236&h=608&s=775060&e=gif&f=24&b=272727)

第二种实现方式如下：

```xml
<!-- App.svelte -->
<script>
  import Input from './Input.svelte';
  let name = $state('YaYu');
  let editing = $state(false);

  function toggleEdit() {
    editing = !editing
  }
</script>

昵称：
{#if editing}
  <Input bind:value={name} />
{:else}
  {name}
{/if}

<button onclick={toggleEdit} style="display: inline">
  {editing ? '确定' : '编辑'}
</button>

<!-- Input.svelte -->
<script>
  let { value = $bindable() } = $props();

  let input = $state();

  import { onMount } from 'svelte';
  onMount(() => {
    input.focus();
  });
</script>

<label>
  <input type="text" bind:this={input} bind:value>
</label>
```

在这段代码中，我们将输入框移到一个单独的组件中，在挂载的时候，将输入框聚焦。

[浏览器效果如下](https://svelte.dev/playground/43914e8b561b43c78d853cec6bd36664?version=5.1.12 "https://svelte.dev/playground/43914e8b561b43c78d853cec6bd36664?version=5.1.12")（其实效果跟上面是一样的）：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdb5a97868af40cea0b000c9e9e625cf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1183&h=745&s=97419&e=gif&f=39&b=272727)

第三种方法是使用 Actions：

```javascript
<script>
  import { tick } from 'svelte';

  let input = $state();
  let name = $state('YaYu');
  let editing = $state(false);

  function toggleEdit() {
    editing = !editing
  }

  function focusOnMount(node) {
    node.focus();
  }
</script>

昵称：
{#if editing}
  <label>
    <input use:focusOnMount type="text" bind:this={input} bind:value={name}>
  </label>

{:else}
  {name}
{/if}

<button onclick={toggleEdit}>
  {editing ? '确定' : '编辑'}
</button>
```

[浏览器效果同上](https://svelte.dev/playground/ed1f4e8be40c475ea4d4a3aa873c75d5?version=5.1.12 "https://svelte.dev/playground/ed1f4e8be40c475ea4d4a3aa873c75d5?version=5.1.12")

这三种方法中，哪种更好一点呢？

理论上，Actions 这个方法更好，因为复用方便。如果还有其他输入框，直接添加 `use:focusOnMount`即可。

借这个简单的例子是为了说明 Actions 可以解决 DOM 操作的复用问题。我们再进阶看一个例子。

## 4\. 用途 2：双击事件

现在我们来实现一个双击效果，即当双击按钮的时候，打印一个值：

```xml
<script>
  function doubleClick(element) {
    let count = 0;
    let timeoutId;

    $effect(() => {
      function onClick() {
        if (++count === 2) {
          element.dispatchEvent(new CustomEvent('doubleClick'));
        }

        timeoutId && clearTimeout(timeoutId);
        timeoutId = setTimeout(() => {
          timeoutId = null;
          count = 0;
        }, 800);
      }

      element.addEventListener('click', onClick);

      return () => {
         element.removeEventListener('click', onClick);
      }
    });
  }
</script>

<button
  use:doubleClick
  ondoubleClick={() => console.log('doubleClick!')}
>doubleClick</button>
```

[浏览器效果如下：](https://svelte.dev/playground/dce940ea61bc48bca389e240c28cedad?version=5.1.12 "https://svelte.dev/playground/dce940ea61bc48bca389e240c28cedad?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1c3f2a70fe942c899b7b027c968b71a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1236&h=608&s=737613&e=gif&f=19&b=222222)

这是 Actions 一个非常常见的例子。当元素加上 `use:doubleClick`指令的时候，元素会监听点击事件，并在双击的时候，发送自定义的 doubleClick 事件，然后元素可以使用 `ondoubleClick`监听该事件。

这也是 [Tutorial](https://learn.svelte.dev/tutorial/actions "https://learn.svelte.dev/tutorial/actions") 中介绍的常见的 4 种用法之一：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d662c27c592c49c2b991139990bf2c9f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1514&h=640&s=75085&e=png&b=242424)

当然不止双击，各种手势操作都可以通过这种方式实现：

```xml
<script>
  import { on } from 'svelte/events';

  function gestures(node) {
    $effect(() => {
      // ...
      node.dispatchEvent(new CustomEvent('swipeleft'));

      // ...
      node.dispatchEvent(new CustomEvent('swiperight'));
    });
  }
</script>

<div
  use:gestures
  onswipeleft={next}
  onswiperight={prev}
>...</div>
```

## 5\. 用途 3：tooltips

tooltips 是 Tutorial 中介绍的例子。我们再列举一遍当做复习了。

有的时候，为了开发方便，你需要引入一些三方库，Actions 就可以搭配三方库一起使用。就比如实现 tooltips 的 [tippy](https://atomiks.github.io/tippyjs/ "https://atomiks.github.io/tippyjs/")。它的基本用法如下：

```javascript
<button id="myButton">My Button</button>;

tippy("#myButton", {
  content: "I'm a Tippy tooltip!",
});
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a7456e181b743c684d3595bf6fa5fdd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1184&h=684&s=916771&e=gif&f=27&b=23252f)

如果我们要在 Svelte 项目中使用：

```xml
<script>
  import tippy from 'tippy.js';

  let content = $state('Hello!');

  function tooltip(node, fn) {
    $effect(() => {
      const tooltip = tippy(node, fn());

      return tooltip.destroy;
    });
  }
</script>

<input bind:value={content} />

<button use:tooltip={() => ({ content })}>
  Hover me
</button>

<style>
  :global {
    [data-tippy-root] {
      --bg: #666;
      background-color: var(--bg);
      color: white;
      border-radius: 0.2rem;
      padding: 0.2rem 0.6rem;
      filter: drop-shadow(1px 1px 3px rgb(0 0 0 / 0.1));

      * {
        transition: none;
      }
    }

    [data-tippy-root]::before {
      --size: 0.4rem;
      content: '';
      position: absolute;
      left: calc(50% - var(--size));
      top: calc(-2 * var(--size) + 1px);
      border: var(--size) solid transparent;
      border-bottom-color: var(--bg);
    }
  }
</style>
```

[浏览器效果如下：](https://learn.svelte.dev/tutorial/adding-parameters-to-actions "https://learn.svelte.dev/tutorial/adding-parameters-to-actions")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c24e8cc24c5444eca2e8ec4a470e0a51~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=984&h=166&s=173870&e=gif&f=25&b=2d2d2d)

当然不止是 tippy，因为在函数中提供了元素的引用，可以搭配使用很多三方库，这极大的拓展了 Svelte 能够实现的功能。

我认为这个例子很好的展示了 [Tutorial](https://learn.svelte.dev/tutorial/actions "https://learn.svelte.dev/tutorial/actions") 中介绍的用法：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10d17ed5bc2449fba6a5a2a33563e15e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1004&h=422&s=47528&e=png&b=242424)

## 6\. 用途 4：clickOutside 与 Modal

这个例子类似于双击事件，只不过是自定义的 clickOutside 事件：

```xml
<script>
  import { scale } from 'svelte/transition'

  let open = $state(false)

  function openModal() {
    open = true
  }

  function closeModal() {
    open = false
  }

  const clickOutside = (element) => {

    $effect(() => {
      function handleClick(event) {
        if (element && !element.contains(event.target)) {
          const clickOutsideEvent = new CustomEvent('outside')
          element.dispatchEvent(clickOutsideEvent)
        }
      }

      document.addEventListener('click', handleClick, true)

      return () => {
        document.removeEventListener('click', handleClick, true)
      }
    });
  }
</script>

{#if open}
  <div class="background">
    <div
      class="modal"
      use:clickOutside
      onoutside={closeModal}
      transition:scale
    >
      <h2>Modal</h2>

      <p>What's up?</p>

    </div>

  </div>

{/if}

<button onclick={openModal}>
  Open
</button>

<style>
  .background {
    position: absolute;
    inset: 0;
    display: grid;
    place-content: center;

    & .modal {
      width: 400px;
      min-height: 300px;
      padding: 2rem;
      background-color: hsl(200 10% 10% / 80%);
      backdrop-filter: blur(20px);
      border: 1px solid hsl(200 10% 12%);
      border-radius: 8px;
      box-shadow: 1px 1px 10px hsl(0 0% 0% / 20%);

      & p {
        margin-top: 1rem;
      }
    }
  }
</style>
```

我们在 modal 元素上添加了 `use:clickOutside`，在 `clickOutside` Action 函数中我们判断 Modal 是否包含点击的元素，如果没有，那说明点击了 Modal 之外的位置，于是发送自定义的 `outside`事件。然后我们在 modal 元素上监听 outside 事件，关闭 modal。

[浏览器效果如下：](https://svelte.dev/playground/43272a97a7ff44b0a56579a82b539190?version=5.1.12 "https://svelte.dev/playground/43272a97a7ff44b0a56579a82b539190?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84ac364dd08e46d59ecb8d943f62e36d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1474&h=690&s=242006&e=gif&f=35&b=282828)

## 7\. 用途 5：图片懒加载

图片懒加载就是当图片出现在视口内的时候，再开始加载图片。我们新建一个 `lazyLoad.svelte.js`：

```javascript
export function lazyLoadImage(node) {
  $effect(() => {
    const src = node.getAttribute("data-src");
    let intersecting = false;

    const handleIntersection = (entries) => {
      intersecting = entries[0].isIntersecting;
      if (entries[0].intersectionRatio > 0) {
        node.src = src;
      }
      if (intersecting) {
        observer.unobserve(node);
      }
    };

    const observer = new IntersectionObserver(handleIntersection);

    observer.observe(node);

    return () => {
      observer.unobserve(node);
    };
  });
}
```

当需要使用懒加载的时候：

```xml
<script>
  import { lazyLoadImage } from './lazyLoad.svelte.js'
</script>

{#each [...Array(15).keys()] as _}
  <p>
    Lorem ipsum dolor sit amet consectetur adipisicing elit. Aspernatur obcaecati ex totam laboriosam, culpa ipsa quis nostrum odio dolore aut eos numquam ratione quam maiores voluptates quas eius labore error?
  </p>

{/each}

<img use:lazyLoadImage width="400px" data-src="https://plus.unsplash.com/premium_photo-1720706435477-bb1d79c2224c?q=80&w=4480&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D" alt="">
```

在图片元素上加上 `use:lazyLoadImage`，并在 `data-src`属性上加上该图片的实际地址。Actions 就会在该元素进入视口的时候，根据 `data-src`属性值加载对应图片。

[浏览器效果如下：](https://svelte.dev/playground/edb85ec58d9045b8aa7a219838234842?version=5.1.12 "https://svelte.dev/playground/edb85ec58d9045b8aa7a219838234842?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58594948c16144e8a6aad15ed643d91e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1650&h=788&s=1812794&e=gif&f=36&b=282828)

这个例子展示了 [Tutorial](https://learn.svelte.dev/tutorial/actions "https://learn.svelte.dev/tutorial/actions") 中介绍的懒加载用法：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4bf3f3e7365435493fc59579bb2d037~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1430&h=432&s=47880&e=png&b=242424)

## 8\. 注意：关于 Svelte 4 中的 Actions 顺序问题

在 Svelte 4 中，使用 Actions 的时候要注意，Bindings 和 Actions 的顺序可能会影响最终的结果。我们举个例子：

```xml
<script>
  let value1 = '';
  let value2 = '';

  function uppercase(element) {
    function onInput(event) {
      element.value = element.value.toUpperCase();
    }
    element.addEventListener('input', onInput);
    return {
      destroy() {
        element.removeEventListener('input', onInput);
      }
    }
  }
</script>

<input
  bind:value={value1}
  use:uppercase
/>
<input
  use:uppercase
  bind:value={value2} />

<div>{value1}</div>

<div>{value2}</div>
```

[浏览器效果如下：](https://svelte.dev/repl/de54ff5709d9457bb54f7166efe52bdd?version=4.2.18 "https://svelte.dev/repl/de54ff5709d9457bb54f7166efe52bdd?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/957d0c568e9f49e2ae75dc9d7a211dd4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1454&h=512&s=720258&e=gif&f=28&b=282828)

我们的第一个输入框使用的是 `<input bind:value={value1} use:uppercase />`，当输入小写字母的时候，虽然输入框显示没有问题，但实际 value1 的值的最后一个字母是小写的。

这是因为按照顺序，我们先进行的双向绑定，然后再使用 Actions 修改的值。当输入第一个小写字符`a`的时候，因为进行了双向绑定，所以 value1 的值显示为 `a`，但输入框有 Action，所以显示为大写。当输入第二个小写字符 `s`的时候，因为输入框之前的值 `A`，加上刚输入的 `s`，双向绑定后，修改 value 的值为 `As`，输入框因为有 Action，继续显示为大写 `AS`。以此类推……所以实际值的最后一个字符总是小写。

第二个输入框使用的是 `<input use:uppercase bind:value={value2} />`，按照顺序，我们先使用 Actions 修改输入框的值，再进行双向绑定，所以字符始终为大写。

但在 Svelte 5 中，使用 $effect 的方式则顺序不会影响结果，都是最后一个字符小写：

```xml
<script>
  let value1 = $state('');
  let value2 = $state('');

  function uppercase(element) {
    $effect(() => {
      function onInput(event) {
        element.value = element.value.toUpperCase();
      }
      element.addEventListener('input', onInput);
      return () => {
        element.removeEventListener('input', onInput);
      }
    });
  }
</script>

<input bind:value={value1} use:uppercase />
<input use:uppercase bind:value={value2} />

<div>{value1}</div>
<div>{value2}</div>
```

[浏览器效果如下：](https://svelte.dev/playground/6ba4231f262742c8bd492706767a9d96?version=5.1.12 "https://svelte.dev/playground/6ba4231f262742c8bd492706767a9d96?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/360be67748f34aa08f4ac44be0f8422f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1216&h=772&s=66125&e=gif&f=33&b=1e1f23)

## 9\. 最后

恭喜您完成了本篇内容的学习！Actions 是一个非常强大的功能，擅用 Actions 可以让我们的代码功能拆分更清晰，复用更方便。