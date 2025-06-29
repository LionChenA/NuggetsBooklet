上节学习了 Node.js 里 sourcemap 的两个作用：调试源码、错误堆栈定位源码。

调试源码是 Debugger 自带的功能，它们能解析文件关联的 sourcemap，定位到源码位置：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a869f09515643c8842c7d3dfa8d7b49~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=824&h=428&s=55109&e=png&b=1f1f1f)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7c9b03f7cc249359fc8943abefc4f34~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1222&h=192&s=29534&e=png&b=1f1f1f)

错误堆栈定位源码，在 node 12 后开启 --enable-source-maps 就可以。

在 node 12 之前可以用 source-map-support 来实现。

上节用了下 source-map-support，这节我们自己来实现一遍。

```perl
mkdir my-source-map-support
cd my-source-map-support
npm init -y
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/addb6f995dc94ff4843f3f2ddd067096~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=896&h=698&s=91792&e=png&b=010101)

安装 typescript：

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
    "skipLibCheck": true,
    "sourceMap": true
  }
}
```

在 package.json 设置 type 为 module：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4bdaf7ef656485fafefeef504c027f3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=642&h=300&s=42087&e=png&b=1f1f1f)

然后写下 src/index.ts

```javascript
function add(a: number, b: number) {
    if(a === 1) {
        throw new Error('xxx');
    }
    return a + b;
}

function main() {
    console.log(add(1,2));
}

main();
```

还是当 a 为 1 的时候，抛一个异常。

跑一下：

```bash
npx tsc -w

node ./dist/index.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b969c4cfb9c4f3ba12f54132b113d57~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1240&h=456&s=94419&e=png&b=191919)

打印的信息分为 error 信息和 stack 这两部分：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/568efd9b00e740a389aeae8ae9652a15~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1200&h=448&s=95368&e=png&b=191919)

我们自己来拼接下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebe7e911a7624627a2ee33a48bef541d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1044&h=940&s=159805&e=png&b=1f1f1f)

定义 Error.prepareStackTrace 就可以重写错误格式：

```javascript
Error.prepareStackTrace = (error, stack) => {
    const name = error.name || 'Error';
    const message = error.message || '';
    const errorString = name + ": " + message;
  
    const processedStack = [];
    for (let i = stack.length - 1; i >= 0; i--) {
      processedStack.push('
    atat ' + stack[i]);
    }

    return errorString + processedStack.reverse().join('');
}
```

我们也是分为两部分来拼接，首先是上面的部分，是 error.name + error.message

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/568efd9b00e740a389aeae8ae9652a15~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1200&h=448&s=95368&e=png&b=191919)

下面的部分就是循环打印 stack。

最后要把数组 reverse 一下，因为我们是从前到后遍历的，而栈是从顶向下，也就是从后向前的顺序。

只是我们把 at 改为了 atat

跑一下：

```bash
node ./dist/index.js
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/967815e4528640e1873569f6bf87d461~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1232&h=472&s=99377&e=png&b=191919)

可以看到，打印的错误格式和之前的一样，除了 at 换成了 atat。

那接下来把 dist/index.js 换成 src/index.ts 之类的不也就是个字符串替换么？

是，但是替换也不是那么简单的，需要解析 sourcemap 才知道怎么替换。

接下来我们定义个 wrapCallSite 方法来处理栈帧：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea40b795b1cc4e89bbebf402b52fc80b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1466&h=610&s=143520&e=png&b=202020)

看上图的 ts 类型，栈帧就是 CallSite 类型。

它有很多方法：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ea2af8c338d43d38face0cef1597186~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1166&h=680&s=155339&e=png&b=202020)

比如 getFileName、getLineNumber、getFunctionName 等，可以拿到文件名、方法名、行列号等。

不用自己从字符串解析。

```javascript
Error.prepareStackTrace = (error, stack) => {
    const name = error.name || 'Error';
    const message = error.message || '';
    const errorString = name + ": " + message;
  
    const processedStack = [];
    for (let i = stack.length - 1; i >= 0; i--) {
      processedStack.push('
    atat ' + wrapCallSite(stack[i]));
    }

    return errorString + processedStack.reverse().join('');
}

function wrapCallSite(frame: NodeJS.CallSite) {
    debugger;
}
```

