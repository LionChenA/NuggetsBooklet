学完线程池后，我们来实战下。

大家觉得 npkill 扫描磁盘上的 node\_modules 目录是不是一个耗时任务？

![2024-09-02 20.39.22.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6232933e080e4b3fa413edf09b009a80~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1726&h=934&s=3705369&e=gif&f=70&b=020202)

明显是啊，扫描的目录越多越耗时。

这个就可以用线程池来做。

而且扫描完以后，结果也就是 node\_modules 路径是异步返回的，这种异步多次返回消息我们可以用 rxjs 来实现。

创建个项目：

```arduino
mkdir scan-thread-pool

cd scan-thread-pool

npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f75743bbb5743118154b99fa89fb244~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=870&h=680&s=135113&e=png&b=000000)

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

在 package.json 设置下 type：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6fd62d028038415ea3e8e9819f121eee~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=678&h=264&s=37574&e=png&b=1f1f1f)

然后我们先用一下 rxjs，和它熟悉下：

```css
npm install --save rxjs
```

创建 src/test.ts

```javascript
import { filter, map, Subject } from 'rxjs';

const stream$ = new Subject<number>();

stream$.subscribe((v) => {
    console.log(`订阅者1: ${v}`)
});

stream$.subscribe((v) => {
    console.log(`订阅者2: ${v}`)
});

stream$.next(1);

setTimeout(() => {
    stream$.next(2);
}, 1000);

setTimeout(() => {
    stream$.next(3);
}, 2000);
```

就是一个 subject，两个订阅者，subject 可以异步的调用 next 返回消息，订阅者可以收到这个消息。

跑一下：

```bash
npx tsc -w
node ./dist/index.js
```

![2024-09-02 22.27.02.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4446306a3a10410d8eda43b4231d7358~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1308&h=590&s=52197&e=gif&f=30&b=181818)

**相比 Promise 只能异步的返回一个值，rxjs 的 Subject 可以异步的返回任意多个值。**

而且，过程中还可以对值做一些修改：

```javascript
import { filter, map, Subject } from 'rxjs';

const stream$ = new Subject<number>();

const result$ = stream$
    .pipe(map((x) => x * x))
    .pipe(filter((x) => x % 2 !== 0));

result$.subscribe((v) => {
    console.log(`监听者1: ${v}`)
});

result$.subscribe((v) => {
    console.log(`监听者2: ${v}`)
});

stream$.next(1);

setTimeout(() => {
    stream$.next(2);
}, 1000);

setTimeout(() => {
    stream$.next(3);
}, 2000);
```

这里在 pipe 里对值做修改，用 map、filter 这种操作符可以对值做一些变换之后再返回。

比如这里我们对值做了乘方运算，然后过滤掉偶数的值。

变换之后会返回新的流 result$，让订阅者订阅这个新的流。

跑一下：

![2024-09-02 22.31.15.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25c71de408514fccbf4b562a5c53b728~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1308&h=590&s=53654&e=gif&f=33&b=181818)

可以看到，乘方和过滤都实现了。

rxjs 就这么简单，就是**发布订阅、可以用 next 返回多个值，用 pipe + 操作符对值做修改。**

然后我们来实现 node\_modules 目录的扫描：

创建 src/scan.ts

```javascript
import { cpus } from 'node:os';
import { MessageChannel, MessagePort, Worker } from "node:worker_threads";
import { Subject } from "rxjs";

interface WorkerJob {
    job: 'scan';
    value: { 
        path: string 
    }
}

export type WorkerMessage = {
    type: 'scanResult'
    value: {
        results: Array<{ 
            path: string; 
            isTarget: boolean 
        }>
    }
} | {
    type: 'scan',
    value: {
        path: string;
    }
} | { 
    type: 'startup', 
    value: { 
        channel: MessagePort 
        id: number
    } 
}

export class ScanService {
    private index = 0;
    private workers: Worker[] = [];
    private tunnels: MessagePort[] = [];

    startScan(stream$: Subject<string>, path: string) {
        this.initWorkers();
        this.listenEvents(stream$);
    
        this.addJob({ job: 'scan', value: { path } });
    }

    private listenEvents(stream$: Subject<string>) {
        this.tunnels.forEach((tunnel) => {
          tunnel.on('message', (data: WorkerMessage) => {
            this.newWorkerMessage(data, stream$);
          });
        });
    }

    private newWorkerMessage(
        message: WorkerMessage,
        stream$: Subject<string>,
    ) {
        const { type, value } = message;
    
        if (type === 'scanResult') {
          const results: Array<{ path: string; isTarget: boolean }> = value.results;
    
          results.forEach((result) => {
            const { path, isTarget } = result;
            if (isTarget) {
              stream$.next(path);
            } else {
              this.addJob({
                job: 'scan',
                value: { path },
              });
            }
          });    
        }
    }

