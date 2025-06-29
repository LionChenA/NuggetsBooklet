如果想在一个项目中用另一个项目的代码，你会怎么做呢？

有同学说，可以发一个 npm 包呀，然后在另一个项目里引入。

这样是可以，但是如果经常需要改动它的源码呢？这样频繁发包就很麻烦。

那可以用 monorepo 的形式来组织呀，也就是一个项目下包含多个包，它们之间可以相互依赖。

这样确实可以频繁改动源码，然后另一个包里就直接可用了。

但如果这个包是一个独立的 git 仓库，我希望它虽然在另一个项目里用了，但要保留 git 仓库的独立性呢？

这种就可以用 git submodule 或者 git subtree 了。这俩都实现了一个 git 项目里引入了另一个 git 项目的功能。

那 submodule 和 subtree 都能做这个，它俩有什么区别呢？我该用哪个好呢？

这节我们就来详细对比下 git submodule 还有 git subtree。

首先我们准备这样一个 git 项目：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d65e83590304940b561e4828d8764b0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1356&h=1104&s=136258&e=png&b=1f1f1f)

3 个 commit，每个文件一个 commit。

然后在另一个项目里引入，该怎么做呢？

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1797ce73bab14596ac7b2a176c1503a0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=568&h=274&s=18087&e=png&b=28282a)

我们先用 git submodule 的方式：

执行

```scss
git submodule add git@github.com:QuarkGluonPlasma/git-research-child.git child
```

这个命令就是添加这个 git 项目到 child 目录下：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67f195746a7a4396bff0ebfdb689d513~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2004&h=998&s=171418&e=png&b=1e1e1e)

然后我们再在 child 目录下再添加一个 git submodule：

```bash
cd child
git submodule add git@github.com:QuarkGluonPlasma/git-research-child.git child2
```

现在就是两级 git submodule 了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/264d7a19f2fa47dc86ed7f8fc2f36440~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2014&h=1340&s=301528&e=png&b=1e1e1e)

在 .gitmodules 里记录着它的 url 和保存的 path：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db25e80a6b504f5e8ddd89a1dbc8358e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1852&h=608&s=110315&e=png&b=1f1f1f)

前面说 submodule 能保留独立性，怎么看出来的呢？

首先，它有独立的 .git 目录，代表是单独 git 项目。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebe7095d233144f2b6c5ed906f318b4b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1260&h=770&s=83076&e=png&b=1f1f1f)

虽然这个 .git 目录是放在根 git 项目的 .git 下的：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb6b91f09352499babbb074e3f7cb7e2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1284&h=962&s=111868&e=png&b=1f1f1f)

这样就保证了它们依然可以独立的 pull 和 push。

比如我在 child 里加了一个 444.md 的文件：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/147e5831167d40a2a4d04de8082a492a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1598&h=1344&s=187034&e=png&b=1e1e1e)

你 git status 只能看到它提示了 submodule 有内容变动，但是根本不会管有什么变动。

你需要进入这个目录执行 git add、git commit、git push 才行。

也就是它依然是独立的项目，父项目只是记录了它关联的 commit id 是啥。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb551e0ca0df4296ae3f5b6036a5f231~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1702&h=1452&s=252841&e=png&b=1f1f1f)

可以看到，子项目可以正常 push 成功。

这时候在 child 目录下执行 git status 就可以看到没有变动了：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ede656cb3faf4e9ab9c3f9ef41507786~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1452&h=804&s=102253&e=png&b=1f1f1f)

但这时候你回到父级目录可以看到提示 submodule 有新的 commit：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e426d31aa21c41ba851698dd2332e835~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1768&h=1310&s=205169&e=png&b=1f1f1f)

我们新生成一个 commit 来保存这次变更：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c64150a36b8402fba9a0f3a7e904c82~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1860&h=1218&s=230455&e=png&b=1f1f1f)

这就是 submodule 的独立性，你可以在这个目录下单独执行 pull、push，单独管理变更，父项目只是加一个 commit 记录 submodule 有个新的 commit。

那如果别人 clone 下这个项目来，还有这个 submodule 么？

我们 clone 下试试：

```bash
git clone git@github.com:QuarkGluonPlasma/git-research.git git-research-2
```

我把这个项目 clone 下来：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e64b007e879349edaad11bbdfc403b7e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1818&h=600&s=95279&e=png&b=1f1f1f)

