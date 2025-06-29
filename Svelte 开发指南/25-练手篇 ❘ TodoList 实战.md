> 推荐学习指数：⭐️️⭐️️⭐️️

## 1\. 前言

在正式开始实战篇之前，我们先使用 Svelte 5 和 SvelteKit 实现一个全栈 TodoList，用于熟悉 Svelte 的基本语法和掌握前后端交互方式。TodoList 的效果并不复杂：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b40be5a983eb4deda1cea03626775c27~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1079&h=704&s=157813&e=gif&f=114&b=fefefe)

虽然效果看起来并不复杂，但麻雀虽小，五脏俱全。大多数的应用本质上就是“增删改查”，所以这样一个 TodoList 是最好的学习方式。

此外，为了让交互效果更加流畅，TodoList 实现了乐观更新：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a4bc6c1ac6744d58903ff6aa47821d6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1080&h=717&s=95208&e=gif&f=51&b=fefefe)

所谓乐观更新，说白了就是乐观的假设操作会成功，先更新 UI，同时发送数据请求。如果数据请求成功，相安无事，用户感受到流畅的操作，提升了用户体验，数据也得到更新。如果更新失败，则视情况对错误进行处理。

以我们实现的 TodoList 为例，如果请求成功，应用操作就会如第一张动图中那样流畅，如果请求失败，则会像第二张动图中那样给与用户错误提示并回退数据。

那就让我们开始吧。

## 2\. 项目初始化

运行：

```bash
npx sv create svelte-todolist
```

选择 **SvelteKit minimal、TypeScript、prettier、eslint、tailwindcss**：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a39e56e71b2463e99f5c1a8b1dd389f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2416&h=1048&s=193501&e=png&b=1e1e1e)

按照命令行中的提示提交代码：

```bash
cd svelte-todolist

git init && git add -A && git commit -m "Initial commit" (optional)  

npm run dev -- --open   
```

浏览器效果如下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b569f8e7dae431d9221fb85ab382d12~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1800&h=638&s=52578&e=png&b=ffffff)

## 3\. 纯前端实现

我们先纯前端实现一遍。因为代码并不复杂，加起来也才一两百行代码，所以我们直接给出最终的代码。

新建 `src/routes/todo/remove.svg`，代码如下：

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
    <path fill="#888" stroke="none" d="M22 4.2h-5.6L15 1.6c-.1-.2-.4-.4-.7-.4H9.6c-.2 0-.5.2-.6.4L7.6 4.2H2c-.4 0-.8.4-.8.8s.4.8.8.8h1.8V22c0 .4.3.8.8.8h15c.4 0 .8-.3.8-.8V5.8H22c.4 0 .8-.3.8-.8s-.4-.8-.8-.8zM10.8 16.5c0 .4-.3.8-.8.8s-.8-.3-.8-.8V10c0-.4.3-.8.8-.8s.8.3.8.8v6.5zm4 0c0 .4-.3.8-.8.8s-.8-.3-.8-.8V10c0-.4.3-.8.8-.8s.8.3.8.8v6.5z" />
</svg>
```

这是我们的删除按钮 SVG 图片。

新建 `src/routes/todo/+layout.svelte`，代码如下：

```xml
<script lang="ts">
  import type { Snippet } from 'svelte';
  const {
  children
  }: {
  children: Snippet;
  } = $props();
</script>

<div class="m-10">
  <h1 class="mb-4 block text-2xl font-bold">今日清单</h1>
  {@render children()}
