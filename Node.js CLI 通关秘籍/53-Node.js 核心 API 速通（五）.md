这节我们来过一下剩下的 Node.js 的内置模块：

## vm

vm 模块可以创建一个 JS 执行环境。

比如我们写单测的时候，你会发现 test、describe 这些都是全局变量，但我们写其他 JS 代码的时候就没有这些。

很明显，test 脚本跑在一个单独的环境里。

那如何创建一个单独的 JS 运行环境呢？

就是用 vm 模块。

创建 vm.mjs

```javascript
import vm from 'node:vm';

const context = {
    console,
    guang: 111,
    dong: 222
}

vm.createContext(context);

vm.runInContext('console.log(guang + dong)', context);
```

我们用 vm.createContext 创建了一个上下文。

有 console、guang、dong 这三个全局变量。

然后用 runInContext 来跑了一段 JS 代码，传入 context。

跑一下：

```
node vm.mjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c1d356271f14158b38b2cb794282d1e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=358&h=84&s=11510&e=png&b=181818)

其实你每天跑的单测，实现原理就是这个。

自定义了 context，所以才可以直接用 test、describe 等全局 api。

## repl

repl 是 Read-Eval-Print-Loop，也就是这个东西：

![2025-01-07 14.04.13.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7849a2a4e4c4f3f9ad9a646b8a57360~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=974&h=664&s=245006&e=gif&f=52&b=020202)

可以交互式的来执行一些脚本。

如果你也想创建这种 repl，直接用 repl 模块就行。

创建 repl.mjs

```javascript
import repl from 'node:repl';

const r = repl.start({ prompt: '> ', eval: myEval});

function myEval(cmd, context, filename, callback) {
    console.log('你输入的命令：' + cmd);
    callback();
}
```

创建一个 repl，指定 eval 的处理函数。

cmd 是用户输入的内容

context 是 node 的所有 api，比如 context.fs

最后调用 callback 才会结束处理，所以可以异步处理，处理完调用 callback

跑一下：

```bash
node ./repl.mjs
```

![2025-01-07 14.44.12.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59f2b27052354bd79f9774c93ee40d52~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=974&h=664&s=140605&e=gif&f=44&b=171717)

我们引入 cfonts 来做下艺术字：

```css
npm install --save cfonts
```

改下代码：

```javascript
import repl from 'node:repl';
import cfonts from 'cfonts';

const r = repl.start({ prompt: '> ', eval: myEval});

function myEval(cmd, context, filename, callback) {
    cfonts.say(cmd, {
        font: '3D',
        colors: ['yellow', 'cyan']
    });
    callback();
}

```

跑一下：

![2025-01-07 14.49.27.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/720ba71b3078448c858e7191d3691f88~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1308&h=784&s=377245&e=gif&f=57&b=171717)

## dns

dns 模块，顾名思义就是用来查询域名对应的 IP 的。

试一下：

创建 dns.mjs

```javascript
import dns from 'node:dns/promises';

async function main() {
    const res = await dns.resolve('juejin.com');
    console.log(res);
}

main();
```

跑一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8309afe562c9452888fe8eeab3addf0f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=392&h=84&s=13451&e=png&b=181818)

## string\_decoder

这个模块是用来从 Buffer 根据编码解析成字符串的。

创建 string\_decoder.mjs

```javascript
import { StringDecoder } from 'node:string_decoder';
import { Buffer } from 'node:buffer';

const decoder = new StringDecoder('utf8');

const buf = Buffer.from('神说要有光', 'utf-8');
console.log(decoder.write(buf));
```

我们学过 Buffer.from 是用来创建 buffer 的，然后用 StringDecoder 就可以把 Buffer 解析为字符串。

跑一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d30724f85d74dbab40497d6d10958c8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=524&h=84&s=16475&e=png&b=191919)

## worker\_threads

我们学过 child\_process 是用来创建子进程的，其实 node 也支持创建工作线程。

就是 worker\_threads 模块。

比如很大计算量的任务，放在主线程会很耗时，我们完全可以把它拆到工作线程中去，异步处理。

创建 worker\_threads\_main.mjs

```javascript
import { Worker, MessageChannel } from 'node:worker_threads';

const { port1, port2 } = new MessageChannel();

const worker = new Worker('./worker_threads_worker.mjs');
worker.postMessage(
    { value: 10*10000*10000, channel: port2 },
    [port2]
);

port1.on('message', (value) => {
    console.log('res', value);
})
```

创建主线程，用 new Worker 创建工作线程。

MessageChannel 是创建通信的通道，它有两个端口 port1 和 port2。

你在 port1 里 postMessage，在 port2 里就可以用 message 事件接收到消息，反过来也是。

很容易理解，就像一个管子的两个口：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fdd2275daac4a04a4dbb62c98a58ee0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=884&h=634&s=849085&e=png&b=979591)

其实和浏览器里的工作线程的 api 是差不多的：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfe7a31e4b374c92ba3854e7a711b4b2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1250&h=572&s=46734&e=png&b=ffffff)

然后创建工作线程 worker\_threads\_worker.mjs

```javascript
import { parentPort } from 'node:worker_threads';

function calc(num) {
    let total = 0;
    for(let i = 0; i< num; i++) {
        total += i;
    }
    return total
}

