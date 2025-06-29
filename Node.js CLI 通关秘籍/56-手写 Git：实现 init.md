这节我们来实现 git init。

其实 init 的原理非常简单，就是在 .git 的目录下写入一堆文件。

创建项目：

```perl
mkdir my-git
cd my-git
npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0253a3fbf674eef8d1feffe14837f19~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=852&h=660&s=83197&e=png&b=010101)

我们先试下 git init：

```csharp
git init
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dcd156a79a174d879b90e92419947431~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1736&h=794&s=147195&e=png&b=1c1c1c)

执行后会创建 .git 目录，下面有一堆文件。

然后我们可以执行 git add、commit、log 等命令：

```sql
git add .
git commit -m 'first commit'
git log
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/090d754af07f4606ac40665d7d2b0084~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=766&h=282&s=58064&e=png&b=191919)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c64d207881174f83b0dd1fb4c9d60bc7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1118&h=394&s=45198&e=png&b=181818)

然后我们删掉 .git 目录，自己来写入这些文件。

创建 src/init.mjs

```javascript
import fs from 'node:fs';

const gitdir = '.git'
const defaultBranch = 'main';

export function init() {
    if (fs.existsSync(gitdir + '/config')) return;

    const folders = [
        'hooks',
        'info',
        'objects/info',
        'objects/pack',
        'refs/heads',
        'refs/tags',
    ].map(dir => gitdir + '/' + dir);

    for (const folder of folders) {
        fs.mkdirSync(folder, {
            recursive: true
        })
    }
    
    fs.writeFileSync(
        gitdir + '/config',
        '[core]
' +
            '\trepositoryformatversion = 0
' +
            '\tbare = false
' +
            '\tfilemode = false
' +
            '\tsymlinks = false
' +
            '\tignorecase = true
'
    )
    fs.writeFileSync(gitdir + '/HEAD', `ref: refs/heads/${defaultBranch}
`)
}
```

写入一堆文件，还有 .git/config 文件

创建 src/test.mjs

```javascript
import { init } from "./init.mjs";

init();
```

跑一下：

```bash
node ./src/test.mjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d65ca24320144ff99d6a59b01dad5dd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1360&h=742&s=128638&e=png&b=1c1c1c)

.git 目录和下面的文件也都创建成功了。

我们试一下：

```sql
git add .
git commit -m '111'
git log
git branch
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7091ffac15e440eb791087aadd2ade9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=620&h=354&s=70184&e=png&b=191919)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91c0c88cb5b84860b077bc68f729e756~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1104&h=274&s=39925&e=png&b=181818)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e70b7f72b35c4379974829be1a67711c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=248&h=136&s=7694&e=png&b=181818)

可以看到，add、commit、log、branch 命令都可以正常执行。

这样 git init 我们就实现了。

接下来继续实现 git add：

// 未完待续