这节我们继续来过 node.js 的核心 API。

## stream

stream 是流相关的 api，前面我们单独一节讲过，这里简单再过一遍：

Node.js 一共有 4 种 stream，也就是 Readable（可读流）、Writable（可写流）、Duplex（双工流）、Transform（转换流）

流和流可以组合，也就是这样：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4197503937946fdbc2439cccd0f2db1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1362&h=214&s=22373&e=png&b=ffffff)

这条 pipeline 的特点是 Readable 产生数据，然后经过任意多个 Duplex（包括 Transform），最后传入 Writable

其实我们经常会用到流：

比如上节我们用 http.request 的时候，返回的 res 就是一个可读流：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cfe613b67af7412c847309000b0bf95c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1206&h=916&s=144557&e=png&b=1f1f1f)

读取文件用的 fs.createReadStream 也是一个可读流：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca93a2dc120e48b7ad1cbf57aab9bde4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1328&h=502&s=95887&e=png&b=1f1f1f)

可写流 Writable 也用的非常多：

fs.createWriteStream 创建的就是可写流：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51d9bbccdd504d04a3aee06deea27e61~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1000&h=494&s=116435&e=png&b=202020)

而 socket 这种可读可写的是双工流 Duplex：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/300c4b871dcf4e62979db31a5f103f0c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=900&h=758&s=141182&e=png&b=1f1f1f)

而 Transform 转换流也是一种 Duplex，它会对内容做转换之后返回新的内容：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4ff6f5e951741708ca025c249636f5c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1050&h=516&s=110491&e=png&b=1f1f1f)

比如 zlib 的 createGzip。

这 4 种流都是可以自定义的：

Readable：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b248412a4c949f8a7ba825f58bc04f0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=728&h=872&s=140276&e=png&b=1f1f1f)

实现 \_read 方法，通过 this.push 返回数据，this.push(null) 代表结束

Writable：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/835340b546ac44ab9d1a8d05d9c685f9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=892&h=1138&s=198607&e=png&b=1f1f1f)

实现 \_write 方法，调用 next() 消费下一个数据，频率自己控制。

Duplex：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f92e98f5cbe246008b5008efec515d27~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=870&h=1386&s=259989&e=png&b=1f1f1f)

同时实现 \_read、\_write 方法

Transform：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e5e972dfd264b4c8151a4c71c987c28~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1078&h=922&s=196906&e=png&b=1f1f1f)

实现 \_transform 方法，用 this.push 返回数据，next(）消费下一个数据

流和流之间可以 pipe 连接，也可以直接 pipeline 连接：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6438944f235b422999ef3bf687e167b3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1022&h=504&s=101839&e=png&b=1f1f1f)

这些我们在 stream 那节讲过，只是简单复习一下。

## http

http 我们常用的就两个功能，一个起 http 服务，另一个就是发送 http 请求：

发送 http 请求我们写过：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4cd4d34a635247caa8af0dbe73d6833d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1234&h=914&s=146097&e=png&b=1f1f1f)

调用 http.reqeust，传入 options。

这个 options 可以用 url.urlToHttpOptions 来解析产生。

返回的是一个可读流。

而创建 http 服务是这样的：

创建 http-server.mjs

```javascript
import http from 'node:http';
import fs from 'node:fs';

const server = http.createServer(async function (req, res) {
    const writeStream = fs.createWriteStream('aaa.txt', 'utf-8');
    req.pipe(writeStream);

    res.write('66666');
    res.end('done');
});

server.listen(8000);
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ac73f9b4447411bbe6d292f946b3849~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1038&h=444&s=79287&e=png&b=1f1f1f)

它的 req 是可读流，我们 pipe 到一个文件写入流。

它的 res 是可写流，我们通过 write 方法返回内容。

跑一下：

```bash
node ./http-server.mjs
```

通过 curl 访问：

```arduino
curl -X POST -d "name=guang&age=20" http://localhost:8000
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f6be9199c9b493abddf202aa0997338~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=924&h=102&s=26961&e=png&b=010101)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7750f5511b740a08bd6f33dfa0d4a37~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=882&h=228&s=30860&e=png&b=1d1d1d)

可以看到，req 的请求体写入了文件，res 返回的内容返回给了客户端。

