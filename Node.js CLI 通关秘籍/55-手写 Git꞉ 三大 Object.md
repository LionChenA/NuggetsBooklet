git 我们每天都在用，但你知道它是怎么实现的么？

git add、git commit 整天都敲，但你知道它底层做了什么么？

commit、branch、暂存区这些都是怎么实现的，怎么做到的版本切换呢？

所有这些疑问，只要搞懂 3 个 object 就全部能解答了。

不信我们来看一下：

首先，执行 git init 初始化 git 仓库。

git 的所有内容都是存储在 .git 这个隐藏目录的，我们先把它给搞出来：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc6dac46a89d44aa80e9bcfc19de59ae~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1638&h=1190&s=196368&e=png&b=262627)

默认隐藏，但只要你把这个 exclude 配置删掉，就显示出来了：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89a1c9fd29134171aa95b1076c70ee47~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1792&h=1122&s=208026&e=png&b=212121)

展开以后可以看到这些东西：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5cad83fefa684a199537d40245681e1d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=420&h=646&s=38976&e=png&b=252526)

重点就是这里的 objects。

它是什么呢？

我们添加一个 object 就知道了：

有这样一个 text.txt 的文件：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06f2cf6b39904a2991dc65fb276ed872~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=948&h=310&s=29006&e=png&b=252526)

执行这个 hash-object 的命令：

```python
git hash-object -w text.txt
```

它会返回一个 hash：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adad8380024f43c5a919daa715348122~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=898&h=130&s=61748&e=png&b=1f1f1f)

然后你会在 objects 目录下发现多了一个目录，目录名是 hash 前两位，剩下的是文件名：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6e9fcccc1da48fdb56e82e708f6ce15~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1718&h=888&s=118292&e=png&b=1e1e1e)

它存了什么内容呢？

可以通过 cat-file 来看：

```css
git cat-file -p 7c4a013e52c76442ab80ee5572399a30373600a2
```

\-p 是 print 的意思。

可以看到文件内容就是 text.txt 的内容：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aeb13e95def5450899bd6dfb22064833~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1982&h=884&s=118989&e=png&b=1e1e1e)

哦，原来 git 存储的文件内容就是放在这里的。

改一下文件内容，再存一下：

```python
git hash-object -w text.txt
```

你会看到多了一个新的目录，同样是 hash 做目录名和文件名：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb7efbb3d8da4e9584896c80687f9128~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1760&h=1128&s=184755&e=png&b=1e1e1e)

就这么一点东西，我们就能实现版本管理了！

怎么做呢？

读取不同 hash 的内容写入文件不就行了？

比如现在内容是 bbb，我想恢复上一个版本的内容是不是只要 cat-file 上个 hash 再写入文件就行了？

```arduino
git cat-file -p 7c4a013e52c76442ab80ee5572399a303 > text.txt
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9cdd71b15fa4b38809818392432f1be~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2260&h=1018&s=1174969&e=gif&f=34&b=1e1e1e)

这就是一个版本管理工具了！

当然，现在还没有存文件名的信息，还有目录信息，这些信息存在哪呢？

这就需要别的类型的 object 了。

刚才我们看的存储文件内容的 object 叫做 blob。

可以通过 cat-file 加个 -t 看出来：

\-t 是 type 的意思。

```matlab
git cat-file -t 7c4a013e52c76442ab80ee5572399a303
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e5f1e21d1024868ae4a28f5958399fa~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1152&h=102&s=20143&e=png&b=1e1e1e)

还有存储目录和文件名的 object，叫做 tree。

tree 和 blob 是咋关联的呢？

找个真实的仓库看看就知道了：

比如我在 react 项目下执行了 cat-file，之前我们用它查看过 blob 对象内容，这次查看的是 main 分支的顶部的 tree 对象。

```css
git cat-file -p main^{tree}
```

可以看到有很多 blob 对象和 tree 对象：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49132ed5177543cebb8da8ab2cd798eb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1554&h=888&s=280835&e=png&b=1f1f1f)

很容易看出来，目录是 tree 对象，文件内容是 blob 对象：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/391ca34d11bd4396a8e36dabf71e1980~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2148&h=1362&s=552484&e=png&b=1f1f1f)

那文件名呢？

文件名不是已经在 tree 对象里包含了么？

我们继续用 cat-file 看下 packages 这个 tree 对象的内容：

