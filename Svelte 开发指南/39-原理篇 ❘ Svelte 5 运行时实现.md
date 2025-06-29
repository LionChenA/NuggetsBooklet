> 推荐学习指数：⭐️️⭐️️

## 1\. 前言

Svelte 5 之于 Svelte 4，并不是在 Svelte 4 上新增了一个符文功能，而是基于 Signals 的彻底重写。本篇我们讲讲 Svelte 5 编译后的代码运行原理。我们先从最基础的代码开始说起。

## 2\. Hello World!

最基础的代码莫过于 `"Hello World!"` 了：

```html
<h1>Hello world!</h1>
```

查看 [REPL 生成的代码](https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAEy3IMQqAMAwF0KvonwvuRQQ3b-AgDmpTKERT2lQQ8e4iOL53wwemDDvdOJadYNHHCAO94od8EivBIEtJ2zftWlTl6AZilmqUxK5umz9hsIsLPpCD1VTomZ8XwTP8TmMAAAA= "https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAEy3IMQqAMAwF0KvonwvuRQQ3b-AgDmpTKERT2lQQ8e4iOL53wwemDDvdOJadYNHHCAO94od8EivBIEtJ2zftWlTl6AZilmqUxK5umz9hsIsLPpCD1VTomZ8XwTP8TmMAAAA=")：

```javascript
import * as $ from "svelte/internal/client";

var root = $.template(`<button>Hello World!</button>`);

export default function App($$anchor) {
  var button = root();

  $.append($$anchor, button);
}
```

`$.template`、`$.append` 到底做了什么呢？

从 `svelte/internal/client`这个导入文件目录对应 [Svelte GitHub 源码](https://github.com/sveltejs/svelte/tree/main "https://github.com/sveltejs/svelte/tree/main")，可以查找到 `$` 的[源码文件](https://github.com/sveltejs/svelte/blob/main/packages/svelte/src/internal/client/index.js "https://github.com/sveltejs/svelte/blob/main/packages/svelte/src/internal/client/index.js")：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9321e07fd5bb4bf987eac9ca331cd848~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2690&h=1190&s=268416&e=png&b=0e1318)

作为汇总的导出函数文件，像 `$.xxx`这样的方法都可以在这个文件查找到对应的源码目录。

所以我们找到 `$.template`和 `$.append`的源码，然后写一个极简版的实现：

新建 `index.html`，代码如下：

```html
<html>
  <head>
    <meta charset="utf-8" />
    <title>svelte-under-the-hood</title>
  </head>
  <body>
    <script src="./svelte5-1.js"></script>
  </body>
</html>
```

新建 `svelte5-1.js`，代码如下：

```javascript
const $ = {};

$.template = template;
$.append = append;

function template(content, flags) {
  var node;
  return () => {
    if (node === undefined) {
      node = create_fragment_from_html(content);
      node = node.firstChild;
    }

    var clone = node.cloneNode(true);
    return clone;
  };
}

function create_fragment_from_html(html) {
  var elem = document.createElement("template");
  elem.innerHTML = html;
  return elem.content;
}

function append(anchor, dom) {
  anchor.before(dom);
}

function mount(Component, { target, anchor }) {
  var anchor_node = anchor ?? target.appendChild(document.createTextNode(""));
  Component(anchor_node);
}

/**
 * 编译后的代码
 */
var root = $.template(`<button>Hello World!</button>`);

function App($$anchor) {
  var button = root();
  $.append($$anchor, button);
}

/**
 * 挂载 DOM
 */
mount(App, {
  target: document.body,
});
```

这段代码并不复杂，`$.template`返回一个函数 `root`，`root()` 函数运行时会创建 DOM 节点，`$.append`将其插入到 `$$anchor`前。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af7a068924614cde9bebf6a2927ed40d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2200&h=910&s=243118&e=png&b=2c2c2c)

## 3\. 加入变量

现在代码改为：

```xml
<script>
  let name = $state('World');
</script>

<button>Hello {name}!</button>
```

