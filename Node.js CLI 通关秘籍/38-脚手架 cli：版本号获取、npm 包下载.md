上节我们搭建了 monorepo 的架构，封装了项目模版和 cli，并发布到了 npm 仓库。

这节我们开始实现 create 命令。

按照这个流程来：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb38b576082a4abf8f3983adbf0a580e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1872&h=974&s=127296&e=png&b=fefdfd)

我们需要从 npm 仓库下载 template 包，并且后面还要拿到最新的版本来实现更新：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/574ce1746f81458590ce11a3b9e646c4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=944&h=520&s=75635&e=png&b=fdfdfd)

我们先把这些 npm 包下载相关的逻辑封装下。

这种逻辑放在 utils 包里。

```bash
mkdir packages/utils

cd packages/utils

npm init -y
```

创建 utils 包，然后改下 package.json

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43e3d46a012846a38d8e278577db21b8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=734&h=332&s=47953&e=png&b=1f1f1f)

```json
"name": "@guang-cli/utils",
"publishConfig": {
    "access": "public"
},
```

在 utils 包下创建 tsconfig.json

```css
pnpm --filter utils exec npx tsc --init
```

改下内容：

```json
{
  "compilerOptions": {
    "outDir": "dist",
    "types": [ "node" ],
    "target": "es2016", 
    "module": "NodeNext", 
    "moduleResolution": "NodeNext",
    "declaration": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "sourceMap": true
  }
}
```

修改 package.json 的 type 为 module

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3003cd57d75453b8baa857c1e3c5917~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=720&h=410&s=57172&e=png&b=1f1f1f)

并且注册下代码入口 main 和 ts 类型 types

```json
"type": "module",
"main": "dist/index.js",
"types": "dist/index.d.ts",
```

然后开始写代码。

我们先写下获取版本的代码，也就是从 registry + 包名的地址获取 json

比如 create-vite 的：

[registry.npmmirror.com/create-vite](https://registry.npmmirror.com/create-vite "https://registry.npmmirror.com/create-vite")

从 dist-tags 的 latest 属性拿到最新版本：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58c9a9a9c27246bd94bd4ef439fc361d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1052&h=596&s=91356&e=png&b=fefefe)

从 versions 属性可以拿到所有的版本号：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0856d820092146e4be4014b805b987b2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1036&h=1144&s=168290&e=png&b=fefefe)

创建 src/versionUtils.ts

```javascript
import axios from 'axios';
import urlJoin from 'url-join';

function getNpmRegistry() {
  return 'https://registry.npmmirror.com';
}

async function getNpmInfo(packageName: string) {
  const register = getNpmRegistry();
  const url = urlJoin(register, packageName);
  try {
    const response = await axios.get(url);

    if (response.status === 200) {
      return response.data;
    }
  } catch(e) {
    return Promise.reject(e);
  }
}

async function getLatestVersion(packageName: string) {
  const data = await getNpmInfo(packageName);
  return data['dist-tags'].latest;
}

async function getVersions(packageName: string) {
  const data = await getNpmInfo(packageName);
  return  Object.keys(data.versions);
}

export {
  getNpmRegistry,
  getNpmInfo,
  getLatestVersion,
  getVersions
}
```

我们封装了几个工具方法，首先是用 axios 访问 registry + packageName 的这个 json：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3338b08fc8d4e9999c4421d9ffd4a98~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1116&h=880&s=125745&e=png&b=ffffff)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b45306e5e884497e913ade93c6339504~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=944&h=868&s=179276&e=png&b=1f1f1f)

这里用 urlJoin 来连接 url，不用自己处理 / 结尾和不是 / 结尾的情况。

然后 getLatestVersion 和 getVersions 分别取 dist-tags.latest 和 versions 属性。

安装下用到的包：

```csharp
pnpm --filter utils add axios url-join
```

我们先来测试下：

创建 src/test.ts

```javascript
import { getNpmInfo } from './versionUtils.js';

const info = getNpmInfo('create-vite');

console.log(info);
```

跑一下：

```lua
pnpm --filter utils exec npx tsc

pnpm --filter utils exec node ./dist/test.js
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c0f9409f7cc406bb09eeeb6cd24101e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=964&h=524&s=72706&e=png&b=181818)

没啥问题。

然后来写下安装逻辑：

然后创建 src/NpmPackage.ts

```javascript
import fs from 'node:fs';
import fse from 'fs-extra';
// @ts-ignore
import npminstall from 'npminstall';
import { getLatestVersion, getNpmRegistry } from './versionUtils.js';
import path from 'node:path';