可以看到确实有 child 这个目录，但是没内容。

这是因为它需要单独初始化一下并更新下代码：

执行

```csharp
git submodule init
git submodule update
```

或者执行

```sql
git submodule update --init
```

就可以看到代码被拉下来了：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5f177a5447c40e6929dd41d4908e493~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2300&h=944&s=215858&e=png&b=1c1c1c)

但只有一层，如果想递归的 init 和 update，可以这样：

```css
git submodule update --init --recursive
```

这样它就会把每一层 submodule 都拉下来：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14eef57104a944a7875cc698f2599615~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1702&h=778&s=172968&e=png&b=1e1e1e)

这样就完整下载了整个项目的代码。

当然，这一步可以提前到 git clone，也就是执行：

```bash
git clone --recursive-submodules xxx
```

这样就不用单独 git submodule init 和 git submodule update 了。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/929df5fdc2c7439899897bac1e0ad3c8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=850&h=94&s=19861&e=png&b=fefefe)

小结下 git submodule 的用法：

* **通过 git submodule add 在一个项目目录下添加另一个 git 项目作为 submodule**

* **submodule 下可以单独 pull、push、add、commit 等**

* **父项目只是记录了 gitmodules 的 url 和它最新的 commit，并不管具体内容是什么**

* **submodule 可以多层嵌套**

* **git clone 的时候可以 --recursive-submodules 来递归初始化 submodules，或者单独执行 git submodule init 和 git submodule update**

可以体会到啥叫复用子项目代码的同时保留项目的独立性了么？

然后我们再来试试 git subtree：

还是这样一个项目：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d43c875bcbf405bba98f27b618f7434~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=438&h=272&s=16699&e=png&b=28282a)

我们用 subtree 的命令添加子项目：

```scss
git subtree add --prefix=child git@github.com:QuarkGluonPlasma/git-research-child.git main
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce22188f446642f18759babfbe71face~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2036&h=914&s=198413&e=png&b=1f1f1f)

这样和 submodule 有什么区别呢？

不知道你有没有发现，child 目录下是没有 .git 的，这代码它不是一个单独的 git 项目，只是一个普通目录：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a951f216e54a4ddb86931079cab4ae04~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=534&h=406&s=32326&e=png&b=282829)

所以你在这个目录下的任何改动都可以被检测到：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6e397463b8a4df09e7edc6aff6fb42f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1590&h=884&s=131236&e=png&b=1f1f1f)

可以和整个项目一起 git add、commit、push 等。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf1cb528541d46c0a59a5b10b7f85ffb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1742&h=1066&s=201871&e=png&b=1f1f1f)

不过 subtree 的方式在创建目录的时候会生成一个 commit：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/603980ed2a244fe5845a5fe3085308b8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1286&h=432&s=77196&e=png&b=1e1e1e)

那这样都作为一个普通目录了，这个子项目还独立么？还能单独 pull 和 push 么？

可以的！

虽然没有单独的 .git 目录，但它依然有独立性。你可以通过 subtree 的命令来 pull 和 push 它的代码：

比如我们先试试 pull。

我在 git-search-child 这个项目下加两个 commit：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82fbc4bec3cd44229b7f3f86f037f1bf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1112&h=1400&s=174285&e=png&b=1f1f1f)

加了 555、666 这俩 commit。

然后我在项目下执行 git subtree pull：

```scss
git subtree pull --prefix=child git@github.com:QuarkGluonPlasma/git-research-child.git main
```

这样子项目的最新改动就拉下来了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a452972ceece4ba584ff53cf9f8adbbe~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2060&h=1348&s=262708&e=png&b=1f1f1f)

所以说 subtree 虽然把它作为普通目录来管理了，但它依然保留着独立 pull 和 push 到单独项目的能力！

上面的 url 如果你觉得敲起来麻烦，可以放到 git remote 里来管理：

```scss
git remote add child git@github.com:QuarkGluonPlasma/git-research-child.git
```

这样就可以只写它的名字了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff73308bf3b642d0ba755f599269a9b6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1502&h=230&s=55705&e=png&b=1e1e1e)

这样 pull，会生成 3 个 commit：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c66c1916b8ac4f009e7c48dade078c1c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1180&h=694&s=109641&e=png&b=1e1e1e)

刚拉下来的 555、666 的 commit，还有一个 merge commit。

你也可以加个 --squash 来合并：

```css
git subtree pull --prefix=child child main --squash
```

（这个 child 是我们前面添加的 git remote）

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4caa70bfcb7f4d2b9bb3039bcb463b78~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1484&h=396&s=80188&e=png&b=1e1e1e)

这样就只有一个合并后的 commit，一个 merge commit 了。这就是 --squash 的作用：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed67cc920c6b44b0871ed7cce941018c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1168&h=710&s=110992&e=png&b=1e1e1e)

再来试下独立的 push。

我这里加了个 444.md 的文件，然后 push

```css
git subtree push --prefix=child child main
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f7b23a6b03a462bb34be9e8028cb654~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2044&h=862&s=195453&e=png&b=1f1f1f)

