我们每天都在用 console.log 打印日志，一般都是从上到下依次打印的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f34817b433614a29af508a63c17a4582~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=496&h=384&s=40319&e=png&b=1c1c1c)

但很多 Node.js 工具可以在任意的位置打印内容：

![2024-09-01 12.24.12.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ff4859a02914df48f6f7e926bdfcf7a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1410&h=942&s=235254&e=gif&f=70&b=020202)

比如 npkill 这个包，它就可以把文件夹大小异步的打印在对应行的右边。

而且还能显示颜色。

都是 console.log，咋做到的呢？

这就涉及到 ANSI 字符了。

其实终端比图形界面出现的早得多，那时候所有的交互都是在终端上进行的。

如果只能从上到下依次打印，还不支持颜色，想象一下，得多难用。

所以 1970 年左右，就出现了 ANSI 标准。

看下[维基百科](https://zh.wikipedia.org/wiki/ANSI%E8%BD%AC%E4%B9%89%E5%BA%8F%E5%88%97 "https://zh.wikipedia.org/wiki/ANSI%E8%BD%AC%E4%B9%89%E5%BA%8F%E5%88%97")怎么说的：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54a3ae6797124edbaa4d08b3a0cd6f87~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2364&h=1264&s=650701&e=png&b=fefefe)

那个年代就是靠这种 ANSI 控制字符移动光标。

它并不像 aaa、bbb 这样会展示对应的字符，而是遇到就会解释为指令，执行相应的代码。

ANSI 字符可以用来改变光标位置、改变颜色。

那具体是什么样子呢？

可以在[这里](https://www2.ccs.neu.edu/research/gpc/VonaUtils/vona/terminal/vtansi.htm "https://www2.ccs.neu.edu/research/gpc/VonaUtils/vona/terminal/vtansi.htm")看到。

可以控制这么多东西：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7b444bde373410c9c84e4f51b72c52b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2126&h=1024&s=287760&e=png&b=fefefe)

光标、擦除、颜色等。

我们简单看几个：

比如 <ESC>\[1K 可以擦除光标到这一行开头的内容：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6dfc626236c2419094fb874446091ec9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1006&h=708&s=97983&e=png&b=fefefe)

试一下：

```bash
mkdir ansi-test
cd ansi-test
npm init -y
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99dc429dc6c341a891beb717459b44d7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=834&h=650&s=84262&e=png&b=010101)

创建 index.js

```javascript
console.log('123\u001B[1K456')
```

跑一下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94b183a4a78c48f7847a936fb7573f0e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=642&h=278&s=33490&e=png&b=1c1c1c)

可以看到前面的内容被擦除了。

其实你每天在用这个，比如 vite：

![2024-09-01 15.07.10.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ee8cf4c28d24c2f864409adfb1320d2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1380&h=594&s=576505&e=gif&f=54&b=171717)

当你执行 npm run dev 的时候，就会滚动终端。

怎么实现的呢？

看下[源码](https://github.com/vitejs/vite/blob/8c661b20e92f33eb2e3ba3841b20dd6f6076f1ef/packages/vite/src/node/logger.ts#L44 "https://github.com/vitejs/vite/blob/8c661b20e92f33eb2e3ba3841b20dd6f6076f1ef/packages/vite/src/node/logger.ts#L44")：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/484a002f6fec45f393497ef8a533ff97~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1134&h=524&s=107651&e=png&b=fffcfc)

首先拿到当前终端能显示的行数 process.stdout.rows

打印这么多空行

然后移动 cursor 到 0 行 0 列的位置，之后清除下面的内容：

我们跑一下：

index2.js

```javascript
import readline from 'node:readline';

const repeatCount = process.stdout.rows - 2;
const blank = repeatCount > 0 ? '
'.repeat(repeatCount) : '';
console.log(blank);
```

要直接执行 es module 的代码，需要在 package.json 里配置下 type 为 module：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d873c4a3aba54d0fb28f6c80c968e31c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=550&h=318&s=44923&e=png&b=202020)

跑一下：

打印空行的效果是这样： ![2024-09-01 15.19.00.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b331c1c9dee7475b8a3d6ea30ef072f9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1882&h=854&s=189631&e=gif&f=45&b=151716)

再加上移动光标和清空下面的内容：

```javascript
readline.cursorTo(process.stdout, 0, 0);
readline.clearScreenDown(process.stdout);
```

![2024-09-01 15.21.17.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f189fe6f99ef4ce3b25da339071833dd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1882&h=854&s=119093&e=gif&f=33&b=171717)

这就是 vite 这个 cli 的实现原理：

![2024-09-01 15.07.10.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ee8cf4c28d24c2f864409adfb1320d2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1380&h=594&s=576505&e=gif&f=54&b=171717)

显然，readline 的底层也是通过 ANSI 的控制字符来修改的光标位置和清除内容的。

平时我们经常用到 ANSI 控制字符来修改光标位置，但是直接写那些字符很麻烦，我们会用封装好的包来做。

node 内置的 readline 模块也可以做这个，比如 cursorTo、clearScreenDown 等方法。

但它并不全。

想做更多的控制，我们会用 [ansi-escapes](https://www.npmjs.com/package/ansi-escapes "https://www.npmjs.com/package/ansi-escapes") 这个包来做：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7cffcd84461b43ceb06f1ab25233c696~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2422&h=1160&s=230250&e=png&b=fcfcfc)

周下载量 3000w，好家伙。

它封装了修改 cursor 位置的方法：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f67c15374b2d443a98edcd115d2704a6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=708&h=1084&s=121149&e=png&b=fefefe)

擦除终端内容的方法：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5309f563addf49cca4fd26ec19469da6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=860&h=1026&s=114702&e=png&b=ffffff)

我们用一下：

```css
npm install --save ansi-escapes
```

index3.js

```javascript
import ansiEscapes from 'ansi-escapes';

const log = process.stdout.write.bind(process.stdout);

log(ansiEscapes.cursorTo(10, 1) + '111');
log(ansiEscapes.cursorTo(7, 2) + '222');
log(ansiEscapes.cursorTo(5, 3) + '333');

setTimeout(() => {
    log(ansiEscapes.cursorTo(0, 2) + ansiEscapes.eraseEndLine);
    log(ansiEscapes.cursorTo(5, 3) + '444')
}, 1000)
```

console.log 会打印一个换行符，所以我们用 processs.stdout.write 打印。

cursorTo 第一个参数是列号、第二个参数是行号。

跑一下：

![2024-09-01 16.06.27.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6177689cf0e74b0fb6efe04e7042599b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1634&h=764&s=50568&e=gif&f=28&b=171717)

可以看到，在不同位置打印了内容，并且 1s 后修改了第二行的内容。

这就是 npkill 这种能够在不同位置打印内容的原理：

![2024-09-01 12.24.12.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ff4859a02914df48f6f7e926bdfcf7a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1410&h=942&s=235254&e=gif&f=70&b=020202)

接下来我们再来看 ANSI 支持的颜色打印：

打印颜色是这样的格式：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e7d7a92f6c0459f83bcb9a9ce779a81~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1390&h=238&s=103978&e=png&b=fffefe)

除了 ESC\[ 外，还要加上前景色、背景色、加粗、下划线等的设置，用 ; 分割，最后用 m 表示结束。

index4.js

```javascript
console.log('\u001b[36;1;4mguang');
```

36 表示前景色为青色、1 表示加粗、4 表示下划线

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e38e036565f74b17a8ea035aba66c465~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=722&h=290&s=37691&e=png&b=1c1c1c)

当然，加了样式还要去掉，再加一个 \\u001b\[0m 就可以了。

```javascript
console.log('\u001b[36;1;4mguang\u001b[0m 666');
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c1a25eeb29c4b169d484c97c120910c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=840&h=286&s=43124&e=png&b=1c1c1c)

