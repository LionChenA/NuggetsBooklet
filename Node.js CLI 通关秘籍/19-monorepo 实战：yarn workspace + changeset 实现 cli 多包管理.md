前面讲过，npm workspace、yarn workspace、pnpm workspace 都可以实现 monorepo。

这节我们用一遍，亲身感受下。

首先是 yarn workspace：

创建个项目：

```bash
mkdir yarn-monorepo-test

cd yarn-monorepo-test

npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/182448ed85714bb085e51b02d50ac969~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=882&h=652&s=90680&e=png&b=010101)

进入项目，修改 package.json

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4a4c9d569bd407491a138981c08cc56~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=674&h=390&s=54633&e=png&b=1f1f1f)

添加 workspaces 配置：

```javascript
"private": "true",
"workspaces": [
    "packages/*"
],
```

一般我们都会在根目录的 package.json 加上 private: true，也就是不把这个目录发到 npm 仓库。

然后初始化两个目录

```bash
npm init -w packages/cli -y
npm init -w packages/core -y
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0800167625454760915363789102079e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1430&h=972&s=162670&e=png&b=1a1a1a)

\-w 是 --workspace，会自动添加这个包到 package.json 的 workspaces 数组里，不过我们添加过了。

接下来在根目录执行 yarn

```
yarn
```

这时候你会发现 node\_modules 下多了两个软链目录：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ab1bef9d5ea4d2fbe3abe8b81531901~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1256&h=698&s=140810&e=png&b=1c1c1c)

![2024-09-15 09.05.02.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48861f1314e949b7821f6979e7f23d4f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1676&h=842&s=606777&e=gif&f=52&b=f4f2f1)

可以看到，就是从 pacakges 下软链过来的。

然后我们在 cli 包安装 core 包的依赖：

```sql
yarn workspace cli add core@1.0.0
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/afc4f2e43adb459ebcd8afb970ff282b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1488&h=1072&s=208872&e=png&b=1b1b1b)

这里必须指明版本号才会安装本地 packages 下的另一个包为依赖，不然会从 npm 仓库来查找依赖。

这是 yarn workspace 的一个[历史 bug](https://github.com/yarnpkg/yarn/issues/3973 "https://github.com/yarnpkg/yarn/issues/3973")。

我们在 core 包下添加 index.js

```javascript
module.exports = 'hello monorepo'
```

然后在 cli 包下添加 index.js

```javascript
const core = require('core');

console.log(core);
```

跑一下：

```bash
node ./packages/cli/index.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4dfcb83fdc24adb9b10822ae66b570b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=542&h=96&s=15733&e=png&b=181818)

没问题，依赖查找成功。

node 查找 require 或者 import 引入的依赖时会层层往上找，统一放在根目录的 node\_modules 就都能找到。

然后在根目录安装 typescript：

```css
npm install --save-dev typescript @types/node
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0fe7a55954341c2b91c43efd6825cbc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1112&h=594&s=126770&e=png&b=1c1c1c)

在 core 包下创建 tsconfig.json

```bash
cd packages/core

npx tsc --init
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2328b00be9943cc9f7ed2eecbd88f55~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=874&h=482&s=70297&e=png&b=181818)

然后改下内容：

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
  }
}
```

这里我们还声明了 declaration 为 true，也就是生成 d.ts 类型文件。

修改 package.json 的 type 为 module

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75383348575f46aabe10ec2371f4e9f1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=686&h=296&s=39997&e=png&b=1f1f1f)

然后把之前的 index.js 删掉，创建 src/index.ts

```javascript
function add(a: number, b: number) {
    return a + b;
}

function minus(a: number, b: number) {
    return a - b;
}

export {
    add,
    minus
}
```

编译下：

```
npx tsc
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f84f40775c2b4972b4b19f217820fda6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1408&h=756&s=128449&e=png&b=1c1c1c) 然后把产生的 dist/index.js 配置为入口：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad238b0dca664d1792c21d47914d6d26~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1240&h=548&s=117520&e=png&b=1c1c1c)

```json
"main": "dist/index.js",
"types": "dist/index.d.ts",
```

