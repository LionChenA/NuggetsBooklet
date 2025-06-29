前两节我们学了 blessed 和 blessed-contrib，也就是 cli 里的组件库、图表库。

这节我们用学到的知识来写一个 cli 仪表盘。

也就是 gtop 这个：

```
npx gtop
```

![2024-10-23 18.43.16.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca3bdd55cc2443afb9f0975bbf915620~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1160&h=1322&s=1874604&e=gif&f=46&b=2a0c1f)

其实展示用的组件都是现成的，我们只要拿到数据渲染就行。

创建项目：

```bash
mkdir cli-dashboard

cd cli-dashboard

npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b9b89c5979d4d0ba831baabb71da696~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=870&h=676&s=131628&e=png&b=000000)

先安装下 typescript

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

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c339c5b04ee244f491822fbf8fe6541c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=544&h=270&s=36125&e=png&b=1f1f1f)

安装 blessed

```css
npm install --save blessed blessed-contrib

npm install --save-dev @types/blessed
```

我们先用 Grid 写下布局：

![2024-10-24 10.03.56.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/183878614f0f4fe49ca68c8ff644a54c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2162&h=1220&s=446947&e=gif&f=28&b=000000)

还是行列分为 12 份，分别算下比例：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d3e5da838dd43ebb31814331cad296f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2162&h=1216&s=390605&e=png&b=000000)

这个很好算，位置也是按照比例来。

我们写下布局代码：

src/index.ts

```javascript
import blessed from 'blessed';
import contrib from 'blessed-contrib';

const screen = blessed.screen({
    fullUnicode: true
});

const grid = new contrib.grid({
  rows: 12,
  cols: 12,
  screen: screen,
});

const cpuLineChart = grid.set(0, 0, 4, 12, contrib.line, {
  label: 'CPU 占用',
  showLegend: true,
});

const memLineChart = grid.set(4, 0, 4, 8, contrib.line, {
  label: '内存和交换分区占用',
  showLegend: true
});

const memDonut = grid.set(4, 8, 2, 4, contrib.donut, {
  radius: 8,
  arcWidth: 3,
  label: '内存占用',
});

const swapDonut = grid.set(6, 8, 2, 4, contrib.donut, {
  radius: 8,
  arcWidth: 3,
  label: '交换分区',
});

const netSpark = grid.set(8, 0, 2, 6, contrib.sparkline, {
  label: '网络使用情况',
  tags: true,
  style: {
    fg: 'blue'
  },
});

const diskDonut = grid.set(10, 0, 2, 6, contrib.donut, {
  radius: 8,
  arcWidth: 3,
  label: '磁盘使用',
});

const processTable = grid.set(8, 6, 4, 6, contrib.table, {
  keys: true,
  label: '进程列表',
  columnSpacing: 1,
  columnWidth: [7, 24, 7, 7],
});

processTable.focus();

screen.render();

screen.key('C-c', function() {
    screen.destroy();
});
```

总共是 12 份，按照比例设置位置、大小。

分别渲染那不同的组件。

跑一下：

```bash
npx tsc -w

node ./dist/index.js
```

![2024-10-24 10.45.19.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9de1526298194a0297f6104b749edb09~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2162&h=1220&s=134577&e=gif&f=30&b=161616)

布局没问题。

然后就是填充数据，渲染组件了。

我们从上往下来写。

首先是 cpu 负载情况，这个我们用 [systeminformation](https://www.npmjs.com/package/systeminformation "https://www.npmjs.com/package/systeminformation") 这个 npm 包来拿到：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd4669ac4bd94582a263472e2381fba0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1704&h=946&s=209817&e=png&b=fefdfd)

周下载量 130w，用的挺多的。

它的功能就是拿到很多硬件、系统的信息：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e1e811c354040cf8a45f3eab342dc16~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1278&h=848&s=186856&e=png&b=fefefe)

我们安装下：

```css
npm install systeminformation --save
```

然后在根目录写个 test.js

```javascript
import si from 'systeminformation';

si.currentLoad(data => {
    console.log(data);
});
```

跑一下：

```
node test.js
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/504f3abaf6454642afeecad97dd390ce~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=678&h=1130&s=189001&e=png&b=181818)

