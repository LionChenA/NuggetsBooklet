前面学了 VSCode 里怎么调试 Node.js ，这节我们来过一遍 VSCode Node Debugger 的常用配置。

首先，从 attch 的方式开始：

## attach

有这样一个 Node.js 文件：

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    data: 'Hello World!'
  }));
});

server.listen(8888);
```

我们以调试模式启动：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f638c12a835d448f9cd7001f53d63e1b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1380&h=172&s=103915&e=png&b=1f1f1f)

会有一个 9229 的调试端口。

我们创建 .vscode/launch.json 的调试配置文件：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01a493e66c044a08a1db582d482c837a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=786&h=714&s=222669&e=png&b=262627)

然后添加一个 attach 类型的 Node 调试配置，端口是 9229:

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93c1c646459740b4b094022bd605787b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=768&h=334&s=51043&e=png&b=262627)

点击调试启动就可以连上。

打个断点：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6632cc7f4454ddf9ab21688febf32c8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1484&h=628&s=130290&e=png&b=222222)

浏览器访问，这时候代码就会在断点处断住：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36c9d0930d014ba7bcc9db8973c2fe60~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1320&h=770&s=184227&e=png&b=202020)

### attach by process id

上面的 attach 方式是连接到了 9229 端口，但其实还有另一种 attch 方式，就是通过进程 id。

比如上面跑在 9229 端口的 node 进程的 id 可以通过命令查看：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/198a0ea1b1db4b0aba1060c1f571f7a1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1542&h=150&s=35834&e=png&b=1f1f1f)

发现是 98700，那么我们就可以通过这个进程 id 来 attach 调试服务：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98279a98b7cf498dadeaadf43da024d1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=750&h=328&s=49413&e=png&b=272728)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ef91d7fc8a341e58bccb6528b885ad2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=782&h=434&s=55241&e=png&b=1e1e1e)

默认值是 ${command: PickProcess} 这个会弹出一个选择窗口：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1b54798d0cc43b692c9df931de681d3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1936&h=1220&s=1057737&e=gif&f=129&b=1e1e1e)

选择 98700 的那个进程，attach 即可。

当然，这里我们已经知道了进程 id，那就不需要选择了，直接指定即可：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2a94db07b484744afecbc07644c674e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=672&h=416&s=46632&e=png&b=1e1e1e)

attch 的方式讲完了，接下来来看下 launch 方式的配置：

## launch

launch 不需要我们自己以调试模式启动，只需要指定 node 程序的地址即可：

### program & args

创建个 test 文件，打个断点：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8bc89e368028412b90d88a36a39dfd06~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=676&h=160&s=19082&e=png&b=212121)

添加一个 launch 类型的 Node 调试配置：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/518990593e484b2d8bb0bc3b035055db~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=696&h=220&s=39191&e=png&b=272829)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c566efb23b6343e7bac667d43b1e3e2e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=800&h=412&s=50465&e=png&b=1e1e1e)

代码会在断点处断住，然后可以查看当前的命令行参数：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0ea10a7d28a4038b7e648a6b5de4b2b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1664&h=1022&s=397593&e=gif&f=97&b=1e1e1e)

命令行参数是这俩：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5df00ad8ce534e6aa0fd2caf1a0b19f9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1144&h=244&s=53842&e=png&b=252526)

可以通过 args 来添加命令行参数：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc6db4727ba2443b9c7bfc18a6d1207b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=854&h=598&s=64275&e=png&b=1e1e1e)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3aaa8f206d94b8a9959ea0f6bf4bd26~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1216&h=352&s=69612&e=png&b=222222)

### runtimeExecutable

runtime 默认是 node，其实这个也是可以改的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc4e9e5a19b4423993cb6d86405bdd7d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1240&h=450&s=80508&e=png&b=1f1f1f)

VSCode Debugger 会从 PATH 的环境变量中查找对应名字的 runtime 启动。

比如调试 npm scripts，比如调试 npm run serve，就是这样配置：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a48d8e4173ab4ce99c10271aedede022~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=626&h=570&s=56070&e=png&b=1e1e1e)

所有 PATH 里有的命令都可以在这里配，比如 npx create-vite、npm run xxx 等，都可以配置在 runtimeExecutable 里直接调试。

我们可以跑 node 调试，是因为 PATH 中有 node。

我们可以安装一个 ts-node：

```
npm install -g ts-node
```

然后把 runtimeExecutable 改为 ts-node：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ec215472230448b9789f5073e94d754~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=814&h=448&s=60177&e=png&b=1e1e1e)

这时候就是用 ts-node 来跑了：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d360c5919e74c48a25e4f56e2bfc149~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2120&h=1332&s=304967&e=png&b=212121)

你还可以通过 runtimeArgs 给它传参数。

VSCode 内置了 launch via npm 的配置，如果你把 runtimeExecutable 换成 pnpm，那就是 launch via pnpm 了。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b790537384694161a250d56f00170172~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=656&h=620&s=65749&e=png&b=1e1e1e)

### skipFiles

这个配置的默认值是 <node\_internal>/\*\*

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e71979b6bac44187963b4e87e596e4f4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=852&h=444&s=59246&e=png&b=1e1e1e)

也就是跳过 node 内部的文件。

效果就是这样的：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad9ace48ec0a4ca69a7162187b71e2be~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1844&h=1060&s=439601&e=png&b=242425)

这样就可以精简掉调用栈，只显示我们关心的部分。

把 skipFiles 置空之后，所有代码就都展示出来了：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/921e1b958c4542859296505adbb08d2f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1756&h=1282&s=277179&e=png&b=212121)

再来测试下，我们添加这样两个文件：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33701d177c0d44b5a7ea0096265f1e4b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=688&h=230&s=29840&e=png&b=1f1f1f)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0c45c402f5c4bf2b7757bb068ddf303~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=784&h=234&s=35501&e=png&b=1f1f1f)

在 add 里打了个断点，然后调试方式启动：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/471c8286e4c2423fa2478531d609cfd1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=824&h=444&s=59449&e=png&b=1e1e1e)

调用栈是这样的：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2400bd78913b40089c530b868b5658b5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1734&h=1058&s=165284&e=png&b=232323)

如果你把 index 添加到 skipFiles 里：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bdc27b7271340a7bdb2f69170ada8ec~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=806&h=482&s=68466&e=png&b=1e1e1e)

那调用栈就是这样了：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/579c20a1dc8b46fd812bc37ce8c4e5d5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1776&h=1026&s=155848&e=png&b=222222)

index.js 也被加到 skipFiles 里折叠起来了。

### stopOnEntry

这个是在首行断住，和 node --inspect-brk 的效果一样，在 chrome 调试时也有同样的配置：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce382b17f8ad41c8bd2db4670905e08f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1804&h=922&s=893312&e=gif&f=169&b=1f1f1f)

## console

默认 debug 模式下，打印的日志是在 console 的，而不是 terminal。而 console 里是不支持彩色的：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dac72025ec614fcd9fe754f3a7d8b5d5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1644&h=1124&s=321760&e=png&b=1f1f1f)

这个可以通过 console 配置设置，有三个选项：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/867c29f9cc8143d2a5bcadd0cdeff5be~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=652&h=434&s=73926&e=png&b=1f1f1f)

internalConsole 就是内置的 debug console 面板，默认是这个。

internalTerminal 是内置的 terminal 面板，切换成这个就是彩色了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f45bc10833ad45988019574c7f4ed302~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2042&h=1340&s=272829&e=png&b=1f1f1f)

externalTerminal 会打开系统的 terminal 来展示日志信息：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5037552e44d2478a88235e3d7112cacc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1902&h=920&s=453505&e=png&b=ffffff)

一般情况下，用 internalTerminal 就好。

### autoAttachChildProcesses

node 里是支持多进程的，可以把一些脚本放在子进程来跑来提高性能，充分利用计算机的资源。

比如这样：

```javascript
const cp = require('child_process');

