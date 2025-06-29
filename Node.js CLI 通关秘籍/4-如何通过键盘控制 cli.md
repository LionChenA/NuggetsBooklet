很多 cli 工具都可以通过键盘来控制。

比如 npill：

![2024-09-03 09.11.50.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/886d9fc7dc4143e996c60c4b1d278d58~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1978&h=916&s=648147&e=gif&f=53&b=020202)

通过上下键切换选中的目录，然后空格键删除。

cli 是怎么接收键盘控制的呢？

我们来试一下：

```bash
mkdir keyboard-control
cd keyboard-control
npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebb10469e1cc4c6589bb45a3c965300b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=846&h=672&s=89846&e=png&b=010101)

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

在 package.json 设置 type 为 module：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02aee9f20dc64a55a28ff6cc90d5d9ec~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=578&h=324&s=44492&e=png&b=202020)

然后创建 src/index.ts

```javascript
import readline from 'node:readline';

readline.emitKeypressEvents(process.stdin);
 
process.stdin.setRawMode(true);
 
process.stdin.on('keypress', (str, key) => {
 
    console.log(str, key)

});
```

readline 模块的 emitKeypressEvents 可以让输入流处理键盘事件。

然后 stdin.setRawMode(true) 是禁用掉内置的一些键盘事件处理，比如 ctrl + c 退出进程

监听 keypress 事件，打印下。

跑一下：

```bash
npx tsc -w
node ./dist/index.js
```

![2024-09-03 11.18.55.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7dc9f4e308648d097639983044914f3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1880&h=742&s=112574&e=gif&f=39&b=171717)

我们分别按了 a、b、c 键，以及 ctrl + c 键。

可以看到，现在 ctrl + c 也不退出进程了。

处理下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd36f4300c7443b99cf1f12448c6b56e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=822&h=572&s=85922&e=png&b=1f1f1f)

```javascript
if(key.sequence === '\u0003') {
    process.exit();
}
```

试一下：

![2024-09-03 11.20.45.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/589017f142ec4b0191e8a649a8e6f2ab~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1880&h=742&s=105225&e=gif&f=30&b=171717)

这样就可以了。

其实键盘控制在 cli 工具里用的很多，比如 pm2

pm2 是一个进程管理工具，可以在后台跑很多 node 进程，进程异常退出会自动重启。

我们试一下：

创建 src/test.ts

```javascript
const arr = [];
 
setInterval(() => {
    arr.push(Math.random());
}, 10)
```

src/test2.ts

```javascript
const arr = [];
 
while(true){
    arr.push(Math.random())
}
```

test2.ts 里有一个死循环，很明显，内存和 cpu 占用都会很高。

用 pm2 跑一下这两个 node 脚本

```bash
npx pm2 start ./dist/test.js
npx pm2 start ./dist/test2.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad802392c4304924a3c2fe883897d306~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1750&h=710&s=187324&e=png&b=191919)

然后用 pm2 monit 看下进程的状态：

```
npx pm2 monit
```

![2024-09-03 11.29.38.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13f35a72946443169e44fb20b44d3d20~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1972&h=942&s=958213&e=gif&f=40&b=171717)

通过上下键来切换查看的进程，右边会展示对应进程的日志，下面会展示进程的内存、cpu 等状态。

通过左右键切换选中的框：

![2024-09-03 11.31.02.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/456291554d1140fcbd4e32a4cdb450f6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1972&h=942&s=6117882&e=gif&f=70&b=171717)

键盘操作让 pm2 monit 很好用。

再比如我们每天在用的 create-vite：

![2024-09-03 11.34.47.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b6c8a3225834611aa7b3dd9d14680ea~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1972&h=942&s=151701&e=gif&f=54&b=171717)

也是通过键盘操作来选择的。

这种通过键盘控制的滚动列表用的还挺多的：

![2024-09-03 13.41.17.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bff2c7e7986446c921ae7c1d2cb629f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1292&h=852&s=1186097&e=gif&f=56&b=030303)

![2024-09-03 13.40.33.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07fdafa6e604435ea52090004542d612~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1292&h=852&s=1972378&e=gif&f=36&b=030303)

我们也来写一下：

首先封装个基础类：

src/base-ui.ts

```javascript
import ansiEscapes from 'ansi-escapes';

export interface Position {
  x: number;
  y: number;
}

export abstract class BaseUi {
  private readonly stdout: NodeJS.WriteStream = process.stdout;

  protected print(text: string) {
    process.stdout.write.bind(process.stdout)(text);
  }

  protected setCursorAt({ x, y }: Position) {
    this.print(ansiEscapes.cursorTo(x, y));
  }

  protected printAt(message: string, position: Position) {
    this.setCursorAt(position);
    this.print(message);
  }

  protected clearLine(row: number) {
    this.printAt(ansiEscapes.eraseLine, { x: 0, y: row });
  }

  get terminalSize(): { columns: number; rows: number } {
    return {
      columns: this.stdout.columns,
      rows: this.stdout.rows,
    };
  }

  abstract render(): void;
}
```

这里用到了 ansi-escapes 包，封装了 setCursorAt、printAt、clearLine 方法。

通过 stdout.columns、stdout.rows 可以拿到终端可以显示的的行列号。

安装下 ansi-escapes 包：

```css
npm install --save ansi-escapes
```

然后继承它来实现 list 渲染

src/scroll-list.ts