这个 cpus 数组里，就是每个 cpu 核心的负载数据。

然后再拿一下其他数据：

```javascript
import si from 'systeminformation';

si.currentLoad(data => {
    // console.log(data);
});

si.fsSize(data => {
    console.log(data);
});
```

fsSize 是拿到 file system，也就是文件系统的信息。

如果在 windows 下，拿到的就是 c 盘、d 盘、e 盘等的信息。

包括每个磁盘分区的大小、可用大小等：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7474bf7aed3442df94530175ed8abd49~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=740&h=862&s=116083&e=png&b=181818)

然后再拿一下内存信息：

```javascript
si.mem(data => {
    console.log(data);
})
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f64850aae8be4d4a99d9b9ba1ff08cca~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=656&h=734&s=95172&e=png&b=1a1a1a)

内存总大小、已用多少等。

接下来是网络信息：

```javascript
si.networkInterfaceDefault(iface => {
    console.log(iface);
    si.networkStats(iface, data => {
        console.log(data);
    });
});
```

我们只需要拿到默认那个网卡的传输速度就行。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/259f96d214c84cf7a182b5e6421d9818~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=546&h=532&s=56445&e=png&b=181818)

这里 rx\_bytes 就是 received bytes，每秒接收的数据量。

tx\_bytes 是 transfer byptes，每秒传输的数据量。

可以用来计算网速：

![2024-10-24 11.36.11.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1319706528d24ef686c773e3a9407a32~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1078&h=262&s=42122&e=gif&f=35&b=030303)

然后是进程列表：

```javascript
si.processes(data => {
    console.log(data);
})
```

会打印进程列表：

![2024-10-24 11.44.58.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91882b3f998d4fe09f1a9cf4a08c3d9c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1868&h=950&s=6015493&e=gif&f=26&b=171717)

只不过太多了，用 console.log 不方便看。

这种情况你会咋办呢？

很明显，要用断点调试。

我们加一个调试配置：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3561863de07f40518168f5634ba7aa2e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=670&h=466&s=59764&e=png&b=191919)

点击调试面板的 create a launch.json file，选择 node 调试配置

![2024-10-24 12.01.48.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/980f7b9a399b4e608b489c50d5b0a096~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2120&h=1020&s=582039&e=gif&f=30&b=1b1b1b)

会生成 .vscode/launch.json

改下调试配置：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "debug test.js",
            "console": "integratedTerminal",
            "program": "${workspaceFolder}/test.js",
        }
    ]
}
```

就是用 node 跑 test.js，然后指定用 vscode 的内置终端跑。

打个断点：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c56534718e2477cac84ef8d0296c8b1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=618&h=352&s=47206&e=png&b=1f1f1f)

点击 debug，代码会在断点处断住：

![2024-10-24 12.04.16.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d5e9c9a780f49c88046b484409cb8df~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2120&h=1020&s=1077946&e=gif&f=70&b=1b1b1b)

这样对象再大也可以一层层查看，比 console.log 好用太多了。

可以拿到每个进程的启动路径，cpu、内存占用等信息。

至此，我们需要的所有数据就全了。

然后把这些数据设置到组件里就行：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bee932366764ea3a1565c970bd81dbe~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2172&h=1476&s=246423&e=png&b=000000)

创建 src/monitor/cpu.ts

```javascript
import si from 'systeminformation';
import contrib from 'blessed-contrib';

const colors = ['magenta', 'cyan', 'blue', 'yellow', 'green', 'red'];

type CpuData = {
    title: string;
    style: {
        line: string;
    }
    x: number[];
    y: number[];
}

class CpuMonitor{

    lineChart: contrib.Widgets.PictureElement;
    cpuData: CpuData[] = [];
    interval: NodeJS.Timeout | null = null;

    constructor(line: contrib.Widgets.PictureElement) {
        this.lineChart = line;
    }

    init() {
        si.currentLoad(data => {
            this.cpuData = data.cpus.map((cpu, i) => {
              return {
                title: 'CPU' + (i + 1),
                style: {
                  line: colors[i % colors.length],
                },
                x: Array(60).fill(0).map((_, i) => 60 - i),
                y: Array(60).fill(0)
              };
            });

            this.updateData(data);

            this.interval = setInterval(() => {
              si.currentLoad(data => {
                this.updateData(data);
              });
            }, 1000);
        });
    }
        
    updateData(data: si.Systeminformation.CurrentLoadData) {
        data.cpus.forEach((cpu, i) => {
            let loadString = cpu.load.toFixed(1).toString();

            while (loadString.length < 6) {
                loadString = ' ' + loadString;
            }

            loadString = loadString + '%';
        
            this.cpuData[i].title = 'CPU' + (i + 1) + loadString;
            this.cpuData[i].y.shift();
            this.cpuData[i].y.push(cpu.load);
        });
        
        this.lineChart.setData(this.cpuData);
        this.lineChart.screen.render();
    }
}

export default CpuMonitor;
```