</div>
```

> 注：这段代码其实并不是必要的，全部放在 +page.svelte 也可以，只是帮助大家再熟悉一下插槽的用法

新建 `src/routes/todo/+page.svelte`，代码如下：

```xml
<script lang="ts">
  import { fly } from 'svelte/transition';
  import DeleteBtn from './remove.svg';

  type Todo = {
    id: string;
    text: string;
    done: boolean;
  };

  type Filters = 'all' | 'todo' | 'done';

  let todos = $state<Todo[]>([
    { id: 'gewrii', text: '阅读', done: false },
    { id: '4m9ha4', text: '写作', done: false },
    { id: 'b9u33z', text: '冥想', done: false }
  ]);

  let filter = $state<Filters>('all');

  let filteredTodos = $derived(filterTodos());

  function filterTodos() {
    switch (filter) {
      case 'all':
        return todos;
      case 'todo':
        return todos.filter((t) => !t.done);
      case 'done':
        return todos.filter((t) => t.done);
      default:
        return todos;
    }
  }

  function onAddTodo(event: KeyboardEvent) {
    if (event.key !== 'Enter') return;

    const todoElement = event.target as HTMLInputElement;

    todos.push({
      id: Math.random().toString(36).slice(-6),
      text: todoElement.value,
      done: false
    });

    todoElement.value = '';
  }

  function onRemoveTodo(event: Event) {
    const buttonEl = event.target as HTMLInputElement;
    const id = buttonEl.dataset.id;
    const index = todos.findIndex((t) => t.id === id);
    todos.splice(index, 1);
  }

  function setFilter(newFilter: Filters) {
    filter = newFilter;
  }

  function remaining(todos: Todo[]) {
    console.log('recalculating');
    return todos.filter((t) => !t.done).length;
  }
</script>

<input
  placeholder="添加待办事项"
  class="mb-4 block w-full rounded-md border-0 p-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
  onkeydown={onAddTodo}
/>