颜色代码[可以搜到](https://zh.wikipedia.org/wiki/ANSI%E8%BD%AC%E4%B9%89%E5%BA%8F%E5%88%97#%E9%A2%9C%E8%89%B2 "https://zh.wikipedia.org/wiki/ANSI%E8%BD%AC%E4%B9%89%E5%BA%8F%E5%88%97#%E9%A2%9C%E8%89%B2")：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88cc6e1961b44c529ccc1888862b7f29~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2036&h=1376&s=507046&e=png&b=e6e8ec)

显然，自己去打印这些颜色也不现实，我们会用一些库来做：

比如 [chalk](https://www.npmjs.com/package/chalk "https://www.npmjs.com/package/chalk")：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1e90fea00ee4bfbb1ff1727372240d7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2256&h=1230&s=482844&e=png&b=fdfcfc)

1.8 亿的周下载量。

chalk 的不同方法就是封装了这些 ASCII 码的颜色控制字符。

试一下：

```css
npm install --save chalk
```

index5.js

```javascript
import chalk from 'chalk';

const log = console.log;

log(chalk.blue('Hello') + ' World' + chalk.red('!'));
log(chalk.blue.bgRed.bold('Hello world!'));
log(chalk.blue('Hello', 'World!', 'Foo', 'bar', 'biz', 'baz'));
log(chalk.red('Hello', chalk.underline.bgBlue('world') + '!'));
log(chalk.green(
	'I am a green line ' +
	chalk.blue.underline.bold('with a blue substring') +
	' that becomes green again!'
));

log(`
    CPU: ${chalk.red('90%')}
    RAM: ${chalk.green('40%')}
    DISK: ${chalk.yellow('70%')}
`);

log(chalk.rgb(123, 45, 67).underline('Underlined reddish color'));
log(chalk.hex('#DEADED').bold('Bold gray!'));
```

跑一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b33466e4fede495ebf100406c5d7636f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=956&h=388&s=53121&e=png&b=181818) 比如我们用 eslint 之类的包会打印这样的错误消息：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d3c6b350af041d680b429a5f0cc55f2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=442&h=196&s=73335&e=png&b=1f1f1f)