    private initWorkers(): void {
        const size = this.getPoolSize();
    
        for (let i = 0; i < size; i++) {
          const { port1, port2 } = new MessageChannel();
          const worker = new Worker('./scan.worker.js');
          
          worker.postMessage(
            { 
                type: 'startup', 
                value: { 
                    channel: port2, 
                    id: i 
                } 
            },
            [port2]
          );

          this.workers.push(worker);
          this.tunnels.push(port1);
        }
    }

    private getPoolSize() {
        return cpus().length;
    }

    private addJob(job: WorkerJob) {
        if (job.job === 'scan') {
          const tunnel = this.tunnels[this.index];
          const message: WorkerMessage = { type: 'scan', value: job.value };
          tunnel.postMessage(message);
          this.index = this.index >= this.workers.length - 1 ? 0 : this.index + 1;
        }
      }
   
}
```

我们从上往下看：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b9cddc3a3b343d0843a2647c0c6df1f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=778&h=224&s=44148&e=png&b=1f1f1f)

这三个属性分别是 wokers 数组、通信用的 port 数组，以及轮询的当前 worker 的 index。

这些和上节一样。

initWorkers 里循环创建 cpu 核心数个 worker，放到 workers 数组里，并创建 channel 放到 tunnels 数组里。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/189762e9dfd94591b45a74845fa04300~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=950&h=850&s=116359&e=png&b=1f1f1f)

这里的 worker 路径是根据当前文件的目录，加上后缀名拼出来的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ed5f7d26303499eb4b579e80769d133~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=982&h=376&s=78964&e=png&b=1f1f1f)

**在 es module 里，不能用 \_\_dirname，可以换成 import.meta.url**

调用 startScan 开始扫描的时候，创建 workers，监听所有 port 的 message 消息，然后给 worker 派发第一个 job：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6af7d403ff804711b39e24cd859b17ec~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=948&h=546&s=107270&e=png&b=1f1f1f)

派发 job 就是用当前 index 的 worker 来跑任务：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16d0a17bff7a41e9883916f88b938459~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1214&h=360&s=82168&e=png&b=1f1f1f)

返回消息的时候，返回的是 scanResult，带着扫描结果：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69b4219703da43179886dc5cdb60e520~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1348&h=1126&s=219961&e=png&b=1f1f1f)

如果 isTarget 是 true，那就找到了目标，用 stream$ 把结果返回给订阅者。

否则，就继续用这个 path 去扫描，添加一个 job。

这样，我们只要调用了 startScan 创建 workers 并放第一个 path 进去，线程池就会自己扫描起来，返回的路径只要不是目标路径就会继续用 workers 去扫描。

然后实现下这个 worker

创建 src/scan.worker.ts

```javascript
import { MessagePort, parentPort } from "node:worker_threads";
import { WorkerMessage } from "./scan.js";
import { opendir } from "node:fs/promises";
import { Dir, Dirent } from "node:fs";
import EventEmitter from "node:events";
import { join } from "node:path";

(() => {
    let id = 0;
    let fileWalker: FileWalker;
    let tunnel: MessagePort;
  
    if (parentPort === null) {
        throw new Error('Worker 只能被 parent thread 启动，不能单独跑');
    }

    parentPort.on('message', (message: WorkerMessage) => {
      if (message.type === 'startup') {
        id = message.value.id;
        tunnel = message.value.channel;

        fileWalker = new FileWalker();

        initTunnelListeners();
        initFileWalkerListeners();
      }
    });
  
  
    function initTunnelListeners(): void {
      tunnel.on('message', (message: WorkerMessage) => {

        if (message?.type === 'scan') {
          fileWalker.enqueueTask(message.value.path);
        }
      });
    }
  
    function initFileWalkerListeners(): void {
      fileWalker.events.on('newResult', ({ results }) => {
        tunnel.postMessage({
          type: 'scanResult',
          value: { results }
        });
      });
    }
})();

interface Task {
    path: string;
}

class FileWalker {
    readonly events = new EventEmitter();
    private readonly taskQueue: Task[] = [];

    enqueueTask(path: string) {
        this.taskQueue.push({ path });
        this.processQueue();
    }

    private processQueue() {
        while (this.taskQueue.length > 0) {
          const path = this.taskQueue.shift()?.path;

          if (path === undefined || path === '') {
            return;
          }

          this.run(path);
        }
    }

    private async run(path: string) {    
        try {
          const dir = await opendir(path);
          await this.analizeDir(path, dir);
        } catch (_) {
        }
    }

    private async analizeDir(path: string, dir: Dir) {
        const results: Array<Record<string, any>> = [];

        let entry: Dirent | null = null;
        while ((entry = await dir.read().catch(() => null)) != null) {
          this.newDirEntry(path, entry, results);
        }
    
        this.events.emit('newResult', { results });
    
        await dir.close();    
    }

