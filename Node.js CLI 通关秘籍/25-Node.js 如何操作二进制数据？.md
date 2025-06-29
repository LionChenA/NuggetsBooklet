这节我们来学习 Node.js 如何操作二进制数据。

我们经常用的 string 存储的是字符串，比如 utf8，他可能一个字符占 1-3 个字节。

如果想操作原始的字节数组呢？

这时候就可以用 js 语言内置的 ArrayBuffer 的 api 了。

我们来用一下：

```arduino
mkdir buffer-test
cd buffer-test
npm init -y
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b30e9ed51c746548210c91308c6c571~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=842&h=666&s=86026&e=png&b=010101)

进入项目，写下 index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        const buffer = new ArrayBuffer(10);

        const arr1 = new Uint16Array(buffer);

        arr1[0] = 256;
        console.log(arr1)

        const arr2 = new Uint8Array(buffer);
        console.log(arr2);
    </script>
</body>
</html>
```

ArrayBuffer 不能直接操作，我们可以通过具体的 TypedArray 来操作。

TypedArray 也就是指定你存的是什么类型的数据，比如 8 位有符号整型、16 位无符号整型等：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84e9703a7d1b4a23a319c3ca00d8dd7f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1466&h=920&s=150785&e=png&b=fefefe)

有符号需要有一位专门来存储 0 1 代表正负，比如 8 位有符号整数，只有 7 位可以用来存二进制数字，所以范围是 2 的 7 次方：-128 到 127 （还需要存储 0）

那无符号就多了一个位来存储数字，所以范围是 2 的 8 次方，也就是 256。

Uint16Array 是用 16 位也就是 2 个字节来存储数字，而 Uint8Array 只用一个字节存储。

我们用 Uint6Array 存储了一个 256 的数字，那就超出了 Uint8Array 的范围了，会作为两个数字。

跑一下：

```vbscript
npx http-server .
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3adb360aac84e0e9a0cf3172d6c1dd1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=750&h=632&s=95036&e=png&b=181818)

访问 [http://127.0.0.1:8080](http://127.0.0.1:8080 "http://127.0.0.1:8080") ，可以看到 Uint16Array 是 5 个元素，Uint8 是 10 个元素：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61ed3284808d48d5aa70125ea3aa12b5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1426&h=766&s=98389&e=png&b=ffffff)

因为每个元素的字节数不同嘛。

Uint16Array 里的 256，在 Unit8Array 里就作为了两个数字，0 和 1。

这也是为啥要有那么多 [TypedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray#typedarray_%E5%AF%B9%E8%B1%A1 "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray#typedarray_%E5%AF%B9%E8%B1%A1")：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/707838094ccf4e2f9c60b16333523440~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1224&h=1218&s=180565&e=png&b=fefefe)

因为不同数字类型数组的长度都不同。

打印下长度：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91d77e7b3dfa4de8add93168c775c6b5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=854&h=764&s=133003&e=png&b=1f1f1f)

```javascript
const buffer = new ArrayBuffer(10);

const arr1 = new Uint16Array(buffer);

arr1[0] = 256;
console.log(arr1)
console.log(arr1.length);
console.log(arr1.byteLength);

const arr2 = new Uint8Array(buffer);
console.log(arr2);
console.log(arr2.length);
console.log(arr2.byteLength);
```

length 是取 TypedArray 的长度，而 byteLength 是取 ArrayBuffer 的长度：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cecc23b671fe40e78f09e6a9f4e65aac~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=800&h=546&s=56718&e=png&b=ffffff)

当然，如果你觉得对同一个 ArrayBuffer 操作，需要转成不同的 TypedArray 太麻烦，也可以不转，用 DataView 来操作。

写下 index2.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        const buffer = new ArrayBuffer(10);

        const dataView = new DataView(buffer);

        dataView.setUint16(0, 256);

        console.log(dataView.getUint16(0));

        console.log(dataView.getUint8(0));
        console.log(dataView.getUint8(8));
    </script>
</body>
</html>
```

我们之前用 Uint8Array、Uint16Array 读写元素，指定下标就行，会自动算出来在哪个字节读写。

但用 DataView，你需要告诉它在哪个字节开始读写，然后读写的是什么类型。

试一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04cb3aed144b40e8a5ef464187a92691~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1052&h=718&s=58441&e=png&b=ffffff)

这样读写和之前的结果一样：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61ed3284808d48d5aa70125ea3aa12b5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1426&h=766&s=98389&e=png&b=ffffff)

DataView 相比 TypedArray 更加灵活。