我们加个 debugger，然后断点调试下。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2f97adef1304859b1787cc48dc55f8e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=710&h=452&s=53442&e=png&b=181818)

添加个调试配置：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7fd6eb4ff0cb446dbaf260252c49d1b1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1418&h=672&s=162445&e=png&b=1d1d1d)

```json
{
    "name": "调试 node",
    "program": "${workspaceFolder}/src/index.ts",
    "request": "launch",
    "console": "integratedTerminal",
    "skipFiles": [
        "<node_internals>/**"
    ],
    "type": "node"
},
```

![2024-10-30 08.59.00.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4165818e954742f5946ec20e3b8ce0a0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2442&h=1508&s=731570&e=gif&f=33&b=1b1b1b)

断住之后在 debug console 执行下这些方法：

```javascript
frame.toString()

frame.getFileName()

frame.getLineNumber()

frame.getColumnNumber()

frame.getFunctionName()
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13e7f26914d048568d81f26bb32bf69d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1400&h=486&s=99394&e=png&b=1a1a1a)

根本不用自己解析字符串，CallSite 的实例里都有这些方法。

我们来重写下这些方法：

```javascript
function wrapCallSite(frame: NodeJS.CallSite) {
    const source = frame.getFileName();

    if(source) {
        const newFrame: Record<string, any> = {};
        newFrame.getFunctionName = function() {
            return frame.getFunctionName();
        };
        newFrame.getFileName = function() { return frame.getFileName(); };
        newFrame.getLineNumber = function() { return 666; };
        newFrame.getColumnNumber = function() { return frame.getColumnNumber() };
        newFrame.toString = function() {
            return this.getFunctionName() 
                + ' (' + this.getFileName() 
                + ':' + this.getLineNumber() 
                + ':' + this.getColumnNumber()
                + ')'
        }
        return newFrame;
    }

    return frame;
}
```

跑一下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2010e9111a85425d9c6c79ec68f77aa2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1534&h=550&s=121168&e=png&b=191919)

除了行号换成了我们自己的以外，其余的都没问题。

接下来就是读取下这个文件，解析下 sourcemap，然后拿到真实的 fileName、lineNumber、columnNumber 替换就好了。

我们先实现从文件内容解析 sourceMappingUrl 的方法：

```javascript
function retrieveSourceMapURL(source: string) {
    const fileData = fs.readFileSync(source, { encoding: 'utf-8' });

    const regex = /# sourceMappingURL=(.*)$/g;
    let lastMatch, match;
    while (match = regex.exec(fileData)) {
        lastMatch = match;
    }
    if (!lastMatch) return null;
    return lastMatch[1];
}
```

也就是解析这个：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a869f09515643c8842c7d3dfa8d7b49~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=824&h=428&s=55109&e=png&b=1f1f1f)

用正则匹配，取最后一个匹配的。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/258e43b22b5f43adb8183e0a5dd92fe2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1120&h=676&s=88908&e=png&b=1d1d1d)

```javascript
console.log(retrieveSourceMapURL('./dist/index.js'))
```

调用下，然后跑一下，可以看到，确实解析出了 sourcemap 的 url。

接下来就是读取 sourcemap，解析出传入的 position 对应的源码里的 position。

解析 sourcemap 需要用 [source-map](https://www.npmjs.com/package/source-map "https://www.npmjs.com/package/source-map") 这个包：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1fbe900922145b0b7ba44b54cb5fd58~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1852&h=896&s=165424&e=png&b=fdfdfd)

1.5 亿次的周下载量，这个包用的非常多。

安装下：

```arduino
npm install --save source-map
```

我们先单独用一下这个包：

在根目录创建 test.js

```javascript
import { SourceMapConsumer } from 'source-map';
import fs from 'node:fs';

