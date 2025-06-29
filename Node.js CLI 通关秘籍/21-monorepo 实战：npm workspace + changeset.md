这节我们来做 npm workspace + changeset 的实战。

它和 pnpm workspace、yarn workspace 差不多，但我们还是做一遍，以后用到可以回来查。

创建个项目：

```bash
mkdir npm-monorepo-test

cd npm-monorepo-test

npm init -y
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6c4a804288a4e1a82266b01a03505a9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=852&h=686&s=136254&e=png&b=010101)

进入项目，修改 package.json，加上 private:true

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e5dbe9f2cc5494c93f435d7710066ab~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=618&h=302&s=39041&e=png&b=1f1f1f)

然后创建两个包：

```bash
npm init -w packages/core -y
npm init -w packages/cli -y
```

它会自动在根目录 package.json 里添加 workspace 配置：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bedc3cea513e45b697172c35c872d7be~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1514&h=1274&s=219229&e=png&b=1b1b1b)

然后在 cli 包添加 core 包为依赖：

```css
npm install core --workspace cli
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/929fe3651b4e47e3baf82c4a0a68b426~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1230&h=924&s=156555&e=png&b=1c1c1c)

可以看到，cli 包的 package.json 添加了依赖，并在根目录 node\_modules 创建了软链。

是从 packages 下 link 过来的：

![2024-09-24 20.29.01.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/493d9390c3f141ae9be3bbe59b922753~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1078&h=838&s=418903&e=gif&f=40&b=fdfcfc)

然后我们改下包名，加上 scope：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86edfa970c1e472f98e1015e9e603116~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=636&h=274&s=41526&e=png&b=1f1f1f)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f9360eecd4d47e1a06762df8c4b85eb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=518&h=202&s=27391&e=png&b=1f1f1f)

再改下依赖名：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75cdbd7aff0a4617a73c64bd0dcadd90~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=784&h=636&s=82779&e=png&b=1f1f1f)

然后再次 npm install

```
npm install
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68f2f5f140a2482dbe8b1333abd16a91~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1064&h=588&s=125461&e=png&b=1c1c1c)

可以看到，node\_modules 下的依赖更新了。

然后我们安装 typescript：

```css
npm install --save-dev typescript @types/node
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e05e5a907a1e4370ac80eb2788586536~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1126&h=658&s=137106&e=png&b=1d1d1d)

在 core 和 cli 包下创建 tsconfig.json

```bash
npm exec --workspace @guang-npm/cli -- npx tsc --init
```

\--workspace 是在 cli 包下执行命令， 如果想在所有 workspace 下执行命令，这样写：

```css
npm exec --workspaces -- npx tsc --init
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67e57550afa444029ce6afc2333d148a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=906&h=380&s=51989&e=png&b=181818) 然后改下内容：

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

修改 package.json 的 type 为 module

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3256f858a07d44f9aacaffeebee4a127~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=570&h=286&s=42482&e=png&b=1f1f1f)

并且注册下代码入口 main 和 ts 类型 types

```json
"type": "module",
"main": "dist/index.js",
"types": "dist/index.d.ts",
```

同样的方式也改下 core 包的：

```bash
npm exec --workspace @guang-npm/core -- npx tsc --init
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c1c3723921a4a319b4f3283d87864bb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=560&h=282&s=42009&e=png&b=1f1f1f)

然后在 core 包下创建 src/index.ts

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

```bash
npm exec --workspace @guang-npm/core -- npx tsc
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db482dbf60d545bf9e2eba9524cc5c8e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1422&h=490&s=88967&e=png&b=1d1d1d)

然后改一下 cli 包：

添加依赖：

```css
npm install --workspace @guang-npm/cli chalk commander
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b29ef65d9d9e4e319519414206c357a6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1030&h=714&s=114001&e=png&b=1f1f1f) 创建 src/index.ts

```javascript
#!/usr/bin/env node
import { Command } from 'commander';
import chalk from "chalk";
import { add, minus } from '@guang-npm/core';

const program = new Command();

program
  .name('num cli')
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

编译下：

```sql
npm exec --workspaces -- npx tsc
```

\--workspaces 是所有 workspace 都执行这个命令。

跑一下：

```bash
npm exec --workspace cli -- node ./dist/index.js add 1 2
npm exec --workspace cli -- node ./dist/index.js minus 1 2
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b92a94c28d67424b9e5cf55ea85077b5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1096&h=174&s=45533&e=png&b=181818)

没啥问题。

在 package.json 配置下 bin：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/465a324247d04ef18f6948867f822aba~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=602&h=382&s=53583&e=png&b=1f1f1f)

```json
"bin": {
    "num-calc": "./dist/index.js"
},
```

接下来就可以把它发布到 npm 了。

版本更新、npm 包的发布，用 changeset 来做。

我们先在 npm 增加一个组织：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23774a8c9e594b04969c958a53dfe919~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=818&h=884&s=72604&e=png)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa79f45490b043609fc6124a471cea2f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1724&h=1350&s=158472&e=png&b=fafafa)

