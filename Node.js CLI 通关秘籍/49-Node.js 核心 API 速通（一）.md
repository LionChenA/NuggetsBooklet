这节开始我们整体过一遍 Node.js 的 api。

其实前面我们用到了很多的 api，但是没有整体梳理一遍，这节开始，整体过一遍。

我们只学习核心的 api，其余的用到查文档就行。

创建个项目：

```bash
mkdir node-api-test
cd node-api-test
npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56dd835f0119445583bddfc5f8c47e34~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=824&h=680&s=86165&e=png&b=010101)

## events

首先是 events 模块：

创建 events.js

```javascript
const EventEmitter = require('node:events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

myEmitter.on('aaa', (data) => {
  console.log('aaa 事件触发', data);
});

myEmitter.once('bbb', (data) => {
    console.log('bbb 事件触发', data);
});

myEmitter.emit('aaa', 1);
myEmitter.emit('aaa', 2);
myEmitter.emit('bbb', 3);
myEmitter.emit('bbb', 4);
```

创建一个 class 继承 EventEmitter，之后 new 一个实例。

on 注册的事件可以多次触发，而 once 注册的只会触发一次。

跑一下：

```bash
node ./events.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7390608ae6c8481da00cb0c503b6bbfc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=420&h=164&s=24005&e=png&b=191919)

可以看到 aaa 事件触发了两次，而 bbb 事件只触发了一次。

EventEmiiter 虽然比较简单，但是用的很多。

你可以安装下 @types/node，会有 node api 的类型提示：

```css
npm install --save-dev @types/node
```

## path

然后是 path 模块，它是用来做文件路径处理的：

创建 path.js

```javascript
const path = require('node:path');

const filePath = __filename;

console.log(filePath)
console.log(path.dirname(filePath));
console.log(path.basename(filePath));
console.log(path.extname(filePath));
```

这里用 \_\_filename 拿到当前文件路径，然后用 dirname、basename、extname 拿到目录名、文件名、后缀名。

跑一下：

```bash
node ./path.js
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6d82e14fb46487bac6f2f8efb928d58~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=600&h=160&s=23183&e=png&b=191919)

当然，\_\_dirname、\_\_filename 只在 commonjs 的模块里有，如果是 es module 就要用 import.meta.url

改下文件后缀名为 path.mjs

```javascript
import path from 'node:path';
import { fileURLToPath } from 'node:url'

const filePath = fileURLToPath(import.meta.url)

console.log(filePath)
console.log(path.dirname(filePath));
console.log(path.basename(filePath));
console.log(path.extname(filePath));
```

再跑下：

```bash
node ./path.mjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb568bf70ac14f29a9597e522aeda27b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=640&h=170&s=24520&e=png&b=191919)

我们可以通过改 package.json 里的 type 为 module 或者 commonjs 来告诉 node 这些 js 文件是什么模块规范：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/924675af3cf24c4b99b690dc28644f11~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=480&h=306&s=36298&e=png&b=202020)

或者也可以通过后缀名改为 mjs 告诉 node 这个文件是 es module 的模块规范。

继续来测试其他 api：

path2.js

```javascript
const path = require('node:path');

const filePath = path.join('../', 'node-api-test', './', 'path2.js');

console.log(filePath);

const filePath2 = path.resolve('../', 'node-api-test', './', 'path2.js');

console.log(filePath2);

console.log(path.relative('/a/b/c', '/a/d'));

console.log(path.parse(__filename));
```

path.join 可以把多个路径连接起来，解析其中的 ../ ./，合并成一个路径。

path.resolve 也是连接多个路径，但最后会返回一个绝对路径。

path.relaive 是 a 路径到 b 路径的相对路径。

path.parse 是解析路径。

跑一下：

```
node path2.js
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6dc9dd2c54ac4a9ba22d52b6baf47f7b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=632&h=320&s=40814&e=png&b=181818)

## import.meta

刚才用到了 import.meta.url，其实还有别的属性、方法可以用。

创建 meta.js

```javascript
console.log(__dirname);
console.log(__filename);
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db5b5a9ffc534f1db414c94720d2b5c3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=682&h=356&s=49873&e=png&b=1c1c1c)

首先，在 commonjs 模块里，可以用 \_\_dirname、\_\_filename 来拿到目录名、文件名。

而在 es module 模块里，是用 import.meta

创建 meta2.mjs

```javascript
import url from 'node:url';

console.log(import.meta.url);
console.log(import.meta.resolve('./a.js'))

console.log(import.meta.dirname);
console.log(import.meta.filename);

console.log(url.fileURLToPath(import.meta.url))
```

import.meta.url 是拿到当前文件以 file:// 开头的路径。

import.meta.resolve 是基于当前目录和传入的路径来解析路径。

这里的 import.meta.dirname 要 node 版本 20.11 以上才有：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96956bc586544d61b52776cd22d0c1f6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1192&h=970&s=124455&e=png&b=fefefe)

文档里也写了，和 url.fileURLToPath(import.meta.url) 等价。

跑一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fbf59db2b31451693d101befd450f78~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=882&h=618&s=115633&e=png&b=1d1d1d)

## url

刚才用到了 url.fileURLToPath，我们再来过一下其他 api：

创建 url.mjs