const mapContent = fs.readFileSync('./dist/index.js.map', 'utf-8');

const map = await new SourceMapConsumer(mapContent);

const position = map.originalPositionFor({
    line: 54,
    column: 0
});

console.log(position);
```

创建 SourceMapConsumer 的实例，调用 originalPositionFor，传入编译后的代码位置，会返回源码里的位置。

跑一下：

```bash
node ./test.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8692c11c2e2e49af9795768f83ea4381~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1106&h=160&s=37169&e=png&b=181818)

可以看到，这行代码对应的源码位置是 index.ts 文件里的第 67 行第 0 列。

我们看一下：

编译后代码的 54 行：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b563715722f342dbb009901460bdb9ac~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1324&h=606&s=143387&e=png&b=1c1c1c)

源码的 67 行：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/812f88aedaf0430e8ee5a17e990a10a9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1286&h=564&s=132958&e=png&b=1c1c1c)

对上了！

sourcemap 的原理就是这样一个个位置的映射，我们用 source-map 包可以解析这种映射。

不过这里是异步的，我么用的是 source-map@0.7，如果想用同步版本，要用 source-map@0.6 版本。

功能是一样的。

关于 source-map 的异步问题感兴趣可以看 [github 的 issue](https://github.com/mozilla/source-map/issues/331 "https://github.com/mozilla/source-map/issues/331")。

改下版本：

```css
npm install --save source-map@0.6
```

去掉 await 再跑下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42bcc4182b544feb99c7fb1c6abe1334~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1416&h=808&s=151382&e=png&b=1e1e1e)

功能正常。

然后我们继续来写后面的逻辑：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f66387a1bf124930bd09f7875faabe9d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1456&h=492&s=109508&e=png&b=191919)

我们要区分下 file:// 开头的路径，这种才是有 sourcemap 的。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3524d0a8b0945daade72a48efe1301e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1510&h=1224&s=250159&e=png&b=1f1f1f)

如果是 file:// 开头的路径，就用 fileURLToPath 去掉前面的 file://

这个是 node:url 的 api：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6f88953480a45ca932c4f4fc5d182f4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=978&h=254&s=54968&e=png&b=1f1f1f)

然后调用 retrieveSourceMapURL 解析出 sourceMapUrl，读取 sourcemap，解析出源码位置返回。

在 wrapCallSite 方法里调用它，拿到源码位置后替换下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d87654bb231846b2a3df4eb4f1ed88b1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1712&h=1168&s=278955&e=png&b=1f1f1f)

不过这里也是只有 file:// 开头的文件才做处理。

```javascript
import fs from 'node:fs';
import { fileURLToPath } from 'node:url';
import path from 'node:path';
import { SourceMapConsumer } from 'source-map';

Error.prepareStackTrace = (error, stack) => {
    const name = error.name || 'Error';
    const message = error.message || '';
    const errorString = name + ": " + message;
  
    const processedStack = [];
    for (let i = stack.length - 1; i >= 0; i--) {
      processedStack.push('
    atat ' + wrapCallSite(stack[i]));
    }

    return errorString + processedStack.reverse().join('');
}

function wrapCallSite(frame: NodeJS.CallSite) {
    const source = frame.getFileName();

    if(source) {
        let position: Record<string, any> | null = {
            source: frame.getFileName(),
            line: frame.getLineNumber(),
            column: frame.getColumnNumber()
        }
        if(source.startsWith('file:/')) {
            position = mapSourcePosition(source, frame.getLineNumber()!, frame.getColumnNumber()!);
        }

        const newFrame: Record<string, any> = {};
        newFrame.getFunctionName = function() {
            return frame.getFunctionName();
        };
        newFrame.getFileName = function() { return position?.source; };
        newFrame.getLineNumber = function() { return position?.line; };
        newFrame.getColumnNumber = function() { return position?.column };
        newFrame.toString = function() {
            return this.getFunctionName() 
                + ' (' + this.getFileName() 
                + ':' + this.getLineNumber() 
                + ':' + this.getColumnNumber()
                + ')'
        }
        return newFrame;
    }

    return frame;
}

function mapSourcePosition(source: string, line: number, column: number) {
    if(source.startsWith('file:/')) {
        source = fileURLToPath(source);
    }
    if(!fs.existsSync(source)) {
        return null;
    }

    const sourceMapUrl = retrieveSourceMapURL(source);

    if (sourceMapUrl) {
        const dir = path.dirname(source);
        const sourceMapPath = path.join(dir, sourceMapUrl);
        
        if (fs.existsSync(sourceMapPath)) {
            const mapContent = fs.readFileSync(sourceMapPath, 'utf-8');
            const map = new SourceMapConsumer(mapContent as any);

            return map.originalPositionFor({
                line,
                column
            });
        }
    }

    return null;
}

function retrieveSourceMapURL(source: string) {
    const fileData = fs.readFileSync(source, { encoding: 'utf-8' });

    const regex = /# sourceMappingURL=(.*)$/g;
    let lastMatch, match;
    while (match = regex.exec(fileData)) {
        lastMatch = match;
    }
    if (!lastMatch) return null;
    return lastMatch[1];
}

function add(a: number, b: number) {
    if(a === 1) {
        throw new Error('xxx2');
    }
    return a + b;
}

function main() {
    console.log(add(1,2));
}

main();

```