并且把 dts 配置为 ts 类型入口。

接下来在 cli 包里用一下：

```bash
cd packages/cli

npx tsc --init
```

同样改下 tsconfig.json

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
  }
}
```

然后改下 package.json

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8ce724b7fbb47e4837ba28f90a83116~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=658&h=322&s=44692&e=png&b=1f1f1f)

```json
"main": "dist/index.js",
"types": "dist/index.d.ts",
"type": "module",
```

删掉之前的 index.js 创建 src/index.ts

```javascript
import { add, minus } from 'core';

console.log(add(1, 2));
console.log(minus(5, 3));
```

编译下，然后跑一下：

```bash
npx tsc

node ./dist/index.js
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0de6b9b198884df5a02b94e2bfc1485b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1164&h=488&s=99903&e=png&b=1b1b1b)

没啥问题。

然后我们在 cli 包安装 chalk 和 commander

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f18e38233948486cabbecf5378adbd02~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=926&h=1088&s=168239&e=png&b=1c1c1c)

```csharp
yarn workspace cli add chalk commander
```

在某个 workspace 下安装依赖用 yarn workspace xxx add

删除依赖用 yarn workspace xxx remove

再改下 src/index.ts

```javascript
#!/usr/bin/env node
import { Command } from 'commander';
import chalk from "chalk";
import { add, minus } from 'core';

const program = new Command();

program
  .name('num-cli')
  .description('计算数字加减')
  .version('0.0.1');

program.command('add')
  .description('加法')
  .argument('a', '第一个数字')
  .argument('b', '第二个数字')
  .action((a: string, b: string) => {
    console.log(chalk.green(add(+a, +b)))
  });

program.command('minus')
  .description('减法')
  .argument('a', '第一个数字')
  .argument('b', '第二个数字')
  .action((a: string, b: string) => {
    console.log(chalk.cyan(minus(+a, +b)))
  });

program.parse();
```

就是用 commander 声明了两个子命令，分别有 a、b 两个参数。

编译然后跑一下：

```bash
npx tsc

node ./dist/index.js

node ./dist/index.js add 1 2

node ./dist/index.js minus 1 2
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd5e029c15754af5b018d1434138a6ea~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=942&h=690&s=116196&e=png&b=181818)

没啥问题。

在 package.json 配置下 bin：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3633b0537ad7496199ddc140ca91c88e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=626&h=404&s=52976&e=png&b=1f1f1f)

就可以发布这个包出去用了。

不过我们现在用的 cli、core 的包名太容易冲突了。

一般 monorepo 的包都是用 @scope/xxx 的命名方式。

比如 [@babel/cli](https://github.com/babel/babel/blob/main/packages/babel-cli/package.json#L2 "https://github.com/babel/babel/blob/main/packages/babel-cli/package.json#L2")

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/843306610a2c48db908228fab5bc73da~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1706&h=984&s=245771&e=png&b=ffffff)

[@babel/core](https://github.com/babel/babel/blob/main/packages/babel-core/package.json#L2 "https://github.com/babel/babel/blob/main/packages/babel-core/package.json#L2")

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3d55e9e6f2e4678aad031f37516968a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1722&h=1052&s=230230&e=png&b=fffefe)

我们也改一下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbfd44473eb14398aaba36f4f37236da~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1290&h=608&s=132984&e=png&b=1d1d1d)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0e4c9a99bc14cb0ba35e5dbc5397fe5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1282&h=638&s=130202&e=png&b=1c1c1c)

分别叫 @guang-yarn/cli、@guang-yarn/core。

并且依赖的名字也要改一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f8ce83671174995a69c0f527c27f0fa~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=910&h=852&s=125631&e=png&b=1f1f1f)

然后改下代码：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f019d4ee193746aeb658d3592f1c0279~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1402&h=792&s=203803&e=png&b=1d1d1d)

改了名字需要再根目录重新执行下 yarn，会自动在 node\_modules 创建新的包的软链。

这样就可以发包了。

进入 packages/cli 目录登录 npm：

```bash
cd packages/cli