代码比较多，我们一个方法一个方法看。

首先声明几个属性，在构造器里传入渲染用的 lineChart 组件实例：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88d1d370aa4c4c25b225e74c740d968e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=874&h=370&s=55621&e=png&b=1f1f1f)

init 里刚开始在不同位置填充 60 个 0，渲染一次。

之后每一秒获取 cpu 数据，重新渲染。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b754c22224e94ed9bea045b049da62c7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=998&h=828&s=112089&e=png&b=1f1f1f)

这里 Array(60).fill(0) 是创建数组的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/906de892f03a4bd49f7834147e1c77d6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=788&h=586&s=65447&e=png&b=000000)

渲染的这么多值为 0 的点就是这个：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8b86bc6d1a44059a36ef81cc75badfc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2154&h=488&s=62204&e=png&b=000000)

然后后面每一秒获取最新数据后，渲染新的值：

![2024-10-24 15.54.52.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f03073bbdc9145719160a9529da91349~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2120&h=1020&s=235833&e=gif&f=50&b=010101)

那如何实现这种不断左移的效果呢？

其实也很简单

比如现在在这个位置：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c76a66d8dbd04e5681681feccba10d76~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=864&h=194&s=10942&e=png&b=ffffff)

下一秒就是在这个位置：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74d53aa0a35d41aca732bb64e8c4b2f4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=866&h=408&s=18829&e=png&b=ffffff)

也就是数组里 shift 删除了一个前面的元素，然后 push 添加了一个后面的元素。

每一秒都前面 shift 一个、后面 push 一个，那不就是不断右移的效果么？

也就是这样：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24fe131db2a742beb9088f2233e06441~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1012&h=742&s=125054&e=png&b=1f1f1f)

上面那个 while 循环是在前面补 0，让长度都是 6 位。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa6ba35f940e4de48fc6b8764d556c20~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=276&h=334&s=37136&e=png&b=000000)

在 index.ts 里调用下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a589eb167c0466b992f97be9c159e33~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=692&h=422&s=57674&e=png&b=1f1f1f)

```javascript
new CpuMonitor(cpuLineChart).init();
```

跑下试试效果：

```bash
node dist/index.js
```

在系统自带的终端跑，效果好一点。

![2024-10-24 16.07.31.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19710a50b93943ed8e12a2752a09a902~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2152&h=1118&s=265213&e=gif&f=70&b=000000)

可以看到，我电脑的 8 个 cpu 的负载都会实时展示。

而且也实现了不断右移的效果。

你应该在很多地方都看到过这种不断右移的图表，就是通过 shift + push 元素实现的。

然后再来写下面的内存部分：

创建 src/monitor/memory.ts