export interface NpmPackageOptions {
    name: string;
    targetPath: string;
}

class NpmPackage {

    name: string;
    version: string = '';
    targetPath: string;
    storePath: string;
    
    constructor(options: NpmPackageOptions) {
        this.targetPath = options.targetPath;
        this.name = options.name;

        this.storePath = path.resolve(options.targetPath, 'node_modules');
    }

    async prepare() {
        if (!fs.existsSync(this.targetPath)) {
            fse.mkdirpSync(this.targetPath);
        }
        const version = await getLatestVersion(this.name);
        this.version = version;
    }

    async install() {
        await this.prepare();

        return npminstall({
            pkgs: [
                {
                    name: this.name,
                    version: this.version,
                }
            ],
            registry: getNpmRegistry(),
            root: this.targetPath
        });
    }

    get npmFilePath() {
        return path.resolve(this.storePath, `.store/${this.name.replace('/', '+')}@${this.version}/node_modules/${this.name}`);
    }

    async exists() {
        await this.prepare();

        return fs.existsSync(this.npmFilePath);
    }

    async getPackageJSON() {
        if(await this.exists()) {
            return fse.readJsonSync(path.resolve(this.npmFilePath, 'package.json'))
        }
        return null;
    }

    async getLatestVersion() {
        return getLatestVersion(this.name);
    }

    async update() {
        const latestVersion = await this.getLatestVersion();
        return npminstall({
            root: this.targetPath,
            registry: getNpmRegistry(),
            pkgs: [
                {
                    name: this.name,
                    version: latestVersion,
                }
            ]
        });
    }
}

export default NpmPackage;
```

前面我们测试过 npminstall，需要传入 name、version、targetPath：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ee22f3bbcbd4a9181d301f2f7bbc00a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=990&h=512&s=84231&e=png&b=1f1f1f)

所以我们接收 name、targetPath，并在 prepare 方法里拿到最新版本号：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebbafb74088b423d8b9edc05c8745674~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1302&h=1128&s=190753&e=png&b=1f1f1f)

targetPath 加上 node\_modules 就是 storePath

然后分别实现了 install、update 方法

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae505355017147d2af704c9ed1e5d0ce~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1020&h=920&s=128635&e=png&b=1f1f1f)

install 的时候调用 npminstall，传入 name、version、root 就可以安装了。

npminstall 安装后的目录结构是这样的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7d89d9b2eb5495f99d0b108e8a0d58b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=618&h=596&s=50491&e=png&b=181818)

.store/xxx@version/node\_modules/xxx

但如果是 @xxx/xxx 的包，目录是这样的：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/387646294fdc4e388f4c09c541497a26~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=546&h=452&s=46180&e=png&b=191919)

最外层目录包名中的 / 会被替换为 +

所以我们拼接一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4be883038b24e11ba44a62616b8f833~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1916&h=796&s=158461&e=png&b=1f1f1f)

exists 方法就是判断这个目录是否存在，从而判断包又没有安装。

getPackageJSON 方法就是用 readJSONSync 读取 package.json 文件。

然后是 update 方法：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48d3842fc6d241acbbb3117b870bc3b9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1216&h=896&s=151227&e=png&b=1f1f1f)

安装的时候安装了 4.4.1 版本，那调用 update 就可以更新到 5.5.5 版本。

所以我们用 getLatestVersion 拿到最新版本之后，再用 npminstall 安装。

此外，我们加了 @ts-ignore 是因为这个包没有 ts 类型，会导致 ts 报错，所以直接忽略类型就好了：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f80a4591618643cbba225cafd7db0d4f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=878&h=312&s=69113&e=png&b=1f1f1f)

在 test.ts 加下测试代码：

```javascript
import NpmPackage from './NpmPackage.js';
import { getLatestSemverVersion, getLatestVersion, getNpmInfo, getNpmLatestSemverVersion, getNpmRegistry, getVersions } from './versionUtils.js';
import path from 'node:path';

