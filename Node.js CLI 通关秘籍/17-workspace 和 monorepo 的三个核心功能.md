monorepo 是多个包在同一个项目中管理的方式，是很流行的项目组织形式。

主流的开源包基本都是用 monorepo 的形式管理的。

为什么用 monorepo 也很容易理解：

比如 babel 分为了 @babel/core、@babel/cli、@babel/parser、@babel/traverse、@babel/generator 等一系列包。

如果每个包单独一个仓库，那就有十多个 git 仓库，这些 git 仓库每个都要单独来一套编译、lint、发包等工程化的工具和配置，重复十多次。

工程化部分重复还不是最大的问题，最大的问题还是这三个：

1. 一个项目依赖了一个本地还在开发的包，我们会通过 npm link 的方式把这个包 link 到全局，然后再 link 到那个项目的 node\_modules 下。

npm link 的文档是这么写的：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4df67cef70e144b0a1638b2d14fadc03~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1342&h=508&s=160553&e=png&b=ffffff)

就是把代码 link 到全局再 link 到另一个项目，这样只要这个包的代码改了，那个项目就可以直接使用最新的代码。

如果只是一个包的话，npm link 还是方便的。但现在有十几个包了，这样来十多次就很麻烦了。

2. 需要在每个包里执行命令，现在也是要分别进入到不同的目录下来执行十多次。最关键的是有一些包需要根据依赖关系来确定执行命令的先后顺序。

3. 版本更新的时候，要手动更新所有包的版本，如果这个包更新了，那么依赖它的包也要发个新版本才行。

这也是件麻烦的事情。

因为这三个问题：npm link 比较麻烦、执行命令比较麻烦、版本更新比较麻烦，所以就有了对 monorepo 的项目组织形式和工具的需求。

比如主流的 monorepo 工具 lerna，它描述自己解决的三个大问题也是这个：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41c8caf21bc3453e85c12577cefb54c6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1658&h=590&s=194411&e=png&b=f7f9fb)

也就是说，把理清了这三个点，就算是掌握了 monorepo 工具的关键了。

我们分别来看一下：

npm link 的流程实际上是这样的：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6b95f886ca84373a332a3995f876fe0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=930&h=622&s=146557&e=png&b=ffffff)

npm 包先 link 到全局，再 link 到另一个项目的 node\_modules。

而 monorepo 工具都是这样做的：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfabfdd5bde84493a424a58eff8cd022~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=470&h=366&s=19915&e=png&b=ffffff)

比如一个 monorepo 项目下有 a、b、c 三个包，那么 monorepo 工具会把它们 link 到父级目录的 node\_modules。

node 查找模块的时候，一层层往上查找，就都能找到彼此了，就完成了 a、b、c 的相互依赖。

比如用 lerna 的 demo 项目试试：

```bash
git clone https://github.com/lerna/getting-started-example.git
```

下载下来是这样的结构：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71c5dfb7b9724d5696b6e68c6abe06ea~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=544&h=660&s=62063&e=png&b=272728)

执行 npm install，在根目录的 node\_modules 下就会安装很多依赖。

包括我们刚说的 link 到根 node\_modules 里的包：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67201d979f5944cf94d0af52dd7a11cc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=540&h=186&s=18477&e=png&b=272829)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97eb88a545a24dbea3c0974aa077cbe5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=512&h=168&s=16395&e=png&b=252526)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00dbcf388b7344768b8d79b41ab798b5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=536&h=194&s=44031&e=png&b=262627)

这个箭头就是软链接文件的意思。

底层都是系统提供的 ln -s 的命令。

比如我执行

```bash
ln -s package.json package2.json
```

那就是创建一个 package2.json 的软连接文件，内容和 package.json 一样。

这俩其实是一个文件，一个改了另一个也就改了：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86cac1aa14564b7094bbd51183ea9784~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1478&h=678&s=216221&e=gif&f=17&b=262626)

原理都是软连接，只不过 npm link 的那个和 monorepo 这个封装的有点区别。

这种功能本来是 lerna 先实现的，它提供了 lerna bootstrap 来完成这种 link：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdf0cbb137e34af4adaf81107a2fa2de~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1630&h=340&s=125570&e=png&b=f0f9fc)

只不过后来 npm、yarn、pnpm 都内置了这个功能，叫做 workspace。就不再需要 lerna 这个 bootstrap 的命令了。

直接在 package.json 里配置 workspace 的目录：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adf415c1727e40e1ae2b7712a5686e22~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=630&h=508&s=54113&e=png&b=1e1e1e)

然后 npm install，就会完成这些 package 的 link。

而包与包之间的依赖，workspace 会处理，本地开发的时候只需要写 \* 就好，发布这个包的时候才会替换成具体的版本。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c2925bb3f764ae0849f903e4d4daeab~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=618&h=382&s=53091&e=png&b=1e1e1e)

这里用的是 npm workspace：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ce6e2e2beaf4c1e978fce3e4c1c24e9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2028&h=566&s=118702&e=png&b=f8fafb)

