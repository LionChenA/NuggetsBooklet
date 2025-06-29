这节开始我们正式进入脚手架 cli 的开发。

流程上节分析过了：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb38b576082a4abf8f3983adbf0a580e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1872&h=974&s=127296&e=png&b=fefdfd)

我们用 monorepo 的形式管理，分为 template-vue、template-react 三个包，以及 cli、create 这几个包。

scope 就用 @guang-cli 吧（你创建的时候要自己换个名字）

登录 [npm 网站](https://www.npmjs.com/ "https://www.npmjs.com/")，创建组织：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50d774cec43b4664bc0429ec5d3ae04b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=618&h=726&s=53588&e=png&b=fcfcfc)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17f53d7113bc4b7ba32e3d99ad271194~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1236&h=1048&s=110293&e=png&b=fbfbfb)

然后我们创建下 monorepo：

```bash
mkdir guang-cli
cd guang-cli
npm init -y
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d63a2e49f0d5437888da5eae8bdceb11~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=850&h=658&s=125211&e=png&b=000000)

进入项目，修改 package.json

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e8ab4a869eb44efa0d12ce3ea1a52f2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=500&h=234&s=29969&e=png&b=1f1f1f)

加上 private: true，也就是不发布这个 package.json 的包到 npm 仓库。

添加 pnpm-workspace.yaml 配置文件：

```yaml
packages:
  - 'packages/*'
```

我们用 pnpm workspace + changeset 来做 monorepo

首先创建 template-react 和 template-vue 包，我们直接复制 create-vite 的项目模版过来（当然，用别的模版也可以）。

安装下 create-vite

```css
npm install --no-save create-vite
```

把 template-react-ts 和 template-vue-ts 复制出来，到项目的 packages 目录下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1b261f7d25546c8acc0e0de42eaaf42~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1256&h=1044&s=151623&e=png&b=1a1a1a)

分别放到 packages/template-vue/template 目录，packages/template-react/template 目录

把模版里的 \_gitignore 改为 .gitignore

然后在两个包下创建 package.json

```bash
cd packages/template-vue

npm init -y

cd ../template-react

npm init -y
```

改下 package.json

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb94763fd2524e5daf61a91cd284c78e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=852&h=352&s=56981&e=png&b=202020)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3fb7dbb311e34b7fa3d3e96682e3c456~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=800&h=332&s=52580&e=png&b=1f1f1f)

name 加上 scope，并且加上 publishConfig 指定这个是公开访问的包。

```json
"publishConfig": {
    "access": "public"
},
```

然后用 changeset 发到 npm 仓库：

登录下 npm

```
npm adduser
```

浏览器登录过 npm，会提示你打开浏览器，授权之后就登录成功了：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dea6fb2bad44461cad9374d97d7fdd0d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1204&h=212&s=44162&e=png&b=181818)

然后回到根目录，安装 changeset，执行 changeset init：

```csharp
pnpm add --save-dev -w @changesets/cli prettier-plugin-organize-imports prettier-plugin-packagejson

npx changeset init
```

\-w 是在根目录安装依赖

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d40b721b5744a0bbab45e8c6fcba5a3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=886&h=402&s=65556&e=png&b=181818)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad5643d18be148b1b12efc2c69c321b0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1204&h=258&s=68507&e=png&b=181818)

会多一个 .changeset 目录：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9ba17534a2c4a3aad3e32d66ddeb791~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=574&h=366&s=37865&e=png&b=181818)

changeset 会基于 git 来判断代码有没有变动。

我们初始化下 git

先在根目录创建 .gitignore

```
node_modules/
dist/
.DS_Store
```

执行 init

```csharp
git init

git add .

git commit -m '初始化项目，创建 template-react template-vue'
```

changeset 会基于上次的 commit 来判断变更，所以我们先创建一个 commit，然后再做下改动：

分别在 template-vue、template-react 的 package.json 里加个换行。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39239f981163421bb11a711b787e99fa~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1682&h=696&s=174064&e=png&b=1c1c1c)

然后执行 npx changeset add

```csharp
npx changeset add
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1591343b382a4574b86179dfce1ab834~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1034&h=328&s=51510&e=png&b=191919)

