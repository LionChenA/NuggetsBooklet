上节我们测试 cli 的时候，通过 npm link 把这个包安装到了全局。

那 npm link 是怎么实现的呢？

答案就是软链接。

软链接类似 windows 下的快捷方式。

我们试一下就知道了：

```bash
cd ~/

mkdir aaa-lib

cd ./aaa-lib

echo "console.log('hello')" > test.js
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/491e192481d24047a8226b98ec1b72e5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=640&h=348&s=60472&e=png&b=000000)

首先创建个目录和文件。

然后随便找一个别的目录，把 aaa-lib 目录软链过去：

```bash
cd ~/

mkdir bbb

cd bbb

ln -s ../aaa-lib ./aaa-lib
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2965cae7142d4d6da5193897b79f8a40~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=484&h=352&s=59684&e=png&b=000000)

打开这个目录看下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/062bd8a43b8c4b5c95bf31d9bb37a686~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1188&h=288&s=53817&e=png&b=fbf9f9)

可以看到， bbb 目录下多了 aaa-lib 这个目录。

但并不是复制，看到这个箭头了么：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7600a264452441a7a9451cf6516e0582~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1122&h=186&s=35469&e=png&b=fbfafa)

这个箭头就是软链接的标记。

跑一下：

```bash
node ./aaa-lib/test.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bc9a930ac9045979bb66a9b72b3686b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=490&h=192&s=36285&e=png&b=010101)

这和复制有区别么？

当然有，复制是两份内容，而软链是同一份内容的两个引用。

我们改下源文件，再跑下 bbb 目录下的那份：

```bash
echo "console.log('hello2')" > ~/aaa-lib/test.js

cd ~/bbb

node ./aaa-lib/test.js
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89abf3b4a3e24be59293f74af63ac7c1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=788&h=174&s=46503&e=png&b=010101)

可以看到，改了源文件，软链文件的内容也跟着改了。

这就是软链。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb0e21e2d6f04f44ab4ec44d98497d1f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1068&h=880&s=52319&e=png&b=fffefe)

软链这种机制有非常多应用，npm link 就是用软链实现的。

我们之前在项目目录下执行了 npm link，其实就是把这个包注册到了全局仓库。

但用的是软链的方式。

不信我们看下：

首先我们找下全局仓库的位置：

```swift
npm get prefix
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65c10af47ea341d6bae5ff321ed988dd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=392&h=80&s=14943&e=png&b=181818)

当然，你的可能和我的不一样。

然后进入 lib/node\_modules 目录，就是全局包的安装目录。

```bash
ls "$(npm get prefix)/lib/node_modules"
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5529f19b4f1d43bf8bb054ed4b1d421c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=684&h=88&s=27010&e=png&b=010101)

可以看到，这个 my-nest-cli 就是我们是上节全局安装的包。

进入 bin 目录，就是全局命令的目录。

```arduino
open "$(npm get prefix)/bin"
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3989b50c6927498e8fe05a8ea6988a57~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=928&h=506&s=90961&e=png&b=fefefe)

其中，可以看到 my-nest-cli 注册的 my-cli 这个命令。

可以跳回原位置：

![2024-09-08 08.40.20.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d3be87608184b64bfabaa1dedb83161~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1360&h=818&s=2522390&e=gif&f=70&b=fbfafa)

也就是说全局安装包的时候做了两件事：

* 往 npm get prefix 下的 lib/node\_modules 安装了这个包
* 往 npm get prefix 下的 bin 里放了这个包里注册的命令

这样我们会把这个 bin 目录加到环境变量 $PATH 里。

自然就可以直接调用这些全局命令了。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee1a4be1f5cc439a84a581d7c0f5a282~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1126&h=228&s=89300&e=png&b=020202)

其实你本地 npm install 的时候也差不多：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8284e975b0be4774be1d45eacee74f04~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=850&h=940&s=100686&e=png&b=191919)

首先在 node\_modules 下安装这个 npm 包

然后在 node\_modules/.bin 下放这个包注册的命令

和全局安装的流程一样，只是位置不同。

![2024-09-08 08.45.16.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8798e9d853894875bfc2a05800890b8f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1360&h=818&s=1728083&e=gif&f=57&b=fbf9f9)

npm link 的实现也是这两步。

我们先从全局仓库把 my-nest-cli 删除。

```perl
npm uninstall -g my-nest-cli
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6de43ab78dd8467eb009ec77f5c6f079~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=722&h=192&s=42859&e=png&b=010101) 再看下那两个目录：