查看 [REPL 生成的代码](https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAEz2LwQoCIRQAf8UewRYIezdb6LZ_0CE7bOtbEN6q6DMI8d9DiI4zzFTYHGEG9ajglx1BwS1GkMCf2CG_kRhBQg4lrd3ovCYXeTJeCEIW_RJXccy8MJ6Ge0hkh_PFeD3-S-P1qzAHP81IFETtUzvo8WdBwh6s2xxaUJwKtmf7Ajf-ANKaAAAA "https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAEz2LwQoCIRQAf8UewRYIezdb6LZ_0CE7bOtbEN6q6DMI8d9DiI4zzFTYHGEG9ajglx1BwS1GkMCf2CG_kRhBQg4lrd3ovCYXeTJeCEIW_RJXccy8MJ6Ge0hkh_PFeD3-S-P1qzAHP81IFETtUzvo8WdBwh6s2xxaUJwKtmf7Ajf-ANKaAAAA")：

```xml
import * as $ from "svelte/internal/client";

var root = $.template(`<button> </button>`);

export default function App($$anchor) {
  let name = 'World';
  var button = root();
  var text = $.child(button);

  $.reset(button);
  $.template_effect(() => $.set_text(text, `Hello ${name ?? ""}!`));
  $.append($$anchor, button);
}
```

这次主要多了一个 `$.template_effect`函数，我们提供一个极简版的代码：

修改 `svelte.html`，改下脚本引入地址，代码如下：

```xml
<html>
  <head>
    <meta charset="utf-8" />
    <title>svelte-under-the-hood</title>
  </head>
  <body>
    <script src="./svelte5-2.js"></script>
  </body>
</html>

```

新建 `svelte5-2.js`，代码如下：

```javascript
const $ = {};

$.template = template;
$.append = append;
$.child = child;
$.reset = reset;
$.template_effect = template_effect;
$.set_text = set_text;

function template(content, flags) {
  var node;
  return () => {
    if (node === undefined) {
      node = create_fragment_from_html(content);
      node = node.firstChild;
    }

    var clone = node.cloneNode(true);
    return clone;
  };
}

function create_fragment_from_html(html) {
  var elem = document.createElement("template");
  elem.innerHTML = html;
  return elem.content;
}

function append(anchor, dom) {
  anchor.before(dom);
}

function child(node) {
  return node.firstChild;
}

let hydrating = false;
function reset(node) {
  if (!hydrating) return;
}

const RENDER_EFFECT = 1 << 3;
const ROOT_EFFECT = 1 << 6;
const DIRTY = 1 << 10;

function template_effect(fn) {
  return render_effect(fn);
}
function render_effect(fn) {
  return create_effect(RENDER_EFFECT, fn, true);
}

let component_context = null;

function create_effect(type, fn, sync) {
  var effect = {
    ctx: component_context,
    deps: null,
    nodes_start: null,
    nodes_end: null,
    f: type | DIRTY,
    first: null,
    fn,
    last: null,
    next: null,
    prev: null,
    teardown: null,
    transitions: null,
    version: 0,
  };

  if (sync) {
    update_effect(effect);
  }

  return effect;
}

function update_effect(effect) {
  update_reaction(effect);
}

function update_reaction(reaction) {
  var result = reaction.fn();
  return result;
}

function set_text(text, value) {
  if (value !== (text.__t ??= text.nodeValue)) {
    text.__t = value;
    text.nodeValue = value == null ? "" : value + "";
  }
}

function mount(Component, { target, anchor }) {
  var anchor_node = anchor ?? target.appendChild(document.createTextNode(""));
  Component(anchor_node);
}

/**
 * 编译后的代码
 */
var root = $.template(`<button> </button>`);

function App($$anchor) {
  let name = "World";
  var button = root();
  var text = $.child(button);

  $.reset(button);
  $.template_effect(() => $.set_text(text, `Hello ${name ?? ""}!`));
  $.append($$anchor, button);
}

/**
 * 挂载 DOM
 */
mount(App, {
  target: document.body,
});
```

`$.template_effect`创建了一个 effect 对象，effect 对象会保存 `effect`函数以及各种信息。

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7c50b6fbd3e48509e38baddeea6ef8b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2224&h=1060&s=270595&e=png&b=2d2d2d)

## 4\. 更新变量

现在我们将代码改为：

```xml
<script>
  let name = $state('World');
  function update() {
    name = 'Svelte';
  }

  setTimeout(update, 1000)
</script>
<button>Hello {name}</button>
```