<div class="mb-4 flex items-center gap-4">
  {#each [['all', '全部'], ['todo', '待办'], ['done', '已完成']] as [filter, filterName]}
    <button
      class="flex w-full justify-center rounded-md bg-indigo-500 px-3 py-1.5 text-sm font-semibold leading-6 text-white"
      onclick={() => {
        setFilter(filter as Filters);
      }}>{filterName}</button
    >
  {/each}
</div>

<ul class="mb-4 flex flex-col gap-4">
  {#each filteredTodos as todo, index (todo.id)}
    <li class="flex items-center gap-4" class:opacity-40={todo.done} in:fly={{ y: -20 }}>
      <input type="checkbox" bind:checked={todo.done} />
      <input
        type="text"
        class="block flex-1 rounded-md border-0 p-1.5 text-gray-900 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
        bind:value={todo.text}
      />
      <button onclick={onRemoveTodo} aria-label="Remove">
        <img src={DeleteBtn} class="w-4" alt="删除按钮" data-id={todo.id} />
      </button>
    </li>
  {/each}
</ul>

<p>剩余待办事项：{remaining(todos)}</p>
```

在这段代码中，因为并没有提交按钮，所以我们监听了输入框的 `onkeydown` 事件，在回车的时候获取输入框中的值并创建 Todo。Todo 创建后，checkbox 和 Todo 修改输入框我们都采用了 bind 进行双向绑定，可以快捷的修改值。点击筛选器按钮，我们通过 `filteredTodos`派生状态获取筛选后的值。点击删除按钮，我们从 todo 数组中删除对应 id 的值。

在这个例子中，我们演示了两种获取对应点击元素数据的的方法，一种是类似于筛选器按钮，通过函数参数传递：

```xml
{#each [['all', '全部'], ['todo', '待办'], ['done', '已完成']] as [filter, filterName]}
  <button
    class="flex w-full justify-center rounded-md bg-indigo-500 px-3 py-1.5 text-sm font-semibold leading-6 text-white"
    onclick={() => {
      setFilter(filter as Filters);
    }}>{filterName}</button
  >
{/each}
```

一种是类似于删除按钮，将数据放到 `data-xxx`属性，然后通过 `event.target.dataset.xxx`获取：

```xml
{#each filteredTodos as todo, index (todo.id)}
  <li class="flex items-center gap-4" class:opacity-40={todo.done} in:fly={{ y: -20 }}>
    <button onclick={onRemoveTodo} aria-label="Remove">
      <img src={DeleteBtn} class="w-4" alt="删除按钮" data-id={todo.id} />
    </button>
  </li>
{/each}
```

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75c83bb4929b40b2963f09e3fac9def1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1075&h=727&s=185628&e=gif&f=84&b=fefefe)

## 4\. 全栈实现

因为是纯前端实现，所以页面刷新后数据就会丢失，现在我们将这个 TodoList 改为全栈实现：

新建 `src/lib/index.ts`，代码如下：

```typescript
export type Todo = {
  id: string;
  text: string;
  done: boolean;
};

export type Filters = 'all' | 'todo' | 'done';
```

因为客户端与服务端都会用到这些类型，所以我们放到 `lib` 目录下方便引用。

新建 `src/lib/utils.ts`，代码如下：

```typescript
export const sleep = (ms: number) => new Promise((resolve) => setTimeout(resolve, ms));
```

`sleep` 函数用于模拟请求延时。

新建 `src/routes/todo/+page.server.ts`，代码如下：

```typescript
import { fail } from "@sveltejs/kit";
import { sleep } from "$lib/utils";
import type { Todo } from "$lib";

const todos: Todo[] = [];

export function load() {
  return {
    todos,
  };
}

export const actions = {
  createTodo: async ({ request }) => {
    await sleep(1000);

    const data = await request.formData();
    const text = data.get("text") as string;

    try {
      const newTodo = {
        id: crypto.randomUUID(),
        text,
        done: false,
      };

      todos.push(newTodo);
      return { todos };
    } catch (error: any) {
      return fail(422, { error: error.message });
    }
  },
  deleteTodo: async ({ request }) => {
    await sleep(1000);

    const data = await request.formData();
    const id = data.get("id") as string;

    try {
      const index = todos.findIndex((todo) => todo.id === id);
      if (index !== -1) todos.splice(index, 1);
    } catch (error: any) {
      return fail(422, { error: error.message });
    }
  },
  toggleTodo: async ({ request }) => {
    await sleep(1000);

    const data = await request.formData();
    const id = data.get("id") as string;

    try {
      const todo = todos.find((todo) => todo.id === id);
      if (todo) {
        todo.done = !todo.done;
      }
    } catch (error: any) {
      return fail(422, { error: error.message });
    }
  },
  editTodo: async ({ request }) => {
    await sleep(1000);

    const data = await request.formData();
    const id = data.get("id") as string;

    try {
      const todo = todos.find((todo) => todo.id === id);
      if (todo) {
        todo.text = data.get("text") as string;
      }
    } catch (error: any) {
      return fail(422, { error: error.message });
    }
  },
};
```

这里我们创建了 4 个用于前后端交互的 Form Actions。

修改 `src/routes/todo/+page.svelte`，完整代码如下：

```typescript
<script lang="ts">
  import type { Todo, Filters } from '$lib';
  import { fly, slide } from 'svelte/transition';
  import { enhance, applyAction, deserialize } from '$app/forms';
  import type { ActionResult } from '@sveltejs/kit';
  import { invalidateAll } from '$app/navigation';
  import DeleteBtn from './remove.svg';

  let { data, form } = $props();
  const { todos: todoList = [] } = data;

  let loading = $state(false);
  let todos = $state<Todo[]>(todoList);
  let filter = $state<Filters>('all');

  let filteredTodos = $derived(filterTodos());

  function filterTodos() {
    switch (filter) {
      case 'all':
        return todos;
      case 'todo':
        return todos.filter((t) => !t.done);
      case 'done':
        return todos.filter((t) => t.done);
      default:
        return todos;
    }
  }

  $effect(() => {
    todos = data.todos;
  });

  const onToggleTodo = (todo: any) => async (event: Event) => {
    event.preventDefault();
    const checkboxEl = event.target as HTMLInputElement;

    const formData = new FormData();
    formData.append('id', todo.id);
    formData.append('done', checkboxEl.checked.toString());
    const response = await fetch('?/toggleTodo', {
      method: 'POST',
      body: formData
    });

    const result: ActionResult = deserialize(await response.text());

    if (result.type === 'success') {
      await invalidateAll();
    }

    applyAction(result);
  };

  const onEditTodo = (todo: any) => async (event: Event) => {
    event.preventDefault();
    const inputEl = event.target as HTMLInputElement;
    const formData = new FormData();
    formData.append('id', todo.id);
    formData.append('text', inputEl.value);
    const response = await fetch('?/editTodo', {
      method: 'POST',
      body: formData
    });

    const result: ActionResult = deserialize(await response.text());

    if (result.type === 'success') {
      await invalidateAll();
    }

    applyAction(result);
  };

  function remaining(todos: Todo[]) {
    return todos.filter((t) => !t.done).length;
  }

  function setFilter(newFilter: Filters) {
    filter = newFilter;
  }
</script>

<form
  action="?/createTodo"
  method="POST"
  use:enhance={() => {
    loading = true;
    return async ({ update }) => {
      loading = false;
      update();
    };
  }}
>
  <input
    placeholder="添加待办事项"
    class="mb-4 block w-full rounded-md border-0 p-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
    name="text"
    disabled={loading}
  />
  <button type="submit" class="hidden">提交</button>
</form>

{#if form?.error}
  <p in:fly={{ y: 20 }} class="mb-4 rounded-md bg-red-400 px-4 py-2 text-white">
    {form.error}
  </p>
{/if}

<div class="mb-4 flex items-center gap-4">
  {#each [['all', '全部'], ['todo', '待办'], ['done', '已完成']] as [filter, filterName]}
    <button
      class="flex w-full justify-center rounded-md bg-indigo-500 px-3 py-1.5 text-sm font-semibold leading-6 text-white"
      onclick={() => {
        setFilter(filter as Filters);
      }}>{filterName}</button
    >
  {/each}
</div>

<ul class="mb-4 flex flex-col gap-4">
  {#each filteredTodos as todo (todo.id)}
    <li class:opacity-40={todo.done} class="flex items-center gap-4">
      <form action="?/toggleTodo" method="POST" use:enhance>
        <input type="hidden" name="id" value={todo.id} />
        <input type="checkbox" checked={todo.done} onclick={onToggleTodo(todo)} />
      </form>

      <input
        type="text"
        class="block flex-1 rounded-md border-0 p-1.5 text-gray-900 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
        value={todo.text}
        onchange={onEditTodo(todo)}
      />

      <form action="?/deleteTodo" method="POST" use:enhance>
        <input type="hidden" name="id" value={todo.id} />
        <button aria-label="Remove" type="submit">
          <img src={DeleteBtn} class="w-4" alt="删除按钮" />
        </button>
      </form>
    </li>
  {/each}
  {#if loading}
    <span class="loading">Loading...</span>
  {/if}
</ul>

<p>剩余待办事项：{remaining(todos)}</p>
```

在这段代码中，我们演示了两种与 Form Actions 交互的方式，一种是传统的使用 `<form>`标签：

```typescript
<form
  action="?/createTodo"
  method="POST"
  use:enhance={() => {
    loading = true;
    return async ({ update }) => {
      loading = false;
      update();
    };
  }}
>
  <input
    placeholder="添加待办事项"
    class="mb-4 block w-full rounded-md border-0 p-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
    name="text"
    disabled={loading}
  />
  <button type="submit" class="hidden">
    提交
  </button>
</form>
```

一种则是通过 `fetch` 的方式：

```xml
<script lang="ts">
const onToggleTodo = (todo: any) => async (event: Event) => {
    event.preventDefault();
    const checkboxEl = event.target as HTMLInputElement;

    const formData = new FormData();
    formData.append('id', todo.id);
    formData.append('done', checkboxEl.checked.toString());
    const response = await fetch('?/toggleTodo', {
      method: 'POST',
      body: formData
    });

    const result: ActionResult = deserialize(await response.text());

    if (result.type === 'success') {
      await invalidateAll();
    }

    applyAction(result);
  };

</script>

{#each filteredTodos as todo (todo.id)}
  <li class:opacity-40={todo.done} class="flex items-center gap-4">
    <input type="checkbox" checked={todo.done} onclick={onToggleTodo(todo)} />
  </li>
{/each}
```

如果能使用第一种 `<form>`标签的形式，还是尽可能使用第一种，因为它支持渐进式增强，即便禁用 JavaScript 也可以使用。第二种方式虽然可以用，但其实跟调用 API 没太大区别，主要的区别在于利用了 `+page.server.js`的 `actions`，省了在 `+server.js`声明 API 接口的功夫。

除了这两种与后端交互的形式，还有就是传统的 fetch API 的形式，后端通过 `+server.js`声明接口，然后前端调用接口。

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18c4ed8ffc574e038da3d50d996c55ce~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1078&h=661&s=123260&e=gif&f=71&b=fefefe)

此时交互效果基本与之前一致，但因为我们在接口中使用了 `sleep(1000)`模拟接口延迟，当我们删除或者切换任务状态的时候，因为有 1s 左右的延迟，所以交互显得有些“卡”。

## 5\. 乐观更新

如果我想要实现乐观更新该如何实现呢？其实并不复杂，我们为 Todo 的切换状态、编辑、删除添加上乐观更新效果：

修改 `src/routes/todo/+page.server.ts`，完整代码如下：

```typescript
import { fail } from "@sveltejs/kit";
import { sleep } from "$lib/utils";
import type { Todo } from "$lib";

const todos: Todo[] = [];

export function load() {
  return {
    todos,
  };
}

export const actions = {
  createTodo: async ({ request }) => {
    await sleep(1000);

    const data = await request.formData();
    const text = data.get("text") as string;

    try {
      const newTodo = {
        id: crypto.randomUUID(),
        text,
        done: false,
      };

      todos.push(newTodo);
      return { todos };
    } catch (error: any) {
      return fail(422, { error: error.message });
    }
  },
  deleteTodo: async ({ request }) => {
    await sleep(1000);

    const data = await request.formData();
    const id = data.get("id") as string;

    try {
      throw new Error("删除事项失败");
      const index = todos.findIndex((todo) => todo.id === id);
      if (index !== -1) todos.splice(index, 1);
    } catch (error: any) {
      return fail(422, { error: error.message });
    }
  },
  toggleTodo: async ({ request }) => {
    await sleep(1000);

    const data = await request.formData();
    const id = data.get("id") as string;

    try {
      throw new Error("修改事项失败");
      const todo = todos.find((todo) => todo.id === id);
      if (todo) {
        todo.done = !todo.done;
      }
    } catch (error: any) {
      return fail(422, { error: error.message });
    }
  },
  editTodo: async ({ request }) => {
    await sleep(1000);

    const data = await request.formData();
    const id = data.get("id") as string;

    try {
      throw new Error("修改事项失败");
      const todo = todos.find((todo) => todo.id === id);
      if (todo) {
        todo.text = data.get("text") as string;
      }
    } catch (error: any) {
      return fail(422, { error: error.message });
    }
  },
};
```

主要是添加了三行 `throw new Error` 模拟出错时的情况。

修改 `src/routes/todo/+page.svelte`，完整代码如下：

```typescript
<script lang="ts">
  import type { Todo, Filters } from '$lib';
  import { fly, slide } from 'svelte/transition';
  import { enhance, applyAction, deserialize } from '$app/forms';
  import type { ActionResult } from '@sveltejs/kit';
  import { invalidateAll } from '$app/navigation';
  import DeleteBtn from './remove.svg';

  let { data, form } = $props();

  const { todos: todoList = [] } = data;

  let loading = $state(false);
  let deleting: string[] = $state([]);

  let todos = $state<Todo[]>(todoList);
  let filter = $state<Filters>('all');

  let filteredTodos = $derived(filterTodos());

  function filterTodos() {
    switch (filter) {
      case 'all':
        return todos;
      case 'todo':
        return todos.filter((t) => !t.done);
      case 'done':
        return todos.filter((t) => t.done);
      default:
        return todos;
    }
  }

  $effect(() => {
    todos = data.todos;
  });

  const onToggleTodo = (todo: any) => async (event: Event) => {
    event.preventDefault();
    const checkboxEl = event.target as HTMLInputElement;

    const formData = new FormData();
    formData.append('id', todo.id);
    formData.append('done', checkboxEl.checked.toString());
    const response = await fetch('?/toggleTodo', {
      method: 'POST',
      body: formData
    });

    const result: ActionResult = deserialize(await response.text());

    if (result.type === 'success' || result.type === 'failure') {
      await invalidateAll();
    }

    applyAction(result);
  };

  const onEditTodo = (todo: any) => async (event: Event) => {
    event.preventDefault();
    const inputEl = event.target as HTMLInputElement;
    const formData = new FormData();
    formData.append('id', todo.id);
    formData.append('text', inputEl.value);
    const response = await fetch('?/editTodo', {
      method: 'POST',
      body: formData
    });

    const result: ActionResult = deserialize(await response.text());

    if (result.type === 'success' || result.type === 'failure') {
      await invalidateAll();
    }

    applyAction(result);
  };

  function remaining(todos: Todo[]) {
    return todos.filter((t) => !t.done).length;
  }

  function setFilter(newFilter: Filters) {
    filter = newFilter;
  }
</script>

<form
  action="?/createTodo"
  method="POST"
  use:enhance={() => {
    loading = true;
    return async ({ update }) => {
      loading = false;
      update();
    };
  }}
>
  <input
    placeholder="添加待办事项"
    class="mb-4 block w-full rounded-md border-0 p-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
    name="text"
    disabled={loading}
  />
  <button type="submit" class="hidden">提交</button>
</form>

{#if form?.error}
  <p in:fly={{ y: 20 }} class="mb-4 rounded-md bg-red-400 px-4 py-2 text-white">
    {form.error}
  </p>
{/if}

<div class="mb-4 flex items-center gap-4">
  {#each [['all', '全部'], ['todo', '待办'], ['done', '已完成']] as [filter, filterName]}
    <button
      class="flex w-full justify-center rounded-md bg-indigo-500 px-3 py-1.5 text-sm font-semibold leading-6 text-white"
      onclick={() => {
        setFilter(filter as Filters);
      }}>{filterName}</button
    >
  {/each}
</div>

<ul class="mb-4 flex flex-col gap-4">
  {#each filteredTodos.filter((todo) => !deleting.includes(todo.id)) as todo (todo.id)}
    <li class:opacity-40={todo.done} class="flex items-center gap-4">
      <input type="checkbox" bind:checked={todo.done} onchange={onToggleTodo(todo)} />

      <input
        type="text"
        class="block flex-1 rounded-md border-0 p-1.5 text-gray-900 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600"
        bind:value={todo.text}
        onchange={onEditTodo(todo)}
      />

      <form
        action="?/deleteTodo"
        method="POST"
        use:enhance={() => {
          deleting = [...deleting, todo.id];
          return async ({ update }) => {
            await update();
            deleting = deleting.filter((id) => id !== todo.id);
          };
        }}
      >
        <input type="hidden" name="id" value={todo.id} />
        <button aria-label="Remove" type="submit">
          <img src={DeleteBtn} class="w-4" alt="删除按钮" />
        </button>
      </form>
    </li>
  {/each}
  {#if loading}
    <span class="loading">Loading...</span>
  {/if}
</ul>

<p>剩余待办事项：{remaining(todos)}</p>
```

我们是如何实现乐观更新的呢？首先是删除按钮，我们添加了一个 `deleting`状态，点击删除按钮后，会立刻进入 `deleting`状态，在遍历的时候，处于 `deleting`状态的 Todo 不会被渲染，于是实现了立刻删除效果。在接口结果返回后，`deleting`状态去除，如果接口出错，删除的 Todo 恢复。如果接口成功，数据重新加载，UI 界面保持一致。

然后是 `<input type="checkbox">` 和 `<input type="text">`，我们改为监听 `onchange`事件，当用户操作时，内容会先修改，然后再触发 `onchange`事件，也就是会提交修改后的值。如果接口失败，我们也调用 `invalidateAll()`，这会导致数据重新加载，因为使用了双向绑定，所以数据会恢复之前的值。

浏览器效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51a5aa45dc16480aa88f7091824bec05~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1080&h=723&s=84586&e=gif&f=56&b=fefefe)

## 6\. 总结

本篇我们使用 Svelte5 和 SvelteKit 实现了一个全栈的 TodoList，麻雀虽小，五脏俱全，应用包含了数据的增删改查，一些大型的应用也不过是增删改查的堆叠，所以这个应用的实现对于上手 Svelte 和 SvelteKit 有很多帮助。但在实际开发中，我们并不会只使用 Svelte，还会搭配一些技术选型实现快速开发。实战篇开始我们会介绍 Svelte 项目主流搭配的一些技术选型，那就让我们继续学习吧！