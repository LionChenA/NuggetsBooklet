这节我们继续来学习 Node.js 核心 API。

## crypto

crypto 模块是用来加密的，有很多加密的 api。

我们平时存入数据库的密码做加密，一般就会用这个模块：

创建 crypto.mjs

```javascript
import crypto from 'node:crypto';

export function md5(str) {
    const hash = crypto.createHash('md5');
    hash.update(str);
    return hash.digest('hex');
}

console.log(md5('123456'));
```

我们不会直接明文存储密码，而是会 md5 然后转为 16 进制字符串再存。

跑一下：

```bash
node ./crypto.mjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08f393f016034c0ebb88782eeccfa4e5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=566&h=72&s=21040&e=png&b=191919)

你还可以用流的方式来处理：

创建 crypto2.mjs

```javascript
import { createHash } from 'node:crypto';
import { Readable } from 'node:stream';

const rs = new Readable();
rs._read = function() {
    this.push('123456');
    this.push(null);
}

const hash = createHash('md5');
rs.pipe(hash).setEncoding('hex').pipe(process.stdout);
```

创建一个 Readable，重写 \_read 方法，通过 this.push 返回内容，this.push(null) 结束。

把这个可读流 pipe 到 crypto 的转换流，最后 pipe 到 process.stdout 标准输出流。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c058c1b8c91541b9abe80a655cfccf4c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=720&h=326&s=55323&e=png&b=1f1f1f)

跑一下：

```
node crypto2.mjs
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1094208c91104445ad0bcfed40039f02~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=548&h=84&s=16691&e=png&b=191919)

除了 md5 外，crypto 还支持 sha256 等加密算法，也很常用：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76968a79a5a34c40b11549bebb71cdfe~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1010&h=334&s=65257&e=png&b=1c1c1c)

此外，你还可以用 crypto 产生随机数：

创建 crypto3.mjs

```javascript
import crypto from 'node:crypto';

console.log(crypto.randomInt(10));
console.log(crypto.randomInt(10));
console.log(crypto.randomInt(10));
console.log(crypto.randomInt(10));

console.log(crypto.randomUUID());
console.log(crypto.randomUUID());
```

产生 0 到 9 的随机数，或者产生唯一的 uuid：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18930ec8758548c2b8c240866b152856~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=588&h=238&s=29674&e=png&b=181818)

## buffer

Buffer 是用来操作二进制数据的，前面单独一节写过。

我们来复习下：

JS 标准里有 ArrayBuffer 的 api 来保存字节数据，但它不能直接读写。

可以用 TypedArray 也就是 Uint8Array、Uint16Array 等以一个字节、两个字节为单位，通过下标读写。

也可以用 DataView 的 setUint16、getUint8 等来灵活读写。

ArrayBuffer 是存储可变数据的，可以读写，而 Blob 是用来不可变的二进制数据，用来传递一些参数之类的很合适，浏览器里的 File 就是 Blob 的子类，有 text、arrayBuffer 等方法来转换成别的格式。

ArrayBuffer、Blob、DataView、TypedArray 都是 JS 标准里的，不管是浏览器还是 Node.js 都有这些 api。

此外，Node.js 扩展了 Buffer 类，它继承了 ArrayBuffer，可以通过 alloc、from 创建 buffer，通过 readUint8、writeUint16LE 等来灵活读写。

Node.js 里很多 api 都是基于 Buffer 的，比如 fs.readFile。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ec1e39885b9475da5e21cd29e3891ea~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1086&h=512&s=78601&e=png&b=1d1d1d)

当你指定字符集的时候，才会根据编码转为字符串：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5862394402b34f7dbdc2d8c60e43ce77~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1096&h=770&s=114220&e=png&b=1b1b1b)

创建 buffer.mjs

```javascript
import { Buffer } from 'node:buffer';

const buf1 = Buffer.alloc(10, 6);

const buf2 = Buffer.from('神说要有光', 'utf-8');

const buf3 = Buffer.from([1, 2, 3]);

console.log(buf1.toString('hex'))

console.log(buf2.toString('utf-8'))
console.log(buf2.toString('base64'))

console.log(buf3.toString('hex'))
```

我们分别用 Buffer.alloc 创建了长度为 10 用 6 填充的 buffer，用 Buffer.from 创建了用数组、字符串来填充的 buffer。

然后转为 hex、utf-8、base64 等编码格式。

跑一下：

```arduino
node buffer.mjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae4ff0b351014644b5ccfa49121dee0b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=398&h=166&s=19617&e=png&b=191919)

buffer 自带和 DataView 一样的灵活读写的 api：

创建 buffer2.mjs

```javascript
const { Buffer } = require('node:buffer');