查看 [REPL 生成的代码](https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAEz2NwQrCMAyGX6UEoRsMNq-zCt68K3iwHuaWQaFrS5sKUvru0jk8hXx_vj8JZqUxQP9IYIYFoYezc9AAfVxZwhs1ITQQbPRjISKMXjk6SSNJI7FisSPbBRoIK363Xk-8PpR4jmYkZQ2LbiphzVLBkjaHX9d2vh5nacoISDe1oI1U_ayG7buuq6UR7f-zeEUia04X1NqyVOqyaDcIDSx2UrPCCXryEfMzfwGGUpqX5wAAAA== "https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAEz2NwQrCMAyGX6UEoRsMNq-zCt68K3iwHuaWQaFrS5sKUvru0jk8hXx_vj8JZqUxQP9IYIYFoYezc9AAfVxZwhs1ITQQbPRjISKMXjk6SSNJI7FisSPbBRoIK363Xk-8PpR4jmYkZQ2LbiphzVLBkjaHX9d2vh5nacoISDe1oI1U_ayG7buuq6UR7f-zeEUia04X1NqyVOqyaDcIDSx2UrPCCXryEfMzfwGGUpqX5wAAAA==")：

```javascript
import * as $ from "svelte/internal/client";

var root = $.template(`<button> </button>`);

export default function App($$anchor) {
  let name = $.state('World');

  function update() {
    $.set(name, 'Svelte');
  }

  setTimeout(update, 1000);

  var button = root();
  var text = $.child(button);

  $.reset(button);
  $.template_effect(() => $.set_text(text, `Hello ${$.get(name) ?? ""}`));
  $.append($$anchor, button);
}
```

这里就涉及 Signals 和 Effect 的代码了。为了方便大家快速理解，依然提供一个极简版的实现：

新建 `svelte5-3.js`，代码如下：