按住空格来选择

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7daf24b958244cb6ada293dfed0cf9de~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1734&h=684&s=198029&e=png&b=181818)

我们改 minor 版本号

在 .changeset 下多了一个临时文件记录着这次变更的信息：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2e7ef9fac1643a79b5cfbd175f67032~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1724&h=376&s=86387&e=png&b=1c1c1c)

然后执行 version 命令来生成最终的 CHANGELOG.md 还有更新版本信息：

```
npx changeset version
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df2c6f70da6a420f9129ba900743fc4a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1040&h=90&s=22662&e=png&b=191919)

更改的包下都多了 CHANGELOG.md 文件：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2cf50f7a3334113a1334a8d2ae7d9c4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1740&h=594&s=168377&e=png&b=1e1e1e)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f11cb5d91eb1496fb5b6caa3c3266614~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1734&h=606&s=168251&e=png&b=1d1d1d)

并且都更新了版本号：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0dc006a248de4a9d91b8adcc71518c5b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1996&h=702&s=235019&e=png&b=1e1e1e)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/115401160101431fb0c4b191f281832e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1914&h=668&s=217687&e=png&b=1d1d1d)

可以看到，变的是我们选择的 minor 版本号。

然后创建一个 commit，发布到 npm 仓库：

```sql
git add .
git commit -m '项目模版 1.1.0'

npx changeset publish
```

创建 commit 是因为 changeset 会给最新的 commit 打 tag。

执行 changeset publish 命令：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b8b8b93c5d54766999c12715fda2f38~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1360&h=820&s=238915&e=png&b=191919)

发布到了 npm，并在这个 commit 上打了两个 tag

去 [npm 网站](https://www.npmjs.com/~quark-gluon-plasma?activeTab=packages "https://www.npmjs.com/~quark-gluon-plasma?activeTab=packages")看一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11ccec8c541741cc879c0c2ade9400d1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1594&h=1190&s=205442&e=png&b=ffffff)

两个包都发布成功了。

[点进去](https://www.npmjs.com/package/@guang-cli/template-react?activeTab=code "https://www.npmjs.com/package/@guang-cli/template-react?activeTab=code")看下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0e808f355124499a1a02557efd95614~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1308&h=950&s=123530&e=png)

没啥问题。

接下来创建 cli 和 create 包：

```bash
mkdir packages/cli packages/create

cd packages/cli

npm init -y

cd ../create

npm init -y
```

改下名字，加上 publishConfig：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/872a44ce1ef544aab24223a13eb5c505~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=790&h=330&s=48910&e=png&b=1f1f1f)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb3b532d8aa14efb87441809ad9bb60a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=662&h=284&s=39591&e=png&b=1f1f1f)

```json
"publishConfig": {
    "access": "public"
},
```

回到根目录，在 cli 包添加 create 包为依赖：

```css
pnpm --filter cli add @guang-cli/create --workspace
```

\--filter 指定在 cli 包下执行 add 命令

加上 --workspace 就是从本地查找

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac51fbefcbf7496484cea9430d9ceb0a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1040&h=838&s=124146&e=png&b=1f1f1f)

这时 node\_modules 下的 @guang-cli/create 包就是从 packages 下软链过来的。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9579ff579c3042de9792f1ff26aec960~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=904&h=528&s=58865&e=png&b=191919)

可以看到，node\_modules 下的依赖也更新了。

然后我们安装 typescript：

```sql
pnpm add typescript @types/node -w --save-dev
```

加上 -w 才能在根目录安装依赖：

在 cli 包下创建 tsconfig.json

```css
pnpm --filter cli exec npx tsc --init
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89913498fc3f423395eb7d6bf0619f3a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=792&h=386&s=48169&e=png&b=181818)

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

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2be974001801418c80bbc8ed5f00b5cd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=814&h=486&s=74038&e=png&b=1f1f1f)

并且注册下代码入口 main 和 ts 类型 types

```json
"type": "module",
"main": "dist/index.js",
"types": "dist/index.d.ts",
```