它所解决的问题正如我们分析的：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64412819e6e84b108aea74c68fd28ad9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1430&h=202&s=154471&e=png&b=ffffff)

在 npm install 的时候自动 link。

yarn workspace 也是一样的方式：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e67a8586bc4b45579f05543ea2c3777a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1540&h=316&s=91218&e=png&b=fefefe)

pnpm 有所不同，是放在一个 yaml 文件里的：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd1799d32aeb4bf789b9be3b52fe15de~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1648&h=856&s=166591&e=png&b=f6f8fa)

此外，yarn 和 pnpm 支持 workspace 协议，需要把依赖改为这样的形式：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/005c71904c51416c849b110f817c4705~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=544&h=324&s=51390&e=png&b=1e1e1e)

这样查找依赖就是从 workspace 里查找，而不是从 npm 仓库了。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4cb634e16642465c93d3a7515470892d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1714&h=152&s=155179&e=png&b=ffffff)

总之，不管是 npm workspace、yarn workspace 还是 pnpm workspace，都能达到在 npm install 的时候自动 link 的目的。

回过头来再来看 monorepo 工具的第二大功能：执行命令

在刚才的 demo 项目下执行

```arduino
lerna run build
```

输出是这样的：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f854a7042d94637810eda90ae9b7b48~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1358&h=392&s=64571&e=png&b=1e1e1e)

lerna 会按照依赖的拓扑顺序来执行命令，并且合并输出执行结果。

比如 remixapp 依赖了 header 和 footer 包，所以先在 footer 和 header 下执行，再在 remixapp 下执行。

当然，npm workspace、yarn workspace、pnpm workspace 也是提供了多包执行命令的支持的。

npm workspace 执行刚才的命令是这样的：

```sql
npm exec --workspaces -- npm run build
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e7e6c6ab0734e54a53d610216fb1040~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1446&h=708&s=117062&e=png&b=1e1e1e)

也可以单独执行某个包下执行：

```css
npm exec --workspace header --workspace footer -- npm run build
```

只不过不支持拓扑顺序。

yarn workspace 可以执行：

```arduino
yarn workspaces run build
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61703b50f5a04fae885916162b2e4ecd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1072&h=812&s=118282&e=png&b=1e1e1e)

但也同样不支持拓扑顺序。

我们再来试试 pnpm workspace。

npm workspace 和 yarn workspace 只要在 package.json 里声明 workspaces 就可以。

但 pnpm workspace 要声明在 pnpm-workspaces.yaml 里：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fad223de1e5d4e72bc70320f4b8f3c35~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=414&h=170&s=18333&e=png&b=1f1f1f)

pnpm 在 workspace 执行命令是这样的：

```arduino
pnpm exec -r pnpm run build
```

\-r 是递归的意思：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bd2ff7a90b442468fd350126a3eb07a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=666&h=196&s=23190&e=png&b=ffffff)

关键是 pnpm 是支持选择拓扑排序，然后再执行命令的：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49747fd7d6f040b2a7d8c13dd8bd2dea~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1368&h=554&s=65878&e=png&b=ffffff)

有时候命令有执行先后顺序的要求的时候就很有用了。

总之，npm、yarn、pnpm 都和 lerna 一样支持 workspace 下命令的执行，而且 pnpm 和 lerna 都是支持拓扑排序的。

再来看最后一个 monorepo 工具的功能：版本管理和发布。

有个工具叫做 changeset 是专门做这个的，我们看下它能做啥就好了。

执行 changeset init：

```css
npm install --save-dev @changesets/cli prettier-plugin-organize-imports prettier-plugin-packagejson

npx changeset init
```

执行之后会多这样一个目录：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cdb1e259e4014ecf842132cce614ea8f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=310&h=152&s=15620&e=png&b=252526)

然后添加一个 changeset。

什么叫 changeset 呢？

就是一次改动的集合，可能一次改动会涉及到多个 package，多个包的版本更新，这合起来叫做一个 changeset。

我们执行 add 命令添加一个 changeset：

```csharp
npx changeset add
```

会让你选一个项目：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf224dc7aa6b4b479adf737e02b0d2df~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=896&h=222&s=33152&e=png&b=1e1e1e)

哪个是 major 版本更新，哪个是 minor 版本更新，剩下的就是 pacth 版本更新。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/304c9f6b5d8543678972b248cd36af28~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1058&h=208&s=118937&e=png&b=1f1f1f)

1.2.3 这里面 1 就是 major 版本、2 是 minor 版本、3 是 patch 版本。

之后会让你输入这次变更的信息：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b90060f61d5e496db1fb208d0d7485df~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1090&h=112&s=70810&e=png&b=1f1f1f)

然后你就会发现在 .changeset 下多了一个文件记录着这次变更的信息：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0939cd02b08b4f42a6ed2e96f5b4d850~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=950&h=254&s=43149&e=png&b=1f1f1f)