```javascript
import si from 'systeminformation';
import contrib from 'blessed-contrib';

const colors = ['magenta', 'cyan', 'blue', 'yellow', 'green', 'red'];

type ChartType = contrib.Widgets.PictureElement;

type MemData = {
    title: string;
    style: {
        line: string;
    }
    x: number[];
    y: number[];
}

class MemoryMonitor{

    lineChart: ChartType;
    memDonut: ChartType;
    swapDonut: ChartType;

    interval: NodeJS.Timeout | null = null;
    memData: MemData[] = []

    constructor(line: ChartType, memDonut: ChartType, swapDonut: ChartType) {
        this.lineChart = line;
        this.memDonut = memDonut;
        this.swapDonut = swapDonut;
    }

    init() {
        si.mem(data => {
            this.memData = [
              {
                title: 'Memory',
                style: {
                  line: colors[0],
                },
                x: Array(60).fill(0).map((_, i) => 60 - i),
                y: Array(60).fill(0),
              },
              {
                title: 'Swap',
                style: {
                  line: colors[1],
                },
                x: Array(60).fill(0).map((_, i) => 60 - i),
                y: Array(60).fill(0),
              },
            ];

            this.updateData(data);

            this.interval = setInterval(() => {
              si.mem(data => {
                this.updateData(data);
              });
            }, 1000);
          });
    }

    updateData(data: si.Systeminformation.MemData) {
        let memPer = +(100 * (1 - data.available / data.total)).toFixed();
        let swapPer = +(100 * (1 - data.swapfree / data.swaptotal)).toFixed();
    
        swapPer = isNaN(swapPer) ? 0 : swapPer;
    
        this.memData[0].y.shift();
        this.memData[0].y.push(memPer);
    
        this.memData[1].y.shift();
        this.memData[1].y.push(swapPer);
    
        this.lineChart.setData(this.memData);

        this.memDonut.setData([
            {
                percent: memPer / 100,
                label: '',
                color: colors[0],
            },
        ]);
        this.swapDonut.setData([
            {
                percent: swapPer / 100,
                label: '',
                color: colors[1],
            },
        ]);
        this.lineChart.screen.render();
    }
}

export default MemoryMonitor;
```

虽然代码多，但是和前面的 CpuMonitor 几乎一摸一样。

首先，要传入三个图表组件的实例：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/839793a895cf46dfad3a84bbb02276f2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1172&h=524&s=85995&e=png&b=1f1f1f)

因为内存的图表有三个：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be594ecd1e4741d2a9bb85bf44a32683~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1010&h=414&s=319543&e=png&b=260e1f)

然后和之前一样，两条折线，初始有 60 个点。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/256ac8893ee34c458b86ae6ba07daaf8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1068&h=1156&s=137992&e=png&b=1f1f1f)

update 的时候通过 shift、push 的方式来不断右移：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb964917933e4c04b3f6148d6193c642~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1092&h=506&s=106457&e=png&b=1f1f1f)

下面两个环形进度条直接设置 data 就好了

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f122af83d0484dbf9c8c3219f7265020~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1134&h=1060&s=170768&e=png&b=1f1f1f)

这里的比例是通过 data.available / data.total 也就是可用内存 / 总内存算出来的。

index.ts 里用一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf28601c3aa94a868bd09a5b0737856a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1028&h=368&s=65487&e=png&b=1f1f1f)

```javascript
new MemoryMonitor(memLineChart, memDonut, swapDonut).init();
```

跑一下：

![2024-10-24 17.55.54.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d9f7f29f8df4ec89193717cea51d9bd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2218&h=1256&s=366674&e=gif&f=70&b=000000)

折线图和环形进度条的数据展示都没问题。

而且不断右移的效果也正常。

继续写下面的网络部分：

创建 src/monitor/net.ts

```javascript
import si from 'systeminformation';
import contrib from 'blessed-contrib';

type ChartType = contrib.Widgets.PictureElement;

class NetMonitor{

    sparkline: ChartType;

    interval: NodeJS.Timeout | null = null;
    netData: number[] = []

    constructor(line: ChartType) {
        this.sparkline = line;
    }

    init() {
        this.netData = Array(60).fill(0)

        si.networkInterfaceDefault(iface => {
            const updater = () => {
                si.networkStats(iface, data => {
                    this.updateData(data[0]);
                });
            };

            updater();
        
            this.interval = setInterval(updater, 1000);
        });
    }

    updateData(data: si.Systeminformation.NetworkStatsData) {
        const rx_sec = Math.max(0, data['rx_sec']);
      
        this.netData.shift();
        this.netData.push(rx_sec);
      
        const rx_label =`Receiving:      ${formatSize(rx_sec)}
Total received: ${formatSize(data['rx_bytes'])}`

        this.sparkline.setData([rx_label], [this.netData]);
        this.sparkline.screen.render();
    }
}

function formatSize(bytes: number) {
    if (bytes == 0) {
        return '0.00 B';
    }

    if(bytes < 1024) {
        return Math.floor(bytes) + ' B'
    }

    let num = bytes / 1024;

    if(num > 1024) {
        return (num / 1024).toFixed(2) + ' MB'
    }

    return  (num / 1024).toFixed(2)+ ' KB'
}

export default NetMonitor;
```