    private newDirEntry(path: string, entry: Dirent, results: any[]): void {
        const subpath = join(path, entry.name);
        const shouldSkip = !entry.isDirectory();
        if (shouldSkip) {
            return;
        }

        results.push({
            path: subpath,
            isTarget: entry.name === 'node_modules'
        });
    }

}
```

首先，我们监听父线程 message 消息，收到 startup 的消息的时候就创建 FileWorker 实例：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91681dc8cad34d448c758044f1c5ae52~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1066&h=772&s=119555&e=png&b=1f1f1f)

然后监听 port 发来的 scan 消息，调用 enqueueTask 加入任务队列。

并且监听 FileWorker 返回的 newResult，用 postMessage 传给父线程：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22a989b3f5054d838ad2c4d5853888b3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=990&h=1116&s=171048&e=png&b=1f1f1f)

这样，和父线程的消息通信就完成了，接收 scan 消息，返回 scanResult。

然后是 FileWorker 的实现：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3609eb334be849069905b543b559ff0a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=858&h=876&s=116296&e=png&b=1f1f1f)

就是放入一个 task，然后循环取队列里的 task 来跑。

就是 run 的过程就是 opendir，然后调用 dir.read 会遍历目录，把遍历到的每个目录都通过 newResult 返回：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/514db29516254f9a88d7b62f778a7bb5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1048&h=760&s=123321&e=png&b=1f1f1f)

这里循环调用 dir.read() 直到返回 null，就是遍历目录的意思。

具体判断的过程就是如果不是目录就过滤是，是目录的话判断下是不是 node\_modules：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10a7302dd65a45d39556b7474fb33082~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1098&h=476&s=77019&e=png&b=1f1f1f)

这样，从启动 worker 到 worker 里跑 node\_modules 扫描目录的任务就完成了。

我们用一下：

src/index.ts

```javascript
import { Subject } from "rxjs";
import { ScanService } from "./scan.js";

const service = new ScanService();

const stream$ = new Subject<string>();

service.startScan(stream$, '/Users/guang');

stream$.subscribe((value) => {
    console.log('订阅者收到了扫描结果：', value)
})
```

传入 rxjs 的 stream$，加一个订阅者。

跑一下：

```bash
node ./dist/index.js
```

![2024-09-03 07.46.22.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43995a235e8147378f085a9d785f394f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1882&h=1136&s=7012054&e=gif&f=27&b=161616)

可以看到，扫描到了所有的 node\_modules，速度还是非常快的。

因为我们用了 8 个线程在同时扫描。

我们再算下 node\_modules 的大小，

用 get-folder-size 这个包：

```arduino
npm install --save get-folder-size
```

```javascript
import { Subject } from "rxjs";
import { ScanService } from "./scan.js";
import getFolderSize from "get-folder-size";

const service = new ScanService();

const stream$ = new Subject<string>();

service.startScan(stream$, '/Users/guang');

stream$.subscribe(async (value) => {
    console.log('订阅者收到了扫描结果：', value, await getSize(value))
})

async function getSize(path: string) {
    const res = await getFolderSize(path);

    return (res.size / 1024 / 1024).toFixed(2) + 'M';
}
```

跑一下：

```bash
node ./dist/index.js
```

![2024-09-03 08.06.20.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95ca71189f2849c58e18ae05859ad463~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1824&h=796&s=1902053&e=gif&f=52&b=171717)

但这样肉眼可见的会影响目录扫描结果的打印速度。

其实可以分开跑，然后有了结果打印在对应位置。

就像 npkill 这样：

![2024-09-02 20.39.22.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6232933e080e4b3fa413edf09b009a80~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1726&h=934&s=3705369&e=gif&f=70&b=020202)

目录扫描和大小计算是分开的。

但你对比下可能会发现我们计算的大小和 npkill 的有点差距：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ece23c87316748ca83d4a06e44c647c7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1738&h=580&s=135563&e=png&b=010101)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fa46d7f483d4adf9462efa9de937723~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1618&h=302&s=143784&e=png&b=191919)

确实。

看下 npkill 咋做的：

它在 mac 下用的是系统的 du 命令来算的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93446c08b1df4e6b8a9975d61fef0704~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1254&h=720&s=182812&e=png&b=1f1f1f)

在 windows 下用的 getFolderSize：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d24d13716bf94199a486d36390e8750a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1100&h=440&s=121251&e=png&b=202020)

那肯定是系统命令的结果更准确。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11b0d72aabcc49f285f6e77dfe408681~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=662&h=330&s=42237&e=png&b=010101)

我们也可以根据系统不同来分来用两种方式算。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/scan-thread-pool "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/scan-thread-pool")

## 总结

这节我们用了下上节学的线程池的知识，实现了多个线程同时扫描 node\_modules 目录。

我们先熟悉了下 rxjs，它就是发布订阅，然后可以异步的返回多个值，过程中可以用 operator 来修改值。

用 cpu 数量个 worker 线程来扫描目录，worker 现成里 opendir 后如果找到子目录，返回给父线程，父线程继续调用别的线程来扫描这个目录。

这样就实现了同时用多个线程扫描目录的功能。

调用者传入 rxjs 的 Subject，然后加一个订阅者来接收这种多次异步返回的结果。

最后我们还用 get-folder-size 来计算了目录大小，但在 mac、linux 等系统中，还是系统的 du -sk 命令更准确。

做完这个实战，相信你对 rxjs 和线程池的应用就有更深的理解了。