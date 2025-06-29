我们学会了如何断点调试 Node.js 代码，但有个问题：

我们用 js 写代码的时候，可以直接在源码打断点调试，可如果源码是 ts 写的呢？

这时候跑的是编译后的 js 代码，在 ts 源码里打断点是不生效的。

调试编译后的 js 代码很不方便，还得找到对应的 ts 源码来改。

能不能直接在 ts 代码里断点调试呢？

当然是可以的，加上 sourcemap 就可以。

我们试下：

```bash
mkdir ts-debug-test
cd ts-debug-test
npm init -y
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e1b027fafa64f8595004fc48293d895~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=908&h=676&s=87898&e=png&b=000000)

然后安装 typescript

```sql
npm install typescript  @types/node --save-dev
```

创建 tsconfig.json

```csharp
npx tsc --init
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c55f9fb2d59f49428da0cac6b8de4b95~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=692&h=366&s=43074&e=png&b=181818)

改一下：

```json
{
  "compilerOptions": {
    "outDir": "dist",
    "types": [ "node" ],
    "target": "es2016", 
    "module": "NodeNext", 
    "moduleResolution": "NodeNext",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true
  }
}
```

在 package.json 设置 type 为 module：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f16e5afbf1149fab45e9d2e6067cd53~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=564&h=278&s=39147&e=png&b=1f1f1f)

然后写下 src/index.ts

```javascript
import { add } from './calc.js';

function test() {
    console.log(add(1, 2));
}

function main() {
    test();
}

main();
```

和 src/calc.ts

```javascript
function add(a: number, b: number) {
    return a + b;
}

export {
    add
}
```

编译下：

```
npx tsc -w
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad4ab0a597404b818ec7fb535c603667~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1236&h=374&s=74378&e=png&b=1d1d1d)

跑一下：

```bash
node ./dist/index.js
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20f20ced773f400ea1348590103bd9e2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=684&h=114&s=18995&e=png&b=191919)

然后调试下：

创建调试配置：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27ba8ce8ae5248da90f9160bcb80b3cd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=604&h=406&s=44142&e=png&b=181818)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c31676c0efb4f68bb0b60ee4d9dc052~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1692&h=748&s=196561&e=png&b=1c1c1c)

```json
{
    "name": "调试 index",
    "program": "${workspaceFolder}/dist/index.js",
    "request": "launch",
    "console": "integratedTerminal",
    "skipFiles": [
        "<node_internals>/**"
    ],
    "type": "node"
}
```

和直接跑 node ./dist/index.js 一样，只不过现在是调试的方式跑。

在 src/clac.ts 打个断点：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/440aea3f217241bd9d91096f17053f52~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1326&h=568&s=96495&e=png&b=1c1c1c)

点击调试启动：

![2024-10-29 12.31.23.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/562f1af8f4b344f9a139b15c82de458b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2442&h=1508&s=347482&e=gif&f=30&b=191919)

你会发现断点并没有断住。

在 dist/calc.js 打个断点再跑：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ad21a1a53574feba09d9ce8cd58c481~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1166&h=464&s=74581&e=png&b=1b1b1b)

![2024-10-29 12.32.48.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49ecbe015f094d67a81b7ea9afcb4cbb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2442&h=1508&s=466847&e=gif&f=52&b=191919)

断点处断住了。

调用栈里的执行路径也是 dist 下的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc14f5a8f662436c9e3ff48f59bf04d2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=764&h=748&s=101706&e=png&b=1b1b1b)

因为本来跑的就是编译后的 js 代码。

如果你想直接调试 ts 源码，就要加上 sourcemap 了。

改下 tsconfig.json，生成 sourcemap

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a909bf2c229b42c38c213310c8c1b12c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=770&h=544&s=82214&e=png&b=1f1f1f)

再重新执行编译：

```
npx tsc -w
```

这时候在 dist 下就生成了每个文件的 sourcemap 并且做了关联：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3fb3a4dfdbc044e3a6680ca6fea35345~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1286&h=400&s=66621&e=png&b=1c1c1c)

你可以简单看下 sourcemap 文件的内容：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68c35fc032af4d63a63bc5721a30ca8f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2152&h=424&s=73173&e=png&b=1d1d1d)

