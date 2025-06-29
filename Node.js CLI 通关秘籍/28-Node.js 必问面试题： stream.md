stream 在 Node.js 面试中出现频率非常高。

那什么是 stream（流）呢？

其实我们写的 Node.js 代码经常会用到流。

直接来看代码吧：

```bash
mkdir stream-test
cd stream-test
npm init -y
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b867930313d54beaabf8e090fdcddee0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=828&h=686&s=83579&e=png&b=010101)

创建 src/test.mjs

```javascript
import http from 'node:http';
import fs from 'node:fs';

const server = http.createServer(async function (req, res) {
    const data = fs.readFileSync(import.meta.dirname + '/index.html', 'utf-8');
    res.end(data);
});

server.listen(8000);
```

可以安装下 node 的类型，这样写的时候会有类型提示：

```css
npm install --save-dev @types/node
```

我们跑了个 http 服务。

用 fs.readFileSync 读取 data.txt 的内容返回。

跑一下：

```bash
node ./src/test.mjs
```

创建 src/data.txt

```javascript
神说要有光
```

然后用 curl 访问下：

```arduino
curl -i http://localhost:8000
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4a2d89e265a4a42b3a563de271e4542~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=602&h=316&s=43420&e=png&b=010101)

因为是全部读完返回的，所以可以知道 Content-Length，也就是响应体的长度。

当文件比较小的时候，这样读取、返回没啥问题。

那如果文件非常大呢？

比如有好几百 M，这时候全部读取完再返回是不是就合适了？

因为要等好久才能读取完文件，之后才有响应。

这就需要用到流了：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c63877a427b4a17a3cf1ff027fc2410~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1390&h=438&s=102339&e=png&b=1f1f1f)

```javascript
const readStream = fs.createReadStream(import.meta.dirname + '/data.txt', 'utf-8');
readStream.pipe(res);
```

我们用 fs.createReadStream 创建文件读取流，然后 pipe 到响应的流。

跑一下：

```bash
node ./src/test.mjs
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c05c99085f8475e8c1c9cec4d0a24c1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=572&h=322&s=46101&e=png&b=010101)

结果一样，但是因为现在是流式返回的，并不知道响应体的 Content-Length。

所以是用 Transfer-Encoding: chunked 的方式返回流式内容。

这个是面试常考题。

从服务器下载一个文件的时候，如何知道文件下载完了呢？

有两种方式：

一种是 header 里带上 Content-Length，浏览器下载到这个长度就结束。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6876968033b44759a5a97f4eeedce012~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1088&h=316&s=63926&e=png&b=fefefe)

另一种是设置 transfer-encoding:chunked，它是不固定长度的，服务器不断返回内容，直到返回一个空的内容代表结束。

比如这样：

```diff
5
Hello
1
,
5
World
1
!
0
```

这里分了 “Hello” “,” “World”“!” 这 4 个块，长度分别为 5、1、5、1

最后以一个长度为 0 的块代表传输结束。

这样，不管内容多少都可以分块返回，就不用指定 Content-Length 了。

这就是大文件的流式传输的原理，就是 transfer-encoding:chunked。

当然，这是 http 传输时的流，在用 shell 命令的时候，也经常会用到流：

比如

```perl
ls | grep pack
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fdd1164dcb514f4684eac655e8e3b629~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=382&h=104&s=17787&e=png&b=191919)

ls 命令的输出流，作为 grep 命令的输入流。

当然，我们也可以把 grep 命令的输出流，作为 node 脚本的输入流。

创建 src/read.mjs

```javascript
process.stdin.on('readable', function () {
    const buf = process.stdin.read();
    console.log(buf?.toString('utf-8'));
});
```

process.stdin 就是输入流，监听 readable 事件，用 read 读取数据。

跑一下：

```bash
ls | grep pack | node src/read.mjs
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ff1cf5d66ec4a78987c0752f3927481~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=602&h=256&s=41427&e=png&b=181818)

可以看到，我们的 node 脚本接收到了 grep 的输出流作为输入流。

这就是管道 pipe 的含义。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36dc568a7abe4029994881a04e1343d2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1376&h=450&s=102567&e=png&b=1f1f1f)

上面我们写的从文件读取流 pipe 到响应流，就是这个意思。

综上，可以小结下我们对流的认识：

**流就是分段的传输内容，比如从服务端像浏览器返回响应数据的流，读取文件的流等。**

**流和流之间可以通过管道 pipe 连接，上个流的输出作为下个流的输入。**

在 node 里，流一共有 4 种：可读流 Readable、可写流 Writable、双工流 Duplex、转换流 Transform

```javascript
import stream from 'node:stream';

