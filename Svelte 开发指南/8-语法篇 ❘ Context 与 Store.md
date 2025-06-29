> 推荐学习指数：⭐️⭐

## 1\. 前言

本篇我们讲解 Svelte 中的重要概念 —— Context 和 Store。Context 负责数据传递，Store 是 Svelte 内置的状态管理方案。在实际开发中，两者可以独自使用，也可以搭配在一起使用。

那就让我们来看看怎么使用吧！

## 2\. Context

Svelte 的 Context 概念类似于 React 的 Context，可以让组件向它的子孙组件们发送数据。举个例子：

```xml
<!-- App.svelte -->
<script>
  import Parent1 from './Parent1.svelte';
  import Parent2 from './Parent2.svelte';
</script>

<Parent1 />
<Parent2 />

<!-- Parent1.svelte -->
<script>
  import Child from './Child.svelte';

  import { setContext } from 'svelte';

  setContext('color', 'red')
</script>

<Child />

<!-- Parent2.svelte -->
<script>
  import Child from './Child.svelte';

  import { setContext } from 'svelte';

  setContext('color', 'blue')
</script>

<Child />

<!-- Child.svelte -->
<script>
  import GrandChild from './GrandChild.svelte';
</script>

<GrandChild />

<!-- GrandChild.svelte -->
<script>
  import { getContext } from 'svelte';
  const color = getContext('color')
</script>

<div style="color: {color}">GrandChild {color}</div>
```

Context 核心就两个 API，父组件 `setContext()`，子孙组件 `getContext()`。在这段代码中，虽然`<GrandChild>` 组件都是`getContext('color')`，但因为我们在 `<Parent>` 组件 `setContext()` 设置的值不同，所以最终取的值也不同。