其实就是一个映射关系，映射编译后的文件的行列号的源码的那个文件的行列号。

然后把这个映射关系做了下编码。

现在我们重新在 src/calc.ts 里打下断点：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab14299625e143e89d75bf6a8d56967c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1448&h=538&s=98678&e=png&b=1c1c1c)

调试配置不变，再跑下：

![2024-10-29 12.45.37.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40dab6acf1254a3e9c533bc28e45dba0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2442&h=1508&s=650739&e=gif&f=39&b=191919)

现在就是 ts 源码里断住了：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53d36136ac88445c932a5f7c3912a0f2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1468&h=956&s=164258&e=png&b=1c1c1c)

这样就可以直接在 ts 源码里断点调试了。

开启 sourcemap 后，调试配置还可以这样写：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/def7f2fe76bc40c6ab9a803295548a6c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1030&h=684&s=104941&e=png&b=1f1f1f)

直接写源码 ts 文件的路径。

也是一样：

![2024-10-29 12.44.56.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a541f92c69174b02a0bd252783b36f62~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2442&h=1508&s=483544&e=gif&f=26&b=191919)

这就是 sourcemap 的一个作用，直接在源码里打断点调试。

其实 sourcemap 还有一个作用，就是错误堆栈里展示源码位置。

比如我们在 add 函数里抛一个错误：

```javascript
function add(a: number, b: number) {
    if(a === 1) {
        throw new Error('xxx');
    }
    return a + b;
}

export {
    add
}
```

再跑下：

```bash
node ./dist/index.js
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e270a585f804b0cb704d280bd0945b4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1326&h=652&s=131774&e=png&b=191919)

可以看到，现在打印的堆栈是编译后的代码的。

这对于我们定位代码哪里有错误很不方便。

我们加上一个 --enable-source-maps 的选项，再跑：

```bash
node --enable-source-maps ./dist/index.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/def52e0cdcba485a97030fc241a53f2d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1290&h=652&s=131431&e=png&b=191919)

可以看到，现在打印的错误堆栈就是 ts 代码里的了。

这样再定位错误代码就很方便了。

![2024-10-29 12.52.17.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b1b1e17bb154c9498ab7315426945fe~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2442&h=1508&s=469752&e=gif&f=39&b=191919)

这个 --enable-source-maps 是 node 12 才有的。

你切换下 node 版本到 11 试下。

我本地是用 n 来管理 node 版本的：

```
npm install -g n

sudo n 11
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7c4818c17bf43c2a3a7fb0a19027ef4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=738&h=160&s=27121&e=png&b=191919)

再跑下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abb5728a703f4991b745efb6092eb11e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=876&h=104&s=21393&e=png&b=191919)

现在就提示没这个选项了。

而且低版本还不支持直接跑 es module 代码，还是切换回去。

那 node12 前都是怎么解决这种问题呢？

有个 [source-map-support](https://www.npmjs.com/package/source-map-support "https://www.npmjs.com/package/source-map-support") 的包来做这个：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83b14f97c0e9429b87e414395a6c891c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1930&h=998&s=237818&e=png&b=fefefe)

现在还有 4000w+ 的周下载量

安装下：

```arduino
npm install --save-dev source-map-support
```

然后跑 node 的时候加上 -r ：

```arduino
node -r source-map-support/register ./dist/index.js
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1754d823f13148fdba1c716a30f124ec~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1352&h=340&s=93044&e=png&b=181818)

可以看到，我们没有用 --enable-source-maps 选项，现在打印的错误堆栈依然是源码里的。

这个 -r 是什么呢？

其实就是 require 一个文件，在跑代码之前跑的。

在根目录加一个 a.cjs

```javascript
console.log('aaa');
```

b.cjs

```javascript
console.log('bbb')
```

因为我们在 package.json 里设置 type 为 module 了，也就是默认 js 都是 es module 的，所以这里要用 cjs 后缀声明是 commonjs 的模块。

跑一下：

