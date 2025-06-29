> 推荐学习指数：⭐️️⭐️️⭐️️

## 1\. 前言

最近大家提起 Signals，都在说：“Signals 是前端框架的未来”。从使用情况来说，还真是这样。Angular、Ember、Preact、Qwik、Solid、Svelte、Vue，甚至 Mobx、Tailwind 都已使用 Signals，甚至已经有了关于 Signals 的 TC39 提案，未来可能直接在 JavaScript 中使用 Signals。然而，Signals 并不是一个新概念，早在 2010 年的 Knockout.js 就有使用。

所以 Signals 到底是什么？又是因为什么流行起来？背后的实现原理是什么？带来了哪些好处，解决了哪些问题？为什么会受到大家的喜欢和推崇？

因为 Svelte 5 的符文功能正是基于 Signals，所以本篇就带大家探究一下 Signals。

## 2\. Signals

什么是 Signals 呢？在我看来，它是一种实现细粒度响应式的技术称呼。在学术论文中经常被称为“ Signals”，也会被称为：Observables、Atoms、Subjects、Refs（想想 Vue 的 Ref）。因为是一种技术名称，所以它没有具体的 API，每个框架都有自己的实现方式。

### 2.1. 响应式

关于“细粒度响应式”，让我们来慢慢解释。

首先是什么是响应式？

JavaScript 的响应式是有限制的，举个例子：

```javascript
let count = 1;
let doubled = count * 2;

count = 2;
console.log(doubled);
```

此时 `doubled`的打印结果为 `2`，但我们想要的是 `4`，为此我们可以这样做：

```javascript
let count = 1;
let doubled = () => count * 2;

count = 2;
console.log(doubled());

count = 3;
console.log(doubled());
```

这当然能解决问题，但实际项目开发中，我们还想要在 `count`值发生修改的时候做一些事情（这就是我们常说的 effect），我们很容易想到使用观察者模式：

```javascript
function observable(value) {
  const subscribers = new Set();
  return {
    subscribe(fn) {
      subscribers.add(fn);
    },
    update(value) {
      subscribers.forEach((fn) => fn(value));
    },
  };
}

let count = observable(0);

count.subscribe((count) => {
  console.log(count);
});

count.update(2);
```

这当然能解决问题，但每次都要手动进行订阅会让代码更加复杂，而且我们该如何处理像 doubled 这种衍生状态呢？我们或许会这样做：

```javascript
let count = observable(0);
let doubled;

count.subscribe((count) => {
  console.log("count =", count);
});

count.subscribe((count) => {
  doubled = count * 2;
  console.log("count * 2 =", doubled);
});

count.update(2);
```

可以看出：对于衍生状态以及其修改时的副作用订阅，这些都会增加代码的复杂度。

为了统一解决这些问题，在不断地实践中便有了“Signals”，它本质还是基于观察者模式，但是借助了 getter 和 setter，所以它解决了手动订阅的问题。

它的使用方式如下：

```javascript
let subscriber = null;

function signal(value) {
  const subscribers = new Set();
  return {
    get value() {
      if (subscriber) {
        subscribers.add(subscriber);
      }
      return value;
    },
    set value(newValue) {
      value = newValue;
      subscribers.forEach((fn) => fn());
    },
  };
}

function effect(fn) {
  subscriber = fn;
  fn();
  subscriber = null;
}

function derived(fn) {
  const derived = signal();
  effect(() => {
    derived.value = fn();
  });
  return derived;
}

let count = signal(0);
let doubled = derived(() => count.value * 2);

effect(() => {
  console.log("count =", count.value);
});

effect(() => {
  console.log("count * 2 =", doubled.value);
});

count.value = 2;
```

这是我们自己定义的 Signals 函数和使用方式，但已经接近主流框架 Signals 的使用方式了。单看 Signals 其实是没有什么特别的，搭配 Reactions（也被称为 Effects、Autoruns、 Watches 或 Computed）和衍生状态的处理会让 Signals 的功能更加强大。其中 Reactions 会监听 Signals 的改变，在其发生更新时重新运行。

