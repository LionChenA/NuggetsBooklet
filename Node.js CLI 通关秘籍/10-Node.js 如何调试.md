调试的好处上节我们已经感受到了：

我们打印一个大的对象的时候，用 console.log 很难理清对象的结构：

![2024-10-24 11.44.58.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91882b3f998d4fe09f1a9cf4a08c3d9c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1868&h=950&s=6015493&e=gif&f=26&b=171717)

而打个断点，用调试方式启动：

![2024-10-24 12.04.16.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d5e9c9a780f49c88046b484409cb8df~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2120&h=1020&s=1077946&e=gif&f=70&b=1b1b1b)

可以轻松的一层层查看对象。

刚重要的是，断点调试可以单步执行，可以看到调用栈，也就是代码的执行路线：

比如这个 App 的函数组件，你可以看到渲染这个函数组件会经历 workLoop、beginWork、renderWithHooks 这些流程：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72588b94a4c54cb9b2fb6cf9d032d66d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2276&h=1086&s=423350&e=png&b=222222)

你可以点开调用栈的每一帧看下都执行了啥逻辑，用到啥数据。比如可以看到这个函数组件的 fiber 节点：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/316982c49ddf4ef28c45ffa672fdb449~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1982&h=1114&s=2629028&e=gif&f=69&b=212121)

再就是作用域，点击每一个栈帧就可以看到每个函数的作用域中的变量：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2809ab2ad48e489aa49ccf7c92730921~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2094&h=1348&s=2777808&e=gif&f=80&b=1f1f1f)

用 Debugger 可以看到代码的执行路径，每一步的作用域信息。而你用 console.log 呢？

只能看到那个变量值而已。

拿到的信息量的差距不是一点半点，调试时间长了，别人会对代码的运行流程越来越清晰，而你用 console.log 呢？还是老样子，因为你看不到代码执行路径。

所以，不管是调试库的源码还是业务代码，不管是调试 Node.js 还是网页，都推荐用 Debugger 打断点。

这节我们就来学下 Node.js 代码的调试。

创建项目：

```bash
mkdir node-debug
cd node-debug
npm init -y
```

我们准备一段 Node.js 的代码：

创建 index.js

```javascript
const fs = require('fs/promises');

(async function() {
    const fileContent = await fs.readFile('./package.json', {
        encoding: 'utf-8'
    });
    
    await fs.writeFile('./package2.json', fileContent);
})();
```

就是简单的文件读写，先用 node index.js 跑一下：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ad44d4b369c44aeb1b027fa1c0ef187~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2348&h=1276&s=295980&e=gif&f=16&b=252525)

然后以调试模式启动，加个 --inspect-brk 参数：

```bash
node --inspect-brk ./index.js
```

\--inspect 是以调试模式启动，--inspect-brk 是以调试模式启动并且在首行断住。

然后你就会发现它打印了 ws 的地址：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6924c300b2a4465d824be62a09aff58c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1596&h=176&s=160036&e=png&b=1f1f1f)

这就是调试的服务端。

接下来找个对接它的调试协议的客户端连上就行了。

Node.js 可以用 Chrome DevTools 来调试：

## 用 Chrome DevTools 调试 node 代码

浏览器打开 [chrome://inspect/#devices]() ，下面列出的是所有可以调试的目标，也就是 ws 服务端：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/366f98f2790d4e48978b0ac010c0bfbf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1178&h=684&s=102576&e=png&b=fefefe)

你会发现这里列出了我们启动的跑的那个 node 脚本。

这是因为我在网络端口里加上了 node 的 ws 调试服务的端口：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8ca24ccbcae45d8a452570ef70c1ad3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1576&h=1078&s=140182&e=png&b=fdfdfd)

node 调试服务默认是跑在 9229 端口，但是也可以换：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdf99777bc5146429e6ca33964a9d06e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1632&h=152&s=74006&e=png&b=1f1f1f)

只要把它的端口加入到配置里就可以了：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b8e06a498bed4aa9970aca8ee89e3ea1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1652&h=976&s=205004&e=gif&f=22&b=fdfdfd)