此外，作为 http 服务，还可以读取 req.headers，设置 res.statusCode 等：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ade95cb559e45e5a55bb839ca77a04f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=988&h=432&s=99969&e=png&b=202020)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f72b2e48a7f452eac21a7f4d11bfe8b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1120&h=460&s=85327&e=png&b=1f1f1f)

用到的时候再细看就行。

## fs

fs 算是最常用的 api 了，就是文件、目录的增删改查。

我们过一遍常用的：

首先是目录的增删改：

创建 fs.mjs

```javascript
import fs from 'node:fs';

fs.mkdirSync('aaa');

setTimeout(() => {
    fs.renameSync('aaa', 'bbb');
}, 1000);

setTimeout(() => {
    fs.rmSync('bbb');
}, 3000);
```

mkdirSync 创建目录、1s 后 renameSync 修改名字，3s 后 rmSync 删除目录。

跑一下：

```
node fs.mjs
```

![2025-01-01 11.33.21.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/786829b1338c44378b5bd7a3d19ff15a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1804&h=748&s=184820&e=gif&f=42&b=191919)

然后是文件的增删改：

创建 fs2.mjs

```javascript
import fs from 'node:fs';
import { EOL } from 'node:os';

fs.writeFileSync('aaa.txt', 'hello' + EOL);

setTimeout(() => {
    fs.appendFileSync('aaa.txt', 'world' + EOL)
}, 2000);

setTimeout(() => {
    fs.unlinkSync('aaa.txt');
}, 4000)
```

前面讲过 fs.createWriteStream 的方式，如果文件内容不大，直接用 fs.writeFileSync 同步写就行，还可以用 fs.appendFileSync 同步追加内容。

最后用 fs.unlinkSync 删除文件

跑一下：

```
node fs2.mjs
```

![2025-01-01 12.19.40.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2839186cf6994fb8b08ddb9746ef9f34~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1768&h=902&s=295175&e=gif&f=54&b=1a1a1a)

再就是复制文件和目录：

创建 fs3.mjs

```javascript
import fs from 'node:fs';

fs.mkdirSync('aaa/bbb/ccc/ddd', { 
    recursive: true
});

fs.writeFileSync('aaa/a.txt', '111');
fs.writeFileSync('aaa/bbb/b.txt', '222');
fs.writeFileSync('aaa/bbb/ccc/c.txt', '333');
fs.writeFileSync('aaa/bbb/ccc/ddd/d.txt', '444');
```

首先，递归创建目录，并写入 4 个文件。

```
node fs3.mjs
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed2b650f60e44a8a804f2a6cd857f9d7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=358&h=450&s=25826&e=png&b=181818)

然后来复制下：

创建 fs4.mjs

```javascript
import fs from 'node:fs';
import path from 'node:path';

function copyDir(srcDir, destDir) {
    fs.mkdirSync(destDir, { recursive: true });

    for (const file of fs.readdirSync(srcDir)) {
      const srcFile = path.resolve(srcDir, file)
      const destFile = path.resolve(destDir, file)
      copy(srcFile, destFile)
    }
}

function copy(src, dest) {
    const stat = fs.statSync(src)
    if (stat.isDirectory()) {
        copyDir(src, dest)
    } else {
        fs.copyFileSync(src, dest)
    }
}

copy('aaa', 'aaa2');
```

用 fs.statSync 拿到文件的信息，如果是目录就递归复制目录，否则就 fs.copyFileSync 复制文件。

目录的复制就是用先用 fs.mkdirSync 创建目录，然后用 fs.readdirSync 读取目录的内容。

用 path.resolve 拼接下路径为绝对路径，之后继续递归 copy

跑一下：

```
node fs4.mjs
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f25d4c0e26e64ffda02ba31f5eb37a27~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=342&h=476&s=28282&e=png&b=181818)

当然，其实 fs 有个 cpSync 的方法可以直接用，不用自己实现；

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b61e8507674455084011c6230251f02~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=546&h=232&s=27445&e=png&b=1f1f1f)

