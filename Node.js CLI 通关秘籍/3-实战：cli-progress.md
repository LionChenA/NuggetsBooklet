上节学了 ANSI 控制字符，这节来写个实战案例： cli-progress 命令行的进度条

![2024-09-01 17.23.14.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30d8ac2286c9427bb81eeae13cff2872~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1612&h=396&s=130382&e=gif&f=38&b=181818)

我们用 console.log 可达不到这样的效果。

明显是通过 ANSI 转义字符修改 cursor 位置实现的。

我们来写一下：

```bash
mkdir cli-progress-test
cd cli-progress-test
npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/492f35ade2fc40b38a74d7657870c96b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=850&h=706&s=90307&e=png&b=000000)

安装下 typescript：

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
  }
}
```

这里设置 module 为 NodeNext，就是会根据情况选择编译为 commonjs 还是 es module。

比如我们在 package.json 里设置 type 为 module，或者文件名用 .mts 结尾，那就会编译成 es module，否则就是 commonjs

在 package.json 设置下 type：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28fae3a74ed44a09aeb7bd8f140ada18~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=582&h=252&s=34282&e=png&b=1f1f1f)

然后来写具体代码：

cli-progress 是一个 npm 包

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b411e992597546ccb8e5f8a36f16bb0f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2034&h=1190&s=266668&e=png&b=fefdfd)

我们先用一下：

```scss
npm install --save cli-progress
```

src/index.ts

```javascript
import { Bar } from 'cli-progress';

const bar = new Bar({
    format: '进度：{bar} | {percentage}% || {value}/{total} || 速度: {speed}',
    barCompleteChar: '\u2588',
    barIncompleteChar: '\u2591',
    hideCursor: true
});

bar.start(200, 0, {
    speed: "0"
});

let value = 0;

const timer = setInterval(function(){
    value++;

    bar.update(value, {
        speed: (60 * Math.random()).toFixed(2) +  "Mb/s"
    });

    if (value >= bar.getTotal()){
        clearInterval(timer);

        bar.stop();
    }
}, 20);
```

调用 start 传入 total、初始 value、初始 speed

在 setInterval 里调用 update 来更新 value，并更新 speed

这里的 speed 是我们设置的一个随机值。

当 value 超过 total，调用 stop 结束进度条。

跑一下：

```bash
npx tsc
node ./dist/index.js
```

![2024-09-01 22.21.25.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49ccbd19230645999b6158c76e6c18e3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1612&h=396&s=147670&e=gif&f=38&b=181818)

用起来还是简单的，我们来实现一下：

首先大概理一下实现原理：

这里需要改光标位置，安装 ansi-escapes

```css
npm install --save ansi-escapes
```

src/test.ts

```javascript
import ansiEscapes from 'ansi-escapes';

process.stdout.write(ansiEscapes.cursorSavePosition)
process.stdout.write('░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░');

setTimeout(() => {
    process.stdout.write(ansiEscapes.cursorRestorePosition)
    process.stdout.write('████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░');
}, 1000)

setTimeout(() => {
    process.stdout.write(ansiEscapes.cursorRestorePosition)
    process.stdout.write('███████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░');
}, 2000)

setTimeout(() => {
    process.stdout.write(ansiEscapes.cursorRestorePosition)
    process.stdout.write('██████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░');
}, 3000)

```

其实就是第一次打印的时候，用 cusorSavePositoin 对应的 ANSI 控制字符保存光标位置。

后面打印的时候就恢复这个光标位置再打印。

对应的 ANSI 控制字符就是这俩：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5cdb0d588a724a6195204ac14b494328~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1314&h=622&s=81881&e=png&b=ffffff)

跑一下：

```bash
npx tsc -w
node ./dist/test.js
```

这里 tsc 加上 -w 就会自动监听文件变动，然后编译。

后面我们就直接执行 dist 下的代码就好了，不用手动编译。

![2024-09-01 22.52.04.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d33fdd5a7992496c87a4e2be7ea39890~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1612&h=396&s=44362&e=gif&f=38&b=181818)

这就是进度条的效果。

但最后还有光标，这个在 cli-progress 是支持隐藏的：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28b60e23a2584e8fb8fdca90bf6b59c7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=892&h=338&s=63673&e=png&b=1f1f1f)

我们也支持下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/465e5c91a4cf4fba9e79c54297c62c60~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1158&h=470&s=99001&e=png&b=1f1f1f)

```javascript
process.stdout.write(ansiEscapes.cursorHide)
```

![2024-09-01 22.54.49.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cfedeb1c483d45adaa78e8d30ffa0f4d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1612&h=396&s=39961&e=gif&f=41&b=171717)

这样就不显示光标了。

大概思路理清了，我们来具体实现下进度条逻辑。

cli-progress 支持 format，也就是会替换字符串中的 {xxx} 为具体的变量：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/108eb780d70e48a88eeb5f880e69ddc0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1226&h=298&s=68466&e=png&b=1f1f1f)

我们简化版就不做了。

src/ProgressBar.ts

```javascript
import ansiEscapes from 'ansi-escapes';
import { EOL } from 'os';

const write = process.stdout.write.bind(process.stdout);

export class ProgressBar {
    total: number = 0;
    value: number = 0;

    constructor() {}

    start(total: number, initVlaue: number) {
        this.total = total;
        this.value = initVlaue;

        write(ansiEscapes.cursorHide)
        write(ansiEscapes.cursorSavePosition)

        this.render()
    }

    render() {
        let progress = this.value / this.total;

        if(progress < 0) {
            progress = 0;
        } else if(progress > 1) {
            progress = 1;
            this.value = this.total;
        }

        const barSize = 40;

        const completeSize = Math.floor(progress * barSize);
        const incompleteSize = barSize - completeSize;

        write(ansiEscapes.cursorRestorePosition)

        write('█'.repeat(completeSize));
        write('░'.repeat(incompleteSize));
        write(` ${this.value} / ${this.total}`)

    }

    update(value: number) {
        this.value = value;

        this.render();
    }

    getTotalSize() {
        return this.total;
    }

    stop() {
        write(ansiEscapes.cursorShow)
        write(EOL)
    }

}
```