```bash
ls "$(npm get prefix)/lib/node_modules"
ls "$(npm get prefix)/bin"
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcc0e5511b7d467caf0a7a0927f78ecd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=700&h=88&s=23883&e=png&b=010101)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e18fb77e434480f86cff3c0b1d46d1a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=996&h=900&s=268773&e=png&b=000000)

确实都没 my-nest-cli 了。

然后我们进入上节的项目目录，执行 npm link：

```bash
npm link
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/370dbcb316b24e5faa6ddd2da79ec469~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1368&h=704&s=154055&e=png&b=1b1b1b)

再看下这两个目录：

```arduino
open "$(npm get prefix)/lib/node_modules"
open "$(npm get prefix)/bin"
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fc51a1512324bf79bdc9e03fc9105fd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1838&h=588&s=134872&e=png&b=fdfcfc)

看到这个箭头标记了么，说明这个 my-nest-cli 是 link 过去的。

看下原位置：

![2024-09-08 08.59.12.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5effa96d96754f18b3b9c9013dc3124a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1906&h=974&s=859797&e=gif&f=46&b=fcfbfb)

这就是你执行 npm link 的那个目录位置。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a3b81a8a05045609e08f56e0100d6dc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1336&h=632&s=128322&e=png&b=fdfcfc)

然后 bin 下的这个 my-cli 的命令，也是从项目目录 link 过来的：

![2024-09-08 09.00.40.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efd11a93f6b849f3a58f459745fcbb25~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1526&h=822&s=1931847&e=gif&f=56&b=faf8f8)

所以说，npm link 会做两件事：

* 往 npm get prefix 下的 lib/node\_modules 安装了这个包（用 ln -s 创建的软链）
* 往 npm get prefix 下的 bin 里放了这个包里注册的命令（用 ln -s 创建的软链）

用起来和 npm install -g 的包的命令没区别。

删除自然也是用 npm uninstall -g xxx 来删，删除软链不影响源文件。

所以说，npm link 就是用软链模拟了 npm install -g。

此外，npm link 还有一个作用。

我们创建个项目：

```bash
mkdir tmp-project
cd tmp-project
npm init -y
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a25e0d0fa0634a14b8df22cb8d5a13c4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=816&h=662&s=128019&e=png&b=000000)

进入项目，执行

```perl
npm link my-nest-cli
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/722989f6e66e4f9faae0a72e0657fbea~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=912&h=348&s=121726&e=png&b=010101)

打印和 npm install 一样，提示添加了一个包。

看下 node\_modules：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50f0b9ad54814ebc909ed31519dec20b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1408&h=618&s=125810&e=png&b=fdfcfc)

看到这个软链没，它是从哪 link 过来的呢？

![2024-09-08 09.08.05.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/130d38cbd16b4bed95c1078404bfc8e0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1622&h=960&s=2130370&e=gif&f=57&b=f8f5f5)

从全局仓库 link 过来的。

还有 .bin 下的命令：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42df885d4f004ebca0e4ebf0502f3c55~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=610&h=374&s=39156&e=png&b=191919)

也是从全局仓库 link 过来的：

![2024-09-08 09.23.46.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a4b62740aa1483cb5dcde9a62e3689a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1834&h=1044&s=2068852&e=gif&f=56&b=fcfbfb)

画个图就清楚了：

npm link 的原理是这样的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/446c3ab14dc64cfd809b01190ca7f1b5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1648&h=1024&s=93777&e=png&b=fffefe)

npm link xxx 的原理是这样的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/385ba63f18c845c5b5a520a02fb0b663~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1916&h=1050&s=103268&e=png&b=dff3f4)

## 总结

上节我们通过 npm link 来把项目安装到了全局，并且注册了全局命令。

用起来很方便。

这节我们探究了下实现原理。

原理就是软链接。

npm link 会做两件事：

* 在 npm get prefix 下的 lib/node\_modules 安装了这个包（用 ln -s 创建的软链）
* 在 npm get prefix 下的 bin 里放了这个包里注册的命令（用 ln -s 创建的软链）

而 npm link xxx 则是再把这个包 link 到项目的 node\_modules 下，并且把命令 link 到项目的 node\_modules/.bin 下。

其实和 npm install 一样，就是用软链模拟的 npm install 的过程。

所以删除自然也是用 npm uninstall

软链在 node 生态里有很多应用，之后我们会继续探究。