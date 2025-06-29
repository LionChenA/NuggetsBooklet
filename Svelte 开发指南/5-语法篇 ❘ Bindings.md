> 推荐学习指数：⭐️⭐️，建议学习

## 1\. 前言

一般来说，数据流应该自上而下，通过 props 从父级流向子级。但有的时候，数据从子级流向父级是很有必要的，就比如 `<input>`元素，正常你需要这样写：

```xml
<script>
  let name = $state('')
  function oninput(e) {
    name = e.target.value
  }
</script>

<input {oninput} value={name} >
```

每个 `<input>`都需要这样处理，多少有些“样板代码”。使用 Bindings 可以将代码简化为：

```xml
<script>
  let name = $state('');
</script>

<input bind:value={name} />
```

如果绑定名和值一致，代码还可以简写为：

```xml
<script>
  let value = $state('');
</script>

<input bind:value />

<!-- 相当于  -->
<input bind:value={value}/>
```

Bindings 看似好用，但其实只能用在特定的元素中。

那 Bindings 可以应用于哪些元素上，绑定哪些值，本篇我们就来详细介绍一下。

## 2\. 绑定表单元素

Bindings 常用于表单元素：

```xml
<input bind:value={name} />

<textarea bind:value={text} />

<!-- 注意绑定的是 checked -->
<input type="checkbox" bind:checked={yes} />

<!-- 注意输入为空的时候，值为 null -->
<input type="number" bind:value={num} />

<input type="range" bind:value={num} />

<!-- 获取的是只读的 FileList 文件  -->
<!-- https://developer.mozilla.org/en-US/docs/Web/API/FileList -->
<input accept="image/png, image/jpeg" bind:files id="avatar" name="avatar" type="file" />
```

当绑定 `<select>` 的时候：

```xml
<select bind:value={selected}>
  <option value={a}>a</option>

  <option value={b}>b</option>

  <option value={c}>c</option>

</select>

<!-- fillings 是一个数组 -->
<select multiple bind:value={fillings}>
  <option value="Rice">Rice</option>

  <option value="Beans">Beans</option>

  <option value="Cheese">Cheese</option>

  <option value="Guac (extra)">Guac (extra)</option>

</select>

<!-- 当 option value 的值与文本值相同的时候，可以简写： -->
<select multiple bind:value={fillings}>
  <option>Rice</option>

  <option>Beans</option>

  <option>Cheese</option>

  <option>Guac (extra)</option>

</select>

```