// 可读流
const Readable = stream.Readable;
// 可写流
const Writable = stream.Writable;
// 双工流
const Duplex = stream.Duplex;
// 转换流
const Transform = stream.Transform;
```

其余的流都是基于这 4 种流封装出来的。

我们分别来试一下：

### Readable

Readable 要实现 \_read 方法，通过 push 返回具体的数据。

```javascript
import { Readable } from 'node:stream';

const readableStream = new Readable();

readableStream._read = function() {
    this.push('阿门阿前一棵葡萄树，');
    this.push('阿东阿东绿的刚发芽，');
    this.push('阿东背着那重重的的壳呀，');
    this.push('一步一步地往上爬。')
    this.push(null);
}

readableStream.on('data', (data)=> {
    console.log(data.toString())
});

readableStream.on('end', () => {
    console.log('done');
});
```

当 push 一个 null 时，就代表结束流。

跑一下：

```bash
node ./src/readable.mjs
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef18c4039e99480f9c76f354a2019281~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=536&h=186&s=28381&e=png&b=191919)

创建 Readable 流也可以通过继承的方式：

新建 src/readable2.mjs

```javascript
import { Readable } from 'node:stream';

class ReadableDong extends Readable {

    _read() {
        this.push('阿门阿前一棵葡萄树，');
        this.push('阿东阿东绿的刚发芽，');
        this.push('阿东背着那重重的的壳呀，');
        this.push('一步一步地往上爬。')
        this.push(null);
    }

}

const readableStream = new ReadableDong();

readableStream.on('data', (data)=> {
    console.log(data.toString())
});

readableStream.on('end', () => {
    console.log('done');
});
```

跑一下：

```bash
node ./src/readable2.mjs
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17bfd77b376540cb82b904541a3224c1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=532&h=180&s=28478&e=png&b=191919)

可读流是生成内容的，那么很自然可以和生成器结合：

创建 src/readable3.mjs

```javascript
import { Readable } from 'node:stream';

class ReadableDong extends Readable {

    constructor(iterator) {
        super();
        this.iterator = iterator;
    }

    _read() {
        const next = this.iterator.next();
        if(next.done) {
            return this.push(null);
        } else {
            this.push(next.value)
        }
    }

}

function *songGenerator() {
    yield '阿门阿前一棵葡萄树，';
    yield '阿东阿东绿的刚发芽，';
    yield '阿东背着那重重的的壳呀，';
    yield '一步一步地往上爬。';
}

const songIterator = songGenerator();

const readableStream = new ReadableDong(songIterator);

readableStream.on('data', (data)=> {
    console.log(data.toString())
});

readableStream.on('end', () => {
    console.log('done');
});
```

\* 和 yield 是 js 的 generator 的语法，它是异步返回 yield 后的内容，通过 iterator 的 next 来取下一个。

跑一下：

```bash
node ./src/readable3.mjs
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a18e170162a4401911005c0f44382d8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=538&h=180&s=27604&e=png&b=191919)

我们封装个工厂方法：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e44791b8630446087d67428fa3fc88c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=922&h=620&s=112499&e=png&b=1f1f1f)

```javascript
function createReadStream(interator) {
    return new ReadableDong(interator);
}

const readableStream = createReadStream(songIterator)

readableStream.on('data', (data)=> {
    console.log(data.toString())
});

readableStream.on('end', () => {
    console.log('done');
});
```

是不是就和 fs.createReadStream 很像了？

试一下：

创建 src/fsReadStream.mjs

```javascript
import fs from 'node:fs';

const readStream = fs.createReadStream(import.meta.dirname + '/data.txt', 'utf-8');

readStream.on('data', (data)=> {
    console.log(data.toString())
});

readStream.on('end', () => {
    console.log('done');
});
```

跑一下：

```bash
node ./src/fsReadStream.mjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c41393c2d8b44493a4aaa03c53f96662~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=512&h=126&s=15756&e=png&b=181818)

其实文件的 ReadStream 就是基于 stream 的 Readable 封装出来的。

这就是可读流。

http 服务的 request 就是 Readable 的实例：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ed7de88a747468588dc79c668ee67d1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=810&h=298&s=55314&e=png&b=1f1f1f)

所以我们可以这样写：

创建 src/test2.mjs

```javascript
import http from 'node:http';
import fs from 'node:fs';

const server = http.createServer(async function (req, res) {
    const writeStream = fs.createWriteStream('aaa.txt', 'utf-8');
    req.pipe(writeStream);
    res.end('done');
});