就是每秒获取一次数据，调用 setData 就好了。

要注意的是要处理下 size 的格式化，这里支持了 B、KB、MB 的展示。

向右移动的效果也是用 shift + push 实现的。

调用下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/580736c526bb417ea58b2533e13cdcdf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1218&h=520&s=91879&e=png&b=1f1f1f)

```javascript
new NetMonitor(netSpark).init();
```

跑一下：

![2024-10-25 16.07.16.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9378676718e441f4a924de241415a091~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2708&h=1510&s=240713&e=gif&f=52&b=010101)

没啥问题。

接下来是磁盘部分。

创建 src/monitor/disk.ts

```javascript
import si from 'systeminformation';
import contrib from 'blessed-contrib';

type ChartType = contrib.Widgets.PictureElement;

type FsSizeData = si.Systeminformation.FsSizeData;

class DiskMonitor{

    donut: ChartType;

    interval: NodeJS.Timeout | null = null;

    constructor(donut: ChartType) {
        this.donut = donut;
    }

    init() {
        const updater = () => {
            si.fsSize('', (data) => {
                this.updateData(data);
            });
        }
        updater();

        this.interval = setInterval(updater, 10000);
    }

    updateData(data: FsSizeData[]) {
        const disk = data[0];

        const label =
            formatSize(disk.used) +
            ' of ' +
            formatSize(disk.size);
      
        this.donut.setData([
            {
                percent: disk.use / 100,
                label: label,
                color: 'green',
            }
        ]);
        this.donut.screen.render();
    }
}

function formatSize(bytes: number) {
    return  (bytes / 1024 / 1024 / 1024).toFixed(2)+ ' GB'
}

export default DiskMonitor;
```

也是每秒获取一次数据，setData 到组件。

label 的显示要格式化一下，显示到 GB。

引用下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d587d84a99c649228fc473c780f89315~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1218&h=418&s=79301&e=png&b=1f1f1f)

```javascript
new DiskMonitor(diskDonut).init();

```

跑一下:

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d27071be898e47e79bdc0529eb138f74~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2660&h=1534&s=169399&e=png&b=000000)

可以看到，数据展示出来了。

只不过下面的 label 看不到。

我们调整下圆环大小：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7f66a030a5246398f51553c806eb638~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1080&h=246&s=39404&e=png&b=1f1f1f)

半径改为 4，圆环宽度改为 2

再试下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8918e357b7e049bdb3feca9b99d4b8bf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1120&h=696&s=45681&e=png&b=000000)

这样就显示全了。

前面那个内存的展示也加上 label：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70c8cc60eacb4c9286ab8691764d1262~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1236&h=1160&s=207101&e=png&b=1f1f1f)

```javascript
const memTitle =
    formatSize(data.total - data.available) +
    ' of ' +
    formatSize(data.total);

const swapTitle =
    formatSize(data.swaptotal - data.swapfree) +
    ' of ' +
    formatSize(data.swaptotal);

this.memDonut.setData([
    {
        percent: memPer / 100,
        label: memTitle,
        color: colors[0],
    },
]);
this.swapDonut.setData([
    {
        percent: swapPer / 100,
        label: swapTitle,
        color: colors[1],
    },
]);
```

```javascript
function formatSize(bytes: number) {
    return  (bytes / 1024 / 1024 / 1024).toFixed(2)+ ' GB'
}
```

也改下圆环大小：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/355597de9be646499289a47dc8762251~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1068&h=502&s=75585&e=png&b=1f1f1f)

跑一下： ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60e028e4d2ae4212b046e85dfe326c83~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2686&h=1514&s=179371&e=png&b=000000)

这样 label 就显示了。

最后，来做一下进程列表的展示。

这个稍微复杂一些：

![2024-10-25 20.33.47.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4f1a61eaf514af3a8b701b12cf62210~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1358&h=540&s=623148&e=gif&f=70&b=020202)

按 p、c、m 键的时候，会分别按照 pid、cpu、mem 来排序。

不只是展示 table。

创建 src/monitor/process.ts