cp.spawnSync('node', ['./index.js'], {
    stdio: 'inherit'
});
```

他就是把 index.js 放到子进程来跑了。

但是默认子进程的输入输出也就是 stdin（标准输入）、stdout（标准输出）不会显示在控制台，这时候可以让它继承父进程的 stdin、stdout 就可以了。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/958ddf2e71f44649b8f3a8703304e5fa~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=828&h=516&s=64981&e=png&b=1f1f1f)

这个输入输出当然也可以定向到别的地方。记得前面我们指定过 console 的输出位置么？

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f7f05c6beb648378e13cc062d859a5c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=594&h=192&s=33936&e=png&b=212121)

这个就是输入输出流 stdin、stdout 的重定向。

当然，这不是我们的重点，我们目的是为了能调试子进程的代码。

index.js 是这样的：

```javascript
const add = require('./add');

console.log(add(1, 2));
```

add.js 是这样：

```javascript
module.exports = function(a, b) {
    return a + b;
}
```

我们在 add.js 里打个断点：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3f2e4e4a2c246e48f161bb04a71d922~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=778&h=228&s=34212&e=png&b=1f1f1f)

它是在子进程里的断点，但是跑调试你会发现能断住：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd08d06193604e1abf9376599a9cdee6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1552&h=798&s=195109&e=gif&f=64&b=222222)

这是因为 autoAttachChildProcesses 的默认值是 true。

调试模式启动的时候，主进程会有调试端口，子进程也会有调试端口，而 autoAttachChildProcess 就是自动连接上子进程的 ws 调试服务的端口。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57deed485cb540419521249d0fa222ca~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=956&h=484&s=71186&e=png&b=262627)

调用栈这里也可以看到是两级的结构，这就是 attch 到了子进程的调试服务。

如果你把 autoAttachChildProcesses 设置为 false，就会发现在子进程打的断点不会生效了。

### cwd

cwd 很容易理解就是 current work directory，当前工作目录。

也就是指定 runtime 在哪个目录运行，默认是项目根目录 workspaceFolder

比如在调试 npm scripts 的时候，指定了 cwd 就是在 cwd 下执行 npm run xxx 了：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56b045c4b4684a0a920fee6b73b5f496~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=702&h=640&s=76713&e=png&b=1f1f1f)

### env

node 程序很多情况下是需要取一些环境变量的，那我们要手动设置了环境变量再跑调试么？

不用，有对应的配置，也就是 env。

比如这样：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f78d548a1e80414bb2f396b769854209~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=850&h=670&s=91422&e=png&b=1e1e1e)

这样在调试的 node 程序里就可以取到这些环境变量：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/410bed6c37474d24add503b0dd978815~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1290&h=598&s=131749&e=png&b=252526)

如果你有一个 envFile 来保存环境变量的话，也可以通过 envFile 的方式：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e60e028e916542eea97c63236f40e6ab~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=828&h=520&s=79665&e=png&b=1e1e1e)

.env 内容如下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3628bda5626f4496b992cfa9bbf2de8b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=528&h=170&s=46450&e=png&b=1f1f1f)

在 node 代码里可以取到对应的环境变量：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24898cdfa7584ff2a43f8067dbf50961~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1166&h=520&s=101760&e=png&b=1f1f1f)

### presentation

当调试配置多了以后，就会比较乱，那能不能做一些配置的分组还有排序呢？

VSCode Debugger 是支持的。

比如这样三个调试配置：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24c1c3f6263949cebd563c01b4841a7d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=842&h=1160&s=155607&e=png&b=1e1e1e)

我们想把调试 chrome 的放一组，调试 node 的放一组，就可以这样写：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0cb7c175a1a74d4dadb11abb437bd085~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=658&h=1166&s=132120&e=png&b=1e1e1e)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3129afe7a5941ad8c52886b32201c62~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=326&h=222&s=30086&e=png&b=bebbbe)

效果就是这样的：

大型项目里调试配置会很多，比如 VSCode，它就有很多调试配置，并用 presentation 做了分组：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6fcbe91d83e41c9a16fa9aea8dbd004~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=972&h=1646&s=579158&e=png&b=b9b5bc)

## 总结

这节我们过了一遍 VSCode Node Debugger 的调试配置：

* processId：除了可以通过网络端口，还可以通过进程 id 的方式 attach 到某个进程的调试服务
* program：调试的 node 代码路径，可以通过 args 传参
* runtimeExecutable：指定跑代码用的 runtime，默认是 node，也可以换成 ts-node、npm、pnpm 等，但要求这些命令在 PATH 环境变量里
* skipFiles：折叠某些路径，不显示在调用栈里，比如 node 内部的一些代码
* stopOnEntry：在首行断住，和 --inspect-brk 一样的效果
* autoAttachChildProcesses：自动 attch 到子进程的调试服务
* console：指定日志输出的位置，是内置的 console、terminal，还是外部的 terminal
* cwd：跑 runtime 的目录
* env：指定环境变量
* presentation：对多个调试配置做分组和排序

理解了这些调试配置，就可以愉快的调试 node 代码了。