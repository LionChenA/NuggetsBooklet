> 推荐学习指数：⭐️⭐

## 1\. 前言

**本篇针对使用过 Svelte 4 的同学**，讲解 Svelte 4 中存在的问题，以及使用符文带来的好处。如果初学就是 Svelte 5，本篇简单了解即可。

## 2\. Svelte 4 中的问题

### 2.1. 问题 1：状态与派生状态不一致

这是一段 Svelte 4 中的代码：

```xml
<script>
  let count = 0;
  $: double = count * 2

  function increment() {
    count += 1
  }
</script>

<button on:click={increment}>Click</button>

{count} * 2 = {double}
```

[浏览器效果如下：](https://svelte.dev/repl/11881df2378146da9f1d2eef1db3dad8?version=4.2.18 "https://svelte.dev/repl/11881df2378146da9f1d2eef1db3dad8?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1da1f0501b6f43b49d3b94443ded5828~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1453&h=495&s=644615&e=gif&f=23&b=262626)

看起来很正常，但其实有一个小问题。

我们在 `increment()` 函数中打印一下 `count` 和 `double` 的值：

```diff
<script>
  let count = 0;
  $: double = count * 2

  function increment() {
    count += 1
+   console.log({count, double})
  }
</script>

<button on:click={increment}>Click</button>

{count} * 2 = {double}
```

[浏览器效果如下：](https://svelte.dev/repl/4f705d1fa36e4090bdb9c22d0da0b0aa?version=4.2.18 "https://svelte.dev/repl/4f705d1fa36e4090bdb9c22d0da0b0aa?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/066afd18646640e78f3f1863067c4d8c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1466&h=541&s=838734&e=gif&f=25&b=212121)

当第一次点击的时候，count 的值是 1，double 的值却是 0。第二次点击的时候，count 的值是 2，double 的值却是 2。你会发现，count 和 double 的值并不同步。

这是为什么呢？

这是因为 Svelte 为了避免重复计算，会确保没有其他值更改的时候，才会重新计算 double，所以它会拖到快要进行 DOM 更新时再重新计算，所以 double 变更的时机要晚于 console.log 执行的时机。

如果你想要获取同步的值，你可以这样写：

```diff
<script>
  let count = 0;
  $: double = count * 2

  function increment() {
    count += 1
+   requestAnimationFrame(() => {
+      console.log({count, double})
+   })
  }
</script>

<button on:click={increment}>Click</button>

{count} * 2 = {double}
```

[浏览器效果如下：](https://svelte.dev/repl/c0c8da888f1841788c9f5b8c2fe7c633?version=4.2.18 "https://svelte.dev/repl/c0c8da888f1841788c9f5b8c2fe7c633?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c936b223205e4dc0841c420cc0edfa74~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1472&h=594&s=876622&e=gif&f=23&b=212121)

### 2.2. 问题 2：依赖追踪只发生在编译时

第二个问题是依赖追踪只发生在编译时。还是上节这个例子，我们用 `doubleCount` 函数优化下响应式声明：

```xml
<script>
  let count = 0;
  $: double = doubleCount()

  function doubleCount() {
    return count * 2
  }

  function increment() {
    count += 1
  }
</script>

<button on:click={increment}>Click</button>

{count} * 2 = {double}
```

如果这样写，double 是不会响应式更新的。[浏览器效果如下：](https://svelte.dev/repl/55461f0b09b24b18ad2cd130bd5bc80b?version=4.2.18 "https://svelte.dev/repl/55461f0b09b24b18ad2cd130bd5bc80b?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8775c119be64a198bc8fc12f85f3262~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1472&h=594&s=632092&e=gif&f=19&b=202020)

这是因为编译器无法看到依赖关系，所以 double 并不会响应式更新。如果你想要更新，你需要这样做：

```diff
<script>
  let count = 0;
-	$: double = doubleCount()
+	$: double = doubleCount(count)

  +function doubleCount() {
    return count * 2
  }

  function increment() {
    count += 1
  }
</script>

<button on:click={increment}>Click</button>

{count} * 2 = {double}
```

通过在参数中传入 count，让 Svelte 在编译的时候知道 double 依赖了 count，才会让 double 在 count 变化的时候重新计算。

### 2.3. 问题 3：学习成本高

还是这个计数器：

```xml
<script>
  let count = 0;

  $: double = count * 2

  function increment() {
    count += 1
  }
</script>

<button on:click={increment}>Click</button>

{count} * 2 = {double}
```

假设有很多计数器，我们创建一个函数将代码复用：

```xml
<script>
  function createCounter() {
    let count = 0;

    $: double = count * 2

    function increment() {
      count += 1
    }
  }
</script>

<button on:click={increment}>Click</button>

{count} * 2 = {double}
```

因为 `let` 没有声明在顶层，所以响应式会失效，`$: double` 也变成了普通的语法，没有什么作用。

为了让 count 值变得响应式，我们需要借助 Store：

```xml
<script>
  import { writable } from "svelte/store";

  function createCounter() {
    const { subscribe, update } = writable(0)
    function increment() {
      update((count) => count+1)
    }
    return {
      subscribe,
      increment
    }
  }

  const counter = createCounter()
</script>

<button on:click={counter.increment}>Click</button>

{$counter}
```

[浏览器效果如下：](https://svelte.dev/repl/b21d11c8598f484e81cbb8b368d4fe81?version=4.2.18 "https://svelte.dev/repl/b21d11c8598f484e81cbb8b368d4fe81?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7576b3d41d0c4d65889e542c960e1d56~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1266&h=547&s=636318&e=gif&f=20&b=272727)

这样做本身没有什么问题，但是对于初学者而言，仅仅是为了解决一个普通的代码复用问题，却要去用复杂的 Store，还要理解 Store contract 以及掌握 `$xxx` 这种自动订阅语法。这会提高初学者的理解和学习成本。

### 2.4. 问题 4：重复计算

这是一个 Bindings 章节用到的例子：

```xml
<script>
  let todos = [
    { done: false, text: 'finish Svelte tutorial' },
    { done: false, text: 'build an app' },
    { done: false, text: 'world domination' }
  ];

  function remaining(todos) {
  console.log('recalculating')
  return todos.filter((t) => !t.done).length;
  }
</script>

<ul class="todos">
  {#each todos as todo}
  <li class:done={todo.done}>
    <input
      type="checkbox"
      bind:checked={todo.done}
      />

      <input
        type="text"
        placeholder="What needs to be done?"
        bind:value={todo.text}
        />
      </li>

  {/each}
</ul>

<p>未完成：{remaining(todos)}</p>
```

[浏览器效果如下：](https://svelte.dev/repl/b5fe143c887a43a89fe2c6480e0537a6?version=4.2.18 "https://svelte.dev/repl/b5fe143c887a43a89fe2c6480e0537a6?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57f9c7ff7d684d1aba7ab6ae813aeebd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1321&h=781&s=2113228&e=gif&f=41&b=232323)

你会发现，当我们修改任务的完成状态时，`remaining(todos)` 重新计算，这是没有问题的，但当我们修改任务的内容时，也会导致 `remaining(todos)` 重新计算，但这是没有必要的。如果列表很长，这会造成性能问题。难道就不能避免这些不必要的计算吗？

## 3\. Runes

Runes 解决了这些问题！

Runes，中文译文“符文”，一个听起来很帅，但又会让人感到有些莫名其妙的名词。想想游戏中的符文作用，往往是有着特殊作用的一个字母或者标记：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e0c5c8d83e946a5b3939dba52fb370e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1280&h=720&s=103815&e=jpg&b=131212)

比如在巫师 3 中，通过给武器嵌入符文石，可以增强武器力量，比如嵌入火焰符文会额外造成火焰伤害。

Svelte 5 中的符文也差不多，也是通过一些字符实现特殊作用，只不过游戏中的符文千奇百怪，影响的是角色属性，Svelte 中的符文是 `$`开头的字符，影响的是 Svelte 编译器。

简单来说，符文就是影响 Svelte 编译器的符号。其实 Svelte 4 使用的 `let`、`=`、`export`、`$:` 就是一种“符文”，但 Svelte 5 新增的符文将使用函数语法替代它们并实现更多的功能。

**当使用 Svelte 的时候，符文就是语言的一部分，无须导入（会编译真的可以为所欲为呀！）。**

### 3.1. $state

首先介绍的是状态符文，这是 Svelte 4 的代码：

```xml
<script>
  let count = 0;

  function increment() {
    count += 1;
  }
</script>

<button onclick={increment}>
  clicks: {count}
</button>

```

这是 Svelte 5 的代码：

```xml
<script>
  let count = $state(0);

  function increment() {
    count += 1;
  }
</script>

<button onclick={increment}>
  clicks: {count}
</button>
```

其实就是改了响应式变量的声明方式，之前是 `let count = 0`，现在是 `let count = $state(0)`。猛一看，代码变得繁琐了一点，但使用状态符文会带来很多好处。

就比如响应式将不限制于在组件顶层进行 `let` 声明，在应用的任何位置都可以（在本篇的“好处 1：通用响应式” 章节我们会举例解释）。而且，`$state` 也可以用于普通对象和数组：

```xml
<script>
  let numbers = $state([1, 2, 3]);
</script>

<button onclick={() => numbers.push(numbers.length + 1)}>
  push
</button>

<button onclick={() => numbers.pop()}> pop </button>

<p>
  {numbers.join(' + ') || 0}
  =
  {numbers.reduce((a, b) => a + b, 0)}
</p>
```

### 3.2. $derived

之前的响应式声明 `$：`改为使用 `$derived` 符文。使用之前：

```xml
<script>
  let count = 0;
  $: double = count * 2

  function increment() {
    count += 1
  }
</script>

<button on:click={increment}>Click</button>

{count} * 2 = {double}
```

使用之后：

```xml
<script>
  let count = $state(0);
  let doubled = $derived(count * 2);

  function increment() {
    count += 1;
  }
</script>

<button onclick={increment}>
  clicks: {count}
</button>

<p>{count} * 2 = {doubled}</p>
```

使用 `$derived` 的好处在于，doubled 的值始终是新的，这就解决了问题 1：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23e2edbe55994aa3b036ad6399703bbd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1255&h=523&s=616237&e=gif&f=19&b=222222)

而且即便你进行重构：

```xml
<script>
  let count = $state(0);
  let doubled = $derived(doubleCount());

  function doubleCount() {
    return count * 2;
  }

  function increment() {
    count += 1;
    console.log({count, doubled})
  }
</script>

<button onclick={increment}>
  clicks: {count}
</button>

<p>{count} * 2 = {doubled}</p>
```

这样也是可以生效的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c2c02423b0e42d3be5397f91fcd4d25~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1255&h=523&s=731974&e=gif&f=22&b=222222)

---

**使用符文，依赖项将在运行时进行跟踪**，这就解决了问题 2。

### 3.3. 好处 1：通用响应式（Universal reactivity ）

使用符文带来的一大好处就是通用响应式，意味着你可以在应用程序的任何地方创建响应状态，而不仅仅是在组件的顶层。比如这是一个普通的计数器：

```xml
<script>
  let count = $state(0);
</script>

<button onclick={() => {
  count += 1
}}>
  clicks: {count}
</button>
```

你可以将逻辑封装到一个函数中，以便它可以在多个地方使用：

```xml
<script>
  function createCounter() {
    let count = $state(0);
    return { count }
  }

  let counter = createCounter()
</script>

<button onclick={() => {
  counter.count += 1
}}>
  clicks: {counter.count}
</button>
```

这样写是不会有效果的，因为响应式并不能跨域函数的边界：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4aa4f1d95fd4ce280679bf153dd0c91~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=933&h=493&s=302210&e=gif&f=12&b=292929)

要是想生效，可以借助 `getter` 和 `setter` 语法：

```xml
<script>
  function createCounter() {
    let count = $state(0);
    return {
      get count() {
        return count
      },
      set count(value) {
        count = value
      }
    }
  }

  let counter = createCounter()
</script>

<button onclick={() => {
  counter.count += 1
}}>
  clicks: {counter.count}
</button>
```

[浏览器效果如下：](https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE11Q3UrFMAx-lRC82HD4czvXgfgY1ovZk0lxJx1tekBK31262enxKmm-v6QJZ7tQwP41IU9nwh6f1xU7lK-1PMKFFiHsMLjoTZkMwXi7yqgZYI5sxDoG42kSenGRhXzTQioowEICpgxBwU2QSah5aJ92zJNEz5UJ8FG5v_I_tA2p09zVLhyiy7REulLW4A05pHuzlay5lGNJ8qD-X6J5uD8O1jy8RxHH4Ngs1nyq1LSgxj31x-Nuz71V8Kg55-2fNnLoIV1xcjHfDUfs8OxOdrZ0wl58pPyWvwGR-cwynAEAAA== "https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE11Q3UrFMAx-lRC82HD4czvXgfgY1ovZk0lxJx1tekBK31262enxKmm-v6QJZ7tQwP41IU9nwh6f1xU7lK-1PMKFFiHsMLjoTZkMwXi7yqgZYI5sxDoG42kSenGRhXzTQioowEICpgxBwU2QSah5aJ92zJNEz5UJ8FG5v_I_tA2p09zVLhyiy7REulLW4A05pHuzlay5lGNJ8qD-X6J5uD8O1jy8RxHH4Ngs1nyq1LSgxj31x-Nuz71V8Kg55-2fNnLoIV1xcjHfDUfs8OxOdrZ0wl58pPyWvwGR-cwynAEAAA==")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a3e0d0b89814d2ab6e199809ec56489~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=933&h=493&s=431204&e=gif&f=19&b=282828)

尽管我们使用了符文，但我们依然可以把函数封装在 `.svelte` 组件之外，比如 `.svelte.js`：

```xml
<!-- App.svelte -->
<script>
  import { createCounter } from './counter.svelte.js'

  let counter = createCounter()
</script>

<button onclick={() => {
  counter.count += 1
}}>
  clicks: {counter.count}
</button>

<!-- counter.svelte.js -->
export function createCounter() {
  let count = $state(0);
  return {
    get count() {
      return count
    },
    set count(value) {
      count = value
    }
  }
}
```

[浏览器效果同上](https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE2VQ0U6EMBD8lU1jchAJp694kBg_w_qAvcX0hC1ptxcN6b-bUsBD-7Lp7Mzszk6i0z06Ub1OgtoBRSWex1EUgr_H-HFX7BlFIZzxVkXk5JTVIzeSAPQwGsswgbLYMr4YT4wWAnTWDHAojyohZbIpL-4gKQp7ZFh6UO_VWS7pdNyGSDq9e2ZDYEj1Wn3WU5ZD3cAUfVb_ucJ9DY-SQph3m8mugmnHCdE8GTaiEIM5607jWVRsPYZiO8J-8d97XNztLfBrzt95UqwN_Q2SdtyyQg13jlvG7CF_ih2L7C0lFsDHyluF8S2UGU9YKFJ1G_3a9h5vNOuwGV9EsQRJ4X_mt_ADbrVjNQQCAAA= "https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE2VQ0U6EMBD8lU1jchAJp694kBg_w_qAvcX0hC1ptxcN6b-bUsBD-7Lp7Mzszk6i0z06Ub1OgtoBRSWex1EUgr_H-HFX7BlFIZzxVkXk5JTVIzeSAPQwGsswgbLYMr4YT4wWAnTWDHAojyohZbIpL-4gKQp7ZFh6UO_VWS7pdNyGSDq9e2ZDYEj1Wn3WU5ZD3cAUfVb_ucJ9DY-SQph3m8mugmnHCdE8GTaiEIM5607jWVRsPYZiO8J-8d97XNztLfBrzt95UqwN_Q2SdtyyQg13jlvG7CF_ih2L7C0lFsDHyluF8S2UGU9YKFJ1G_3a9h5vNOuwGV9EsQRJ4X_mt_ADbrVjNQQCAAA=")

而在 Svelte 4 中，相同的功能我们需要借助 Store 来实现：

```xml
<script>
  import { writable } from "svelte/store";
  function createCounter() {
    return writable(0)
  }

  let count = createCounter()
</script>

<button onclick={() => {
  $count += 1
}}>
  clicks: {$count}
</button>
```

这就是问题 3 反映的问题，原本很简单的功能，我们却需要完全改变代码的编写方式，导入 `writeable`，理解 Store 相关的 API，知道背后的 subscribe 和 update 功能，还要使用 `$store`实现自动订阅。这增加了很多需要理解的内容。所以使用符文后，对于初学者而言，学习成本更少，心智负担更小。

### 3.4. 好处 2：细粒度响应式（Fine-grained reactivity）

使用符文的另一大好处是细粒度的响应式，我们以问题 4 为例，现在我们将原本的代码改为：

```diff
<script>
+  let todos = $state([
+    { done: false, text: 'finish Svelte tutorial' },
+    { done: false, text: 'build an app' },
+    { done: false, text: 'world domination' }
+	]);

-  let todos = [
-    { done: false, text: 'finish Svelte tutorial' },
-    { done: false, text: 'build an app' },
-    { done: false, text: 'world domination' }
-  ];

  function remaining(todos) {
    console.log('recalculating')
    return todos.filter((t) => !t.done).length;
  }
</script>

<ul class="todos">
  {#each todos as todo}
    <li class:done={todo.done}>
      <input
        type="checkbox"
        bind:checked={todo.done}
      />

      <input
        type="text"
        placeholder="What needs to be done?"
        bind:value={todo.text}
      />
    </li>

  {/each}
</ul>

<p>未完成：{remaining(todos)}</p>
```

[浏览器效果如下](https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE4WRv47UMBDGX2UwSLsrRbt9LgniGSgozld47cnGYnYc2ePjUJSegoIa8QQUPAGvw7W8AnLCwgHS0dnz55v5fjOp3hMmVV9Pis0ZVa1ejKOqlLwdyyfdIgmqSqWQoy2RJtnoR-k0AxAKSHAhQQvPkhjB7XWJA0zgAmMNvaGEFQjeSQ2b3rNPA7xcREGyhOgNbWCuHus6Zk8ODIMZx__VvgmRHLhw9mzEB97ArFnLze5Kc-nrM9sSh4hn49nzabsY2MG0ytrAKRDuKZy2m4jWkM1kxPNps1srIkqOvPre954E43YrO2g7eCL7stRuT8gnGa5Kw6y5OfxiprnJBJZMSq1Wi4ZWC8vpKRo7_MRp0vKY14kN-bWlLurtVFLLoLlbCwAaz2OWyw-gnK_Vyg5oXx_DnVa_U0fPrl4S6B5qXSoO3YrqEdkC-6HkSMbiEMhhbLV6NRgBRnTFBBxxOdTzf1a4NZQvZorgHwssww_kVzSHgmYBmWmFOHb3nz5_-_L-_t2H718_Tn9fc24OY6cqdQ7O9x6dqiVmnG_mH6t-pyDwAgAA "https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE4WRv47UMBDGX2UwSLsrRbt9LgniGSgozld47cnGYnYc2ePjUJSegoIa8QQUPAGvw7W8AnLCwgHS0dnz55v5fjOp3hMmVV9Pis0ZVa1ejKOqlLwdyyfdIgmqSqWQoy2RJtnoR-k0AxAKSHAhQQvPkhjB7XWJA0zgAmMNvaGEFQjeSQ2b3rNPA7xcREGyhOgNbWCuHus6Zk8ODIMZx__VvgmRHLhw9mzEB97ArFnLze5Kc-nrM9sSh4hn49nzabsY2MG0ytrAKRDuKZy2m4jWkM1kxPNps1srIkqOvPre954E43YrO2g7eCL7stRuT8gnGa5Kw6y5OfxiprnJBJZMSq1Wi4ZWC8vpKRo7_MRp0vKY14kN-bWlLurtVFLLoLlbCwAaz2OWyw-gnK_Vyg5oXx_DnVa_U0fPrl4S6B5qXSoO3YrqEdkC-6HkSMbiEMhhbLV6NRgBRnTFBBxxOdTzf1a4NZQvZorgHwssww_kVzSHgmYBmWmFOHb3nz5_-_L-_t2H718_Tn9fc24OY6cqdQ7O9x6dqiVmnG_mH6t-pyDwAgAA")，仅仅使用 `$state` 包裹一下 `todos`，你就会发现不再进行重复计算：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a111e395972140bc84dea9bb1a0ae711~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1191&h=624&s=2393084&e=gif&f=47&b=242424)

这就是使用符文的好处，将响应式状态的颗粒度进一步细化，只会在需要的时候进行更新。

> 注：你可能会好奇，这些特点到底是怎么实现的？简单来说，就是借助了 Signals 这一概念，我们会在原理篇中详细介绍。

## 4\. 最后

当然，Svelte5 的符文可不止这两个，本篇主要是为大家讲解符文所解决的问题，以及理解使用符文的必要性。符文改变了 Svelte 项目的开发方式，让你的应用程序更小、更快、代码更容易理解、更容易重构。下篇我们会详细介绍符文的完整用法。