首先，在 start 方法里，write 一个 cursorHide 和 cursorSavePosition 的 ANSI 控制字符，隐藏光标、保存光标位置。

然后 render 当前进度条。

在 update 方法里，更新 value，render 当前进度条。

render 的时候就是就算 progress，算出完成的和未完成的字符的个数，然后打印。

打印前要用 cursorRestorePosition 恢复光标位置。

最后在 stop 方法里，要打印 cursorShow，不然代码跑完了还是没有光标。

并且 stop 的时候打印一个换行。

在 src/index2.ts 里用一下：

```javascript
import { clearTimeout } from 'timers';
import { ProgressBar } from './ProgressBar.js';

const bar = new ProgressBar();

bar.start(200, 0);

let value = 0;

const timer = setInterval(function(){
    value++;

    bar.update(value);

    if(value > bar.getTotalSize()) {
        clearTimeout(timer);
        bar.stop();
    }
}, 20);
```

![2024-09-01 23.19.48.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa6f9b94646442e396398c279f4e1267~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1612&h=396&s=77550&e=gif&f=48&b=181818)

这样，简化版 cli-progress 就完成了。

可以再用 chalk 加上颜色：

```css
npm install --save chalk
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ee81be6cd4f49b3ae42d44798f78f76~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=886&h=614&s=102988&e=png&b=1d1d1d)

然后我们用这个来做一个真实下载过程的进度条：

src/index3.ts

```javascript
import https from 'node:https';
import fs from 'node:fs';

const downloadURLs = {
    linux: 'https://storage.googleapis.com/chromium-browser-snapshots/Linux_x64/970501/chrome-linux.zip',
    darwin: 'https://storage.googleapis.com/chromium-browser-snapshots/Mac/970501/chrome-mac.zip',
    win32: 'https://storage.googleapis.com/chromium-browser-snapshots/Win/970501/chrome-win32.zip',
    win64: 'https://storage.googleapis.com/chromium-browser-snapshots/Win_x64/970501/chrome-win32.zip',
};

 https.get(downloadURLs.darwin, response => {

    const file = fs.createWriteStream('./chromium.zip');
    response.pipe(file);

    const totalBytes = parseInt(response.headers['content-length']!, 10);
    
    response.on('data', function (chunk) {
        console.log(totalBytes, chunk.length)
    });
});
```

downloadURLs 是 chrome 的下载 url。

我们用 https 模块的 get 方法请求它，把响应流写入文件。

拿到响应 header 里的 content-length，就是总长度，

而每次触发 data 事件的时候，参数 chunk 的长度就是每次传输的长度。

跑一下：

![2024-09-01 23.41.38.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9f0933d2be040ec90a8c40af15afb09~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1612&h=396&s=86594&e=gif&f=26&b=171717)

然后我们加上我们的 ProgressBar

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6233e8f6ac354f4bb56c2dd1079af5cf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1174&h=1090&s=210462&e=png&b=1f1f1f)

```javascript
import https from 'node:https';
import fs from 'node:fs';
import { ProgressBar } from './ProgressBar.js';

const downloadURLs = {
    linux: 'https://storage.googleapis.com/chromium-browser-snapshots/Linux_x64/970501/chrome-linux.zip',
    darwin: 'https://storage.googleapis.com/chromium-browser-snapshots/Mac/970501/chrome-mac.zip',
    win32: 'https://storage.googleapis.com/chromium-browser-snapshots/Win/970501/chrome-win32.zip',
    win64: 'https://storage.googleapis.com/chromium-browser-snapshots/Win_x64/970501/chrome-win32.zip',
};

const bar = new ProgressBar();

let value = 0;

 https.get(downloadURLs.darwin, response => {

    const file = fs.createWriteStream('./chromium.zip');
    response.pipe(file);

    const totalBytes = parseInt(response.headers['content-length']!, 10);

    bar.start(totalBytes, 0);
    
    response.on('data', function (chunk) {
        value += chunk.length

        bar.update(value);

        if(value > bar.getTotalSize()) {
            bar.stop();
        }
    });
});

```

![2024-09-01 23.47.57.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ff05c564118443293c0c94c05fc3332~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1612&h=396&s=109616&e=gif&f=70&b=171717)

等它下载完：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37d6ebff97864f848bd209f9da6999fd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1618&h=558&s=91336&e=png&b=1b1b1b)

解压这个 zip：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4c945ccae4e4f54879e15f73814b23a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1052&h=544&s=79093&e=png&b=fdfcfc) 打开 chromium 看下:

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9635863d0ef040d39950f53002d3a7a9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2404&h=1542&s=297446&e=png&b=ffffff)

文件下载也没问题。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/cli-progress-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/cli-progress-test")

## 总结

这节我们练习了下上节学的 ANSI 控制字符，实现了一个命令行的进度条。

它的核心就是 save 上次的光标位置，下次更新进度的时候 restore 就好了。

用 ansi-escapes 包的 saveCursorPosition 和 restoreCursorPosition。

此外还要隐藏光标，用 cusorHidden、cursorShow 来控制。

最后，我们用自己写的 ProgressBar 结合 chromium 的下载实际测了一遍，没啥问题。

当然，平时写工具直接用 cli-progress 包就好了，不用自己写。