```javascript
import url from 'node:url';

const myURL =
  new url.URL('https://user:pass@sub.example.com:8080/xxx/yyy?a=1&b=2#hash'); 

console.log(myURL.hash, myURL.host, myURL.searchParams);

console.log(myURL.searchParams.get('a'));

myURL.searchParams.set('b', 222);
myURL.searchParams.append('c', 333);

console.log(myURL.searchParams.toString())
```

创建一个 URL 实例，传入 url 字符串。

它会解析出 url 中各部分的内容，并且会把 query string 封装成 URLSearchParams 的实例。

URLSearchParams 有 get、set、append、toString 等方法。

跑一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/406482827b6c4e03b33cea703b7d8dfe~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1106&h=624&s=123074&e=png&b=1d1d1d)

当然，你也可以直接 new URLSearhParams

```javascript
const params = new url.URLSearchParams('?aa=1&bb=2');
console.log(params);
for (const [name, value] of params) {
    console.log(name, value);
}
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cac4eb12f47242d48ace98fba5db926b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1038&h=508&s=92548&e=png&b=1b1b1b)

还有一个 api 也比较常用，就是 url.urlToHttpOptions

```javascript
console.log(url.urlToHttpOptions(myURL));
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9838b55c27674dda84abb89f871cea54~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1164&h=644&s=101369&e=png&b=1a1a1a)

当你使用 http.request 或者 https.reqeust 的时候，需要传递对象形式的 options。

这时候就可以用 url.urlToHttpOptions 来解析 url 字符串来生成：

创建 url2.mjs

```javascript
import http from 'node:http';
import url from 'node:url';

const options = {
    method: 'GET',
    host: 'www.baidu.com',
    port: 80,
    path: '/'
};

const req = http.request(options, res => {
    res.on('data', (chunk) => {
        console.log(chunk.toString());
    });
    res.on('end', () => {
        console.log('done');
    });
});

req.end();
```

调用 http.request 发请求，返回的 res 是个可读流，监听 data 事件，打印返回的内容。

跑一下：

```
node url2.mjs
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/677b319684e74dca8e980c9e9eaac7a2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1312&h=514&s=102852&e=png&b=181818)

当然，你也可以用刚才的 url.urlToHttpOptions 方法来写：

```javascript
const options = url.urlToHttpOptions(new URL('http://www.baidu.com:80/'));

console.log(options);
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98fa0a66c85c4d79bab645969c3a66d7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1378&h=1012&s=172948&e=png&b=1c1c1c)

## os

os 模块可以拿到很多系统的信息，比如 cpu、内存、home 目录等。

创建 os.mjs

```javascript
import os from 'node:os';

console.log('aaa' + os.EOL + 'bbb' + os.EOL);
```

跑一下：

```lua
node os.mjs
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec82d5f2e69a4f40a36a4224c8e7307d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=356&h=128&s=10356&e=png&b=181818)

os.EOL 是换行符，在 windows 上是 \\r ，在其他系统上是 ，所以直接用 os.EOL 更好

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2dd548ef250943ee852f7e6ccb5770f1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=772&h=458&s=41916&e=png&b=fefefe)

os.cpus() 返回的是 cpu 内核的信息：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06f1c424f12e4c10a2aa835a82ee1f7e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1278&h=1342&s=210447&e=png&b=191919)

model 是型号。

speed 是频率。

timers.user 是在用户模式下运行的毫秒数，timers.sys 是在 sys 模式下运行的毫秒数，idle 是空闲的毫秒数

具体是啥意思用到的时候再看就行。

其它的常用 api 也都比较简单，我们快速过一遍：

```javascript
console.log(os.type());
console.log(os.userInfo())
console.log(os.freemem(), os.totalmem());

console.log(os.homedir());
console.log(os.networkInterfaces())
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95659b52e851416f96e652c86caa11f6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1010&h=962&s=131355&e=png&b=1a1a1a)

os.type() 是系统类型，有 Darwin（也就是 mac）、Windows\_NT、Linux 等值。

os.userInfo() 是拿到当前用户相关的信息，比如 homedir、username 等。

os.freemem() 是可用内存、os.totalmem() 是总内存

os.homedir() 是 home 目录

os.networkInterfaces() 是网卡信息。

当然，你要是想拿到更丰富的系统信息，可以用 [systeminformation](https://www.npmjs.com/package/systeminformation "https://www.npmjs.com/package/systeminformation")这个包，我们在写 cli 仪表盘的时候用过。

![2024-10-23 18.43.16.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca3bdd55cc2443afb9f0975bbf915620~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1160&h=1322&s=1874604&e=gif&f=46&b=2a0c1f)

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/node-api-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/node-api-test")

## 总结

这节我们过了一些 node 常用的 api。

有这些模块：

* **events**：提供了 EventEmitter 类，可以用 on、once 注册监听器，用 emit 触发事件
* **path**：用来处理文件路径的，dirname、basename、extname 方法分别拿到目录名、文件名、后缀名，而 join、resolve、relative、parse 等方法是用来连接、解析路径的
* **import.meta**：在 commonjs 里用 \_\_dirname、\_\_filename 等变量来拿到当前文件名、当前目录，而在 es module 里是用 import.meta.dirname、import.meta.filename，这要 node 20 以上才有，低版本可以用 url.fileURLToPath(import.meta.url)
* **url**：用来解析 URL 的，可以 new URL 来拿到各部分的内容，还可以 new URLSearchParams 来处理 query string
* **os**：拿到系统信息，比如 cpu、内存、homedir、网卡信息、EOL 等。

这些 api 虽然简单，但都是很常用到的。