[浏览器效果如下](https://svelte.dev/repl/6ad5c97f65fd4feb9042c620aaf47811?version=4.2.18 "https://svelte.dev/repl/6ad5c97f65fd4feb9042c620aaf47811?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8210b1e482ad41a4a2fd3dd655b6760b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2476&h=572&s=116296&e=png&b=262626)

使用 Context 时要注意：

1. **与生命周期函数一样，setContext 和 getContext 要在组件初始化的时候调用**

```xml
// ❌ 这样写是会报错的
<button
  on:click={() => {
    setContext("color", "green");
  }}
>
  Set New Color
</button>
```

2. **Context 并不是响应式的，也就是说修改 context 的值，子组件并不会重新渲染**

```xml
// ❌ 这样写是不会生效的
<script>
  import Child from './Child.svelte';
  let color = 'red'
  import { setContext } from 'svelte';

  setContext('color', color)
</script>

<Child />

<button on:click={() => {
  color = 'green'
}}>Set New Color</button>
```

Context 既不能修改，又不是响应式，功能好像立刻就显得“鸡肋”了些。先不要着急，如果要实现修改和响应式，可以搭配接下来要讲的 Store 一起实现。

## 3\. Stores

### 3.1. Store 示例

如果要使用 Store，首先要进行声明：

```jsx
// stores.js
import { writable } from "svelte/store";

export const count = writable(0);
```

这里我们就创建了一个 `Store`，`writable(0)` 表示可读可写，初始值为 `0`。

如果有组件想要监听 Store 中值的变化，需要调用 Store 的 `subscribe()` 函数订阅 `count` 值的更新：

```xml
// App.svelte
<script>
  import { count } from './stores.js';
  import Incrementer from './Incrementer.svelte';
  import Decrementer from './Decrementer.svelte';
  import Resetter from './Resetter.svelte';

  let countValue;

  const unsubscribe = count.subscribe((value) => {
    countValue = value;
  });
</script>

<h1>The count is {countValue}</h1>

<Incrementer />
<Decrementer />
<Resetter />
```

如果要更改 Store 中的值，需要调用 Store 的 `update()` 方法：

```xml
<!-- Decrementer.svelte -->
<script>
  import { count } from './stores.js';

  function decrement() {
    count.update((n) => n - 1);
  }
</script>

<button on:click={decrement}> - </button>

<!-- Incrementer.svelte -->
<script>
  import { count } from './stores.js';

  function increment() {
    count.update((n) => n + 1);
  }
</script>

<button on:click={increment}> + </button>


<!-- Resetter.svelte -->
<script>
  import { count } from './stores.js';

  function reset() {
    count.set(0);
  }
</script>

<button on:click={reset}> reset </button>
```

[浏览器效果如下](https://svelte.dev/examples/writable-stores "https://svelte.dev/examples/writable-stores")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f150886f22e84afd9fc4a1cdb55893ab~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1370&h=708&s=1312789&e=gif&f=26&b=272727)

### 3.2. Store 实现原理

从 `subscribe()` 和 `update()` 函数名想必大家已经想到了，Store 本质就是一个发布订阅模式，就算不用 Svelte 提供的 Store，我们自己也可以写一个。

依然是用刚才的示例代码，我们将 `stores.js` 的代码改为：

```javascript
let subscribes = [];

export const count = {
  value: 0,
  set: function (newValue) {
    this.update(() => newValue);
  },
  update: function (fn) {
    this.value = fn(this.value);
    subscribes.forEach((fn) => {
      fn(this.value);
    });
  },
  subscribe: function (fn) {
    subscribes.push(fn);
    this.update(() => this.value);

    return function unsubscribe(fn) {
      subscribes.splice(subscribes.indexOf(fn), 1);
    };
  },
};
```

[浏览器效果不变](https://svelte.dev/repl/9d6eb40deb784100ae93fbcccd8792d8?version=4.2.18 "https://svelte.dev/repl/9d6eb40deb784100ae93fbcccd8792d8?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2372667f33943a8be3afe61f33ebc2a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1370&h=708&s=126948&e=gif&f=51&b=272727)

我们自己就实现了一个简易的 Store！

### 3.3. 自动订阅（Auto-subscriptions）

但想必你也发现了，使用 Store 有些麻烦……每次监听 Store 值的变化，还要调用订阅函数。举个例子，这是原本要写的代码：

```xml
<!-- App.svelte -->
<script>
   import Input from './Input.svelte'
   import Title from './Title.svelte'
</script>

<Input />
<Title />

<!-- Input.svelte -->
<script>
  import { count } from './stores';
</script>

<input on:input={(e) => { count.set(e.currentTarget.value) }} />

<!-- Title.svelte -->
<script>
  import { count } from './stores';
  import { onMount } from 'svelte';

  let count_value;

  onMount(() => {
    return count.subscribe(value => {
      count_value = value;
    })
  });
</script>

<h1>{count_value}</h1>

<!-- stores.js -->
import { writable } from 'svelte/store';

export const count = writable('');
```

在这段代码中，为了获取 Store 中的 `count` 值，需要先声明一个 `count_value`，然后在 `onMount` 的时候调用 Store 的 `subscribe()` 函数，修改 `count_value`。

[浏览器效果如下](https://svelte.dev/repl/d7c2c685bb114cafa4fe0d5239e8b3b2?version=4.2.18 "https://svelte.dev/repl/d7c2c685bb114cafa4fe0d5239e8b3b2?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e57c2fedb7fa49299575c1ec041823c7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1296&h=543&s=511039&e=gif&f=16&b=282828)

每次这样写着实有些麻烦，Svelte 提供了自动订阅语法，在 Store 变量前添加 `$` 前缀即可。还是刚才的例子，使用自动订阅语法后：

```diff
// Input.svelte
<script>
  import { count } from './stores';
</script>

-<input on:input={(e) => { count.set(e.currentTarget.value) }} />
+<input on:input={(e) => { $count = e.currentTarget.value }} />

// Title.svelte
<script>
  import { count } from './stores';
  import { onMount } from 'svelte';

- let count_value;

- onMount(() => {
-      return count.subscribe(value => {
-            count_value = value;
-      })
- });

</script>

-<h1>{count_value}</h1>

+<h1>{$count}</h1>
```

[浏览器效果不变](https://svelte.dev/repl/b90f26dddb3a4ef9af1e9c234e406a26?version=4.2.18 "https://svelte.dev/repl/b90f26dddb3a4ef9af1e9c234e406a26?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f24f81a4cd7245ac96928f4d5f85269b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1336&h=325&s=304810&e=gif&f=14&b=262626)

**使用 **`$`**，可以自动建立订阅、自动注销订阅，set 值的时候也更加简洁**。

### 3.4. Store contract

初学的时候，可能会以为 `$`就是专门用于为 Svelte 的 Store 实现自动订阅，但实际上，`$`也可以用于符合规范的 Store，这个规范就叫做 “Store contract”。

这个规范内容很简单：

```typescript
store = { subscribe: (subscription: (value: any) => void) => (() => void), set?: (value: any) => void }
```

简单来说就是 3 点要求：

1. Store 必须包含一个 `subscribe()` 函数，该函数接收一个订阅函数作为参数
2. `subscribe()` 函数必须返回一个取消订阅函数
3. 此条可选，包含一个 `set`方法

只要你的 Store 符合这个规范，那就可以使用 `$`自动订阅。

比如刚才的例子，我们把 `stores.js` 中的代码换成我们手写的 Store，同样可以使用 `$`。[浏览器效果如下](https://svelte.dev/repl/d809e6a0c8b1448eb3df1726c73729dc?version=4.2.18 "https://svelte.dev/repl/d809e6a0c8b1448eb3df1726c73729dc?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/663543f14fa44907a3df75ad2891d4ce~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1361&h=383&s=1249220&e=gif&f=51&b=272727)

**掌握 Store contract 和 `$`的使用很重要。** 我给大家举一个实际开发中可能会遇到的例子，比如监听鼠标移动，正常你会这样写：

```xml
<script>
  import { onMount } from 'svelte';
  let x = 0, y = 0;

  onMount(() => {
    function move({clientX, clientY}) {
        x = clientX
        y = clientY
    }
    window.addEventListener("mousemove", move)
    return () => {
      window.removeEventListener("mousemove", move)
    }
  })
</script>

<h1>x: {x} y: {y}</h1>
```

[浏览器效果如下](https://svelte.dev/repl/8525d5883064421b920e59a03fa91adc?version=4.2.18 "https://svelte.dev/repl/8525d5883064421b920e59a03fa91adc?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58a3e725602447738a3348be3b4ebcd6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1356&h=647&s=692048&e=gif&f=17&b=242424)

但如果多个组件都需要用到这个值呢？如果是 React，我们很可能会写一个 useMousemove Hook，在 Svelte 中该怎么办呢？

让我们再细看上面的这段代码，你会发现，其实它跟 Store 的逻辑很像！都是监听值的变化。都是组件初始化的时候，注册一个事件，这个例子中是 addEventListener，Store 中是 subscribe。组件注销的时候，都要取消，这个例子中是 removeEventListener，Store 中是调用 subscribe 返回的 unsubscribe 函数。

这个时候就可以按照 Store contract 创建一个自己的 Store！

```xml
<!-- App.svelte -->
<script>
  import mousePostion from './useMousemove.js'
</script>

<h1>x: {$mousePostion.x} y: {$mousePostion.y}</h1>

<!-- useMousemove.js -->
const store = {
  subscribe(fn) {
    fn({ x:0, y:0 })

    function move({clientX, clientY}) {
      fn({ x:clientX, y:clientY })
    }
    window.addEventListener("mousemove", move)
    return () => {
      window.removeEventListener("mousemove", move)
    }
  }
}

export default store
```

[浏览器效果如下](https://svelte.dev/repl/bc0afebde86e44a18895ace79f6d0b9a?version=4.2.18 "https://svelte.dev/repl/bc0afebde86e44a18895ace79f6d0b9a?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd241cff7960446ba82744ffa46da1fb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1380&h=579&s=864293&e=gif&f=25&b=252525)

### 3.5. Readable Store

Svelte 的 Store 有 Writable 的，也有 Readable 的。它们的差别顾名思义，那就是 Readable 的值无法从外部修改，也就是没有 set 方法。

文档中举的例子都是在内部设置定时器，外部读取。这可能让开发者觉得 readable Store 有些鸡肋。其实监听鼠标移动的这个例子就很适合用 Readable，我们用 Readable Store 重新实现下刚才的例子：

```xml
<!-- App.svelte -->
<script>
  import mousePostion from './useMousemove.js'
</script>

<h1>x: {$mousePostion.x} y: {$mousePostion.y}</h1>

<!-- useMousemove.js -->
import { readable } from 'svelte/store';

export default readable({ x:0, y:0 }, (set) => {
  window.addEventListener("mousemove", move);

  function move({ clientX, clientY }) {
    set({
      x: clientX,
      y: clientY,
    });
  }

  return () => {
    window.removeEventListener("mousemove", move);
  }
})
```

[浏览器效果如下](https://svelte.dev/repl/981d2cd12d5643f7a2bf4680a0a44bc9?version=4.2.18 "https://svelte.dev/repl/981d2cd12d5643f7a2bf4680a0a44bc9?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8749b4357a0949259bb4e34073a8d404~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1380&h=579&s=1141608&e=gif&f=30&b=252525)

### 3.6. Derived Store

Derived Store 类似于响应式声明，相当于响应式声明的 Store 版。我们举个例子：

```xml
<script>
  import { writable, derived } from 'svelte/store';
  const count = writable(0);
  const doubled = derived(count, ($count) => $count * 2);
</script>

<input type="number" bind:value={$count}>
<p>{$count} * 2 = {$doubled}</p>
```

响应式声明是监听值的变化，Derived Store 是监听 Store 值的变化。[浏览器效果如下](https://svelte.dev/repl/40860b7fdab245a081818309d868d622?version=4.2.18 "https://svelte.dev/repl/40860b7fdab245a081818309d868d622?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9408613077db44fabcb78e18b86abe3f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1359&h=351&s=711793&e=gif&f=23&b=272727)

其实 Tutorial 中介绍的有些单薄，如果查看 [derived Store 的 API 文档](https://svelte.dev/docs/svelte-store#derived "https://svelte.dev/docs/svelte-store#derived")，介绍更加全面，它支持单个值，也支持多个值，支持同步，也支持异步。我们举个更复杂的例子：

```xml
<script>
  import { writable, derived } from 'svelte/store';
  const count1 = writable(0);
  const count2 = writable(0);
  // const doubled = derived([count1, count2], ([$count1, $count2]) => $count1 * $count2);
  const doubled = derived([count1, count2], ([$count1, $count2], set) => {
    const timeout = setInterval(() => {
      set($count1 * $count2);
    }, 1000);

    return () => {
      console.log('cancel')
      clearTimeout(timeout);
    };
  }, 'loading');
</script>

<input type="number" bind:value={$count1}> * <input type="number" bind:value={$count2}>
<p>{$count1} * {$count2} = {$doubled}</p>
```

在这段代码中，我们使用 `derived()` 监听了 `count1` 和 `count2` 值的变化。`derived()` 函数的第二个参数是一个函数，我们可以直接返回一个值，表示计算的最终值。但如果要在异步函数中设置计算的最终值，则要使用 `set()`函数 。

[浏览器效果如下](https://svelte.dev/repl/b827aac00dd74573863f4597671c8c98?version=4.2.18 "https://svelte.dev/repl/b827aac00dd74573863f4597671c8c98?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/973197218ff3425d8f047ff76cb87043~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1799&h=687&s=197628&e=gif&f=47&b=272727)

### 3.7. High Order Store

Svelte 提供了一个 [readonly](https://svelte.dev/docs/svelte-store#readonly "https://svelte.dev/docs/svelte-store#readonly") 辅助函数，可以将 Store 的值变为只读：

```typescript
import { readonly, writable } from "svelte/store";

const writableStore = writable(1);
const readableStore = readonly(writableStore);

readableStore.subscribe(console.log);

writableStore.set(2); // console: 2
readableStore.set(2); // ERROR
```

重点不在于 readonly 的作用，而是 readonly 怎么实现的。其实我们自己也可以实现一个 readonly 辅助函数：

```xml
<!-- App.svelte -->
<script>
  import {writable} from 'svelte/store'; import readonly from './readonly';
  const writableStore = writable(1); let readableStore =
  readonly(writableStore); readableStore.subscribe(console.log);
  writableStore.set(2); // console: 2 readableStore.set(2); // ERROR
</script>;

<!-- readonly.js -->
export default function readonly(store) {
  return {
    subscribe: store.subscribe,
    set() {
      throw new Error("Unable to set value of a readonly store");
    },
  };
}
```

[浏览器效果如下：](https://svelte.dev/repl/8b3be76e1fe94dfe8b73d8e6e8c68ab8?version=4.2.18 "https://svelte.dev/repl/8b3be76e1fe94dfe8b73d8e6e8c68ab8?version=4.2.18")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d06a8171e304af9864ca1de7ed4e8a4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=3208&h=950&s=180968&e=png&b=202020)

这个 readonly 的实现就是我们要介绍的内容。我们知道 High Order Component 是返回组件的组件，High Order Store 与之类似，是返回 Store 的 Store，作用在于加强 Store 的功能。

> 注意：当然 Svelte 官方文档里并没有 High Order Store 的介绍。只不过是借鉴了 High Order Component 取了这个名字。你可以把它理解为一种 Svelte 小技巧。

比如我们之前实现过一个监听鼠标值的 useMousemove，但它会频繁的触发更新，如果我们想要加一个节流的效果呢？

我们当然可以直接在原代码里加上，但更好的做法是写一个专门实现节流的 Store，这样“关注点分离”，代码更加清晰合理：

```javascript
<!-- App.svelte -->
<script>
  import mousePostion from './useMousemove.js'
  import withThrottle from './withThrottle.js'

  const postion = withThrottle(mousePostion)
</script>

<h1>x: {$postion.x} y: {$postion.y}</h1>

<!-- useMousemove.js -->
import { readable } from 'svelte/store';

export default readable({ x:0, y:0 }, (set) => {
  window.addEventListener("mousemove", move);

  function move({ clientX, clientY }) {
    set({
      x: clientX,
      y: clientY,
    });
  }

  return () => {
    window.removeEventListener("mousemove", move);
  }
})

<!-- withThrottle.js -->
import { derived } from 'svelte/store';

export default function (store) {
  let previous = 0;
  return derived(store, (value, set) => {
    let now = Date.now();
    if (now - previous > 1000) {
      set(value);
      previous = now;
    }
  });
}
```

在这段代码中，我们实现了一个 `useMousemove` 获取鼠标的值，又实现了一个 `withThrottle` 实现节流。当调用 `withThrottle(mousePostion)`的时候，它会返回一个名为 `position` 的 Store，我们使用 `$postion`就可以获取节流后的鼠标位置。

[浏览器效果如下](https://svelte.dev/repl/f9057c03d2da401184c777a10fbaf6fb?version=4.2.18 "https://svelte.dev/repl/f9057c03d2da401184c777a10fbaf6fb?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/741ba5d269ef420dbf612b3059816810~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1799&h=618&s=184187&e=gif&f=59&b=262626)

### 3.8. get()

Svelte Store 提供了 `get()` 方法用于获取 Store 的值：

```javascript
import { get } from "svelte/store";

const value = get(store);
```

正如[官方文档](https://svelte.dev/docs/svelte-store#get "https://svelte.dev/docs/svelte-store#get")里说明的那样，`get()` 是通过创建订阅、读取值，然后取消订阅来实现的。所以它相当于：

```javascript
function get(store) {
  let _value;
  const unsubscribe = store.subscribe((value) => {
    _value = value;
  });
  unsubscribe();
  return _value;
}
```

## 4\. Context 与 Store

### 4.1. Context vs Store

那么你可能要问了，Context 和 Store 的区别是什么呢？

Context 解决的是组件和所有子组件共享数据的问题，比如一个博客组件，它需要将 BlogId 传给博客组件下的评论组件、点赞组件，这些组件才能知道评论点赞的是哪篇博客，这个时候就需要 Context。

Store 通过发布订阅实现响应式，所有引入 Store 的组件都可以获取其值，修改其值，从而实现了数据共享。比如页面的主题色，如果修改了，所有组件的样式都应该随之修改。

我个人认为非要用 Store 替代 Context 也不是不可以，但很多时候没有必要。比如当我们使用双向绑定：

```xml
<input type="number" bind:value={$data}>
```

如果 `$data` 来自于 Store，修改输入框的值会修改所有用到这个 Store 的组件。但如果 `$data` 来自于 Context，则不会产生任何影响。

实际上，Context 和 Store 使用上并不冲突，完全可以一起使用，这就是 Reactive Context。它结合了两者的优点，Context 控制了组件的影响范围，Store 将状态共享。

### 4.2. Reactive Context

之前讲 Context 的时候讲到目前 Context 的实现并不是响应式的，举个例子：

```xml
<!-- App.svelte -->
<script>
  import Parent from './Parent.svelte';
  import { setContext } from 'svelte';

  let count = 0;
  setContext('count', count)
</script>

From App.svelte: {count}
<button on:click={() => {count += 1}}>
  Increment
</button>

<Parent />

<!-- Parent.svelte -->
<script>
  import Child from './Child.svelte';
</script>

<Child />

<!-- Child.svelte -->
<script>
  import { getContext } from 'svelte';

  let count = getContext('count');
</script>

<div>
  From Child.svelte: { count }
</div>
```

当顶层的 count 值发生改变的时候，子组件中的 count 值并不会改变。[浏览器效果如下](https://svelte.dev/repl/eb00d4a2711e4bf7b87d90beaae37d44?version=4.2.18 "https://svelte.dev/repl/eb00d4a2711e4bf7b87d90beaae37d44?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc5c3f136ca44fdb9cb076a374b6e125~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1068&h=545&s=1570418&e=gif&f=48&b=272727)

如何让子组件的 count 值也随之变化呢？Context + Store 就可以成为 Reactive Context。修改代码如下：

```xml
<!-- App.svelte -->
<script>
  import { setContext } from 'svelte';
  import { writable } from 'svelte/store';

  import Parent from './Parent.svelte';
  let count = writable(0);
  setContext('count', count)
</script>

From App.svelte: {$count}
<button on:click={() => {$count += 1}}>
  Increment
</button>

<Parent />

<!-- Parent.svelte -->
<script>
  import Child from './Child.svelte';
</script>

<Child />

<!-- Child.svelte -->
<script>
  import { getContext } from 'svelte';

  let count = getContext('count');
</script>

<div>
  From Child.svelte: { $count }
</div>
```

当顶层的 count 值发生改变的时候，子组件中的 count 值也会随之改变。[浏览器效果如下](https://svelte.dev/repl/d3f4ee2660e24163bd7c278fa6891961?version=4.2.18 "https://svelte.dev/repl/d3f4ee2660e24163bd7c278fa6891961?version=4.2.18")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55ef2a0169a5413787cd8182c26fc8b6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1141&h=572&s=130395&e=gif&f=40&b=272727)

### 4.3. Context or Store

那么在我们的实际开发中，什么时候使用 Context、什么时候使用 Store、什么时候使用 Context + Store 呢？这取决于很多情况，但我们可以简单的讨论一下。

这要看你所需要的数据是静态的还是动态的，以及数据的影响范围，是全局的还是部分的。

如果是静态数据 + 影响全局，那相当于配置文件，你甚至都不需要 Context、Store。正如我们项目开发中经常会建立一个 `consts.js` 用于声明常量，需要的时候直接引入该文件即可。

如果是静态数据 + 影响部分，Context 是一个不错的选择，比如博客组件将博客 ID 发送给评论组件、点赞组件等。

如果是动态数据 + 影响全局，Store 是一个不错的选择，比如页面的主题色。

如果是动态数据 + 影响部分，Store + Context 是一个不错的选择。Store 负责状态管理，Context 负责影响范围，组件在各自的范围内实现数据共享。

## 5\. 最后

恭喜您完成了第二篇内容的学习！Context 和 Store 是 Svelte 的内置状态管理工具，需要学习掌握其用法。