点击 inspect 就可以调试这个 node 脚本了：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d59d2a9dfef84987bff9d3cfcf7c3d8f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2210&h=1252&s=343210&e=png&b=fffdfd)

但在 Chrome DevTools 里调试，在 VSCode 里写代码，这俩是分离开的，切来切去也挺不方便的。

那 VSCode 能不能调试 node 代码呢？

明显是可以的：

## 用 VSCode Debugger 调试 node 代码

我们创建 .vscode/launch.json 的调试配置文件：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01a493e66c044a08a1db582d482c837a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=786&h=714&s=222669&e=png&b=262627)

添加一个 node 类型的 attach 的调试配置：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ba365c024b64cb29ffd7bc03a107338~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1110&h=644&s=109845&e=png&b=1e1e1e)

端口改成 ws 服务启动的端口 8888:

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6da10b859a242c8b7016ff23823713e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1294&h=990&s=171380&e=png&b=202020)

然后点击 debug 启动：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85322b982c194281ae6316cde09c62c2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2472&h=1384&s=575313&e=gif&f=23&b=262626)

VSCode Debugger 就会连接上 8888 的调试端口开始调试，可以边调试边修改代码。

能调试还能修改代码，这是比 Chrome DevTools 更好用的地方，因此我们一般使用 VSCode Debugger 来调试 Node.js 代码的。

有的同学可能会说，每次都要先 node --inspect xxx.js 把调试服务跑起来，然后再手动 attach，比较麻烦。能不能自动做呢？

当然是可以的：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f7ab6af77824b7ba5bf0c279b50fc6a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=894&h=560&s=93131&e=png&b=1f1f1f)

创建调试配置的时候选择 launch program，指定 program 的路径：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13660bbc88344b7e9cdb630c8113b184~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1026&h=488&s=63531&e=png&b=1e1e1e)

VSCode Debugger 就会自动以调试模式跑这个 node 脚本，并且自动 attach 上：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e36ca648c5b441d8c8d2694fd4936af~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2334&h=1212&s=350489&e=gif&f=8&b=222524)

这样就方便多了。

如果你也想首行断住，可以加一个 stopOnEntry 的配置：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8a30e1d3d9e450db0de50b45ba26291~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=828&h=512&s=77334&e=png&b=1e1e1e)

和 node --inspect-brk 启动一样的效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0af21fd24edc4fdfb85463d461fa93bb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2220&h=1200&s=353805&e=gif&f=9&b=252525)

回过头来我们看一下，为什么 VSCode Debugger 和 Chrome DevTools 都可以调试 node 代码呢？

这是因为都对接了 node 的调试协议，只是实现了各自的 UI：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee1c4a38e33c4f70aeabb27dbe8c550b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2080&h=638&s=90302&e=png&b=ffffff)

有的同学可能会问，为啥图上画的是 Chrome Devtools Protocol 呢？node 也是用 CDP 作为调试协议么？

确实，node 也是基于 CDP 调试的。

至于为什么要用 CDP 来调试 node 代码，这就涉及到一段历史了：

## node debugger 的历史

我们知道 Node.js 是基于 V8 的，V8 本身有调试协议 V8 Debug Protocol，所以 Node.js 最早的调试协议也就是 V8 Debug Protocol。

当时调试是这样的：

通过 node debug 来跑 js 文件，会在首行断住：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e37669f98647414cb49101c5d0758a83~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1494&h=398&s=114794&e=png&b=1f1f1f)

然后可以通过 run、cont、next、step 等命令来实现单步调试，通过 backtrace 打印调用栈，通过 setBreakPoint 等设置断点：

比如用 setBreakPoint（sb）命令在第四行打个断点：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cccf6ccb6f3c42d1ba6e21729461bcbb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=546&h=256&s=82922&e=png&b=1f1f1f)

然后 cont(c) 命令继续执行，backtrace(bt) 打印调用栈：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7092059ba5546a18eefa62ba3593ef1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=480&h=342&s=100017&e=png&b=1f1f1f)