**如果你需要灵活的读写 ArrayBuffer 里的元素的时候就用 DataView，否则用 TypedArray 更简单。**

刚才这些代码在 node 里也一样能跑：

创建 index.js

```javascript
const buffer = new ArrayBuffer(10);

const arr1 = new Uint16Array(buffer);

arr1[0] = 256;
console.log(arr1)
console.log(arr1.length);
console.log(arr1.byteLength);

const arr2 = new Uint8Array(buffer);
console.log(arr2);
console.log(arr2.length);
console.log(arr2.byteLength);
```

跑一下：

```bash
node ./index.js
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62838c410085497aa4c1bf0cdf42669d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=724&h=378&s=36695&e=png&b=181818)

创建 index2.js

```javascript
const buffer = new ArrayBuffer(10);

const dataView = new DataView(buffer);

dataView.setUint16(0, 256);

console.log(dataView.getUint16(0));

console.log(dataView.getUint8(0));
console.log(dataView.getUint8(8));
```

跑一下：

```bash
node ./index2.js
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49f082c1be744596876172843eb54d25~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=728&h=184&s=21578&e=png&b=181818)

结果一样。

因为 node 也是用 v8 引擎来跑 js 代码的，JS 的 api 在 node 里自然也都可以用。

说回这节要学的 Buffer

它继承了 js 的 ArrayBuffer，是它的子类。

在 node 里非常多的 api 都用到了 buffer，比如你用 fs.readFile 的时候，读出来的就是 buffer：

创建 index3.js

```javascript
const fs = require('node:fs/promises');

(async function(){
    const res = await fs.readFile('./package.json');
    console.log(res);
})();
```

跑一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fee4afeaf39d4c9b9ec6ed91f7e0746b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1086&h=512&s=85831&e=png&b=1d1d1d)

当你指定字符集的时候，才会根据编码转为字符串：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5862394402b34f7dbdc2d8c60e43ce77~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1096&h=770&s=114220&e=png&b=1b1b1b)

**在 Node.js 里，操作二进制数据都是用 Buffer**

我们来学一下它的 api：

index4.js

```javascript
const { Buffer } = require('node:buffer');

const buf1 = Buffer.alloc(10, 6);

const buf2 = Buffer.from('神说要有光', 'utf-8');

const buf3 = Buffer.from([1, 2, 3]);

console.log(buf1.toString('hex'))

console.log(buf2.toString('utf-8'))
console.log(buf2.toString('base64'))

console.log(buf3.toString('hex'))
```

new Buffer 的方式被废弃了，创建 Buffer 一般用 Buffer.alloc 或者 Buffer.from

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba44fe52184049a2806f5c7eadf45f49~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1032&h=300&s=68822&e=png&b=212121)

Buffer.alloc(10, 6) 是分配一个 10 个字节的 buffer，用 6 填充。

Buffer.from('神说要有光', 'utf-8') 是把神说要有光按照 utf-8 转为字节数组，创建的对应的 buffer。

Buffer.from(\[1, 2, 3\]) 是根据传入的字节数组来创建 buffer。

然后我们调用了 toString 方法，分别按照 hex、utf-8、base64 的格式打印。

跑一下：

```bash
node ./index4.js
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74af50464f154745be15b9f3756fe505~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=718&h=504&s=79035&e=png&b=1c1c1c)

可以看到，可以转成不同的格式来打印。

这里的 utf-8 换成 utf8、UTF8 等都一样。

Buffer 对 ArrayBuffer 做了这些扩展，但它毕竟是 ArrayBuffer 的子类，所以可以作为 ArrayBuffer 用：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e08187a53f14f48ab0534c6e5d80fa8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=774&h=698&s=114757&e=png&b=1c1c1c)

```javascript
console.log(new Uint8Array(buf3));
```

此外，Buffer 自带了 DataView 的 api：

创建 index5.js

```javascript
const { Buffer } = require('node:buffer');

const buffer = Buffer.alloc(10);

buffer.writeUint16LE(256, 0)