同样的方式也改下 create 包的：

```css
pnpm --filter create exec npx tsc --init
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
    "skipLibCheck": true,
    "sourceMap": true
  }
}
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/495726195dce499483137cfd0fc344e2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=760&h=442&s=70367&e=png&b=1f1f1f)

然后在 create 包下创建 src/index.ts

```javascript
async function create() {
    console.log('create 命令执行中...')    
}

export default create;

```

编译下：

```lua
pnpm --filter create exec npx tsc
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a55a80e26f9f4da7b6d34cb76116595b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1634&h=778&s=226972&e=png&b=1c1c1c)

然后在 cli 包下也创建 src/index.ts

```javascript
#!/usr/bin/env node
import create from '@guang-cli/create';
import { Command } from 'commander';
import fse from 'fs-extra';
import path from 'node:path';

const pkgJson = fse.readJSONSync(path.join(import.meta.dirname, '../package.json'));

const program = new Command();

program
    .name('guang-cli')
    .description('脚手架 cli')
    .version(pkgJson.version);

program.command('create')
    .description('创建项目')
    .action(async () => {
        create();
    });

program.parse();
```

用 commander 解析命令行，注册 create 命令。

用 fs-extra 读取 package.json 里的 version

安装下这两个包：

```css
pnpm --filter cli add commander fs-extra

pnpm --filter cli add --save-dev @types/fs-extra
```

跑一下：

```lua
pnpm --filter cli exec npx tsc 

pnpm --filter cli exec node ./dist/index.js create
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c50ab9dd234345f98726dbda0c540899~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1034&h=88&s=25067&e=png&b=191919)

没啥问题。

这样，monorepo 的基本结构就搭建完成了。

改下 cli 包的 package.json

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c82f0143bd84fc5b870759fdc8ec6db~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=860&h=602&s=88753&e=png)

版本号改为 0.0.1，加上 bin 的配置：

```json
"bin": {
    "guang-ci": "./dist/index.js"
},
```

create 包的版本号也改下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d00bbce4aff46afa17035463dc5c51a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=838&h=462&s=74634&e=png&b=202020)

然后把这两个包发到 npm 仓库：

```sql
git add .
git commit -m 'cli 包、create 包初始化'
```

执行 npx changeset add

```csharp
npx changeset add
```

选择 cli、create 两个包：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a643d065b9054edeb366c470d8a2a7e1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=930&h=340&s=59515&e=png&b=191919)

这里选择改 patch 版本号：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/833766c8960f46b7b45ad233004938db~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1474&h=764&s=211133&e=png&b=181818)

然后执行 npx changeset version 更新版本

```
npx changeset version
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbf066330c0841d3be632469dd6265c1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1662&h=552&s=189983&e=png&b=1d1d1d)

登录 npm：

```
npm adduser
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66216dd0b2774f86a7cfa00f1bde9840~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1404&h=234&s=52867&e=png&b=181818)

然后执行 npx changeset publish 发布 npm 包：

```sql
git add .
git commit -m 'cli create 0.0.2'

npx changeset publish
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6033eea9aca4a9498045da18dc4dc2a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1276&h=698&s=201094&e=png&b=191919)

在 npm 网站看下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bb471ac33bd43fba8ef401f9c2e45bc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1458&h=868&s=141624&e=png&b=ffffff)

然后用 npx 执行下：

```sql
npx @guang-cli/cli create
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78feaf7089014314aab0bf42a3f31ab9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=570&h=192&s=31634&e=png&b=010101)

没啥问题，流程跑通了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/guang-cli "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/guang-cli")

## 总结

这节我们搭建了 monorepo 的基本结构，用 pnpm workspace + changeset 的方案。

创建了 template-vue、template-react、cli、create 这 4 个包，并用 changeset 发到了 npm 仓库，创建了单独的 @guang-cli 的 scope。

安装了 commander 用来解析命令行，并且调用 create 包成功。

最后，我们本地 npx @guang-cli/cli create 试了下，整个流程是通的。

下节我们继续实现 create 命令的逻辑。