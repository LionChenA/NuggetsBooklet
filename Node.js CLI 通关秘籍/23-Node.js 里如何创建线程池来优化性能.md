不知道大家有没有用过浏览器里的多线程。

我们可以把一些计算量大的逻辑拆分到 worker 线程来做。

直接上代码：

```bash
mkdir worker-test
cd worker-test
npm init -y
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a468632fb61449a8c2dfdc9b179306e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=838&h=704&s=86838&e=png&b=010101)

创建 index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>worker performance optimization</title>
</head>
<body>
    <script>
        function calc() {
            let total = 0;
            for(let i = 0; i< 10*10000*10000; i++) {
                total += i;
            }
            return total
        }

        document.write(calc());
    </script>
</body>
</html>
```

有一个大计算量的逻辑 calc。

跑一下：

```vbscript
npx http-server .
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/249282e63c234436a2d2246e17a14b88~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=756&h=594&s=92177&e=png&b=181818)

浏览器访问：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b07063c1e95f4a8cba559423d38dc8f6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=686&h=216&s=21431&e=png&b=fdfdfd)

我们用 Performance 分析下性能：

打开 chrome devtools 的 Performance 面板，点击 reload 按钮，会重新加载页面并开始记录耗时：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca45ac441bd04db683793db8c4401a33~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1924&h=724&s=294489&e=png&b=ffffff)

过几秒点击结束。

![2024-09-02 18.15.11.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/342e26b735a9451eb604290b0de1df8b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2714&h=1432&s=658087&e=gif&f=56&b=252525)

这里的 main 就是主线程：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f1c7b75213a464db0c56ad8d25a6d26~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2180&h=902&s=190689&e=png&b=303030)

其余的 Frames、Network 等是浏览器的其他线程。

可以看到，执行 calc 这个宏任务被标为了 long task，也就是需要优化的长任务。

超过 50ms 就是长任务了，这都执行了 1s 多了。

所以需要性能优化。

创建 index2.html

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>worker performance optimization</title>
</head>
<body>
    <script>
        function runWorker(url, num) {
            return new Promise((resolve, reject) => {
                const worker = new Worker(url);
                worker.postMessage(num);
                worker.addEventListener('message', function (evt) {
                    resolve(evt.data);
                });
                worker.onerror = reject;
            });
        };

        runWorker('./worker.js', 10*10000*10000).then(res => {
            document.write(res);
        });        
    </script>
</body>
</html>
```

我们封装了一个 runWorker 的方法，通过 new Worker 来起一个工作线程。

通过 postMessage 向工作线程发消息，监听 message 事件来接收工作线程发回的消息。

然后创建 worker.js

```javascript
function calc(num) {
    let total = 0;
    for(let i = 0; i< num; i++) {
        total += i;
    }
    return total
}

addEventListener('message', function(evt) {
    postMessage(calc(evt.data));
});
```

也是通过 postMessage 向主线程发消息，监听 message 事件接收主线程发来的消息。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfe7a31e4b374c92ba3854e7a711b4b2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1250&h=572&s=46734&e=png&b=ffffff)

然后 b 和 c 函数就可以改成这样了：

```javascript
function b() {
    runWorker('./worker.js', 10*10000*10000).then(res => {
        console.log('b:', res);
    });
}
```

耗时逻辑转移到了工作线程。

我们再跑一下试试：

![2024-09-02 18.30.16.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/702243fe6fc141ce903ae4d218ca0d62~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2714&h=1432&s=617957&e=gif&f=50&b=262626)

可以看到，主线程的 long task 消失了，转移到了 worker 线程。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4b647e5cbdb4257a626dbc5d576a4eb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1874&h=1368&s=200121&e=png&b=2b2b2b)

这样，我们就通过 worker 线程完成了网页的性能优化。

Node.js 自然也有工作线程的概念。

我们用一下：

创建 index.js

```javascript
const { Worker, MessageChannel } = require('node:worker_threads');

const { port1, port2 } = new MessageChannel();

const worker = new Worker('./node-worker.js');
worker.postMessage(
    { value: 10*10000*10000, channel: port2 },
    [port2]
);

port1.on('message', (value) => {
    console.log('res', value);
})
```

这里用到了 node 的 worker\_threads 模块的 api。

Worker 是用于创建工作线程的。

MessageChannel 是创建通信的通道，它有两个端口 port1 和 port2。