server.listen(8000);
```

跑起来：

```bash
node ./src/test2.mjs
```

访问下：

```less
curl -X POST -d "a=1&b=2" http://localhost:8000
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e2a1a8dfb9c4ddd8eb5d9cbcb4e5d07~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=786&h=94&s=32492&e=png&b=010101)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/852a4398dcd64f21a21a434ef65e4948~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=826&h=208&s=23661&e=png&b=1c1c1c)

可以看到，从 request 的流中读出的内容写入了文件的 WriteStream

#### Writable

Readable 是实现 \_read 方法，通过 this.push 返回内容

Writable 则要实现 \_write 方法，接收写入的内容。

创建 src/writable.mjs

```javascript
import { Writable } from 'node:stream';

class WritableDong extends Writable {

    constructor(iterator) {
        super();
        this.iterator = iterator;
    }

    _write(data, enc, next) {
        console.log(data.toString());
        setTimeout(() => {
            next();
        }, 1000);
    }
}

function createWriteStream() {
    return new WritableDong();
}

const writeStream = createWriteStream();

writeStream.on('finish', () => console.log('done'));

writeStream.write('阿门阿前一棵葡萄树，');
writeStream.write('阿东阿东绿的刚发芽，');
writeStream.write('阿东背着那重重的的壳呀，');
writeStream.write('一步一步地往上爬。');
writeStream.end();
```

Writable 的特点是可以自己控制消费数据的频率，只有调用 next 方法的时候，才会处理下一部分数据。

我们每 1s 处理一次写入。

跑一下：

![2024-12-17 09.25.23.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a0d635a19b7484eadd84d0569a45d4e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=894&h=512&s=87394&e=gif&f=47&b=181818)

其实我们常用的 fs.createWriteStream 就是这样封装出来的。

用一下试试：

创建 src/fsWriteStream.mjs

```javascript
import fs from 'node:fs';

const writeStream = fs.createWriteStream('tmp.txt', 'utf-8');

writeStream.on('finish', () => console.log('done'));

writeStream.write('阿门阿前一棵葡萄树，');
writeStream.write('阿东阿东绿的刚发芽，');
writeStream.write('阿东背着那重重的的壳呀，');
writeStream.write('一步一步地往上爬。');
writeStream.end();
```

跑一下：

```bash
node ./src/fsWriteStream.mjs
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68ab34e8046b4d5e9bbc58873ef9f2e3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=550&h=78&s=13326&e=png&b=181818)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e47d9829c704649ac2b964ed807858f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1122&h=194&s=23662&e=png&b=202020)

这就是可写流。

http 服务的 response 就是 Writable 的实例：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8dacaf01b7c46a5834ab4d5cb048930~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1248&h=398&s=90013&e=png&b=1f1f1f)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7fde969091074db09a4b1be76d26afaf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1510&h=588&s=125133&e=png&b=1f1f1f)

#### Duplex

Duplex 是可读可写，同时实现 \_read 和 \_write 就可以了，也就是双工流

创建 src/duplex.mjs

```javascript
import { Duplex } from 'node:stream';

class DuplexStream extends Duplex {

    _read() {
        this.push('阿门阿前一棵葡萄树，');
        this.push('阿东阿东绿的刚发芽，');
        this.push('阿东背着那重重的的壳呀，');
        this.push('一步一步地往上爬。')
        this.push(null);
    }

    _write(data, enc, next) {
        console.log(data.toString());
        setTimeout(() => {
            next();
        }, 1000);
    }
}

const duplexStream = new DuplexStream();

duplexStream.on('data', data => {
    console.log(data.toString())
});
duplexStream.on('end', data => {
    console.log('read done')
});

duplexStream.write('阿门阿前一棵葡萄树，');
duplexStream.write('阿东阿东绿的刚发芽，');
duplexStream.write('阿东背着那重重的的壳呀，');
duplexStream.write('一步一步地往上爬。');
duplexStream.end();

duplexStream.on('finish', data => {
    console.log('write done')
});
```

跑一下：

```bash
node ./src/duplex.mjs
```

![2024-12-17 11.41.14.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48756ed4b09f4f40bfdf8bab4e755742~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=894&h=512&s=77520&e=gif&f=45&b=181818)

整合了 Readable 流和 Writable 流的功能，这就是双工流 Duplex。

TCP 协议会用 socket 来做双向通信，它就是 Duplex 的实现：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d042c1a6bdd445a7aecad81f21777f97~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1146&h=988&s=215928&e=png&b=202020)

试一下：

创建 src/socket-server.mjs

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

```bash
node ./src/socket-server.mjs
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95adfda021d2446785cc8af598566609~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=980&h=116&s=22098&e=png&b=181818)

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

![2024-12-17 12.10.59.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/939187815eb6492a976b5af905007f32~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1580&h=570&s=159314&e=gif&f=55&b=171717)