```javascript
import si from 'systeminformation';
import contrib from 'blessed-contrib';

const parts: Record<string, any> = {
  p: 'pid',
  c: 'cpu',
  m: 'mem',
};

class ProcessMonitor{

  table: contrib.Widgets.TableElement;

  interval: NodeJS.Timeout | null = null;

  pSort: string = parts.c;
  reIndex: boolean = false;
  reverse: boolean = false;

  constructor(table: contrib.widget.Table) {
      this.table = table;
  }

  init() {
    const updater = () => {
      si.processes(data => {
        this.updateData(data);
      });
    };

    updater();

    this.interval = setInterval(updater, 3000);

    this.table.screen.key(['m', 'c', 'p'], (ch) => {
      if (parts[ch] == this.pSort) {
        this.reverse = !this.reverse;
      } else {
        this.pSort = parts[ch] || this.pSort;
      }
  
      this.reIndex = true;
      updater();
    });
  }

  updateData(data: si.Systeminformation.ProcessesData) {
    const part = this.pSort;

    const list = data.list
      .sort(function(a: any, b: any) {
        return b[part] - a[part];
      })
      .map(p => {
        return [
          p.pid + '',
          p.command,
          ' ' + p.cpu.toFixed(1),
          p.mem.toFixed(1),
        ];
      });

    var headers = ['PID', 'Command', '%CPU', '%MEM'];

    const position = {
      pid: 0,
      cpu: 2,
      mem: 3,
    }[this.pSort]!

    headers[position] += this.reverse ? '▲' : '▼';

    this.table.setData({
      headers: headers,
      data: this.reverse ? list.reverse() : list,
    });

    if (this.reIndex) {
      (this.table as any).rows.select(0);
      this.reIndex = false;
    }

    this.table.screen.render();
  }
}

export default ProcessMonitor;
```

从上到下来看：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2b0fc3909704de79c6518b2c4118706~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=838&h=576&s=84196&e=png&b=1f1f1f)

table 是用于渲染的组件实例，interval 是重绘的定时器，这些大家比较熟悉了。

pSort 是根据哪个字段排序，默认是 cpu

当按下 m、c、p 这三个键的时候，会记录 pSort，也就是根据什么排序：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ce09493a03044e38df39f90c1e163ba~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1036&h=614&s=79617&e=png&b=1f1f1f)

如果已经按过了，就设置 reverse，也就是正序还是倒序。

然后设置 reIndex 为 true，下次渲染根据这个来设置选中 index 为 0

第一次渲染和定时重新渲染和之前一样，只不过多了键盘控制：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02bb57ae3321437089641188774c6624~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1022&h=660&s=92924&e=png&b=1f1f1f)

渲染的时候，首先根据选中的字段来排下序，取出要渲染的字段：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55ac8c1a2340484e90d5ff766761d317~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1076&h=814&s=114380&e=png&b=1f1f1f)

之后设置 data，并且根据是否需要翻转来切换选中的 index 为 0

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c93c78bd83d49e68508362cac63bc5b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1154&h=902&s=129531&e=png&b=1f1f1f)

这里用了 as any 是因为 table 的类型有些问题，运行时是有 rows 属性的，但类型里没有。

在 index.ts 里用一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bb85ec998b24d04834488db5513d025~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1210&h=756&s=132690&e=png&b=1f1f1f)

```javascript
new ProcessMonitor(processTable).init();
```

然后跑下：

![2024-10-25 22.03.27.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e76500967a4447082bb9695eb9938f0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2300&h=774&s=762881&e=gif&f=63&b=010101)

进程列表的渲染，以及根据 pid、cpu、mem 的排序都没问题。

这样，我们的系统监控仪表盘就完成了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/cli-dashboard "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/cli-dashboard")

## 总结

这节我们用前面学的 blessed、blessed-contrib 来实现了 cli 里的系统监控仪表盘。

布局用的 Grid 组件，折线图、环形进度条、Table 等都是 blessed-contrib 的组件。

要注意的就是图表不断右移的效果是在每次重新渲染的时候，在前面 shift 一个元素，在后面 push 一个元素，这样就达到了不断右移的效果。很多你看到的图表都是这么做的。

当你需要在 cli 里展示仪表盘的时候，就可以用 blessed-contrib 来做。