然后你可以执行 version 命令来生成最终的 CHANGELOG.md 还有更新版本信息：

```
npx changeset version
```

之后那些临时的 changeset 文件就消失了：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df0d5ee559a7446a96af4a54639fd6a3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=340&h=136&s=15596&e=png&b=272829)

更改的包下都多了 CHANGELOG.md 文件：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a74f4d8e89046c2a40e82a98026f2dd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1274&h=404&s=120796&e=png&b=312e2a)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51e981fc8215434c89819ddb4497dc21~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1264&h=394&s=117489&e=png&b=322f2b)

并且都更新了版本号：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7ebcdaea5984fc8b8ffa712a589ccfc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1372&h=334&s=124055&e=png&b=232222)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/daec963bcff84613b199d2ccfc376cf3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1344&h=388&s=133466&e=png&b=252424)

而且 remixapp 这个包虽然没有更新，但是因为依赖的包更新了，所以也更新了一个 patch 版本：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bbddb9f8f9ce4000a14bc73248fc179f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1288&h=438&s=133593&e=png&b=272728)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11c7cf2aa8b14ec2b63686b2aadf8f54~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1380&h=444&s=147742&e=png&b=222222)

这就是 changeset 的作用。

如果没有这个工具呢？

你要自己一个个去更新版本号，而且你还得分析依赖关系，知道这个包被哪些包用到了，再去更改那些依赖这个包的包的版本。

就很麻烦。

这就是 monorepo 工具的版本更新功能。

更新完版本自然是要 publish 到 npm 仓库的。

执行 changeset publish 命令就可以，并且还会自动打 tag：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b710f3d8e8ba478e858d44f581ae5e38~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2048&h=272&s=174054&e=png&b=ffffff)

如果你不想用 changeset publish 来发布，想用 pnpm publish，那也可以用 changeset 来打标签：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72b5a22f70454600b6ee74e8d8d53530~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2064&h=316&s=242789&e=png&b=ffffff)

```
npx changeset tag
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6457b12988194697ab7c07c97bf0f680~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=892&h=118&s=70885&e=png&b=1f1f1f)

这就是 monorepo 工具的版本更新和发布的功能。

lerna 是自己实现的一套，但是用 pnpm workspace + changeset 也完全可以做到。

回过头来看下这三个功能：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13392df2ddc748cf9a9fdb492a584fd1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1718&h=534&s=334033&e=png&b=f8fafc)

不同包的自动 link，npm workspace、yarn workspace、pnpm workspace 都可以做到，而 lerna bootstrap 也废弃了，改成基于 workspace。

执行命令这个也是都可以，只不过 lerna 和 pnpm workspace 都支持拓扑顺序执行命令。

版本更新和发布这个用 changeset 也能实现，用 lerna 的也可以。

整体看下来，似乎没啥必要用 lerna 了，用 pnpm workspace + changeset 就完全能覆盖这些需求。

那用 lerna 的意义在哪呢？

虽然功能上没啥差别，但性能还是有差别的。

lerna 还支持命令执行缓存，再就是可以分布式执行任务。

执行 lerna add-caching 来添加缓存的支持:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6cdfc6e092b4078b589952a4c20f373~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1064&h=456&s=84166&e=png&b=1e1e1e)

指定 build 和 test 命令是可以缓存的，输出目录是 dist。

那当再次执行的时候，如果没有变动，lerna 就会直接输出上次的结果，不会重新执行命令。

下面分别是第一次和第二次执行：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4cf40cf9ff14fea862eca9567585a21~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1264&h=668&s=120325&e=png&b=1e1e1e)

至于分布式执行任务这个，是 nx cloud 的功能，貌似是可以在多台机器上跑任务。

所以综合看下来，lerna 在功能上和 pnpm workspace + changeset 没啥打的区别，但是在性能上更好点。

如果项目比较大，用 lerna 还是不错的，否则用 pnpm workspace + changeset 也完全够用了。

## 总结

monorepo 是在一个项目中管理多个包的项目组织形式。

它能解决很多问题：工程化配置重复、link 麻烦、执行命令麻烦、版本更新麻烦等。

lerna 在文档中说它解决了 3 个 monorepo 最大的问题：

* 不同包的自动 link
* 命令的按顺序执行
* 版本更新、自动 tag、发布

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0d55f3588cb4342b83e0d20e316f7c3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1682&h=506&s=155846&e=png&b=f7f9fb)

这三个问题是 monorepo 的核心问题。

第一个问题用 pmpm workspace、npm workspace、yarn workspace 都可以解决。

第二个问题用 pnpm exec 也可以保证按照拓扑顺序执行，或者用 npm exec 或者 yarn exec 也可以。

第三个问题用 @changesets/cli 就可以做到。

lerna 在功能上和 pnpm workspace + changeset 并没有大的差别，主要是它做了命令缓存、分布式执行任务等性能的优化。

总之，monorepo 工具的核心就是解决这三个问题。