npm adduser
```

执行 npm adduser 命令，会让你输入用户名、密码、邮箱、验证码：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/326f90a67bdb41158dc75adfc490abf7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=988&h=244&s=55915&e=png&b=181818)

如果你还没账号，就先去 [www.npmjs.com/](https://www.npmjs.com/ "https://www.npmjs.com/") 注册一个。

如果你的浏览器登录过 npm，会提示你打开浏览器，授权之后就登录成功了：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dea6fb2bad44461cad9374d97d7fdd0d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1204&h=212&s=44162&e=png&b=181818)

```
npm publish
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de42505c86db4b30ab09b124fa40b6f5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1746&h=686&s=192014&e=png&b=171717)

这时候会报错，因为默认 @scope/xxx 的包是私有包，npm 仓库发公共的包免费，但是私有是付费的。

改成公有就可以了：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55c1237f423f4f40b27e5055a78626f9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=718&h=484&s=69390&e=png&b=1f1f1f)

```json
"publishConfig": {
    "access": "public"
},
```

再次 publish 会提示你 scope not found：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44ee8d68772241dbb821730a4312da72~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1474&h=874&s=223435&e=png&b=171717)

想发 @scope/xx 的包需要先在 npm 注册个组织。

在 [npm 网站](https://www.npmjs.com/ "https://www.npmjs.com/") 点击创建组织：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4619848f442f4d43b6bf1ff34caf3207~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2876&h=976&s=218363&e=png&b=fdfdfd)

选择公共的：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adee642386844bd09beb7a470aa4e598~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1646&h=1298&s=154951&e=png&b=fbfbfb)

提示创建成功：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2bf4aa776bd64f108c951fa746f84055~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1124&h=470&s=41648&e=png&b=fefaf3)

然后再发布就可以了。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d70822069cc498c95857cd29209bf6a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1272&h=670&s=166885&e=png&b=171717)

点击 profile：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01f877be2511451eb03fdfcbce56ccdd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=640&h=892&s=68310&e=png&b=fcfcfc)

可以看到你创建的组织列表：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/332e7058e2604b3ebf3106457d2901ed~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2490&h=996&s=104761&e=png&b=fdfcfc)

你发布的包：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a509f863a0a8400c990e3ea086473bac~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2502&h=1166&s=204273&e=png&b=fdfdfd)

然后我们用同样的方式把 @guang-yarn/core 包发布下：

在 core 包的 package.json 加一下 publicConfig

```bash
cd ../core

npm publish
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d2a56bb8b694ba28feee4ad6fe885d7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1340&h=1118&s=243460&e=png&b=1a1a1a)

[看一下](https://www.npmjs.com/~quark-gluon-plasma "https://www.npmjs.com/~quark-gluon-plasma"):

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/605aaec780414f38ac1267f842d0ddec~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2482&h=1174&s=214286&e=png&b=ffffff)

这样就发布成功了。

我们直接用 npx 来跑下这个包

```sql
npx @guang-yarn/cli

npx @guang-yarn/cli add 1 2 

npx @guang-yarn/cli minus 3 4
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97c154101893484aa07ef0bc0d28d82c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=782&h=570&s=68703&e=png&b=010101)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4f75d02693b4f4996fa7cceafda41b7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=542&h=190&s=24344&e=png&b=010101)

或者全局安装也可以：

```sql
npm install -g @guang-yarn/cli

calc add 1 2

calc minus 1 2
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d3c05db98fe49338a592a335f0f8b2b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=704&h=764&s=81760&e=png&b=010101)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fa8d1833fec448088a25c63240a2a80~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=384&h=190&s=18044&e=png&b=000000)

这个 calc 是我们注册的命令的名字：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e277c791f08444eeba9b7a7c818f49ca~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=608&h=320&s=45383&e=png&b=1f1f1f)

一个 npm 包可以注册很多个命令。

这样，基于 yarn workspace 怎么开发 npm 包我们就会了。

不过大家觉得发包麻烦不？

现在其实还好，只有 cli 和 core 两个 package，手动 publish 一下就行。

如果像 babel 这种 packages 下有[上百个包](https://github.com/babel/babel/tree/main/packages "https://github.com/babel/babel/tree/main/packages")的呢？

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5102bc9a08024c259ae482506d67d884~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1132&h=1332&s=209159&e=png&b=ffffff)

想想维护起来就很痛苦。

其实只是 publish 还好，上百个包的 publish 写个脚本自动做就可以了。

但最麻烦的是版本的变更。

比如 cli 包依赖了 core 包，那 core 包版本变了，是不是就得更新 cli 的 package.json 的 dependencies 里 core 包的版本，然后更新 version 发一个新的包？

上百个包的版本变更呢？

那显然就不能靠手动做这件事了。

这时候为啥用 changeset 是不是就很清楚了？

执行 changeset init：

```csharp
yarn add --dev -W @changesets/cli prettier-plugin-organize-imports prettier-plugin-packagejson