```javascript
const $ = {};

$.template = template;
$.append = append;
$.child = child;
$.reset = reset;
$.template_effect = template_effect;
$.set_text = set_text;
$.state = state;
$.get = get;
$.set = set;

function state(v) {
  return {
    f: 0,
    v,
    reactions: null,
    equals: (value) => value === this.v,
    version: 0,
  };
}

function template(content, flags) {
  var node;
  return () => {
    if (node === undefined) {
      node = create_fragment_from_html(content);
      node = node.firstChild;
    }

    var clone = node.cloneNode(true);
    return clone;
  };
}

function create_fragment_from_html(html) {
  var elem = document.createElement("template");
  elem.innerHTML = html;
  return elem.content;
}

function append(anchor, dom) {
  anchor.before(dom);
}

function child(node) {
  return node.firstChild;
}

let hydrating = false;
function reset(node) {
  if (!hydrating) return;
}

const RENDER_EFFECT = 1 << 3;
const ROOT_EFFECT = 1 << 6;
const DIRTY = 1 << 10;

function template_effect(fn) {
  return render_effect(fn);
}
function render_effect(fn) {
  return create_effect(RENDER_EFFECT, fn, true);
}

let component_context = null;
let active_effect = null;
function create_effect(type, fn, sync, push = true) {
  var effect = {
    ctx: component_context,
    deps: null,
    nodes_start: null,
    nodes_end: null,
    f: type | DIRTY,
    first: null,
    fn,
    last: null,
    next: null,
    prev: null,
    teardown: null,
    transitions: null,
    version: 0,
  };

  if (sync) {
    update_effect(effect);
  }

  return effect;
}

function update_effect(effect) {
  update_reaction(effect);
}

let new_deps = null;

function update_reaction(reaction) {
  new_deps = null;

  // 这个函数会触发 get 函数，从而触发 new_deps 的赋值
  var result = reaction.fn();
  var deps = reaction.deps;

  if (new_deps !== null) {
    if (deps == null) {
      reaction.deps = deps = new_deps;
    }

    // 这里建立了 signals 和 effect 的依赖关系
    for (var i = 0; i < deps.length; i++) {
      (deps[i].reactions ??= []).push(reaction);
    }
  }

  return result;
}

function set_text(text, value) {
  if (value !== (text.__t ??= text.nodeValue)) {
    text.__t = value;
    text.nodeValue = value == null ? "" : value + "";
  }
}

const all_registered_events = new Set();
const root_event_handles = new Set();

function get(signal) {
  if (new_deps === null) {
    new_deps = [signal];
  }
  return signal.v;
}

function set(source, value) {
  if (!source.equals(value)) {
    source.v = value;
    mark_reactions(source, DIRTY);
  }

  return value;
}

function mark_reactions(signal, status) {
  signal.f |= status;
  var reactions = signal.reactions;
  if (reactions === null) return;

  for (var i = 0; i < reactions.length; i++) {
    var reaction = reactions[i];
    schedule_effect(reaction);
  }
}

let queued_root_effects = [];
function schedule_effect(signal) {
  queueMicrotask(process_deferred);
  var effect = signal;
  queued_root_effects.push(effect);
}

function process_deferred() {
  const previous_queued_root_effects = queued_root_effects;
  queued_root_effects = [];
  flush_queued_root_effects(previous_queued_root_effects);
}

function flush_queued_root_effects(root_effects) {
  var length = root_effects.length;
  if (length === 0) {
    return;
  }

  for (var i = 0; i < length; i++) {
    var effect = root_effects[i];
    process_effects(effect);
  }
}

function process_effects(effect) {
  update_effect(effect);
}

function mount(Component, { target, anchor }) {
  var anchor_node = anchor ?? target.appendChild(document.createTextNode(""));
  Component(anchor_node);
}

// 编译后代码：

var root = $.template(`<button> </button>`);

function App($$anchor) {
  let name = $.state("World");

  function update() {
    $.set(name, "Svelte");
  }

  setTimeout(update, 1000);

  var button = root();
  var text = $.child(button);

  $.reset(button);
  $.template_effect(() => $.set_text(text, `Hello ${$.get(name) ?? ""}`));
  $.append($$anchor, button);
}

mount(App, {
  target: document.body,
});
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ec7eb605f424d31bc54abf01233cebb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1048&h=344&s=58874&e=gif&f=16&b=2e2e2e)

页面刷新后，1s 后更改了 DOM 内容。

在这段代码中，当我们使用 `$.state()`的时候，就会创建一个 Signal，在 Svelte 中，它本质是一个这样的对象（查看 `state` 函数）：

```javascript
{
  f: 0,
  v,
  reactions: null,
  equals: (value) => value === this.v,
  version: 0,
}
```

其中 `f` 是 `flags` 的缩写，依然采用了位掩码来标记 Signals 的状态。`v`是 `value`的缩写，表示 Signal 的值，`reactions` 表示副作用，当 Signals 的值发生修改时，会执行这些 reactions。

而在 Svelte 中，effect 本质是这样一个对象（查看 `create_effect`函数）：

```javascript
var effect = {
  ctx: component_context,
  deps: null,
  nodes_start: null,
  nodes_end: null,
  f: type | DIRTY,
  first: null,
  fn,
  last: null,
  next: null,
  prev: null,
  teardown: null,
  transitions: null,
  version: 0,
};
```

其中 `f`是 `flags`的缩写，也是位掩码形式。`fn`是具体执行的函数，`deps` 是依赖的 Signals，是一个数组形式。

当运行 `template_effect`的时候，会调用 `create_effect`创建 effect，在调用 `update_reaction`的时候：

```javascript
function update_reaction(reaction) {
  new_deps = null;

  // 这个函数会触发 get 函数，从而触发 new_deps 的赋值
  var result = reaction.fn();
  var deps = reaction.deps;

  if (new_deps !== null) {
    if (deps == null) {
      reaction.deps = deps = new_deps;
    }

    // 这里建立了 signals 和 effect 的依赖关系
    for (var i = 0; i < deps.length; i++) {
      (deps[i].reactions ??= []).push(reaction);
    }
  }

  return result;
}
```

执行 `reaction.fn()`即调用 `$.set_text(text,`Hello {.get(name) ?? ""}`)`，此时会触发 `$.get`，此时会触发 reaction 对应 Signals 的收集，也就是 `new_deps`这个数组。

当 `reaction.fn()`执行完毕后，我们会将 `new_deps`赋值到 `reaction.deps`中。然后我们遍历 `reaction.deps`再将 reaction 添加到这些 Signals 的 reactions 中，这样我们就建立了一个 signals 和 effect 之间的循环依赖：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc8fb830ad5843b3a089e7664f7651c0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2500&h=1232&s=249362&e=png&b=2c2c2c)

当然我们的目的并不是要建立循环依赖，只是要知道用到 Signals 的 Reactions 有哪些，Reactions 依赖的 Signals 有哪些。

当 1s 后调用 `update` 函数的时候，会调用：

```javascript
$.set(name, "Svelte");
```

name 就是 `Signals`，Svelte 会标记该 Signals 为 dirty，并且会调用该 Signals 对应的 Reactions。但 Svelte 并不会直接执行，而是调用了 `schedule_effect`，在该函数中：

```javascript
function schedule_effect(signal) {
  queueMicrotask(process_deferred);
  var effect = signal;
  queued_root_effects.push(effect);
}
```

关键在于 `queueMicrotask(process_deferred)`，`queueMicrotask` 是 JavaScript 提供的使用微任务的 API。

再看 `process_deferred`函数，最终会调用 `update_reaction`，在 `update_reaction` 函数中，我们会执行 `reaction.fn()`，也就是 `$.set_text(text,`Hello {.get(name) ?? ""}`)`，此时调用 `$.get`会获取到最新的值，也就是 `Svelte`，所以成功更新 DOM。

## 5\. 绑定事件

现在我们加入事件，将代码改为：

```xml
<script>
  let name = $state('World');
  function update() {
    name = 'Svelte';
  }