这种双向通信就是基于 Duplex 做的。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efc1fbfbe85642869f344ce8f33c654f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=884&h=520&s=141248&e=png&b=1f1f1f)

看下这些方法：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94778c75d3bb49319245f6470101243f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=954&h=602&s=107099&e=png&b=1f1f1f)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/531499959aa846d183181f870c961087~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=764&h=780&s=115006&e=png&b=1f1f1f)

write 是可写流的， data、end 事件是可读流的。

Socket 就是 Duplex 的实现。

#### Transform

Transform 也是 Duplex 双工流，只不过它会对写入的内容做一些转换之后提供给消费者来读

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa59f48a1be9483280fa6a28e1b62127~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1068&h=334&s=88379&e=png&b=1f1f1f)

Transform 流要实现 \_transform 的 api。

试一下：

创建 src/transform.mjs

```javascript
import { Transform } from 'node:stream';

class ReverseStream extends Transform {

  _transform(buf, enc, next) {
    const res = buf.toString().split('').reverse().join('');
    this.push(res);

    next()
  }
}

var transformStream = new ReverseStream();

transformStream.on('data', data => console.log(data.toString()))
transformStream.on('end', data => console.log('read done'));

transformStream.write('阿门阿前一棵葡萄树');
transformStream.write('阿东阿东绿的刚发芽');
transformStream.write('阿东背着那重重的的壳呀');
transformStream.write('一步一步地往上爬');
transformStream.end()

transformStream.on('finish', data => console.log('write done'));
```

在 \_transform 方法里用 push 来产生可读流数据，然后 next 是消费下一个写入的数据。

push 和 next 方法在前面的 Readable、Writable 里都用过。

跑一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9451c1e016854adcaba0e0d848b2042b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=540&h=220&s=33429&e=png&b=191919)

这就是转换流，它也是一种双工流。

那在 node.js 的内置模块里，有哪些 api 是转换流呢？

zlib

试一下：

创建 src/zlib.mjs

```javascript
import {
    createReadStream,
    createWriteStream,
} from 'node:fs';
import { createGzip } from 'node:zlib';
  
const gzip = createGzip();
const source = createReadStream(import.meta.dirname + '/data.txt');
const destination = createWriteStream('data.txt.gz');

source.pipe(gzip).pipe(destination);
```

从文件的 ReadStream，pipe 到 Gzip 转换流，然后 pipe 到文件的 WriteStream。

这感觉是不是和我们写 shell 命令时一样？

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d467425262314ee899fbedbfee8b94f9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1052&h=238&s=85730&e=png&b=1a1a1a)

跑一下：

```bash
node ./src/zlib.mjs
```

![2024-12-17 12.34.03.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/805022c12bc645a184fa99ae1ffe83f4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1600&h=860&s=372079&e=gif&f=30&b=f9f6f6)

压缩成功了。

点击解压，就可以看到其中的文件。

这个 gzip 是用于 http 的压缩传输的：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/beb4378064914db1a27bcb4725dd03d3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1100&h=228&s=48631&e=png&b=fefefe)

很常用，所以 Node.js 内置了这个模块。

这里的多次 pipe 也可以用 stream 的 pipeline 的 api 简化：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/001a523738a34487a274e15d3fbc763e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=828&h=538&s=104113&e=png&b=1f1f1f)

这条 pipeline 的特点是 Readable 产生数据，然后经过任意多个 Duplex（包括 Transform），最后传入 Writable

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4197503937946fdbc2439cccd0f2db1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1362&h=214&s=22373&e=png&b=ffffff)

至此，四种 stream 我们就都用了一遍。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/stream-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/stream-test")

## 总结

stream 是 Node.js 的非常常用 API，也是面试必问的点。

我们每天敲的 shell 命令，就是基于流的概念，上个进程的输出可以做为下个进程的输入。

写 Node.js 代码的时候，文件读写、网络通信等都是基于流。

虽然各种流有很多，但底层的 stream 只有 4 种：

* Readable：实现 \_read 方法，通过 push 传入内容
* Writable：实现 \_write 方法，通过 next 消费内容
* Duplex：实现 \_read、\_write，可读可写
* Transform：实现 \_transform，对写入的内容做转换再传出去，继承自 Duplex

面试问的话，除了说出这 4 种 stream 外，最好举一个具体的 api 来说明。

比如 fs.createReadStream、http 的 request 是 Readable 的实现，fs.createWriteStream、http 的 response 是 Writable 的实现，net 的 Socket 是 Duplex 的实现，zlib.createGzip 是 Transform 的实现。

理解这 4 种 stream，能自己实现，也能知道哪些 api 是哪种流，就算掌握的差不多了。