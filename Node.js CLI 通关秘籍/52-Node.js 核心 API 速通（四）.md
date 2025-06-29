这节我们继续来过 Node.js 内置模块。

## child\_process

child\_process 是用来跑子进程、和子进程通信的模块。

它有 4 个 api：

* spawn：执行 shell 命令，参数通过数组传入
* exec：执行 shell 命令，整个作为字符串传入
* execFile：执行可执行文件
* fork：跑 js 子进程

试一下：

创建 cp1.mjs

```javascript
import cp from "node:child_process";

const ls = cp.spawn('ls', ['-l', './']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`进程退出 ${code}`);
});
```

spawn 可以执行 shell 命令，参数通过数组传入。

每个进程都有标准输入流 stdin、标准输出流 stdout、标准错误流 stderr

stdout 和 stderr 都是可读流，我们监听 data 事件，拿到内容。

跑一下：

```bash
node ./cp1.mjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d7d2a36e2b4405ba8b996f6a930ed87~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=992&h=1042&s=251620&e=png&b=181818)

exec 是基于 spawn 封装出来的，功能一样：

创建 cp2.mjs

```javascript
import cp from "node:child_process";

const ls = cp.exec('ls -l');

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`进程退出 ${code}`);
});
```

它的参数不用单独数组传入了，可以整个作为字符串传入。

跑一下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4281101d40b54ed08acf3a7613752fda~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=994&h=462&s=112481&e=png&b=181818)

execFile 是跑可执行文件的：

比如这是我本地 chrome 浏览器的可执行文件路径：

```
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome
```

我指定一个 --user-data-dir=./aaa 它会跑一个新的浏览器，数据存在 aaa 目录下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2b0f4a41f4d4961b97261639592b65a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2656&h=1488&s=337519&e=png&b=1c1c1c)

那如何通过 node 跑这个可执行文件呢？

就是用 execFile。

创建 cp3.mjs

```javascript
import cp from 'node:child_process';

const child = cp.execFile('/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome', ['--user-data-dir=./aaa']);
```

跑一下：

![2025-01-07 10.20.27.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e292c2b64fe8493abe2972db71964cf9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2804&h=1470&s=300578&e=gif&f=56&b=1a1a1a)

fork 是用来跑 js 进程的：

创建 cp4.mjs

```javascript
import cp from 'node:child_process';

cp.fork('./cp1.mjs');
```

跑一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b5dd3a9919724a3f9bf19a80cc9bd57e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1050&h=548&s=132053&e=png&b=191919)

child\_process 创建子进程的方法就这 4 个。

## cluster

child\_process 只是创建子进程，父子进程之间的通信。

如果要一次性跑很多进程呢？

这种就要用 cluster 了。

比如 node.js 单线程，但是我们想利用多核 cpu 的能力，通过多进程来提升性能。

可以这样：

创建 cluster.mjs

```javascript
import cluster from 'node:cluster';
import http from 'node:http';
import { cpus } from 'node:os';
 
const numCPUs = cpus().length;
 
if (cluster.isPrimary) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('fork', (worker) => {
    console.log('worker 创建成功', worker.id);
  });

  cluster.on('exit', (worker, code, signal) => {
    console.log('worker 退出:', worker.id);
  });
} else {
  const server = http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world
');
  })
  
  server.listen(8000);

  setTimeout(()=> {
    process.exit();
  }, 3000)
}
```

我们根据 cpu 个数来跑了 n 个进程，每个进程跑了一个服务器，这样请求会分配到多个服务器中去。

然后子进程里 3s 后退出进程。

父进程监听 fork、exit 事件在进程创建、退出时做处理。

跑一下：

![2025-01-07 12.07.26.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dedef678ca3945dd80eaf5461f16bc52~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=870&h=622&s=88429&e=gif&f=44&b=181818)

不过一般我们不会自己调用 cluster 模块，而是用 pm2 这种进程管理器来做。

创建 cluster2.mjs

```javascript
import http from 'node:http';

const server = http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello guang
');
})