看下创建的组织：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e268b8d5d7df414b9469c29c5ed865ed~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2666&h=1054&s=148787&e=png&b=fdfdfd)

然后我们就可以在 @guang-npm 这个 scope 下发包了。

还需要在 cli 和 core 都加一下 publishConfig 配置为公有的：

```json
"publishConfig": {
    "access": "public"
},
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c86df5e79229469d96800d58a848a9bd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=720&h=370&s=55534&e=png&b=1f1f1f)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac7a2226a28045ff811f80d4bc5aa977~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=688&h=498&s=69652&e=png&b=202020) 登录下 npm

```
npm adduser
```

浏览器登录过 npm，会提示你打开浏览器，授权之后就登录成功了：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dea6fb2bad44461cad9374d97d7fdd0d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1204&h=212&s=44162&e=png&b=181818)

没账号的话需要先去注册一个。

然后安装 changeset，执行 changeset init：

```css
npm install --save-dev @changesets/cli prettier-plugin-organize-imports prettier-plugin-packagejson

npx changeset init
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e26d2d8fe8d4dc8bc7018ef82f05bf7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1220&h=252&s=73605&e=png&b=181818)

会多一个 .changeset 目录：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/466e6836f1b2411084d5d0cbe40530bb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1454&h=450&s=115396&e=png&b=1d1d1d)

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

changeset 要基于上次的 commit 来判断变更，所以我们先创建一个 commit。

然后执行 changeset add

```csharp
npx changeset add
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5b6d008c5014ee9856554b141ee953c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=772&h=192&s=30907&e=png&b=191919)

没有变化的包。

我们在 cli 和 core 的 index.ts 加个空行就好了。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0cdf865d6b5649af8f8f4f6d6b9566c5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=782&h=194&s=30883&e=png&b=191919)

按住空格来选择

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8633c0be71a145abb615be5fce81ebdf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1210&h=578&s=159048&e=png&b=191919)

选择更改 minor 版本号。

在 .changeset 下多了一个临时文件记录着这次变更的信息：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/609c671325ec47ec8ab00535e4fc22c0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1044&h=324&s=64604&e=png&b=1c1c1c)

然后执行 version 命令来生成最终的 CHANGELOG.md 还有更新版本信息：

```
npx changeset version
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df2c6f70da6a420f9129ba900743fc4a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1040&h=90&s=22662&e=png&b=191919)

更改的包下都多了 CHANGELOG.md 文件：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85bcab1ce8fa4f84b3ba899d8c1f0a36~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1320&h=558&s=136356&e=png&b=1e1e1e)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c4bb6479eb34b0c9059d3b242281ffc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1926&h=698&s=183748&e=png&b=202020) 并且都更新了版本号：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9f34086d09442839a60d7856dac1532~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1162&h=674&s=165866&e=png&b=1e1e1e)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c9391954ce445e38c20b2fcf5d142ed~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1282&h=1010&s=236778&e=png&b=1d1d1d)

@guang-npm/core、@guang-npm/cli 的版本，dependencies 里的版本号都变了。

然后发布到 npm 仓库：

```sql
git add .
git commit -m 'second commit'

npx changeset publish
```

执行 changeset publish 命令就可以，并且还会自动打 tag：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f950c22ec5d84c3ca3673f7a0390a757~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=966&h=620&s=156384&e=png&b=191919)

发布到了 npm，并在这个 commit 上打了两个 tag

去 [npm 网站](https://www.npmjs.com/~quark-gluon-plasma?activeTab=packages "https://www.npmjs.com/~quark-gluon-plasma?activeTab=packages")看一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff151f3b70284cd0b7dd9c549ffbb67d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2122&h=1290&s=258848&e=png&b=fefefe)

两个包都发布成功了。

测试下：

```sql
npx @guang-npm/cli minus 3 4
npx @guang-npm/cli add 3 4
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f581579078a47f2bcfdec79742d5240~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=550&h=186&s=43107&e=png&b=010101)

```dart
npm install -g @guang-pnpm/cli
num-calc
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75a99c867ce847abbb80d57fbdbc11fe~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=674&h=450&s=76824&e=png&b=010101)

没啥问题。

这样，基于 npm workspace + changeset 的 monorepo 项目就完成了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/npm-monorepo-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/npm-monorepo-test")

## 总结

这节我们用 npm workspace + changeset 实现了 monorepo 形式的 npm 包的开发、构建、发布、版本变更。

在 packages.json 的 workspaces 里配置工作区。

然后我们用到了这些命令：

在某个 workspace 下安装依赖：

```css
npm install core --workspace cli
```

在某些包下执行命令：

```bash
npm exec --workspace @guang-npm/cli -- npx tsc --init
```

在全部包下执行命令

```css
npm exec --workspaces -- npx tsc --init
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

我们这节用 npm changeset + workspace 来实现了前面两节的 cli。

虽然都差不多，但是在一些具体命令上不同，我们都做一遍之后，以后用到就可以回头来查了。