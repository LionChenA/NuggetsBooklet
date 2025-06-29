上节用了一遍 yarn workspace + changeset 来做 cli 的多包管理。

这节用一下 pnpm workspace + changeset。

pnpm workspace 更好一点，但也差不多。

不过我们还是得用一遍，不能眼高手低。

创建个项目：

```bash
mkdir pnpm-monorepo-test

cd pnpm-monorepo-test

npm init -y
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e763db533dc465886e360317ca90ed8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=800&h=672&s=79887&e=png&b=010101)

进入项目，修改 package.json

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/482ddfd917364a2d83caf9379dbff4fd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=604&h=254&s=36854&e=png&b=202020)

加上 private: true，也就是不发布这个 package.json 的包到 npm 仓库。

添加 pnpm-workspace.yaml 配置文件：

```yaml
packages:
  - 'packages/*'
```

npm 和 yarn workspace 都是配置在 package.json 里的，而 pnpm workspace 是配置在 yaml 文件里的。

初始化两个包

```bash
 mkdir packages packages/core packages/cli
 
 cd packages/core
 
 npm init -y
 
 cd ../cli
 
 npm init -y
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d938d907a256454abdde0a64d2358ec4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1528&h=1064&s=177995&e=png&b=1c1c1c)

如果你没安装 pnpm 的话需要先安装下：

```
npm install -g pnpm
```

然后在 cli 包添加 core 包为依赖：

```css
pnpm --filter cli add core --workspace
```

\--filter 指定在哪个包下执行 add 命令

加上 --workspace 就是从本地查找

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd064554942546a299a077e99c147c7c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=726&h=504&s=63531&e=png&b=1f1f1f) 这里 workspace: 前缀就是本地的 workspace 的意思。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18c35bd2d1d442b28c7cf92d61c06a4b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1444&h=950&s=187788&e=png&b=1c1c1c)

可以看到在 packages/cli 包下的 node\_modules 多了一个 core 的软链。

yarn workspace 是放在根目录下，而 pnpm 是放在 workspace 目录下，都可以。

node 查找依赖会从当前目录一层层往上查找 node\_modules，所以都能找到。

可以看到，就是从 packages/core 软链过来的：

![2024-09-18 17.30.58.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58d734b652044c7db7f76af85809a359~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1794&h=850&s=556508&e=gif&f=40&b=fcfbfb)

我们改下包名，加上 scope：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23731e47b63d47d8bcc9d989c152e734~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=866&h=482&s=70675&e=png&b=1f1f1f)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91718e4b0ab4423ea484fb0ea7169465~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=894&h=566&s=82706&e=png&b=1f1f1f)

然后再 pnpm install

```
pnpm install
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cbd4524908d44a09a2485a372b0ac34~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1428&h=554&s=117473&e=png&b=1d1d1d)

可以看到，node\_modules 下的依赖也更新了。

然后我们安装 typescript：

```sql
pnpm add typescript @types/node -w --save-dev
```

加上 -w 才能在根目录安装依赖：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63942aea5fa14628b8022e13722e775c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1932&h=158&s=55499&e=png&b=181818)

然后在 cli 包下创建 tsconfig.json

```css
pnpm --filter cli exec npx tsc --init
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89913498fc3f423395eb7d6bf0619f3a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=792&h=386&s=48169&e=png&b=181818)

pnpm exec 来执行命令，用 --filter 来指定在哪些包执行命令。

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

修改 package.json 的 type 为 module

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/813de7ed0a1744f1999c71158e3d6d03~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=650&h=322&s=48927&e=png&b=1f1f1f)

并且注册下代码入口 main 和 ts 类型 types

```json
"type": "module",
"main": "dist/index.js",
"types": "dist/index.d.ts",
```

同样的方式也改下 core 包的：

```css
pnpm --filter core exec npx tsc --init
```

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
    "skipLibCheck": true
  }
}
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac564e10b9ef449d96a747cdadeab33e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=582&h=262&s=41798&e=png&b=202020)

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

```lua
pnpm --filter core exec npx tsc
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6dc5f512f0c4e70b2fdf1c892c78fd0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1478&h=550&s=96494&e=png&b=1d1d1d)

然后改一下 cli 包：

添加依赖：

```lua
pnpm --filter cli add chalk commander
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/438acbfc44c843a6a739803a278d43ef~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=908&h=688&s=106907&e=png&b=1f1f1f)

创建 src/index.ts

```javascript
#!/usr/bin/env node
import { Command } from 'commander';
import chalk from "chalk";
import { add, minus } from '@guang-pnpm/core';

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

```bash
pnpm -r exec npx tsc
```

\-r 是递归的意思：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bd2ff7a90b442468fd350126a3eb07a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=666&h=196&s=23190&e=png&b=ffffff)

也就是在每个包下执行命令，特定的包下执行命令才需要 --filter

而且 pnpm workspace 比 yarn workspace 好的是支持拓扑顺序执行命令：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49747fd7d6f040b2a7d8c13dd8bd2dea~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1368&h=554&s=65878&e=png&b=ffffff)

有时候命令有执行先后顺序的要求的时候就可以用这个。

不过这里用不到。

然后跑一下：

```bash
pnpm --filter cli exec node ./dist/index.js add 1 2
pnpm --filter cli exec node ./dist/index.js minus 1 2
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4316c97c52048709ed0a64b361d9233~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=880&h=162&s=28429&e=png&b=181818)

没啥问题。

在 package.json 配置下 bin：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46efcb6b8fda4a7691fadedc2ba88305~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=776&h=402&s=62050&e=png&b=202020)