const buffer = Buffer.alloc(10);

buffer.writeUint16LE(256, 0)

console.log(buffer.readUInt16LE(0));
console.log(buffer.readUint8(0), buffer.readUint8(1));
```

在下标 0 的位置写入占 2 个字节的 256、然后在 0 和 1 的位置分别读取两个字节的内容。

跑一下：

```
node buffer2.mjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18f729cee88b4a38a7b63b8681f35cba~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=394&h=106&s=12368&e=png&b=181818)

此外，这里的 LE、BE 我们也学过：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb300a4c391643ecaf3c8070b322c58a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=928&h=354&s=68167&e=png&b=1f1f1f)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/638516f95079456db8d4ec43030a7803~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=616&h=322&s=53734&e=png&b=202020)

这是不同的数据存储顺序：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abd48566da4c492b9e60d062265d3d0b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1174&h=826&s=284774&e=png&b=fdfdfd)

解析网络协议的时候都是大端的顺序（BE），而在处理本机数据的时候可能就要用小端的顺序（LE）了。

这些用到的时候再细看就行。

## util

util 模块顾名思义就是一些工具方法。

我们用一下：

创建 util.mjs

```javascript
import util from 'node:util';

console.log(util.format(
`这是一个数字：%d 
这是一个字符串：%s
这是一个 JSON：%j
这是一个 对象：%o`, 111, '神说要有光', {
    a: 1, 
    b: { 
        c: 2 
    }
}, {
    a: 1, 
    b: { 
        c: 2 
    }
}));
```

util.format 就是可以在字符串中加一些占位符，然后后面的参数传入这些占位符。

不同占位符含义不同：

* %s：字符串
* %d：数字
* %i：整数
* %f：浮点数
* %j：JSON
* %o：对象
* %%：%

跑一下：

```bash
node ./util.mjs
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b11631edeae1447f88c76a22fc1ad463~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=592&h=172&s=29831&e=png&b=191919)

当然，如果你是要打印日志的话，console.log 自带了 format 的支持：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f8cfbeb658641c6a192c424365fca72~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=812&h=930&s=111987&e=png&b=1e1e1e)

没必要调用 util.format。

但如果你想拿到 format 之后的字符串，就可以用它了。

util.getCallSites 可以拿到调用堆栈

创建 util2.mjs

```javascript
import util from 'node:util';

function bbb() {
  const callSites = util.getCallSites();

  callSites.forEach((callSite, index) => {
    console.log(`CallSite ${index + 1}:`);
    console.log(`Function Name: ${callSite.functionName}`);
    console.log(`Script Name: ${callSite.scriptName}`);
    console.log(`Line Number: ${callSite.lineNumber}`);
    console.log(`Column Number: ${callSite.column}`);
    console.log()
  });
}

function aaa() {
  bbb();
}

aaa(); 
```

跑一下：

```
node util2.mjs
```

这个 api 要切换到 node 22 以上才有。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d931bbf714b498fb1f35c30b0244d82~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=998&h=732&s=104063&e=png&b=181818)

记得我们之前写过一个重写 error stack，换成 sourcemap 之后的文件名和行列号的过功能么？

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b1bc3afd17545189f51fcaeb56e85fd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1556&h=1432&s=384540&e=png&b=1f1f1f)

那个是通过重写 Error.prepareStackTrace 才拿到的报错时的调用堆栈。

而如果代码没报错，就可以用 util.getCallSite 拿到调用堆栈。

util.debug 也挺常用：

创建 util3.mjs

```javascript
import util from 'util';

const flag = util.debug('flag');

if (flag.enabled) {
  console.log('这是一条 debug 日志 111');
}

const flag2 = util.debug('flag2');

if (flag2.enabled) {
  console.log('这是一条 debug 日志 222');
}
```

当你希望有环境变量 flag 的时候，才打印日志 111，有环境变量 flag2 的时候打印日志 222，就可以用 util.debug

跑一下：

```bash
node ./util3.mjs
export NODE_DEBUG=flag && node ./util3.mjs
export NODE_DEBUG=flag2 && node ./util3.mjs
```

直接跑没任何打印：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/496c69a983554560b04f46756f9fddc1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=426&h=66&s=14263&e=png&b=181818)

当你加上对应的环境变量的时候

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc0305c2c9f94330ae17ec6a59361dcf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=760&h=168&s=44871&e=png&b=191919)

但你加上环境变量之后，就会执行对应的逻辑了。

还有一个方法也比较常用，就是 promisify

创建 util4.mjs

```javascript
import util from 'node:util';
import cp from 'node:child_process';