parentPort.on('message', (message) => {
    const res = calc(message.value);

    message.channel.postMessage(res);
});
```

工作线程通过 parentPort 和父线程通信。

跑一下：

```bash
node ./worker_threads_main.mjs
```

![2025-01-07 15.08.43.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/031b7ecab8ff4bcab6b1d2c230befcf2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1338&h=606&s=38031&e=gif&f=30&b=181818)

可以看到，工作线程计算出了结果，并且异步的返回了。

## readline

readline 字面意思来看是一行一行的读取，这确实是它的功能之一。

创建 readline.mjs

```javascript
import { createReadStream } from 'node:fs';
import { createInterface } from 'node:readline';

const rl = createInterface({
  input: createReadStream('./repl.mjs')
});

rl.on('line', (line) => {
  console.log(`Line from file: ${line}`);
});
```

按照行来读取文件可读流里的内容。

跑一下：

```
node readline.mjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1aa8d385fe34565b0b8e483ab725a90~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=986&h=410&s=67951&e=png&b=181818)

当然，它的 input 也可以是标准输入 stdin，这样就变成了类似 repl 的效果：

创建 readline2.mjs

```javascript

import { createInterface } from 'node:readline';
import { exit, stdin, stdout } from 'node:process';

const rl = createInterface({
  input: stdin,
  output: stdout,
  prompt: 'guang> ',
});

rl.prompt();

rl.on('line', (line) => {
  switch (line.trim()) {
    case 'hello':
      console.log('world!');
      break;
    default:
      console.log(`你说啥？我听到的是 '${line.trim()}'`);
      break;
  }
  rl.prompt();
}).on('close', () => {
  console.log('bye!');
  exit(0);
});
```

跑一下：

![2025-01-07 22.21.42.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd56b083c69a457d806510c21816252a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1338&h=606&s=69102&e=gif&f=50&b=181818)

同样是按行读取，只不过现在读取的是 stdin，也就是用户输入。

当用户按回车的时候来处理。

而且还支持输出 prompt，这样确实和 repl 效果一样了。

此外，readline 还可以支持键盘输入。

之前我们写过一个例子：

创建 readline3.mjs

```javascript
import readline from 'node:readline';

readline.emitKeypressEvents(process.stdin);

process.stdin.setRawMode(true);
 
process.stdin.on('keypress', (str, key) => {
 
    console.log(str, key)

    if(key.sequence === '\u0003') {
        process.exit();
    }
});
```

readline 模块的 emitKeypressEvents 可以让输入流处理键盘事件。

然后 stdin.setRawMode(true) 是禁用掉内置的一些键盘事件处理，比如 ctrl + c 退出进程

监听 keypress 事件，打印下。

我们自己判断输入的是 ctrl + c 的时候，退出。

![2025-01-07 22.27.03.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8e684b8204a44509668d8704da593a6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1338&h=606&s=159220&e=gif&f=48&b=171717)

这就是 readline 模块的三个功能：按行读取文件、按行读取用户输入（类似 repl）、处理键盘事件

## querystring

这个模块也很简单，就是用于 query string 的解析和格式化的。

创建 query\_string.mjs

```javascript
import queryString from 'node:querystring';

const res = queryString.parse('a=1&b=2&c=xxx');

console.log(res);

const res2 = queryString.stringify({ aaa: '111', bbb: ['222', '33'], ccc: '444' });

console.log(res2);
```

跑一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ad42245c5954d36b0616cd53637a3a5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=784&h=106&s=22949&e=png&b=191919)

## v8

我们知道通过 os 模块可以拿到系统的 cpu、内存、网卡等信息。

那如果想知道 Node.js 的 v8 的内存等信息呢？

这种就要用 v8 模块了。

它提供了查询内存状况、设置运行参数等的 api。

创建 v8.mjs

```javascript
import v8 from 'node:v8';

console.log(v8.getHeapSpaceStatistics());

console.log(v8.getHeapStatistics());
```

getHeapSpaceStatistics 是拿到堆内存的每一部分的统计信息。

getHeapStatistics 是拿到堆内存整体的统计信息。

跑一下：

```
node v8.mjs
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eda93273eab146e0a5b25c8aa4295372~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=770&h=1064&s=148139&e=png&b=181818)

可以看到，v8 的堆内存划分了这些部分，并且给出了每一部分的 size 等信息。

而下面是整个堆内存的统计信息：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eec2fe5ecc634f8e96df91c78c1a78e9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=708&h=528&s=88794&e=png&b=191919)

你还可以运行时给 v8 设置一些 flags，这样：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb21947260e344cf9df90f52f16a614a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=792&h=322&s=49749&e=png&b=1f1f1f)

全部的 v8 flags 可以看这里：

[gist.github.com/andrewiggin…](https://gist.github.com/andrewiggins/68c3165d47769a39eb5ae16e3001d6c6 "https://gist.github.com/andrewiggins/68c3165d47769a39eb5ae16e3001d6c6")

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/node-api-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/node-api-test")

## 总结

这节我们过了一下剩下的几个模块：

* **vm**：创建一个 JS 运行环境，在这个环境里跑 JS 代码，比如 jest 等指定上下文就是通过这个
* **repl**：自己创建 repl 的交互的时候，用这个模块
* **dns**：查询 dns 也就是域名到 ip
* **string\_decoder**：用于解码 Buffer 为字符串
* **worker\_threads**：创建工作线程来提升性能
* **readline**：按行读取输入流，可以读取文件、读取用户输入（类似 repl）、还可以处理键盘事件
* **querystring**：用来解析和生成 querystring 的
* **v8**：拿到 v8 的内存等统计信息，还可以设置 v8 flags

至此，我们就把 Node.js 的所有核心 API 过了一遍。