</script>
<button onclick={update}>Hello {name}</button>
```

查看 [REPL 生成的代码](https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAEz2MwQrCMBBEfyUsQiwUeq9pwZt3Dx6sh5psIZgmIdkIEvLvklY8zsybl2HRBiP09wx2XhF6OHsPLdDH1xDfaAihhehSkLURUQbtaZzsRAaJ1Rcb2CHSTHjkNxeM4s2pzkuykrSzLHlVx4blWk_0-_DrZucbXCYrur9bPBORs8xZabR8DXlXlPGCxjiWq6GIbqdGaGF1Si8aFfQUEpZH-QIEI7Uk2gAAAA== "https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAEz2MwQrCMBBEfyUsQiwUeq9pwZt3Dx6sh5psIZgmIdkIEvLvklY8zsybl2HRBiP09wx2XhF6OHsPLdDH1xDfaAihhehSkLURUQbtaZzsRAaJ1Rcb2CHSTHjkNxeM4s2pzkuykrSzLHlVx4blWk_0-_DrZucbXCYrur9bPBORs8xZabR8DXlXlPGCxjiWq6GIbqdGaGF1Si8aFfQUEpZH-QIEI7Uk2gAAAA==")：

```javascript
import * as $ from "svelte/internal/client";

function update(_, name) {
  $.set(name, 'Svelte');
}

var root = $.template(`<button> </button>`);

export default function App($$anchor) {
  let name = $.state('World');
  var button = root();

  button.__click = [update, name];

  var text = $.child(button);

  $.reset(button);
  $.template_effect(() => $.set_text(text, `Hello ${$.get(name) ?? ""}`));
  $.append($$anchor, button);
}

$.delegate(["click"]);
```

这个事件注册看起来有些奇怪，我们在 《Svelte5 | Snippets、事件处理程序以及其他更新》篇讲到：

> 如果要为元素添加事件，不要使用 addEventListener，而是使用 Svelte 提供的 `on`事件：

```javascript
import { on } from 'svelte/events';

const off = on(element, 'click', () => {
  console.log('element was clicked');
});