```json
"bin": {
    "num-cli": "./dist/index.js"
},
```

接下来就可以把它发布到 npm 了。

接下来的版本更新、npm 包的发布，用 changeset 来做。

我们先在 npm 增加一个组织：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23774a8c9e594b04969c958a53dfe919~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=818&h=884&s=72604&e=png)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84c676728e2642a1bfe69203ac6e71c1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1696&h=1344&s=158127&e=png&b=fbfbfb)

看下创建的组织：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/987c5050fc874c5a99b260b7ad6c5b4c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2564&h=848&s=105605&e=png&b=fcfbfb)

然后我们就可以在 @guang-pnpm 这个 scope 下发包了。

不过还需要在 cli 和 core 都加一下 publishConfig 配置为公有的：

```json
"publishConfig": {
    "access": "public"
},
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/988baf40deb8440f894c4e55a3493710~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=666&h=486&s=64334&e=png&b=1f1f1f)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/238c6a3d83674424b96c6a03436d0207~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=528&h=348&s=49839&e=png&b=202020)

登录下 npm

```
npm adduser
```

浏览器登录过 npm，会提示你打开浏览器，授权之后就登录成功了：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dea6fb2bad44461cad9374d97d7fdd0d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1204&h=212&s=44162&e=png&b=181818)

然后安装 changeset，执行 changeset init：

```csharp
pnpm add --save-dev -w @changesets/cli prettier-plugin-organize-imports prettier-plugin-packagejson

npx changeset init
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d40b721b5744a0bbab45e8c6fcba5a3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=886&h=402&s=65556&e=png&b=181818)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad5643d18be148b1b12efc2c69c321b0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1204&h=258&s=68507&e=png&b=181818)

会多一个 .changeset 目录：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60bb542348c9448da6d7fc9216a2206a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=646&h=224&s=26914&e=png&b=191919)

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

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d8266d946a4432b96514fe27211325b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1296&h=562&s=155584&e=png&b=181818)

同样，我们改 minor 版本号。

在 .changeset 下多了一个临时文件记录着这次变更的信息：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f75ced181fa44c68dfd8aa93989b4bf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1180&h=320&s=63793&e=png&b=1d1d1d)

然后执行 version 命令来生成最终的 CHANGELOG.md 还有更新版本信息：

```
npx changeset version
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df2c6f70da6a420f9129ba900743fc4a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1040&h=90&s=22662&e=png&b=191919)

更改的包下都多了 CHANGELOG.md 文件：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a379b36d544c4c1b83c260a528180c56~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2352&h=668&s=218968&e=png&b=1f1f1f)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/662b1c74567c41dd8d9769c31fa7e365~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2284&h=562&s=166018&e=png&b=202020)

并且都更新了版本号：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6372c44db1844deea5589b017f1ed915~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1460&h=500&s=151772&e=png&b=1e1e1e)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cca9e40e0d3457e9f75cf00fdf7a822~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1426&h=546&s=165962&e=png&b=1e1e1e)

@guang-yarn/core、@guang-yarn/cli 的版本，dependencies 里的版本号都变了。

然后发布到 npm 仓库：

```sql
git add .
git commit -m 'second commit'

npx changeset publish
```

执行 changeset publish 命令就可以，并且还会自动打 tag：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eea13ef4e1d94cf6a500a222925d3eb1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1054&h=664&s=162381&e=png&b=191919)

发布到了 npm，并在这个 commit 上打了两个 tag

去 [npm 网站](https://www.npmjs.com/~quark-gluon-plasma?activeTab=packages "https://www.npmjs.com/~quark-gluon-plasma?activeTab=packages")看一下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2df9299ffc24648abb1a64f969d5f3f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2376&h=1336&s=253560&e=png&b=fefefe)

两个包都发布成功了。

测试下：

```sql
npx @guang-pnpm/cli minus 3 4
npx @guang-pnpm/cli add 3 4
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30288f1e660049d982c9b6ce24706905~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=600&h=296&s=45918&e=png&b=010101)

```dart
npm install -g @guang-pnpm/cli
num-cli
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8dcbb81a080b4f42af23e6113458de8c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=676&h=454&s=50511&e=png&b=010101)

没啥问题。

这样，基于 pnpm workspace + changeset 的 monorepo 项目就完成了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/pnpm-monorepo-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/pnpm-monorepo-test")

## 总结

这节我们用 pnpm workspace + changeset 实现了 monorepo 形式的 npm 包的开发、构建、发布、版本变更。

pnpm workspace 在 pnpm-workspace.yaml 下配置 packages

然后我们用到了这些命令：

安装本地的其他包为依赖：

```css
pnpm --filter cli add core --workspace
```

\--workspace 指定从本地查找包，--filter 指定哪些包下执行命令。

在某些包下安装依赖：

```csharp
pnpm --filter cli add chalk
```

在根目录安装依赖：

```sql
pnpm add typescript -w --save-dev
```

在某些包下执行命令：

```css
pnpm --filter cli exec npx tsc --init
```

在全部包下执行命令

```bash
pnpm -r exec npx tsc
```

还可以通过 --sort 指定拓扑顺序执行命令，这个是 yarn workspace 不支持的

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

我们这节用 pnpm changeset + workspace 来实现了一遍上节的 cli。

整体功能都差不多，就像前面说的，monorepo 的核心就是三点：

* 本地包的自动 link
* 在某些包下执行命令，最好支持拓扑顺序
* 版本变更、发布到 npm

用 pnpm workspace + changeset 能够完美实现这三点功能，相比 pnpm workspace 来说，更好一点。