```css
git cat-file -p 2889ab8f0ef04484849c40d3eebe330ec25bbe1c
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a11d5c82b0f34a0983fd05a684ba0b59~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2354&h=992&s=417414&e=png&b=1f1f1f)

很容易就可以看出来 git 是怎么存储一个目录的了：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9517cef18734296996560917a569b43~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1860&h=1208&s=563598&e=png&b=fcfbfa)

在 tree 对象里存储每个子目录和文件的名字和 hash。

在 blob 对象里存储文件内容。

tree 对象里通过 hash 指向了对应的 blob 对象。

这样是不是就串起来了！

这就是 git 存储文件的方式。

那这个 hash 是怎么算出来的呢？

也很简单，是对“对象类型 内容长度\\0内容” 的字符串 sha1 之后的值转为 16 进制字符串。

比如 aaa 的 hash 就是这样算的：

```javascript
const crypto = require('crypto');

function hash(content) {
    const sha1 = crypto.createHash('sha1');
    sha1.update(content);
    return sha1.digest('hex');
}

console.log(hash('blob 3\0aaa'))
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b9fc4a48bb345ebb5ffbe1081d15029~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1946&h=984&s=216653&e=png&b=1f1f1f)

是不是一毛一样！

所有的 object 都是这么算 hash 的。

继续来讲 tree 对象：

其实我们放到暂存区的内容就相当于一个新的目录，也是通过 tree 对象存储的。

更新暂存区用 update-index 这个命令：

```scss
git update-index --add --cacheinfo 100644 7c4a013e52c76442ab80ee5572399a30373600a2 text.txt
```

\--add --cacheinfo 就是往暂存区添加内容。

指定文件名和 hash，这里我们把 aaa 那个文件放进去了。

前面的 100644 是文件模式：

100644 是普通文件，100755 是可执行文件，120000 是符号链接文件。

添加之后就可以看到 .git/index 这个文件了，暂存区的内容就是放在这：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7656007e624d4db9b51203d89def32ec~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2350&h=1042&s=208678&e=png&b=1f1f1f)

这时候你执行 git status 就可以看到暂存区已经有这个文件了：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47a4ca4cac9440b28cf1857cf30e6f64~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1638&h=1186&s=150812&e=png&b=1f1f1f)

所以说，git add 的底层就是执行了 git update-index。

然后暂存区的内容写入版本库的话只要执行下 write-tree 就好了：

```arduino
git write-tree
```

然后你就会发现它返回了一个 hash，并且 objects 目录下多了一个 object：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c9f03be8a39418a8b69c7e99d4371ce~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1726&h=1062&s=227741&e=png&b=1e1e1e)

这个对象是啥类型呢？

通过 cat-file -t 看下就知道了：

```matlab
git cat-file -t 9ef7e5a61a3b70ff7149805fc86a4c26e953bb3f
```

可以看到，是个 tree 对象：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2edf214e725e4ce9af07699e80163713~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1282&h=124&s=22608&e=png&b=1e1e1e)

所以说，暂存区的内容是作为 tree 对象保存的。

再 cat-file -p 看下它的内容：

```matlab
git cat-file -t 9ef7e5a61a3b70ff7149805fc86a4c26e953bb3f
```

可以看到是这样的：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12695690b3a544db88f9d8bf07c69ec7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1406&h=150&s=104838&e=png&b=1f1f1f)

这就是 git commit 的原理了。

现在假设有个需求，让你找到某个版本的某个文件的内容，恢复回去。

是不是就很简单了？

只要找到对应版本的那个 tree 的 hash，然后再一层层找到对应的 blob 对象，读取内容再写入文件就好了！

这就是 git revert 的原理了。

当然，要是每个版本都要自己记住顶层 tree 的 hash 也太麻烦了。

所以 git 又设计了 commit 对象。

可以通过 commit-tree 命令把某个 tree 对象创建一个 commit 对象。

```bash
echo 'guang 111' | git commit-tree 9ef7e5
```

这里的参数就是上面的 tree 对象的 hash：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f9e5cf51625403ba427bddb9116ebb8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1334&h=340&s=74710&e=png&b=1e1e1e)

再用 cat-file -t 看看返回的对象的类型：

```matlab
git cat-file -t b5f92e68912595dbb3b6cbda9123838546b18f7d
```