// 如果需要删除的话
off();
```

> 这是因为当我们使用 `onclick`的时候，其实并没有为元素本身创建事件处理程序，而是为了提升性能在根 DOM 元素上创建，所以如果你通过 `addEventListener`添加的事件一定比通过 `onclick`添加的事件早执行。可以查看这个 [Demo](https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE41Sy2rDMBD8lUUXJxDiu-sYeugt_YK6h8RaN6LyykgrQzH6965shxJooQc_RhrNzA6aVW8sBlW9zYouA6pKPY-jOij-GjMIE1pGwcFF3-WVOnTejNy01LIZRucZZnD06iIxJOi9G6BYjxVPmZQfiwzaTBkL2ti73R5ODcwLiftIHRtHcLuQtuhlc9tpuSyBbyZAuLloNfhIELBzpO8E-Q_O4tG6j13hIqO_y0BvPOpiv0bhtJ1Y3pLoeNH6ZULiswmMJLZFZ033WRzuAvstdMseOXqCh9SriMfBTfgPnZxg-aYM6_KnS6pFCK6GdJVHPc0C01JyfY0slUnHi-JpfgjwSzUycdgmfOjFEP3RS1qdhJ8dYMDFt1yNmxxU0jRyCwanTW9Qq4p9xPSevgHI3m43QAIAAA== "https://svelte-5-preview.vercel.app/#H4sIAAAAAAAAE41Sy2rDMBD8lUUXJxDiu-sYeugt_YK6h8RaN6LyykgrQzH6965shxJooQc_RhrNzA6aVW8sBlW9zYouA6pKPY-jOij-GjMIE1pGwcFF3-WVOnTejNy01LIZRucZZnD06iIxJOi9G6BYjxVPmZQfiwzaTBkL2ti73R5ODcwLiftIHRtHcLuQtuhlc9tpuSyBbyZAuLloNfhIELBzpO8E-Q_O4tG6j13hIqO_y0BvPOpiv0bhtJ1Y3pLoeNH6ZULiswmMJLZFZ033WRzuAvstdMseOXqCh9SriMfBTfgPnZxg-aYM6_KnS6pFCK6GdJVHPc0C01JyfY0slUnHi-JpfgjwSzUycdgmfOjFEP3RS1qdhJ8dYMDFt1yNmxxU0jRyCwanTW9Qq4p9xPSevgHI3m43QAIAAA==")。为了保持相对顺序，使用 `on`。

所以当你添加了 `onclick`这样的时间处理程序，会使用事件委托技术，为根 DOM 元素上的每个事件类型创建一个处理程序，而不是为每个元素创建一个处理程序，这是为了更好的性能和内存使用率。所以你通过`addEventListener`添加的事件一定比通过 `onclick`添加的事件更早执行，无论在 DOM 中的位置如何。

让我们提供一个完整可运行的极简版代码，可以看到它背后的实现机制：

新建 `svelte5-4.js`，完整可运行代码如下：

```javascript
const $ = {};

$.template = template;
$.append = append;
$.child = child;
$.reset = reset;
$.template_effect = template_effect;
$.set_text = set_text;
$.state = state;
$.get = get;
$.set = set;
$.delegate = delegate;

function state(v) {
  return {
    f: 0,
    v,
    reactions: null,
    equals: (value) => value === this.v,
    version: 0,
  };
}

function template(content, flags) {
  var node;
  return () => {
    if (node === undefined) {
      node = create_fragment_from_html(content);
      node = node.firstChild;
    }

    var clone = node.cloneNode(true);
    return clone;
  };
}

function create_fragment_from_html(html) {
  var elem = document.createElement("template");
  elem.innerHTML = html;
  return elem.content;
}

function append(anchor, dom) {
  anchor.before(dom);
}

function child(node) {
  return node.firstChild;
}

let hydrating = false;
function reset(node) {
  if (!hydrating) return;
}

const RENDER_EFFECT = 1 << 3;
const ROOT_EFFECT = 1 << 6;
const DIRTY = 1 << 10;

function template_effect(fn) {
  return render_effect(fn);
}
function render_effect(fn) {
  return create_effect(RENDER_EFFECT, fn, true);
}

let component_context = null;
let active_effect = null;
function create_effect(type, fn, sync, push = true) {
  var effect = {
    ctx: component_context,
    deps: null,
    nodes_start: null,
    nodes_end: null,
    f: type | DIRTY,
    first: null,
    fn,
    last: null,
    next: null,
    prev: null,
    teardown: null,
    transitions: null,
    version: 0,
  };

  if (sync) {
    update_effect(effect);
  }

  return effect;
}

function update_effect(effect) {
  update_reaction(effect);
}

let new_deps = null;