```javascript
fs.cpSync('aaa', 'aaa3', {
    recursive: true
});
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4bc37b01295484083728d62a6be002b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=362&h=516&s=30589&e=png&b=191919)

不过 fs 这个 cpSync 方法直到 node 22 才不再是实验性的：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb82b01eb79e4e8c8b464d3b8f0f8f94~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1372&h=666&s=90225&e=png&b=fefefe)

所以低版本 node 还是要自己实现，或者用 fs-extra：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2d0c361cfd340fdb8d4b18b2a42e18c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=674&h=190&s=36260&e=png&b=202020)

## zlib

zlib 是压缩和解压的，但它不是用于我们常用的那种压缩包，而是用于 http 传输数据时的 gzip、deflate、brotli 等压缩格式

也就是这个：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/beb4378064914db1a27bcb4725dd03d3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1100&h=228&s=48631&e=png&b=fefefe)

http 请求的时候会带上 Accept-Encoding 表示自己支持的压缩格式。

服务端会通过 Content-Encoding 的 header 来标识响应使用的压缩格式。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/683e9d2f466e40a0bff5fa946ad0d0f4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1454&h=446&s=97916&e=png&b=fefefe)

前面我们单独用过 gzip 的压缩，是这样用的：

```javascript
import {
    createReadStream,
    createWriteStream,
} from 'node:fs';
import { pipeline } from 'node:stream/promises';
import { createGzip } from 'node:zlib';
  
const gzip = createGzip();
const source = createReadStream(import.meta.dirname + '/data.txt');
const destination = createWriteStream('data.txt.gz');

await pipeline(source, gzip, destination);
```

![2024-12-17 12.34.03.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/805022c12bc645a184fa99ae1ffe83f4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1600&h=860&s=372079&e=gif&f=30&b=f9f6f6)

压缩成功了。

点击解压，就可以看到其中的文件。

但最常用的还是在 http 服务上。

创建 zlib-http-server.mjs

```javascript
import zlib from 'node:zlib';
import http from 'node:http';
import fs from 'node:fs';
import { pipeline } from 'node:stream/promises';

const server = http.createServer(async (request, response) => {
    const raw = fs.createReadStream('index.html');

    const acceptEncoding = request.headers['accept-encoding'] || '';

    try {
        if (/\bdeflate\b/.test(acceptEncoding)) {
            response.writeHead(200, { 'Content-Encoding': 'deflate' });
            await pipeline(raw, zlib.createDeflate(), response);
        } else if (/\bgzip\b/.test(acceptEncoding)) {
            response.writeHead(200, { 'Content-Encoding': 'gzip' });
            await pipeline(raw, zlib.createGzip(), response);
        } else if (/\bbr\b/.test(acceptEncoding)) {
            response.writeHead(200, { 'Content-Encoding': 'br' });
            await pipeline(raw, zlib.createBrotliCompress(), response);
        } else {
            response.writeHead(200, {});
            await pipeline(raw, response);
        }
    } catch(err) {
        response.end();
        console.error('An error occurred:', err);
    }
})

server.listen(8080);
```

用 http.createServer 创建一个 http 服务。

拿到请求的 accept-encoding 的 header。

根据支持的压缩格式，分别用 zlib.createDeflate、zlib.createGzip、zlib.createBrotliComporess 压缩

具体的用法就是 stream 的 pipeline

这里的 \\b 是正则表达式里的单词边界的语法，也就是逗号、空格这些。

然后创建 index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    hello
</body>
</html>
```

跑一下：

```bash
node ./zlib-http-server.mjs
```

浏览器访问：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af5fc2305c0a4f71ade065b6d4cbd8b8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1906&h=1080&s=163521&e=png&b=2c2c2c)

可以看到，返回的内容解析出来了，并且用的是 deflate 压缩格式。

我们用 curl 测试下：

```arduino
curl -H "accept-encoding: gzip" --compressed -i http://localhost:8080
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f9f74fe395047aa9d5f0cde8349c7da~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1144&h=682&s=150384&e=png&b=010101)

```arduino
curl -H "accept-encoding: deflate" --compressed -i http://localhost:8080
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/502229320dde4a85a69c149b4f40ab33~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1144&h=684&s=125419&e=png&b=010101)

没啥问题。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/node-api-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/node-api-test")

## 总结

这节我们又过了一些 node 内置模块的 api：

* **stream**：流相关的 api，主要有 Readable、Writable、Duplex、Transform 4 种流，以及可以通过 pipeline 把流连接起来，很多 node 的 api 都是基于流的
* **http**：主要是通过 http.createServer 创建 http 服务，通过 http.request 发送 http 请求，请求响应也是基于 stream 实现的
* **fs**：文件、目录的增删改查
* **zlib**：用于 http 服务的 deflate、gzip、br 等压缩算法

这些模块的 api 用的都很常用，有必要好好掌握。