确实，这是一个 commit 对象：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c61b0041ea142ef99419c135863bc2b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1310&h=302&s=65380&e=png&b=1e1e1e)

那 commit 对象都存了啥呢？

还是用 cat-file -p 看看：

```css
git cat-file -p b5f92e68912595dbb3b6cbda9123838546b18f7d
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9dfe62e12b34f33bf17ed1a7d9f197a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1374&h=272&s=60016&e=png&b=1e1e1e)

下面的内容很熟悉，但是多了一个 tree 节点的指向，这个很正常，commit 的内容就是某个 tree 所对应的版本嘛。

commit、tree、blob 三个对象就是这样的关系：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25bee8ce3df34bd59f60fe6c69020d1a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1742&h=1162&s=716596&e=png&b=fdfbfb)

commit 之间还能关联，也就是有先后顺序。

这个用 commit-tree -p 来指定：

比如我们再创建两个 commit：

```bash
echo 'guang 111' | git commit-tree 9ef7e5 -p b5f92e6
echo 'guang 222' | git commit-tree 9ef7e5 -p c3f9f5
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d81ee4f072f4bf184b359733c0ea2f6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1346&h=504&s=337881&e=png&b=1f1f1f)

这时你用 git log 看看：

```bash
git log 1d1234
```

你会看到平时经常看到的 commit 历史：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e60317c7ea14ed0973d1393f7d5f391~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1168&h=772&s=107056&e=png&b=1e1e1e)

这就是 commit 的实现原理！

当然，这里要记 commit 的 hash 同样也很麻烦。

平时我们怎么用呢？

用 branch 或者 tag 呀！

branch 和 tag 其实就是记录了这个 commit 的 hash。

这部分就不是 object 了，叫做 ref：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e35ef2beb1f4972aab42d52c580336a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=562&h=712&s=46554&e=png&b=252526)

创建 ref 使用 update-ref 的命令：

```sql
git update-ref refs/heads/guang 1d1234e77de6de0bb8edcf90cbd1a9546d7b1d9a
```

比如我创建了一个叫做 guang 的指向一个 commmit 对象的 ref。

这里就会多一个文件，内容存着指向的 commit 是啥：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce024530f40e452b94c1bc95b9e01828~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1562&h=904&s=125208&e=png&b=1e1e1e)

然后你 git branch 看看：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e2845059fce43ae9cad78cbc620902f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1706&h=1136&s=595123&e=gif&f=38&b=1e1e1e)

其实这就是创建了一个新的分支。

这就是 branch 的原理。

tag 也是一样，只不过它是放在 refs/tags 目录下的：

```sql
git update-ref refs/tags/v1.0 1d1234e77de6de0bb8edcf90cbd1a9546d7b1d9a
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39adf5025fef493aad6bc8689207da4c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1796&h=1146&s=522923&e=gif&f=21&b=1e1e1e)

blob、tree、commit 和 ref 的关系就是这样的：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6149526139b48899eb09d572c632977~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1708&h=880&s=556477&e=png&b=fdfbfb)

## 总结

今天我们探究了 git 的实现原理，主要是 3 个 object 以及两个 ref。

3 个对象是：

* blob： 存储文件内容
* tree： 存储目录结构和文件名，指向 blob 和 tree
* commit：存储版本信息，指向不同版本的入口 tree

2 个 ref 是：

* branch：指向某个 commit
* tag：指向某个 commit

此外，暂存区放在 .git/index 文件里，内容其实也是个 tree 对象的内容。

还有，hash 的计算方式是类似 blob 3\\0aaa 这样 “对象类型 内容长度\\0内容”的格式，对它做 sha1 然后转为十六进制。

基本看懂这张图就好了：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d3bc4b49b6a44028b5d5db603536d0e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1708&h=880&s=441017&e=png&b=fdfbfb)

用到了这几个命令：

* hash-object：创建 blob 对象
* cat-file -t: 查看对象类型
* cat-file -p: 查看对象内容
* update-index: 更新暂存区
* commit-tree: 创建 commit 节点
* write-tree: 暂存区写入版本库
* update-ref：创建和更新 ref

理解了这些，你就能理解 git add、git commit、git log、git revert、git branch、git tag 等等绝大多数 git 命令的实现原理了。

甚至按照这个思路来，自己写一个 git 是不是也不难呢？