function update_reaction(reaction) {
  new_deps = null;

  // 这个函数会触发 get 函数，从而触发 new_deps 的赋值
  var result = reaction.fn();
  var deps = reaction.deps;

  if (new_deps !== null) {
    if (deps == null) {
      reaction.deps = deps = new_deps;
    }

    // 这里建立了 signals 和 effect 的依赖关系
    for (var i = 0; i < deps.length; i++) {
      (deps[i].reactions ??= []).push(reaction);
    }
  }

  return result;
}

function set_text(text, value) {
  if (value !== (text.__t ??= text.nodeValue)) {
    text.__t = value;
    text.nodeValue = value == null ? "" : value + "";
  }
}

function get(signal) {
  if (new_deps === null) {
    new_deps = [signal];
  }
  return signal.v;
}

function set(source, value) {
  if (!source.equals(value)) {
    source.v = value;
    mark_reactions(source, DIRTY);
  }

  return value;
}

function mark_reactions(signal, status) {
  signal.f |= status;
  var reactions = signal.reactions;
  if (reactions === null) return;

  for (var i = 0; i < reactions.length; i++) {
    var reaction = reactions[i];
    schedule_effect(reaction);
  }
}

let queued_root_effects = [];
function schedule_effect(signal) {
  queueMicrotask(process_deferred);
  var effect = signal;
  queued_root_effects.push(effect);
}

function process_deferred() {
  const previous_queued_root_effects = queued_root_effects;
  queued_root_effects = [];
  flush_queued_root_effects(previous_queued_root_effects);
}

function flush_queued_root_effects(root_effects) {
  var length = root_effects.length;
  if (length === 0) {
    return;
  }

  for (var i = 0; i < length; i++) {
    var effect = root_effects[i];
    process_effects(effect);
  }
}

function process_effects(effect) {
  update_effect(effect);
}

const all_registered_events = new Set();
const root_event_handles = new Set();
function delegate(events) {
  for (var i = 0; i < events.length; i++) {
    all_registered_events.add(events[i]);
  }

  for (var fn of root_event_handles) {
    fn(events);
  }
}

function handle_event_propagation(event) {
  var handler_element = this;
  var owner_document = /** @type {Node} */ (handler_element).ownerDocument;
  var event_name = event.type;
  var path = event.composedPath?.() || [];
  var current_target = /** @type {null | Element} */ (path[0] || event.target);

  // composedPath contains list of nodes the event has propagated through.
  // We check __root to skip all nodes below it in case this is a
  // parent of the __root node, which indicates that there's nested
  // mounted apps. In this case we don't want to trigger events multiple times.
  var path_idx = 0;

  // @ts-expect-error is added below
  var handled_at = event.__root;

  current_target = /** @type {Element} */ (path[path_idx] || event.target);
  // there can only be one delegated event per element, and we either already handled the current target,
  // or this is the very first target in the chain which has a non-delegated listener, in which case it's safe
  // to handle a possible delegated event on it later (through the root delegation listener for example).
  if (current_target === handler_element) return;

  var define_property = Object.defineProperty;
  var is_array = Array.isArray;
  // Proxy currentTarget to correct target
  define_property(event, "currentTarget", {
    configurable: true,
    get() {
      return current_target || owner_document;
    },
  });

  try {
    var throw_error;
    var other_errors = [];

    while (current_target !== null) {
      /** @type {null | Element} */
      var parent_element =
        current_target.assignedSlot ||
        current_target.parentNode ||
        /** @type {any} */ (current_target).host ||
        null;

      try {
        // @ts-expect-error
        var delegated = current_target["__" + event_name];
        if (
          delegated !== undefined &&
          !(/** @type {any} */ (current_target).disabled)
        ) {
          if (is_array(delegated)) {
            var [fn, ...data] = delegated;
            fn.apply(current_target, [event, ...data]);
          } else {
            delegated.call(current_target, event);
          }
        }
      } catch (error) {
        if (throw_error) {
          other_errors.push(error);
        } else {
          throw_error = error;
        }
      }
      if (
        event.cancelBubble ||
        parent_element === handler_element ||
        parent_element === null
      ) {
        break;
      }
      current_target = parent_element;
    }

    if (throw_error) {
      for (let error of other_errors) {
        // Throw the rest of the errors, one-by-one on a microtask
        queueMicrotask(() => {
          throw error;
        });
      }
      throw throw_error;
    }
  } finally {
    // @ts-expect-error is used above
    event.__root = handler_element;
    // @ts-ignore remove proxy on currentTarget
    delete event.currentTarget;
  }
}