console.log(buffer.readUInt16LE(0));
console.log(buffer.readUint8(0), buffer.readUint8(1));
```

跑一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1563b4a70646448fa60dc6b880858db5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=910&h=374&s=60596&e=png&b=1c1c1c)

这其实就是我们之前用 DataView 做的事情：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4705f6a55354457ebbab0ac7335bd5c7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=706&h=458&s=75740&e=png&b=1f1f1f)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04cb3aed144b40e8a5ef464187a92691~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1052&h=718&s=58441&e=png&b=ffffff)

Buffer 内置了 DataView 这些灵活读写字节数组的 api。

有的同学可能会问，这里多了个 LE 是啥：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb300a4c391643ecaf3c8070b322c58a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=928&h=354&s=68167&e=png&b=1f1f1f)

其实除了 LE 还有 BE：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/638516f95079456db8d4ec43030a7803~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=616&h=322&s=53734&e=png&b=202020)

这是不同的数据存储顺序：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abd48566da4c492b9e60d062265d3d0b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1174&h=826&s=284774&e=png&b=fdfdfd)

解析网络协议的时候都是大端的顺序（BE），而在处理本机数据的时候可能就要用小端的顺序（LE）了。

当然，不用细究，等用到再说。

此外，这个 Buffer 对象被挂到了全局，就算不引入 node:buffer 模块也可以用：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3db9c7621c654ea58e2d513756e881e5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1136&h=704&s=110765&e=png&b=1d1d1d)

但最好还是显式引入。

此外，Buffer 因为有 readUint8、writeUint8 这种读写的 api，所以是可变的。

有的时候，我们要求二进制数据不可变，不可以读写，这时候就要用 [Blob 的 api](https://nodejs.org/docs/latest/api/buffer.html#class-blob "https://nodejs.org/docs/latest/api/buffer.html#class-blob") 了。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a8ffe0c62794408b976d3ca5b574cc9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1472&h=524&s=89262&e=png&b=fefefe)

涉及到多线程的数据，基本都是用 Blob 而不是 Buffer。

为什么呢？

如果多个线程并发的改一个可变的数据，那你怎么知道读取的数据是不是被哪个线程改过了的？

所以都是用 Blob，不让它变。

在 [Node.js 文档](https://nodejs.org/docs/latest/api/buffer.html "https://nodejs.org/docs/latest/api/buffer.html")里，你可以看到 buffer 有这么多读写方法：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12772654ae394160ba396e5a48bcf4c0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1588&h=1340&s=214256&e=png&b=ffffff)

而 Blob 没有：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9356c16f472f4a6fb91268ed59b8ce86~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1408&h=1354&s=208730&e=png&b=ffffff)

之前用 node:workder-threads 包写多线程案例的时候，我们用过 MessageChannel。

它有 port1、port2 两个端口，在一边 postMessage、另一边可以通过 message 事件拿到消息：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fdd2275daac4a04a4dbb62c98a58ee0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=884&h=634&s=849085&e=png&b=979591)

当传递二进制数据的时候，最好用 Blob 而不是 Buffer：

index6.js

```javascript
const { Blob } = require('node:buffer');

const blob = new Blob(['神说要有光']);

const { port1, port2 } = new MessageChannel();

port1.onmessage = async ({ data }) => {
  console.log(data);
  console.log(await data.text())
  console.log(await data.arrayBuffer())
};

port2.postMessage(blob);
```

跑一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/656fe420595343ee8696d89ead1dad84~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1152&h=876&s=156859&e=png&b=1d1d1d)

可以通过 blob.text、blob.arrayBuffer 来把二进制数据转为文本、ArrayBuffer。

当你想传递不可变二进制数据的时候，就用 Blob。

其实浏览器里也有 Blob 对象，也是用来放不可变数据的，比如 File 对象就是它的子类：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3366c1cc90e4224ba11d9c22881b9fe~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1540&h=666&s=113578&e=png&b=ffffff)

api 都是一样的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4b3086ca57f4a59ba80cf00769eaf9b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1106&h=948&s=132314&e=png&b=fefefe)

因为都是 JS 语言标准里的 API。

至此，Buffer、Blob 我们就都会用了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/buffer-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/buffer-test")

## 总结

不管是浏览器还是 Node.js 都有操作二进制数据的需求，所以 JS 标准里有 ArrayBuffer、Blob、DataView、TypedArray 等 api。

* ArrayBuffer 是用来存储可变的二进制数据的，通过 Uint8Array 等 TypedArray 来通过下标读写，或者通过 DataView 的 setUint16、getUint8 等来灵活读写。

* Blob 是不可变的二进制数据，用来传递一些参数之类的很合适，浏览器里的 File 就是 Blob 的子类，有 text、arrayBuffer 等方法来转换成别的格式。

* Node.js 里继承 ArrayBuffer 实现了 Buffer 的 api，可以通过 alloc、from 创建 buffer，通过 readUint8、writeUint16LE 等来灵活读写

Node.js 里很多 api 都是基于 Buffer 的，比如 fs.readFile。

但当你需要传输不可变数据的时候，还是用 Blob 更合适一点。

可能大家平时很少会操作二进制数据，但这个是必须掌握的知识点，不管是写页面还是写 Node.js。