async function main() {
    const pkg = new NpmPackage({
        targetPath: path.join(import.meta.dirname, '../aaa'),
        name: 'create-vite'
    });

    if(await pkg.exists()) {
        pkg.update();
    } else {
        pkg.install();
    }

    console.log(await pkg.getPackageJSON())
}

main();
```

如果这个包安装过，就 update 更新版本，否则 install 安装这个版本。

跑一下：

```lua
pnpm --filter utils exec npx tsc
pnpm --filter utils exec node ./dist/test.js
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ffe59c7f5c5423980433cb8eaac6796~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1482&h=722&s=189721&e=png&b=1b1b1b)

第一次读取到的 package.json 是 null，安装的最新版本 5.5.5 到 aaa 目录

再跑一次：

```bash
pnpm --filter utils exec node ./dist/test.js
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8bd8943cd3134294993f7a6c32e4a846~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1410&h=724&s=192612&e=png&b=1a1a1a)

这次获取到了 package.json，说明是更新。

如果有新版本，那就会下载最新的版本的包。

然后安装下 @babel/core 包：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59c14c9a949e488088b7fba6363a1139~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1090&h=654&s=107269&e=png&b=1f1f1f)

```lua
pnpm --filter utils exec npx tsc
pnpm --filter utils exec node ./dist/test.js
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6fcdf279bf1e4b4db74b3c43d7d1df64~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1078&h=450&s=101344&e=png&b=191919)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce95232adbfa4a1d93afade82843f823~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=696&h=488&s=62183&e=png&b=1a1a1a)

再次跑：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93ad83325d9e43af98bc102d48000791~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1128&h=456&s=90604&e=png&b=181818)

然后添加一个 src/index.ts

```javascript
import NpmPackage from "./NpmPackage.js";
import * as versionUtils from './versionUtils.js';

export {
    NpmPackage,
    versionUtils
}
```

我们把这个 utils 包也发到 npm 仓库。

先把本地的代码 commit 一下。

然后执行 npx changeset add

```csharp
npx changeset add
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9a9abf896c140e3aad2ab2deba128b4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=922&h=326&s=58423&e=png&b=191919)

选择发布 utils 包，按住空格来选择

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4dfcbac6c5364c5fbda97d84b8aff167~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1550&h=660&s=179374&e=png&b=191919)

选择改 minor 版本号

在 .changeset 下多了一个临时文件记录着这次变更的信息：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6dc9ccc5420742f7b5cfa264229a8bfc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1330&h=306&s=62305&e=png&b=1c1c1c)

然后执行 version 命令来生成最终的 CHANGELOG.md 还有更新版本信息：

```
npx changeset version
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df2c6f70da6a420f9129ba900743fc4a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1040&h=90&s=22662&e=png&b=191919)

utils 包下多了 CHANGELOG.md 文件：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1682a612ffac419ba22f486bc9670129~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1596&h=460&s=119471&e=png&b=202020)

并且更新了版本号：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cde29de74d564a318791a9ea0e1ec49e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1492&h=468&s=130553&e=png&b=1f1f1f)

可以看到，变的是我们选择的 minor 版本号。

然后创建一个 commit，发布到 npm 仓库：

```sql
git add .
git commit -m 'utils 1.1.0'

npx changeset publish
```

创建 commit 是因为 changeset 会给最新的 commit 打 tag。

执行 changeset publish 命令：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8214aee0b474542b971c9c907b69b78~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1274&h=620&s=183074&e=png&b=191919)

发布到了 npm，并在这个 commit 上打了两个 tag

去 [npm 网站](https://www.npmjs.com/~quark-gluon-plasma?activeTab=packages "https://www.npmjs.com/~quark-gluon-plasma?activeTab=packages")看一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa7e2b88daa1412ea0f7410e534b7519~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1214&h=464&s=72035&e=png&b=fefefe)

发布成功了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/guang-cli "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/guang-cli")

## 总结

这节我们创建了 utils 包，封装了 versionUtils、NpmPakcage。

versionUtils 是用来获取 npm 包的版本号的，通过 resitry + 包名 的 url 返回的 json 里的 dist-tags、versions 等获取版本号。

NpmPackage 封装了 npminstall 的安装逻辑，实现了 install、update、exists、getPackageJSON 等方法。

之后通过 changeset 把它发布到了 npm 仓库。

npm 包的下载完成了，下节来继续实现 create 命令。