你在 port1 里 postMessage，在 port2 里就可以用 message 事件接收到消息，反过来也是。

很容易理解，就像一个管子的两个口：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fdd2275daac4a04a4dbb62c98a58ee0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=884&h=634&s=849085&e=png&b=979591)

其实和浏览器里的 api 是差不多的：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfe7a31e4b374c92ba3854e7a711b4b2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1250&h=572&s=46734&e=png&b=ffffff)

然后写下 node-worker.js

```javascript
const { parentPort } = require('node:worker_threads');

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

用 parentPort 监听传过来的 message 消息。

然后接收 message 传过来的 channel 来 postMessage 发送消息。

跑一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cf31571443b48c18210352dd5c32c14~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=728&h=200&s=22624&e=png&b=181818)

可以看到，计算出了正确的结果。

所以，不管是浏览器里，还是 node 里，遇到大计算量的任务都可以通过工作线程来优化。

我们知道，线程数的上限就是 cpu 的核心数。

一个工作线程哪够用？

我们得多创建几个工作线程来备用。

我们可以创建一个线程池，充分利用 cpu 来最大程度优化代码。

创建 pool.js

```javascript
const os = require('node:os');

console.log(os.cpus().length);
```

首先看下我电脑有几个 cpu 核心：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e16ade0e99b34ccfbc514e621e584a09~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=634&h=328&s=42111&e=png&b=1c1c1c)

8 个。

这样我们可以创建一个 8 个工作线程的线程池。

创建 pool.js

```javascript
const os = require('node:os');
const { Worker, MessageChannel } = require('node:worker_threads');

const poolSize = os.cpus().length;

const workers = [];
const tunnels = [];

for (let i = 0; i < poolSize; i++) {
    const { port1, port2 } = new MessageChannel();

    const worker = new Worker('./pool-worker.js');
    worker.postMessage(
        { 
            type: 'startup', 
            id: i,
            channel: port2 
        },
      [port2],
    );

    tunnels.push(port1);
    workers.push(worker);
}

for (let i = 0; i < tunnels.length; i ++) {
    tunnels[i].on('message', (msg) => {
        console.log(`线程 ${msg.id} 计算出了结果 ${msg.res}`);
    });
}

let curIndex = 0;

function addJob(num) {
    const tunnel = tunnels[curIndex];

    tunnel.postMessage({ 
        value: num
    });

    curIndex = curIndex >= workers.length - 1 ? 0 : curIndex + 1;
}

for(let i = 0; i< 100; i++) {
    addJob(Math.floor(Math.random() * 1000000));
}
```

根据 cpu 核心数循环创建 8 个 Worker，id 为 index，也就是 0 到 7。

把 Worker 实例和通信用的 port 放到数组里。

循环监听所有 port 的消息，打印下计算结果。

然后封装一个 addJob 方法来发消息，从 0 到 7 依次调用工作线程来处理。

我们循环调用了 100 次来计算。

接下来写下 pool-worker.js

```javascript
const { parentPort } = require('node:worker_threads');

function calc(num) {
    let total = 0;
    for(let i = 0; i< num; i++) {
        total += i;
    }
    return total
}

let tunnel;
let id;

parentPort.on('message', (message) => {
    if(message.type === 'startup') {
        id = message.id;
        tunnel = message.channel;

        tunnel.on('message', (msg) => {
            tunnel.postMessage({
                id,
                res: calc(msg.value)
            });
        })
    }
});
```

就是接收 parentPort 传过来的 message，拿到 id、通信的 channel。

监听 channel 发过来的 message 消息，计算结果后通过 postMessage 传回去。

跑一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f10e7424f6644f194054dfabbb86e38~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=914&h=960&s=228703&e=png&b=191919)

可以看到，我们用线程池实现了 100 个计算任务的计算。

策略是轮询调用 8 个线程。

这样，我们就充分利用 cpu 来实现了 node 的性能优化。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/worker-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/worker-test")

## 总结

这节我们学了浏览器和 node 里的工作线程。

它们很类似，都是通过 new Worker("worker文件路径") 创建工作线程，然后通过 postMessage 发消息，通过 message 事件接收传过来的消息。

在 node 里要用 new MessageChannel() 创建 port1、port2 作为两个端口来收发消息，在一方 postMessage，另一方就可以 message 事件监听消息。

node 里可以用 os 模块的 cpus 拿到 cpu 核心数量，从而创建对应数量的线程池，最大化利用机器的能力来做优化。