所以再看这段代码，其实还是很神奇的，因为修改 count 的值和 effect 表面上没有什么直接关联的地方，但在底层的实现中，effect 的回调函数会调用 count，因为调用了 count，所以自动订阅了 effect 函数。当 count 的值发生修改时，就会重新运行 effect 函数。

这就是 Signals 所带来的响应式，其实很多框架都已经实现了。

比如这是 2010 年的 Knockout：

```javascript
const count = ko.observable(0);

const doubleCount = ko.pureComputed(() => count() * 2);

// logs whenever doubleCount updates
ko.computed(() => console.log(doubleCount()));
```

Vue：

```javascript
import { ref, watchEffect } from "vue";

const count = ref(0);

watchEffect(() => {
  document.body.innerHTML = `Count is: ${count.value}`;
});

// 更新 DOM
count.value++;
```

Solid.js：

```javascript
import { createSignal } from "solid-js";

const [count, setCount] = createSignal(0);

function Counter() {
  return (
    <div
      onClick={() => {
        setCount((c) => c + 1);
      }}
    >
      Count: {count()}
    </div>
  );
}
```

[Signals TC39 草案](https://eisenbergeffect.medium.com/a-tc39-proposal-for-signals-f0bedd37a335 "https://eisenbergeffect.medium.com/a-tc39-proposal-for-signals-f0bedd37a335")：

```javascript
const counter = new Signal.State(0);
console.log(counter.get()); // 0
counter.set(1);
console.log(counter.get()); // 1
const isEven = new Signal.Computed(() => (counter.get() & 1) == 0);
console.log(isEven.get()); // false
counter.set(2);
console.log(isEven.get()); // true
```

### 2.2. 细粒度

那细粒度又是指什么呢？这就不得不提 React 了，我们举个例子：

```jsx
export function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

在 React 中，useState() 返回一个状态值和修改状态值的方法，当你调用 setCount 告诉 React 有状态修改的时候，React 必须重新渲染整个组件。

但使用 Signals 是不会的，我们以 Solid.js 为例：

```jsx
import { createSignal } from "solid-js";

function Counter() {
  const [count, setCount] = createSignal(0);
  return <button onClick={() => setCount(count() + 1)}>{count()}</button>;
}
```

在 Solid.js 中，虽然代码看起来极其相似，但当你调用 setCount 更新状态的时候，组件并不会重新渲染，它会直接更新 DOM 内容。

让我们举个更明显的例子：

这是 React：

```jsx
import { useState, useEffect } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount((c) => c + 1);
    }, 1000);
    return () => clearInterval(intervalId);
  }, []);

  console.log(count);
  return <h1>{count}</h1>;
}
```

你可以想到，React 每秒都会打印一次 count，因为 setCount 会触发组件重新渲染。

但如果是在 Solid.js 中：

```jsx
import { render } from "solid-js/web";
import { createSignal, onCleanup } from "solid-js";

function Counter() {
  const [count, setCount] = createSignal(0);

  const timer = setInterval(() => setCount((c) => c + 1), 1000);
  onCleanup(() => clearInterval(timer));
  console.log(count());
  return <h1>{count()}</h1>;
}

render(() => <Counter />, document.getElementById("app"));
```

count 只会打印一次，如果你想每秒都打印，可以这样写：

```jsx
import { render } from "solid-js/web";
import { createSignal, onCleanup } from "solid-js";

function Counter() {
  const [count, setCount] = createSignal(0);

  const timer = setInterval(() => setCount((c) => c + 1), 1000);
  onCleanup(() => clearInterval(timer));
  return <h1>{console.log(count())}</h1>;
}