```javascript
import { BaseUi } from './base-ui.js';
import chalk from 'chalk';

export class ScrollList extends BaseUi{
    curSeletecIndex = 0;
    scrollTop = 0;

    constructor(private list: Array<string>= []) {
        super()

        this.render()
    }

    onKeyInput(name: string) {
        if(name !== 'up' && name !== 'down') {
            return;
        }
    
        const action: Function = this.KEYS[name];
        action();
        this.render();
    }

    private readonly KEYS = {
        up: () => this.cursorUp(),
        down: () => this.cursorDown()
    }

    cursorUp() {
        this.moveCursor(-1);
    }

    cursorDown() {
        this.moveCursor(1);
    }

    private moveCursor(index: number): void {
        this.curSeletecIndex += index;

        if (this.curSeletecIndex < 0 ) {
            this.curSeletecIndex = 0;
        }

        if (this.curSeletecIndex >= this.list.length) {
            this.curSeletecIndex = this.list.length - 1
        }

        this.fitScroll();
    }

    fitScroll() {
        const shouldScrollUp = this.curSeletecIndex < this.scrollTop;
    
        const shouldScrollDown = this.curSeletecIndex > this.scrollTop + this.terminalSize.rows - 2;
    
        if(shouldScrollUp) {
            this.scrollTop -= 1;
        } 

        if(shouldScrollDown) {
            this.scrollTop += 1;
        }

        this.clear();
    }

    clear() {
        for (let row = 0; row < this.terminalSize.rows; row++) {
            this.clearLine(row);
        }
    }

    bgRow(text: string) {
        return chalk.bgBlue(text + ' '.repeat(this.terminalSize.columns - text.length))
    }

    render() {
        const visibleList = this.list.slice(this.scrollTop, this.scrollTop + this.terminalSize.rows)

        visibleList.forEach((item: string, index: number) => {
            const row = index;
            
            this.clearLine(row);

            let content = item;

            if (this.curSeletecIndex === this.scrollTop + index) {
                content = this.bgRow(content);
            }
            
            this.printAt(content, {
                x: 0,
                y: row,
            });
        });
    }
}
```

首先，定义当前选中的 index 和 scrollTop 两个属性：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79df10383c8c433badaee9c9ede10148~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=884&h=482&s=79097&e=png&b=1f1f1f)

监听键盘事件， up 的时候向上移动 curSelectedIndex，down 的时候向下移动：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/acb2756c91dd421dad884db77072b2c6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=872&h=848&s=123999&e=png&b=1f1f1f)

修改 curSelectedIndex，如果向下超出了 scrollTop + 一屏能显示的行数，那就要滚动，scrollTop + 1，反过来，向上的话就是 scrollTop - 1

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b2cf992dc4e4804af325b906d3db34c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1540&h=1110&s=196364&e=png&b=1f1f1f)

之后清空屏幕：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/010afd74d65547b3a1ee10f691530c66~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1108&h=538&s=70158&e=png&b=1f1f1f)

接下来开始 render：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea52641e6431450bb26600a3da7b9392~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=752&h=404&s=55054&e=png&b=1f1f1f)

根据 scrollTop 从 list 中截取当前渲染的部分，依次渲染出来：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e61c7445f5b945d9a15edaabce2d23b0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1478&h=690&s=111632&e=png&b=1f1f1f)

并且选中的行要加一个背景：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10c228a1d81a40a79601c22d8d4f9ca6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1456&h=830&s=140898&e=png&b=1f1f1f)

用 chalk 加一个蓝色背景，文字后面的部分用空格填充

安装用到的包：

```css
npm install --save chalk
```

用一下

src/list-test.ts

```javascript
import ansiEscapes from 'ansi-escapes';
import { ScrollList } from "./scroll-list.js";
import readline from 'node:readline';

readline.emitKeypressEvents(process.stdin);
 
process.stdin.setRawMode(true);

const list = new ScrollList([
    "红楼梦",
    "西游记",
    "水浒传",
    "三国演义",
    "儒林外史",
    "金瓶梅",
    "聊斋志异",
    "白鹿原",
    "平凡的世界",
    "围城",
    "活着",
    "百年孤独",
    "围城",
    "红高粱家族",
    "梦里花落知多少",
    "倾城之恋",
    "悲惨世界",
    "哈利波特",
    "霍乱时期的爱情",
    "白夜行",
    "解忧杂货店",
    "挪威的森林",
    "追风筝的人",
    "小王子",
    "飘",
    "麦田里的守望者",
    "时间简史",
    "人类简史",
    "活着为了讲述",
    "白夜行",
    "百鬼夜行"
]);

process.stdin.on('keypress', (str, key) => {
 
    if(key.sequence === '\u0003') {
        process.stdout.write(ansiEscapes.clearTerminal)
        process.exit();
    }

    list.onKeyInput(key.name);
});
```

new 一个 ScrollList 的实例，传入 list。

监听键盘事件，交给 list 的 onKeyInput 处理。

还有当输入 ctrl + c 时，清空 terminal

跑一下：

```bash
node ./dist/list-test.js
```

![2024-09-03 16.55.36.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc3892457ce7420486b5ec3cee24ba00~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1768&h=578&s=765382&e=gif&f=70&b=171717)

这样，我们就在 cli 实现了列表。

其实原理和网页里的虚拟列表很类似，都是只渲染列表在可视区域的部分元素：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1280fba848be41529db0a6b0e065ce79~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=398&h=676&s=58626&e=png&b=141414)

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/keyboard-control "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/keyboard-control")

## 总结

这节我们实现了通过键盘控制 cli。

比如在 terminal 渲染一个列表，通过上下键切换选中的条目。

比如有两个窗口，通过左右键切换选中的窗口。

在 pm2、create-vite、npkill 等 cli 工具里，大量依赖了键盘控制。

加上键盘控制后，cli 的交互性会更强。