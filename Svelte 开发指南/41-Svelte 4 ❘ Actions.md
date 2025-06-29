> 推荐学习指数：⭐️️，了解即可

## 1\. 前言

本篇我们讲解 Svelte 中的重要概念 —— Actions。虽然本身比较简单，但它极大的增强了 Svelte 的功能。简单来说，就是能玩出花儿……本篇我们会举出丰富的例子帮助大家理解 Actions 的作用。

那就让我们来看看怎么使用吧！

## 2\. Actions 介绍

**Actions 是元素级别的生命周期函数。** 也就是说，当它搭配 `use`指令用在 HTML 元素时，可设置该元素的生命周期函数。其用法如下：

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

举个最简单的例子：

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

函数的第一个参数为 `node`，表示引用的 DOM 节点。该函数会在元素挂载到 DOM 上的时候执行，相当于元素的 `onMount` 函数。

[浏览器效果如下：](https://svelte.dev/repl/0c5a2b9d42ed46859f942e473bdd9be0?version=4.2.18 "https://svelte.dev/repl/0c5a2b9d42ed46859f942e473bdd9be0?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e26961a98bd54f9cb60a00e715b729fc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3128&h=814&s=182036&e=png&b=292929)

我们举个更复杂的例子，包含挂载、更新、销毁：

```xml
<script>
  let name = 'world';
  let checked = true;

  function log(node, params) {
    node.innerHTML = params
    return {
      update(newParams) {
        console.log(newParams)
        node.innerHTML = newParams
      },
      destroy() {
        console.log('destroy')
      }
    }
  }
</script>

<label>
  <input type="checkbox" bind:checked={checked} />
  show
</label>

{#if checked}
  <input bind:value={name} />
  <h1 use:log={name}>'loading'</h1>
{/if}
```

[浏览器效果如下：](https://svelte.dev/repl/8a044663ca744ac2ac25f0efa0edb2f7?version=4.2.18 "https://svelte.dev/repl/8a044663ca744ac2ac25f0efa0edb2f7?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92049603ebf1489ea2049d6b61e8ba57~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1394&h=782&s=111243&e=gif&f=41&b=212121)

当元素挂载的时候的时候，运行 `log()` 函数，打印 `"mounted"`，当元素销毁的时候，触发 `destroy()` 函数，打印 `"destroy"`。当自定义传入 Action 的 name 值发生改变的时候，触发 update 函数。

Actions 的用法就这么多，Svelte Tutorial 中介绍的也不多，但**实际上 Actions 是一个非常强大的功能**。我们举几个例子，大家可以在实际操作中体会。

## 3\. 用途 1：输入框聚焦

现在我们来实现这样一个效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e3f4b3bf3d54283a783b80f48566018~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=630&h=105&s=12153&e=gif&f=17&b=313131)

效果本身是比较简单的，实现的重点在于**当点击编辑按钮的时候，输入框要聚焦，方便用户输入**。

第一种实现方式如下：

```xml
<script>
  import { tick } from 'svelte';

  let input;
  let name = 'YaYu';
  let editing = false;

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

<button on:click={toggleEdit} style="display: inline">
  {editing ? '确定' : '编辑'}
</button>
```

在这段代码中，我们首先使用 `bind:this` 获取 `<input>`引用，然后在点击编辑的时候，调用 `.focus()`方法。

[浏览器效果如下：](https://svelte.dev/repl/9e2f5e53f7b34d6385e39de6afc75e03?version=4.2.18 "https://svelte.dev/repl/9e2f5e53f7b34d6385e39de6afc75e03?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/705a8c1a33ac43af991e3a3fe16e4143~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1236&h=608&s=60983&e=gif&f=24&b=272727)

第二种实现方式如下：

```xml
<!-- App.svelte -->
<script>
  import Input from './Input.svelte';
  let name = 'YaYu';
  let editing = false;

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

<button on:click={toggleEdit} style="display: inline">
  {editing ? '确定' : '编辑'}
</button>

<!-- Input.svelte -->
<script>
  export let value;
  let input;

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

[浏览器效果如下](https://svelte.dev/repl/be46883f681a44e68ec14bf344bdfa16?version=4.2.18 "https://svelte.dev/repl/be46883f681a44e68ec14bf344bdfa16?version=4.2.18")（其实效果跟上面是一样的）：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1dbb7ed0f024b21977d4eacdb2479dd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1183&h=745&s=97419&e=gif&f=39&b=272727)

第三种方法是使用 Actions：

```javascript
<script>
  import { tick } from 'svelte';

  let input;
  let name = 'YaYu';
  let editing = false;

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

<button on:click={toggleEdit}>
  {editing ? '确定' : '编辑'}
</button>
```

[浏览器效果同上](https://svelte.dev/repl/9de235b2f51d45bdae2f0f7cb4fc0546?version=4.2.18 "https://svelte.dev/repl/9de235b2f51d45bdae2f0f7cb4fc0546?version=4.2.18")

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
    return {
      destroy() {
        element.removeEventListener('click', onClick);
      }
    }
  }
</script>

<button
  use:doubleClick
  on:doubleClick={() => console.log('doubleClick!')}
>doubleClick</button>
```

[浏览器效果如下：](https://svelte.dev/repl/fa522461330d45659a27bc2f187816de?version=4.2.18 "https://svelte.dev/repl/fa522461330d45659a27bc2f187816de?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/021aefc68f8c46e08c41a0a5b3ff51ad~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1236&h=608&s=51187&e=gif&f=19&b=222222)

这是 Actions 一个非常常见的例子。当元素加上 `use:doubleClick`指令的时候，元素会监听点击事件，并在双击的时候，发送自定义的 doubleClick 事件，然后元素可以使用 `on:doubleClick`监听该事件。

这也是 [Tutorial](https://learn.svelte.dev/tutorial/actions "https://learn.svelte.dev/tutorial/actions") 中介绍的常见的 4 种用法之一：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad59f711f480457b9d78a58b4d7279ce~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1514&h=640&s=86265&e=png&b=242424)

当然不止双击，各种手势操作都可以通过这种方式实现。

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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a3f96aea7b64d26ba397b0c08a13e94~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1184&h=684&s=79595&e=gif&f=27&b=23252f)

如果我们要在 Svelte 项目中使用：

```xml
<script>
  import tippy from 'tippy.js';
  import 'tippy.js/dist/tippy.css';
  import 'tippy.js/themes/material.css';

  let content = 'Hello!';

  function tooltip(node, options) {
    const tooltip = tippy(node, options);

    return {
      update(options) {
        tooltip.setProps(options);
      },
      destroy() {
        tooltip.destroy();
      }
    };
  }
</script>

<input bind:value={content} />

<button use:tooltip={{ content, theme: 'material' }}>
  Hover me
</button>
```

[浏览器效果如下：](https://learn.svelte.dev/tutorial/adding-parameters-to-actions "https://learn.svelte.dev/tutorial/adding-parameters-to-actions")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cbc738b026044f1aef0b42d241cf097~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=984&h=166&s=34791&e=gif&f=25&b=2d2d2d)

当然不止是 tippy，因为在函数中提供了元素的引用，可以搭配使用很多三方库，这极大的拓展了 Svelte 能够实现的功能。

我认为这个例子很好的展示了 [Tutorial](https://learn.svelte.dev/tutorial/actions "https://learn.svelte.dev/tutorial/actions") 中介绍的用法：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/314226e66bb444cf972b438ddfd669b6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1004&h=422&s=50082&e=png&b=242424)

## 6\. 用途 4：clickOutside 与 Modal

这个例子类似于双击事件，只不过是自定义的 clickOutside 事件：

```xml
<script>
  import { scale } from 'svelte/transition'

  let open = false

  function openModal() {
    open = true
  }

  function closeModal() {
    open = false
  }

  const clickOutside = (element) => {
    function handleClick(event) {
      if (element && !element.contains(event.target)) {
        const clickOutsideEvent = new CustomEvent('outside')
        element.dispatchEvent(clickOutsideEvent)
      }
    }

    document.addEventListener('click', handleClick, true)

    return {
      destroy() {
        document.removeEventListener('click', handleClick, true)
      },
    }
  }
</script>

{#if open}
  <div class="background">
    <div
      class="modal"
      use:clickOutside
      on:outside={closeModal}
      transition:scale
    >
      <h2>Modal</h2>
      <p>What's up?</p>
    </div>
  </div>
{/if}

<button on:click={openModal}>
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

[浏览器效果如下：](https://svelte.dev/repl/4613dd3be38b4abfb4cc4c468776422c?version=4.2.18 "https://svelte.dev/repl/4613dd3be38b4abfb4cc4c468776422c?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29fdf221a31d4b09af1afd6af54199d1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1474&h=690&s=242006&e=gif&f=35&b=282828)

## 7\. 用途 5：图片懒加载

图片懒加载就是当图片出现在视口内的时候，再开始加载图片。我们新建一个 `lazyLoad.js`：

```javascript
export function lazyLoadImage(node) {
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

  return {
    destroy() {
      observer.unobserve(node);
    },
  };
}
```

当需要使用懒加载的时候：

```xml
<script>
  import { lazyLoadImage } from './lazyLoad.js'
</script>

{#each [...Array(15).keys()] as _}
  <p>
    Lorem ipsum dolor sit amet consectetur adipisicing elit. Aspernatur obcaecati ex totam laboriosam, culpa ipsa quis nostrum odio dolore aut eos numquam ratione quam maiores voluptates quas eius labore error?
  </p>
{/each}

<img use:lazyLoadImage width="400px" data-src="https://plus.unsplash.com/premium_photo-1720706435477-bb1d79c2224c?q=80&w=4480&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D" alt="">
```

在图片元素上加上 `use:lazyLoadImage`，并在 `data-src`属性上加上该图片的实际地址。Actions 就会在该元素进入视口的时候，根据 `data-src`属性值加载对应图片。

[浏览器效果如下：](https://svelte.dev/repl/f12988de576b4bf9b541a2a59eb838f6?version=3.23.2 "https://svelte.dev/repl/f12988de576b4bf9b541a2a59eb838f6?version=3.23.2")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b56726ec5be49fca8151b9f9fc066c6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1650&h=788&s=1812794&e=gif&f=36&b=282828)

这个例子展示了 [Tutorial](https://learn.svelte.dev/tutorial/actions "https://learn.svelte.dev/tutorial/actions") 中介绍的懒加载用法：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8c51f420bd245939039fe406d987094~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1430&h=432&s=53798&e=png&b=242424)

## 8\. 注意 1：顺序问题

使用 Actions 的时候要注意，Bindings 和 Actions 的顺序可能会影响最终的结果。我们举个例子：

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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31ff2a3ecf464987a44e00310c7bea24~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1454&h=512&s=43436&e=gif&f=28&b=282828)

我们的第一个输入框使用的是 `<input bind:value={value1} use:uppercase />`，当输入小写字母的时候，虽然输入框显示没有问题，但实际 value1 的值的最后一个字母是小写的。

这是因为按照顺序，我们先进行的双向绑定，然后再使用 Actions 修改的值。当输入第一个小写字符`a`的时候，因为进行了双向绑定，所以 value1 的值显示为 `a`，但输入框有 Action，所以显示为大写。当输入第二个小写字符 `s`的时候，因为输入框之前的值 `A`，加上刚输入的 `s`，双向绑定后，修改 value 的值为 `As`，输入框因为有 Action，继续显示为大写 `AS`。以此类推……所以实际值的最后一个字符总是小写。

第二个输入框使用的是 `<input use:uppercase bind:value={value2} />`，按照顺序，我们先使用 Actions 修改输入框的值，再进行双向绑定，所以字符始终为大写。

## 9\. 最后

恭喜您完成了第四篇内容的学习！Actions 是一个非常强大的功能，擅用 Actions 可以让我们的代码功能拆分更清晰，复用更方便。