render(() => <Counter />, document.getElementById("app"));
```

那么问题来了，为什么两者的效果截然不同呢？

这是因为 React 返回的是**状态值（StateValue）**，而 Solid.js 返回的是**状态引用（StateReference）**。

对于 React 而言，useState 是不知道状态值在组件中是如何使用的，当你通过 `setCount`通知 React 发生状态更改，因为 React 不知道页面的哪一部分发生了变化，因此必须重新渲染整个组件，这个计算代价是高昂的。

而对于 Solid.js 而言，为了能够做出反应，信号会在状态被调用时（`count()`）收集上下文信息，换句话说，使用 `count()` 时会自动创建一个 subscription，所以当 count 的值发生修改的时候，Solid.js 已经知道哪里要发生修改，所以无须渲染整个组件，只更改对应部分的值即可。

这就是 Signals 带来的细粒度。介绍 Svelte 5 的文档经常会有 **“Fine-grained reactivity”** 这个名词，其实说的就是细粒度响应式。Svelte 5 正是基于 Signals 对代码进行了重构。

## 3\. Signals 的简单历史

Signals 为什么逐渐火起来呢？这离不开各个框架的支持以及一些关键节点事件的发生。所以简单说一下（权当八卦听了）：

最早是被追溯到 2010 年的 Knockout.js：

```javascript
const count = ko.observable(0);

const doubleCount = ko.pureComputed(() => count() * 2);

ko.computed(() => console.log(doubleCount()));
```

后来 React 流行，这种响应式成为小众。

2014 年，Vue 一开始就采用了响应式，但只是将其作为内部的实现机制。Vue 现在的官网上有[《与信号 (signal) 的联系》](https://cn.vuejs.org/guide/extras/reactivity-in-depth "https://cn.vuejs.org/guide/extras/reactivity-in-depth")章节，与主流框架的 Signals 做了对比。

2019 年，Svelte3 展示了编译器的能力，从编译的角度实现了响应式，这让大家更加注重编译所带来的优化效果在，这也将成为未来框架的趋势。

正式火起来应该是 Solid.js 的诞生以及它将 Signals 这个概念推出来，这也是 Solid.js 的基础。Solid.js 的作者 [Ryan Carniato](https://dev.to/ryansolid "https://dev.to/ryansolid") 发布了很多介绍响应式和 Signals 的文章，受到大家的欢迎。

2023 年，Builder.io 的 CTO、Angular、Qwik 的作者 Miško Hevery 发文[《useSignal() is the Future of Web Frameworks》](https://www.builder.io/blog/usesignal-is-the-future-of-web-frameworks "https://www.builder.io/blog/usesignal-is-the-future-of-web-frameworks")。这应该就是“Signals 是前端框架的未来”这种说法的来源了。它的主要观点就是 React 的 useState 返回的响应值在改变时会导致组件重新渲染，基于 Signals 会自动创建订阅，通知改变，带来更高的性能。加上随着这么多年各个框架的实践，Signals 的开发者使用体验已经很好了，不比传统框架差。所以未来应该是基于 Signals 的。

随着 Signals 的概念逐渐深入人心，Angular、Ember、Preact、Qwik、Solid、Svelte、Vue，甚至 Mobx、Tailwind 都已使用 Signals，现在已经有了关于 Signals 的 TC39 提案，未来可能直接在 JavaScript 中使用 Signals。

React 考虑在底层使用：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a93ff9a9e10344f784bf01ee6f79c04b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1674&h=886&s=250832&e=png&b=f7f8f8)

## 4\. 参考链接

1. [A Hands-on Introduction to Fine-Grained Reactivity](https://dev.to/ryansolid/a-hands-on-introduction-to-fine-grained-reactivity-3ndf "https://dev.to/ryansolid/a-hands-on-introduction-to-fine-grained-reactivity-3ndf")
2. [The Evolution of Signals in JavaScript](https://dev.to/this-is-learning/the-evolution-of-signals-in-javascript-8ob "https://dev.to/this-is-learning/the-evolution-of-signals-in-javascript-8ob")
3. [Implementing Signals from Scratch](https://dev.to/ratiu5/implementing-signals-from-scratch-3e4c "https://dev.to/ratiu5/implementing-signals-from-scratch-3e4c")
4. [Patterns for Reactivity with Modern Vanilla JavaScript – Frontend Masters Boost](https://frontendmasters.com/blog/vanilla-javascript-reactivity/#toc-11 "https://frontendmasters.com/blog/vanilla-javascript-reactivity/#toc-11")