```bash
node -r ./a.cjs ./b.cjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d707298919b400e8fd9dbc8cd636d32~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=614&h=126&s=13058&e=png&b=181818)

可以看到，在跑 b.cjs 之前，先跑了 ./a.cjs

这就是 -r 的作用，在执行代码前预先执行的一些代码，一般用来做初始化之类的事情。

那 source-map-support/register 里做了啥呢？

在 source-map-support/source-map-support.js 打个断点：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6aa7940c556d47e2a4fb897d36530ab7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1730&h=768&s=247105&e=png&b=1d1d1d)

这里显然是处理 file:/ 的路径的，在这里打个断点。

改下调试配置：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7138e94ba37a45418ad176aafdf675d7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1238&h=752&s=125826&e=png&b=1f1f1f)

```json
{
    "name": "调试 index",
    "program": "${workspaceFolder}/dist/index.js",
    "runtimeArgs": [
        "-r",
        "source-map-support/register"
    ],
    "request": "launch",
    "console": "integratedTerminal",
    "skipFiles": [
        "<node_internals>/**"
    ],
    "type": "node"
},
```

再跑下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad8ed4ea44274ee28d9f76c7fe0f0691~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2124&h=1116&s=444112&e=png&b=1d1d1d)

断住之后我们来看下调用栈就能理明白。

node 内部代码会调用 Error.prepareStackTrace 这个方法：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a74a223702d4ea7addcc66dc55517ad~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2366&h=1136&s=530730&e=png&b=1d1d1d)

而 source-map-support 重写了 Error 的这个方法：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1471ddc6828041cda685af2892581eca~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1086&h=546&s=127268&e=png&b=1f1f1f)

接下来就是对调用栈的每一帧做下处理：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69067b043b9b43feb91835f870c53ff6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2202&h=1018&s=427944&e=png&b=1d1d1d)

其实很容易想到，就是读取 sourcemap，拿到打印的文件行列号对应的源码里的行列号。

也就是这样：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0a6a38cbd614f5881a7fcef90012a97~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=826&h=448&s=56759&e=png&b=1f1f1f)

先解析出文件的 sourcemap 文件的位置

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/158bf856b4284c42a530a873309f6941~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1490&h=864&s=242031&e=png&b=1f1f1f)

这里就是正则解析。

然后读取 sourcemap 的内容：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e116e9c291348248bde408e6a99d971~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2354&h=708&s=295661&e=png&b=1e1e1e)

这里用 source-map 这个包来解析 sourcemap，拿到对应的源码里的行列号。

之后返回新的栈帧就好了。

你可以和我一样在这两个位置打个断点：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b124b7fe1c98434b9a08a4e431eab257~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1538&h=1126&s=314678&e=png&b=1f1f1f)

点击跳断点执行：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f334556b420c4605bf0e33b3a6bbc526~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=460&h=96&s=11381&e=png&b=191919)

可以看到，执行到上面的断点的时候，frame 是编译后文件的位置：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d8bc9b5c9bc4a079ea54b1c6dfb8d43~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1790&h=932&s=409793&e=png&b=212121)

而执行到下面代码的时候，frame 就变了：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06d61db2e8de48d68f66ad51865ca120~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1512&h=916&s=330578&e=png&b=212121)

小结一下：

source-map-support 重写了 Error.prepareStackTrace 来处理调用栈，通过正则拿到文件的 sourcemap 文件，解析 sourcemap，拿到源码位置，替换调用栈。

这样，代码里打印的错误就都是源码位置了。

这就是 sourcemap 的一个很重要的作用，定位出错的源码位置。

线上报错就是通过 sourcemap 来定位源码的。

## 总结

这节我们学了如何直接在 ts 源码里断点调试，只要生成 sourcemap 就行，debugger 会自动解析编译后文件对应的源码位置。

此外，打印错误信息的时候，可以通过 sourcemap 直接定位出错的源码。

在 node 12 之后，用 node --enable-source-maps 跑就行。

在 node 12 之前，一般都是用 source-map-support 这个包来做。

用 node -r source-map-support/register 来跑。

\-r 是在代码执行前预先执行的一些逻辑，用来做一些初始化之类的事情。

source-map-support 的实现原理就是重写了 Error.prepareStackTrace 来处理调用栈，通过正则拿到文件的 sourcemap 文件，解析 sourcemap，拿到源码位置，替换调用栈。

sourcemap 在 node 调试、错误定位等方面都有很重要的应用，需要好好掌握。