跑一下：

```bash
node ./dist/index.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26530a71425946318edb42d49a4a9ba2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1414&h=448&s=86118&e=png&b=191919)

可以看到，确实变了。

不过路径不大对，我们要改为绝对路径

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3618e2bb82494c8794fbb4345e4b12e9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1348&h=1004&s=191445&e=png&b=1f1f1f)

```javascript
const position = map.originalPositionFor({
    line,
    column
});

return {
    source: path.join(dir, position.source),
    line: position.line,
    column: position.column
}
```

再跑下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90c303c90a304abb98c68e72c97769c5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1408&h=458&s=101128&e=png&b=191919)

然后点击文件路径，看下映射的对不对：

![2024-10-30 11.55.12.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66e16a3d11ea45fd8906a15cd062f5e2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2442&h=1508&s=836182&e=gif&f=56&b=191919)

可以看到，非常精准！

不过我们还是把它抽离出一个 register 文件来：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f61c0c11d474c69a515ba6c56962b97~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1298&h=682&s=129836&e=png&b=1c1c1c)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90af8473b291458fb391d4650a1af656~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1638&h=868&s=261918&e=png&b=1d1d1d)

然后这样用：

```bash
node ./dist/index.js

node --import ./dist/register.js ./dist/index.js
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b4ee86d81ba4ac6b12bbd223eb67610~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1466&h=484&s=108223&e=png&b=191919)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29ebf8810a384314baff6dd2438170ea~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1396&h=490&s=109223&e=png&b=191919)

可以看到，用之前和用之后，确实错误堆栈的路径变了。

这里的 --import 和之前的 -r（也就是 --require）一样，都是在代码执行前执行一些代码，不过 --import 是引入 es module 代码用的。

这样，我们的 source-map-support 就完成了。

再和 node 自带的那个对比下：

```bash
node --enable-source-maps ./dist/index.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8248ac2f639b4eca90d3273f80065fe9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1504&h=954&s=223711&e=png&b=191919)

列差了一行，这个可以加一，不过改不改都行。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/my-source-map-support "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/my-source-map-support")

## 总结

node 12 实现了 --enable-source-maps 的功能，开启后错误堆栈会换为源码位置。

node 12 之前是用 source-map-support 来实现。

这节我们实现了一下 source-map-support。

过程就是重写 Error.prepareStackTrace，它可以拿到错误堆栈的信息，每个栈帧是 CallSite 实例，有 getFileName、getLineNumber、getFunctionName 等方法。

我们读取文件内容，用正则解析出 sourceMapUrl，读取 sourcemap，用 source-map 这个包来解析源码位置，之后替换栈帧就可以了。

然后用 node -r 或者 node --import 这种方式来预执行，就可以修改代码打印的错误堆栈。