这样就把它 push 上去了。

注意，这里可不是整个项目的 push，而是把那个子项目目录的改动 push 到了子项目里去。

另一个项目里就可以把它拉下来：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/100a7a279a1d4112a38ffb106e3d6e9a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1340&h=1024&s=164949&e=png&b=202020)

那问题来了，不是都没有 .git 目录了么？

那 subtree 是怎么知道哪些 commit 是新的，是属于这个子项目的呢？

还记得 subtree add 的时候单独生成了一个 commit 么？

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5febe5761ba242deb636da23b3160715~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1286&h=394&s=75717&e=png&b=1f1f1f)

git 会遍历 git log，直到找到这个 commit，然后把之间的 commit 里涉及到那个目录的改动摘出来，单独 push 到子项目。

因为有个遍历 commit 的过程，所以这一步可能会比较慢。

当然也有优化的方式，当 commit 多的时候，你可以执行 git subtree split：

```css
git subtree split --prefix=child --rejoin
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e89eebe51b04965ba980df2f9f23c64~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1486&h=188&s=42529&e=png&b=1e1e1e)

这样会单独生成一个这样的 commit：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cead1823d14e427c851dca59eba7c6ba~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1302&h=402&s=79575&e=png&b=1e1e1e)

因为 subtree push 的时候会从上往下找 commit，直到找到这样的 commit 结束。

所以 split 命令就可以指定找到哪个 commit，之前的就不找了，从而优化性能。

最后，git submodule 在 clone 的时候需要单独拉一下子项目代码，那 git subtree 呢？

我们试试：

```bash
git clone git@github.com:QuarkGluonPlasma/git-research.git git-research-3
```

可以看到，拉下来的就是全部的代码：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cb4ca4d49fb44f2b801e4ab0b1f627f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=488&h=548&s=41869&e=png&b=252526)

也就是说它真的就是个普通目录，只不过可以单独的作为子项目 pull 和 push 而已。

这就 git subtree 的使用方式。

小结一下：

* **git subtree add 可以在一个目录下添加另一个子项目**
* **子项目目录和别的目录没有区别，目录下改动会被 git 检测到**
* **可以用 git subtree pull 和 git subtree push 单独提交和拉取子项目代码**
* **git subtree pull 加一个 --squash 可以合并拉下来的 commit**
* **add 的时候会创建一个 commit，这是 push 的时候搜索 commit 的终点，你也可以用 git subtree split --rejoin 来单独生成一个这样的 commit**

## 总结

当你想一个项目加入到另一个项目里来复用，并且还有保持这个项目可以作为独立 git 仓库管理的时候，就可以用 git submodule 或者 git subtree 了。

git submodule 会把子项目作为独立 git 仓库，你可以在这个目录下 pull、push、add、commit，父项目只记录着关联的 commit 是啥，并不关心子项目的具体变动。

git subtree 则是把子项目作为普通目录来管理，和别的文件没啥区别，都可以 add、commit 等。只不过依然保留了这个目录下的改动单独 pull、push 到子项目 git 仓库的能力。

这两种方式都可以复用项目代码的同时，保留子项目独立性。

不过 submodule 的方式耦合比较低，你能感觉出来它就是一个独立的 git 项目，你需要单独操作。

subtree 的方式，你根本感觉不到子项目的存在，它彻底融入了父项目。只是你依然可以单独的对它 pull、push 到子项目仓库。

我个人更喜欢 subtree 的方式，它更无感一点。

管理 git 项目里的 git 项目，你会选择 git submodule 还是 git subtree 呢？