被高亮过以后的代码是这样的：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d91383b4f5854e178c16be243a09a851~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1856&h=136&s=173738&e=png&b=262528)

就是通过加上 ANSI 控制字符实现的颜色。

当然，你也可以用别的包，比如 [ansi-colors](https://www.npmjs.com/package/ansi-colors "https://www.npmjs.com/package/ansi-colors")：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8aeae3d62394bf7928ef4dae4978ee8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2102&h=1352&s=420998&e=png&b=fbfbfb)

[colors](https://www.npmjs.com/package/colors "https://www.npmjs.com/package/colors")：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41b332f20bc240e6a2a2617b2f3d81b1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2012&h=1324&s=523825&e=png&b=fcfcfc)

实现原理一样，也用的挺多的。

ansi-escapes 也有替代的包，比如 [sisteransi](https://www.npmjs.com/package/sisteransi "https://www.npmjs.com/package/sisteransi")

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/948d9a3cc1bd44cca5e45b9b263e29fb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2382&h=1088&s=234066&e=png&b=fcfcfc)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fd5226e3b1649fbb2e8a2178f1cb393~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1450&h=1384&s=192909&e=png&b=ffffff)

也用的挺多的，毕竟只是对 ANSI 转义码的封装，没多少东西。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/ansi-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/ansi-test")

## 总结

我们每天都在用各种 cli 工具，它们打印的内容和我们 console.log 的不一样，可以在各处打印、可以打印各种颜色等。

原理就是 ANSI 的转义字符，这个是 1970 年左右就有的标准，历史悠久。

当然，自己写这种转义字符比较麻烦，我们会用 node 的 readline 包、三方的 ansi-escapes 包来做光标控制，用 chalk 包来实现颜色打印。

ANSI 转义字符我们每天都在用，后面写 Node.js 工具也会大量用到。