server.listen(8000);
```

然后用 pm2 来跑多个进程：

```arduino
npx pm2 start -i max ./cluster2.mjs
```

\-i max 就是按照 cpu 个数来跑多个进程：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d286a53c8a7c40f0a26dec4607dd4c39~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1932&h=530&s=195178&e=png&b=191919)

可以看到，也是一次性跑了 8 个。

我们浏览器访问下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/539d41bf5a824e0bb1fdb437b2c30b9c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=666&h=248&s=21048&e=png&b=ffffff)

没啥问题。

一般真要做多进程扩展，直接用 pm2 就行，没必要自己调用 cluster 模块。

## net

http 模块是创建 http 服务器、发送 http 请求相关的。

而 net 则是处理 TCP 相关的。

我们知道， TCP 是双向通信的：

创建服务端 net-tcp-server.mjs

```javascript
import net from 'node:net';

const server = net.createServer(function(clientSocket){
    console.log('新的客户端 socket 连接');

    clientSocket.on('data', function(data){
        console.log(data.toString());

        clientSocket.write('hello');
    });

    clientSocket.on('end', function(){
        console.log('连接中断');
    });
});

server.listen(6666, 'localhost', function(){
    const address = server.address();

    console.log('被监听的地址为：%j', address);
});
```

跑一下：

```vbscript
node net-tcp-server.mjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcd0f3c83ba04dc88f15e8af736bb3ef~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=986&h=104&s=21714&e=png&b=181818)

我们跑了一个 tcp 的服务。

然后再创建个 tcp 的客户端；

net-tcp-client.mjs

```javascript
import net from 'node:net';

const socket = net.createConnection({ 
    host: 'localhost',
    port: 6666 
}, () => {
  console.log('连接到了服务端!');

  socket.write('world!
');

  setTimeout(()=> {
    socket.end();
  }, 2000);
});

socket.on('data', (data) => {
  console.log(data.toString());
});

socket.on('end', () => {
  console.log('断开连接');
});
```

连上 tcp 服务端，发送条消息，然后 2s 后断开连接。

跑一下：

![2025-01-07 12.26.22.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d3dde8a2d924247874ff3745f5efeef~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1986&h=622&s=150995&e=gif&f=46&b=171717)

## dgram

TCP 是双向实时通信的，而 UDP 是单向的数据报。

dgram 就是用来处理 UDP 通信的。

DNS 协议就是基于 UDP 的，我们之前跑过一个 DNS 服务器

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe3af37c969b4406bd74843e21ab0d20~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=810&h=612&s=108105&e=png&b=1f1f1f)

首先跑一个 udp 服务器，通过 Buffer 解析二进制的协议。

根据域名分别做转发、自己返回协议两种处理。

处理协议的过程就是用之前学的 Buffer 的 api 来读写：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a4f7cd61d3749f08075b63ef7a8210b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=996&h=630&s=77395&e=png&b=1f1f1f)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c701b8e32da04e33944d938269f5418a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=936&h=1258&s=222611&e=png&b=1f1f1f)

转发的时候，这样发送数据报：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f65146ca75545e19f64bb666f311386~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=802&h=558&s=85066&e=png&b=1f1f1f)

我们修改里系统偏好设置的本地 DNS 服务器地址指向本机（windows 下也有修改 DNS 的地方）：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65a66b629101403aa4ff471f0ef56704~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=696&h=448&s=71432&e=png&b=f4f4f4)

然后用 nslookup 测试过：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b569d0c7df7548e282cfe52b5110d548~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=572&h=554&s=104935&e=png&b=010101)

返回的 ip 地址是我们自定义的。

这就是 UDP 相关 api 的应用场景。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/node-api-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/node-api-test")

## 总结

这节我们又学了 4 个 Node.js 内置模块。

* **child\_process**：创建子进程的，有 exec、spawn、execFile、fork 这 4 个 api，前两个是跑 shell 命令的，execFile 是跑可执行文件的，fork 是创建 js 进程的
* **cluster**：创建多个进程，比如服务器的多进程扩展，不过一般不用自己写，直接用 pm2 就行
* **net**：创建 TCP 服务，发送 TCP 消息
* **dgram**：创建 UDP 服务，发送 UDP 消息，比如 DNS 协议就是基于 UDP

这些模块可能不常用，但还是要过一遍。