npx changeset init
```

会多一个 .changeset 目录：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91a320efa07346e990de79d47f6db0a2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1678&h=520&s=144124&e=png&b=1a1a1a)

打印的信息也说了 changeset 就是用来处理版本变更、包发布的。

changeset 会基于 git 来判断代码有没有变动。

我们初始化下 git

创建 .gitignore

```
node_modules/
.DS_Store
```

执行 init

```csharp
git init

git add .

git commit -m 'first commit'
```

然后改一下代码：

在 core 包加一个 devide 方法：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5df8fb129bfa4c889ec5d443913732f6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1244&h=692&s=131435&e=png&b=1d1d1d)

```javascript
function add(a: number, b: number) {
    return a + b;
}

function minus(a: number, b: number) {
    return a - b;
}

function devide(a: number, b: number) {
    return a / b;
}

export {
    add,
    minus,
    devide
}
```

在 core 包和 cli 包下加一个 npm scripts

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1185c1c413e94aa6b1c02e1496d60d55~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=684&h=500&s=77046&e=png&b=202020)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75fc83914d844690935256794302ac4c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=528&h=566&s=67239&e=png&b=1f1f1f)

```json
"build": "tsc",
```

这样就不用来回切目录了。

可以直接指定 workspace 来跑 npm scripts，或者全部都跑：

```arduino
yarn workspace @guang-yarn/core run build

yarn workspaces run build
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a975b8ffc16748b2b56d6feaf547f3ec~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=818&h=656&s=93536&e=png&b=181818)

然后再改下 cli 包：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1ba057a9a93407198e893689e0f26cc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=826&h=480&s=92782&e=png&b=1f1f1f)

```javascript
program.command('devide')
  .description('除法')
  .argument('a', '第一个数字')
  .argument('b', '第二个数字')
  .action((a: string, b: string) => {
    console.log(chalk.cyan(devide(+a, +b)))
  });
```

编译下：

```arduino
yarn workspaces run build
```

这时候我们要发新版本了。

手动 publish 怎么做呢？

改一下 @guang-yarn/core 的版本、改一下 @guang-yarn/cli 的版本还有 dependencies 里的 @guang-yarn/core 的版本，之后分别 publish

现在用 changeset 来做：

changeset 管理版本号是遵循 semver 规范的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3262f97034f44a7dad898857c2614fae~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1148&h=398&s=105382&e=png&b=fefefe)

我们做了向下兼容的功能新增，改次版本号。

执行 changeset add

```csharp
npx changeset add
```

changeset 就是一次改动的集合，可能一次改动会涉及到多个 package，多个包的版本更新，这合起来叫做一个 changeset。

按住空格来选择:

![2024-09-18 12.06.03.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ebdc6c4e9d34e74aecac56aac334147~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1000&h=554&s=137316&e=gif&f=63&b=151716)

会列出所有的改动的包，这个是基于 git 来检测的，所以本地代码不要自己 commit。

选择这两个包都是 minor 版本号变更，然后输入改动的描述：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2021dad1c12e47a587ccd7dc4c839631~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1250&h=562&s=154456&e=png&b=181818)

这时你就会发现在 .changeset 下多了一个临时文件记录着这次变更的信息：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c0535ec7264476d9b1a1e305214f509~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1126&h=352&s=76802&e=png&b=1b1b1b)

然后你可以执行 version 命令来生成最终的 CHANGELOG.md 还有更新版本信息：