虽然该有的调试功能都有，但是这样调试还是比较费劲的。

怎么能不用命令行调试，而是用 UI 来调试呢？

当时 Node 就瞄准了 Chrome DevTools，它的调试 UI 就很不错。

但是 Chrome DevTools 的调试协议是 Chrome DevTools Protocol，和 V8 Debug Protocol 还是有些差距的，怎么能用上 Chrome DevTools 的调试工具来调试 Node 呢？

其实还挺容易想到的，就是加一个中间的服务来做转换：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09db8c36ef444c4fab5a2fec9a933b4d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2296&h=336&s=71268&e=png&b=fefefe)

这个服务是 node-inspector 这个包提供的。

所以当时 node debug 服务跑起来之后，还要要再跑一个 node-inspector 服务，这样才能用 chrome devtools 来调试 Node.js 代码。

后来维护 Node.js 的那些人觉得这样也太麻烦了，要不让 Node.js 提供的调试协议就直接就是兼容 Chrome Devtools Protocol 的吧。

当时就有了这样一个 pr，把 v8 inspector 集成到 Node.js 中：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d52712367cf435b886cbf8cd6756ef7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1846&h=1114&s=820363&e=png&b=ffffff)

这个 v8 inspector 就是从 chrome 的内核 blink 里剥离出来的让 v8 支持 chrome devtools protocol 的部分。

很明显这需要 v8 团队的配合，所以说 Node.js 的发展还是很依赖 v8 团队的支持的。

之后 Node.js 就在 v6.3 中加入了这个功能：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0d03538aed94ba9805eabae1d8c48e6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2010&h=294&s=239816&e=png&b=fefefe)

并且在成熟之后去掉了对 v8 debug protocol 的支持，也就是废弃了 node debug 命令，改为了 node inspect。

启动 ws 调试服务的方式就是 node --inspect 或者 node --inspect-brk。

当然，之前作为两个协议的中转的服务 node-inspector 也就退出了历史舞台。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7a0001990da4f6aa238c7f795415aa8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1522&h=870&s=502395&e=png&b=ffffff)

所以今天，我们可以轻易的用 Chrome DevTools 来调试 Node.js 代码。

当然，这里只是说 Chrome DevTools 调试 Node.js，在 VSCode 里调试 Node.js 的话还要做一些额外的处理：

调试的原理我们已经知道了，就是 ws 客户端和服务器的通信，然后基于调试协议来完成不同的功能。Node.js 是这样，其他语言也是这样。

VSCode 是一个通用的编辑器，是要支持多种语言的，也就是它的调试 UI 要支持多种调试协议。

要同一个调试工具同时支持不同的协议有点不太现实，那怎么办呢？

可以加一个中间层，VSCode 的调试 UI 只要支持这个中间的调试协议就可以了，其余的调试协议适配到这个调试协议上来：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1385a2ba0734218943690a0495a8513~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=690&h=558&s=98189&e=png&b=fefefe)

这就是 DAP 协议，debugger adpater protocol。

Node.js 在把调试工具的协议换成 Chrome Devtools Protocol 之后，只要实现个 DAP 的 adapter 就可以对接到 VSCode 的调试工具了。

这样我们就可以在 VSCode 里调试 Node.js 了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d97b87356f6e412b814dcd100ac43731~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2080&h=638&s=84441&e=png&b=ffffff)

## 总结

我们分别用 Chrome DevTools 和 VSCode Debugger 调试了下 node 代码：

用 node --inspect 或 node --inspect-brk 以调试模式跑 node 脚本，然后用调试的 UI 连接上这个调试服务就可以进行调试了。

在 VSCode Debugger 里调试 node 代码还可以边调试边修改，更方便。

除了自己启动 node 调试服务，然后手动 attach 外，还可以选择 launch 类型的调试配置，自动进行这个过程。

node 调试的协议也是用的 Chrome DevTools Protocol，就是为了能够复用 Chrome DevTools 的 UI 的，中间还有一段历史是用 node inspector 做中转。

学会了调试 node 代码，各种命令行工具、node 服务，我们就都可以调试了。