function mount(Component, { target, anchor }) {
  var anchor_node = anchor ?? target.appendChild(document.createTextNode(""));
  Component(anchor_node);
  document.addEventListener("click", handle_event_propagation, {
    passive: false,
  });
}

// 编译后代码：
function update(_, name) {
  $.set(name, "Svelte");
}

var root = $.template(`<button> </button>`);

function App($$anchor) {
  let name = $.state("World");
  var button = root();

  button.__click = [update, name];

  var text = $.child(button);

  $.reset(button);
  $.template_effect(() => $.set_text(text, `Hello ${$.get(name) ?? ""}`));
  $.append($$anchor, button);
}

$.delegate(["click"]);

mount(App, {
  target: document.body,
});
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/269e4f8572e54e42b216da52fa9bd63d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1048&h=344&s=64137&e=gif&f=20&b=2d2d2d)

主要的实现代码在 `handle_event_propagation`函数中，当点击的时候，获取点击元素，从该元素的 `__click`属性中获取处理程序和数据，然后调用 `update`函数，剩下的流程与上节相同。

## 6\. 最后

至此我们就完成了 Svelte5 一个最基本的例子的源码流程分析。如果对细节不太清楚，可以使用提供的极简版代码调试查看运行流程，或者参考《原理篇 | 如何调试 Svelte5 的代码》调试完整的源码。极简版被我删减了非常多的地方，部分地方可能看起来有些多余，如果不清楚作用，可以参照源码对比。

可以看到 Svelte 5 就是基于 Signals 的设计思路进行实现，会在组件首次运行的时候收集 Signals 依赖的 Reactions，在 Signals 的值发生更改的时候，就会运行这些 Reactions，从而直接完成 DOM 更新。

可是 Svelte 4 不也是直接更新 DOM 吗？所以到底有什么改变呢？

从表面上看，Svelte4 是在编译的时候分析依赖项，而 Svelte5 允许 Svelte 在运行时跟踪依赖项，这会带来性能上的优化，可以参考《Svelte5 | Runes 初识 》中关于细粒度响应式的例子。

而在更深层面，Svelte4 实现响应式是通过覆盖 JavaScript 原生行为来实现的，也就是通过劫持 `let` 和 `$`，让开发者写着原生 JavaScript 的语法，但却能实现响应式更新。本质写的并不是 JavaScript，而是一种类似于 JavaScript 的语言，所以那些 let 声明只能放在 `.svelte`组件顶层，放在其他地方 Svelte 就不会做特殊处理了。

但 Svelte5 虽然使用符文，让原本的代码稍微长了一点，但并没有覆盖 JavaScript 的原生行为，所以 Svelte 5 的代码看起来不像 JavaScript，但行为上却是 JavaScript，所以你写的符文代码可以在 `.svelte`文件中使用，也可以在 `.svelte.js`、`.svelte.ts`文件中使用，所以创建状态可以在一个单独的 js 文件中，然后通过导入的方式使用，这让 Svelte 的代码更加灵活。 这是两个版本非常大的不同之处。

## 恭喜你！

最后的最后，恭喜你完成了第四阶段 —— Svelte 原理的学习：

1. 第一阶段：Svelte 5 🎉
2. 第二阶段：SvelteKit 🎉
3. 第三阶段：项目实战 🎉
4. 第四阶段：Svelte 原理 🎉

靡不有初，鲜克有终。我相信绝大部分的读者应该都没有完整阅读完小册内容，但没有关系，因为也没有必要完全看完，我们学习技术的目的在于应用，正确的学习方式应该是边学边用，边用边学，在实际项目开发中解决问题才会帮助你真正理解掌握，所以对于这本小册大部分的内容只用了解即可，知道有这么个东西，真正用到的时候再来这本小册细查用法即可。

感谢阅读这本小册的你，通过文字建立交流本身就是一种缘分，感谢你的阅读和支持。由于作者水平有限，书中难免有错误或不妥之处，欢迎广大读者批评指正。