[浏览器效果如下：](https://svelte.dev/playground/2f1bd3f8b0054cd6a9ddca49c7045cd0?version=5.1.12 "https://svelte.dev/playground/2f1bd3f8b0054cd6a9ddca49c7045cd0?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb62330196494428914cf4781e611fa2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2266&h=1142&s=181834&e=png&b=272727)

Bindings 也可用于 Each 逻辑块中：

```xml
<script>
  let todos = $state([
    { done: false, text: 'finish Svelte tutorial' },
    { done: false, text: 'build an app' },
    { done: false, text: 'world domination' }
  ]);

  $inspect(todos);
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
```

[浏览器效果如下：](https://svelte.dev/playground/072b757410c6472ca2bba84e8c229f7e?version=5.1.12 "https://svelte.dev/playground/072b757410c6472ca2bba84e8c229f7e?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c80adbb31643452d993e8d77f67c8e6c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1474&h=525&s=1804714&e=gif&f=37&b=252525)

## 3\. 绑定组

`bind:group` 适用于单选框和复选框：

```xml
<script>
  let tortilla = 'Plain';
  let fillings = [];
</script>

<!-- 成组的 radio 输入会互斥 -->
<input type="radio" bind:group={tortilla} value="Plain" />
<input type="radio" bind:group={tortilla} value="Whole wheat" />
<input type="radio" bind:group={tortilla} value="Spinach" />

<!-- 成组的 checkbox 输入会放入一个数组 -->
<input type="checkbox" bind:group={fillings} value="Rice" />
<input type="checkbox" bind:group={fillings} value="Beans" />
<input type="checkbox" bind:group={fillings} value="Cheese" />
<input type="checkbox" bind:group={fillings} value="Guac (extra)" />
```

[浏览器效果如下：](https://svelte.dev/playground/e328b087c5844bf593addd84edf2cb15?version=5.1.12 "https://svelte.dev/playground/e328b087c5844bf593addd84edf2cb15?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b3fb565d9a045adb8f6d547fab58a07~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1163&h=508&s=1250910&e=gif&f=30&b=242424)

## 4\. 绑定特殊元素

### 4.1. contenteditable

Bindings 也可用于具有 `contenteditable`属性的元素：

```xml
<div contenteditable="true" bind:innerHTML={html} />
```

具体支持绑定的属性有：

1. `innerHTML`
2. `innerText`
3. `textContent`

它们的差别在出现换行的时候比较明显：

```xml
<script>
  let html = $state(''), text = $state(''), content = $state('');
  $inspect(html, text, content);
</script>

<div contenteditable="true"
  bind:innerHTML={html}
  bind:innerText={text}
  bind:textContent={content}
>
</div>
```

[浏览器效果如下：](https://svelte.dev/playground/79b0d2f7864a452999779f123febb2df?version=5.1.12 "https://svelte.dev/playground/79b0d2f7864a452999779f123febb2df?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39b1664b3d8644ba9708387309cc311e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2402&h=1080&s=251858&e=png&b=1d1f22)

具体差别可以[查看 MDN 关于 Node.textContent 的讲解](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/textContent#differences_from_innertext "https://developer.mozilla.org/zh-CN/docs/Web/API/Node/textContent#differences_from_innertext")。

### 4.2. details

也支持 `<details>` 元素：

```xml
<script>
  let isOpen = $state(false)
  $inspect(isOpen);
</script>

<details bind:open={isOpen}>
  <summary>Details</summary>

  <p>Something small enough to escape casual notice.</p>

</details>
```

[浏览器效果如下：](https://svelte.dev/playground/6c243bc98cb04eab9f782ff5ff1c594d?version=5.1.12 "https://svelte.dev/playground/6c243bc98cb04eab9f782ff5ff1c594d?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/15d5693ed5c94959b29853b88aa90418~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1172&h=482&s=45716&e=gif&f=15&b=1d1e22)

### 4.3. audio 和 video

`<audio>` 有 7 个只读绑定（[currentTime](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/currentTime "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/currentTime")、[playbackRate](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/playbackRate "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/playbackRate")、[paused](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/paused "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/paused")、[volume](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/volume "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/volume")、[muted](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/muted "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/muted")）和 5 个双向绑定（[duration](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/duration "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/duration")、[buffered](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/buffered "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/buffered")、[paused](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/paused "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/paused")、[seekable](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/seekable "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/seekable")、[seeking](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/seeking_event "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/seeking_event")、[ended](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/ended "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/ended")、[readyState](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/readyState "https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/readyState")）：

```xml
<audio
  src={clip}
  bind:duration
  bind:buffered
  bind:played
  bind:seekable
  bind:seeking
  bind:ended
  bind:readyState
  bind:currentTime
  bind:playbackRate
  bind:paused
  bind:volume
  bind:muted
/>
```

`<video>` 相比 `<audio>` 多两个只读绑定 [videoWidth](https://developer.mozilla.org/en-US/docs/Web/API/HTMLVideoElement/videoWidth "https://developer.mozilla.org/en-US/docs/Web/API/HTMLVideoElement/videoWidth") 和 [videoHeight](https://developer.mozilla.org/en-US/docs/Web/API/HTMLVideoElement/videoHeight "https://developer.mozilla.org/en-US/docs/Web/API/HTMLVideoElement/videoHeight")：

```xml
<video
  src={clip}
  bind:duration
  bind:buffered
  bind:played
  bind:seekable
  bind:seeking
  bind:ended
  bind:readyState
  bind:currentTime
  bind:playbackRate
  bind:paused
  bind:volume
  bind:muted
  bind:videoWidth
  bind:videoHeight
/>
```

### 4.4. image

`<img>` 有 2 个只读绑定：

1. [naturalWidth](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/naturalWidth "https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/naturalWidth")：图像的原始宽度，在图像加载后可用
2. [naturalHeight](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/naturalHeight "https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/naturalHeight")：图像的原始高度，在图像加载后可用

```xml
<img
  bind:naturalWidth
  bind:naturalHeight
/>
```

### 4.5. 可视元素

所有可视元素都有 4 个只读绑定，底层使用 [ResizeObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/ResizeObserver "https://developer.mozilla.org/zh-CN/docs/Web/API/ResizeObserver") API 进行测量：

1. [clientWidth](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientWidth "https://developer.mozilla.org/en-US/docs/Web/API/Element/clientWidth")
2. [clientHeight](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientHeight "https://developer.mozilla.org/en-US/docs/Web/API/Element/clientHeight")
3. [offsetWidth](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetWidth "https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetWidth")
4. [offsetHeight](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetHeight "https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetHeight")

```xml
<div bind:offsetWidth={width} bind:offsetHeight={height}>
  <Chart {width} {height} />
</div>
```

> 注意：`display:inline` 的元素没有高度或宽度（特殊元素如 img、canvas 除外），建议先修改属性为如 `display:inline-block`

## 5\. 绑定组件

Binding 本质是建立双向绑定，input 的值由 value 确定，在 input 输入的值也会修改 value。Binding 不止可以用于元素上，也可以用在组件上。

为了帮助大家理解组件的双向绑定，我们先举一个传统的实现组件双向绑定的例子：

```xml
<!-- App.svelte -->
<script>
  import Child from "./Child.svelte"

  let count = $state(0);

  function oncountchange(value) {
    count = value
  }
</script>

<Child {count} {oncountchange} />

<button onclick={() => {
  count = 0
}}>Reset {count}</button>

<!-- Child.svelte -->
<script>
  const { oncountchange, count } = $props();
</script>

<button onclick={() =>{
  oncountchange(count + 1)
}}>
  Increament {count}
</button>
```

**在这段代码中，子组件的 count 值来自于父组件，当子组件发生点击事件的时候，调用父组件的 oncountchange 函数，修改父组件的 count 值，进而改变子组件的 count 值**

[浏览器效果如下：](https://svelte.dev/playground/e6f6274b4faa4d2aa3b5f5ff9accd319?version=5.1.12 "https://svelte.dev/playground/e6f6274b4faa4d2aa3b5f5ff9accd319?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ff37e8539cc4a599401663fb5d21296~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1049&h=564&s=52446&e=gif&f=34&b=1e2024)

**然而换成 Bindings 后，代码会更加简洁：**

```xml
<!-- App.svelte -->
<script>
  import Child from "./Child.svelte"

  let count = $state(0);

  function oncountchange(value) {
    count = value
  }
</script>

<Child bind:count />

<button onclick={() => {
  count = 0
}}>Reset {count}</button>

<!-- Child.svelte -->
<script>
  let { count = $bindable() } = $props();
</script>

<button onclick={() =>{
  count = count + 1
}}>
  Increament {count}
</button>
```

[浏览器效果如下：](https://svelte.dev/playground/1c843c22978e405cb99d42efea42a325?version=5.1.12 "https://svelte.dev/playground/1c843c22978e405cb99d42efea42a325?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6ea87e70ede4f6b91b3914eac6bae91~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1049&h=564&s=49867&e=gif&f=30&b=1e2024)

## 6\. 绑定 this

当 `bind:this` 用于元素时可获取元素的引用，功能类似于 React 的 ref。举个例子：

```xml
<script>
  import { onMount } from "svelte";

  let element = '';

  onMount(() => {
    console.log(element)
  })
</script>

<input bind:this={element} />

<button onclick={() => {
  element.value = ''
}}>Reset</button>

<button onclick={() => {
  element.focus()
}}>Focus</button>
```

**在这段代码中，如果一开始就打印 element 的值，它会是 undefined，只有当挂载的时候，它才会是元素的引用。获取元素的引用后，你就可以操作元素，比如设置元素的值，设置聚焦状态等。**

[浏览器效果如下：](https://svelte.dev/playground/0a90b0cdee1245b297254041efc71960?version=5.1.12 "https://svelte.dev/playground/0a90b0cdee1245b297254041efc71960?version=5.1.12")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2c0593d72604ca79414af99cf2faf9f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1438&h=733&s=84488&e=gif&f=33&b=252525)

`bind:this` 也可用于组件，用于获取组件实例，调用组件上的方法：

```xml
<!-- App.svelte -->
<script>
  import Child from './Child.svelte';
  let element = '';

  function reset() {
    element.reset()
  }
</script>

<Child bind:this={element} />

<button onclick={reset}>Reset</button>

<!-- Child.svelte -->
<script>
  let count = $state();

  export function reset() {
    count = '';
  }
</script>

<input bind:value = {count} />
```

在这段代码中，父组件通过 `bind:this` 获取到子组件的实例，然后调用了子组件导出的 `reset()` 方法，重置了表单的值。

[浏览器效果如下](https://svelte.dev/playground/d02882303f2241b9bcd5c46660da2fa3?version=5.1.12 "https://svelte.dev/playground/d02882303f2241b9bcd5c46660da2fa3?version=5.1.12")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b20dabdd70bc404788f149ccb6ea1063~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1191&h=537&s=806577&e=gif&f=27&b=242424)

## 7\. 最后

恭喜您完成了本篇内容的学习！本篇我们介绍了双向绑定 Bindings 的用法，使用 Bindings 有时可以简化代码，让代码更加直观。Bindings 可绑定表单元素，也可以绑定一些特殊元素。可绑定 DOM 元素，也可以绑定组件。虽然最常用的也就是绑定一下表单元素……