```
npx changeset version
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/930fc381fb9b4fe58267fd62fc7fd5de~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1076&h=104&s=22734&e=png&b=181818)

那些临时的 changeset 文件就消失了：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df0d5ee559a7446a96af4a54639fd6a3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=340&h=136&s=15596&e=png&b=272829)

更改的包下都多了 CHANGELOG.md 文件：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d65e5142fd404542a71ece79a137ee76~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2292&h=714&s=217726&e=png&b=1f1f1f)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7348a783be6147f99cd2d20e0781b6ca~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2392&h=796&s=258811&e=png&b=1e1e1e)

并且都更新了版本号：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b67d6cae3c924af58b05741454d570d7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2368&h=726&s=259018&e=png&b=1e1e1e)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e4f94ebca5f4a539b7382356ba1d681~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2354&h=994&s=300353&e=png&b=1e1e1e)

@guang-yarn/core、@guang-yarn/cli 的版本，dependencies 里的版本号都变了。

然后发布到 npm 仓库：

```sql
git add .
git commit -m 'second commit'

npx changeset publish
```

执行 changeset publish 命令就可以，并且还会自动打 tag：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c005b96f0eb4dab82449c779b934833~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1484&h=388&s=116357&e=png&b=191919)

（因为会打 git tag 所以要 commit一下）

去 npm 网站看一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9846f97daf324259bf53d4dba10a97cc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2008&h=1356&s=205922&e=png&b=fdfdfd)

两个包都发布成功了。

然后看下本地的 git tag：

![2024-09-18 12.47.39.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a75b3eb13d24bf29efb133c41b56aa5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1000&h=554&s=41278&e=gif&f=19&b=181818)

tag 可以用来轻松地找到哪个版本对应哪个 commit，还是很有必要的。

用 changeset 是不是超级方便！

比手动搞简单多了，包越多收益越大。

这就是 changeset 的两个功能：

* 根据用户选择来更新版本号，并且自动更新依赖的版本号和 dependencies 里的版本
* 自动发布所有改动的包到 npm 仓库，并打 tag，和生成 CHAGELOG.md

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07e2e76ac6b94ba4bdf2500090da4888~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1380&h=1106&s=121229&e=png&b=fefdfd)

最后提一下 yarn workspace 的 info 命令：

```
yarn workspaces info
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/437dcfa5ea06443db1abd0b516c701ba~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=670&h=502&s=64618&e=png&b=191919)

可以查看本地的包的依赖关系

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/yarn-monorepo-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/yarn-monorepo-test")

## 总结

这节我们用 yarn workspace + changeset 实现了 monorepo 形式的 npm 包的开发、构建、发布、版本变更。

yarn workspace 首先在 pacakge.json 的 workspaces 下配置哪些包是 workspace，然后执行 yarn install 就会在 node\_modules 下自动 link 这些包。

我们用到了这些命令：

安装本地的其他包为依赖：

```sql
yarn workspace cli add core@1.0.0
```

在某个 workspace 下安装依赖：

```csharp
yarn workspace cli add chalk
```

在根目录安装依赖：

```sql
yarn add typescript -W --dev
```

在某个 worksapce 下执行 npm scripts

```arduino
yarn workspace @guang-yarn/core run build
```

在全部 workspace 执行 npm scripts

```arduino
yarn workspaces run build
```

查看本地 workspace 的依赖树：

```
yarn workspaces info
```

然后用 @changesets/cli 来管理版本变更和发布：

初始化：

```csharp
npx changeset init
```

创建一次变更：

```csharp
npx changeset add
```

改动版本号并生成 CHANGELOG.md

```
npx changeset version
```

发布到 npm 仓库并自动打 tag：

```
npx changeset publish
```

当然，如果包比较少，你也可以不用 changeset，直接自己管理版本和发布：

```
npm adduser

npm publish

git tag v1.0
```

不过这种方式比较原始，成熟的方案还是用 changeset 来自动做。

总之，用 yarn workspace 来实现包之间的依赖、命令的执行，用 changeset 来实现版本变更、npm 包的发布。

两者结合起来，非常好用。