cp.exec('ls -l', (stderr, stdout) => {
    console.log(stdout);
});
```

用 child\_process 模块的 exec 跑 shell 命令的时候，通过第二个参数的回调函数拿到返回值。

跑一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85cd90d559814c8c9c8c82de69444262~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=968&h=584&s=144718&e=png&b=191919)

这样写起来不方便，我们可以用 util.promisify 转成 promise 版本：

```javascript
import util from 'node:util';
import cp from 'node:child_process';

// cp.exec('ls -l', (stderr, stdout) => {
//     console.log(stdout);
// });

const exec = util.promisify(cp.exec);

async function main() {
  const { stdout, stderr } = await exec('ls -l');
  console.log('stdout:', stdout);
  console.error('stderr:', stderr);
}
main();
```

跑一下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/682282dba87a4b97b0a9e966204e99f3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=976&h=532&s=129068&e=png&b=191919)

## sqlite

[Node.js 22](https://nodejs.org/docs/latest/api/sqlite.html "https://nodejs.org/docs/latest/api/sqlite.html") 之后，内置了 sqlite 模块，可以用来存储一些复杂的关系型数据：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed6a6f1069f848f0b17ff682ad09d9da~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1760&h=1086&s=214310&e=png&b=fcfcfc)

创建 sqlite.mjs

```javascript
import { DatabaseSync } from 'node:sqlite';
const database = new DatabaseSync('data.db');

database.exec(`
  CREATE TABLE student(
    id INTEGER PRIMARY KEY,
    name TEXT,
    age INT
  ) STRICT
`);

const insert = database.prepare('INSERT INTO student (id, name, age) VALUES (?, ?, ?)');
insert.run(1, '张三', 20);
insert.run(2, '李四', 21);
insert.run(3, '王五', 22);

const query = database.prepare('SELECT * FROM student ORDER BY id');
console.log(query.all());
```

创建 DatabaseSync 的实例，指定存储的文件位置。

exec 方法是执行 sql

prepare 方法也是准备 sql，其中 ? 是占位符，后面调用 run 方法传入具体的值才会执行。

我们先创建了 student 表，插入了三条数据，然后查询出来。

跑一下：

```bash
node --experimental-sqlite ./sqlite.mjs
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89a54c67f5fe4b9b802c42c7c7933ae4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=956&h=192&s=39998&e=png&b=181818)

记得要切换下 node 版本到 22 以上再跑，而且要加上 --experimental-sqlite 才行，现在还是实验性的。

执行后创建了表、并插入了几条数据，然后用 sql 查询了出来。

数据都存放在 data.db 这个文件里，二进制存储的：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bccb1980747d44cfb8e31e84ce02d19c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=746&h=398&s=42331&e=png&b=1d1d1d)

我们还可以用一些 GUI 客户端来可视化的管理。

比如 DB Browser for SQLite

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad8dfe48141540f5ad5926aa6b29a268~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=424&h=302&s=57015&e=png&b=ac4410)

点击 open database，选择刚才的文件，就可以看到 db browser 把其中的表解析了出来：

![2024-12-08 10.12.02.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ba6a131e42c4d41a70e8a52c990dd8c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2062&h=1296&s=1289267&e=gif&f=70&b=eeeded)

可以看到所有的表的数据：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/899a1e0dc36f496e90603d5cf3657155~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=564&h=346&s=30898&e=png&b=f4f4f4)

还可以执行 sql，按 cmd + enter 执行：

```sql
select * from student
select * from student WHERE name = "张三"
```

![2024-12-08 10.14.53.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e639f5ed6c9a455292f8d968c49130b8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2062&h=1296&s=356461&e=gif&f=40&b=f0efef)

增删改数据后，点击 write changes 按钮，才会把改动写入文件。

当然，现在 node:sqlite 还不稳定，目前我们最好还是用 sqlite 这个包来写。

用法都差不多：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cdf0130f9204319b417c768e61b910c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1364&h=1360&s=291001&e=png&b=1f1f1f)

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/node-api-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/node-api-test")

## 总结

这节我们又学了一些 node 的内置模块：

* **crypto**：用来加密（md5、sha256 等）、产生随机数（randomInt、randomUUID）的。
* **buffer**：用来操作二进制数据，是对 ArrayBuffer 的扩展，node.js 里的很多 api 都是基于 Buffer 来读写二进制数据的
* **util**：一些工具方法，比如 util.format、util.getCallSites、util.debug、util.promisify 等
* **sqlite**：内置的 sqlite 数据库，现在还不稳定，可以先用 sqlite 这个三方包

这些模块也都是常用的，需要好好掌握。