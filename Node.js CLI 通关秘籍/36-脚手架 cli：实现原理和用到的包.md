我们每天都在用脚手架创建项目：

比如你可以用 create-vite 创建 vite + react/vue 项目：

```lua
npx create-vite
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32529cda648c46688913ae90f88507e4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=838&h=448&s=50950&e=png&b=000000)

或者用 create-react-app 创建 webpack + react 项目：

```ini
npx create-react-app --template=typescript test-project2
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/389fb08d3e254bdeb74ca92b9fc8f1f1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1152&h=342&s=51246&e=png&b=000000)

或者用 @vue/cli 创建 vue 项目：

```sql
npm install -g @vue/cli
vue create vue-project
```

![2024-10-31 16.35.50.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d2b6dea92f545929c6f80e910820dd3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1284&h=664&s=81153&e=gif&f=60&b=020202)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ea0caf847ec4ae2bcc9725b791ed48f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=988&h=900&s=122490&e=png&b=000000)

既然有开源的脚手架，为啥还要自己写呢？

因为 create-vite、cra、@vue/cli 等脚手架创建的项目都只是基本代码，而我们实际项目开发会封装很多东西，需要在生成的代码基础上做很多修改。

每个项目都要从 0 到 1 做很多事情，当项目多了就很繁琐。

那为啥我们不把封装好的项目结构做成脚手架的模版，然后直接用这个模版创建项目呢？

这就是为啥需要自己实现脚手架。

大公司的基建基本都有自己的脚手架，封装一些项目模版，开箱即用。

前面我们实现过 create-vite，它是直接把模版放在本地的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c08185b0f0644b9e9fb56e1d353dd607~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=486&h=536&s=48468&e=png&b=191919)

这样不好单独管理模版的版本。

这次我们把每个 template 单独发包。

因为 npm 包是有版本历史功能的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7703f8607ebd45e396aa9861418f8794~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1304&h=1362&s=143419&e=png&b=fefefe)

而且通过 [registry.npmjs.org/@vue/cli](https://registry.npmjs.org/@vue/cli "https://registry.npmjs.org/@vue/cli") 可以拿到 npm 包的信息，比如 dist-tags.latest 可以拿到最新版本号：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe5279ff2fd1496cb3dc7867dff99d29~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1014&h=570&s=95428&e=png&b=fdfdfd)

[registry.npmjs.org/create-vite](https://registry.npmjs.org/create-vite "https://registry.npmjs.org/create-vite")

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/574ce1746f81458590ce11a3b9e646c4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=944&h=520&s=75635&e=png&b=fdfdfd)

所以用 npm 包的方式管理 template 自带版本功能。

相比放在项目本地目录，或者放在 github，通过 npm 包的方式能够独立管理版本，更好一些。

然后我们用 prompts 或者 inquirer 来做交互

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efa0d3f2124240d7abcdef205ee14ba4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=544&h=162&s=26610&e=png&b=010101)

选择一个模版后，从 npm 包下载模版，这里就直接用 npm install 下载就行。

把它下载到一个临时的缓存目录。

之后复制到目标目录即可。

也就是这样的流程：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb38b576082a4abf8f3983adbf0a580e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1872&h=974&s=127296&e=png&b=fefdfd)

原理比较简单，接下来我们把用到的一些包过一遍：

创建个项目：

```bash
mkdir scaffold-pkg-test
cd scaffold-pkg-test
npm init -y
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc7027271d0141308e0c6844ae3a4bd6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=890&h=688&s=138390&e=png&b=000000)

### fs-extra

首先是 fs-extra 这个包：

这个包是对 fs 的扩展，扩展了很多有用的方法。

比如复制目录，之前在 create-vite 实战的时候写过，这样写的：

创建 fs.js

```javascript
const fs = require('fs');
const path = require('path');

function copyDir(srcDir, destDir) {
    fs.mkdirSync(destDir, { recursive: true })
    for (const file of fs.readdirSync(srcDir)) {
      const srcFile = path.resolve(srcDir, file)
      const destFile = path.resolve(destDir, file)
      copy(srcFile, destFile)
    }
}

function copy(src, dest) {
    const stat = fs.statSync(src)
    if (stat.isDirectory()) {
        copyDir(src, dest)
    } else {
        fs.copyFileSync(src, dest)
    }
}

copyDir('./src', './aaa/bbb');
```

就是先创建目标目录，然后递归读取源目录，如果是文件，就 fs.copyFileSync 复制文件，如果是目录就继续递归复制。

跑一下：

```bash
node ./src/fs.js
```

![2024-11-01 09.56.58.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cf0f8aa1d674df38636021a465ba2b5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1632&h=1094&s=420897&e=gif&f=46&b=1a1a1a)

这样写起来还是有点麻烦的，明显可以再封装一下。

fs 其实有个 cpSync 方法，但直到 node 22，这个 api 才不再是实验性的：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb82b01eb79e4e8c8b464d3b8f0f8f94~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1372&h=666&s=90225&e=png&b=fefefe)

如果是低版本 node，还是不能用这个 api

而 fs-extra 就扩展了这个 api：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3682c4e54ad44d158e417ed4e5802ea5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=784&h=354&s=49028&e=png&b=202020)

安装下：

```css
npm install --save fs-extra
```

创建 src/fs-extra.js

```javascript
const fse = require('fs-extra');

fse.copySync('./src', './aaa/bbb/')
```

![2024-11-01 10.01.09.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4033dff39c4942fdbeffceda7b91f5a5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1632&h=1094&s=285768&e=gif&f=37&b=181818)

一个 api 搞定，是不是比原生的写法简单多了？

类似这样的 [api](https://www.npmjs.com/package/fs-extra "https://www.npmjs.com/package/fs-extra") 还有好几个：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94f86182a3fa4edc93e1b86c948c0401~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=542&h=830&s=77061&e=png&b=fffefe)

比如直接读写 json，这个很常用吧，fs 就没有，需要自己读取之后 JSON.parse 一下，用 fs-extra 就可以直接用。

这几个扩展的 api 都挺好用，所以 fs-extra 这个包用的挺多的:

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ac0beb2a7e345f29c045b598d979224~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2018&h=1140&s=243703&e=png&b=fefefe)

有 9000 万的周下载量呢。

### glob

这个包可以根据通配符匹配某个目录下的文件。

比如 \*\* 匹配所有文件和目录，而 \*\*/\*.jpg 会匹配所有的 jpg 图片文件。

在 [npm 网站](https://www.npmjs.com/package/glob "https://www.npmjs.com/package/glob")上看下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c0e64f0685b443d9c70a2eebdc22bc4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1722&h=1114&s=329526&e=png&b=fefefe)

好家伙，1.6 亿的周下载量。

我们安装下：

```css
npm install --save glob
```

创建 src/glob.js

```javascript
const { glob } = require('glob');

async function main() {
    const files = await glob('**', {
        cwd: process.cwd(),
        ignore: 'node_modules/**'
    })
    console.log(files);
}

main();
```

匹配当前目录下除了 node\_modules 下的所有文件和目录。

跑一下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e89cdddbd8ed4cb7b5b7f7d2a316a1bb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=798&h=422&s=49856&e=png&b=181818)

通过 nodir:true 排除目录再跑下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17ad136f566d4319906b2b5f257a6f80~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=826&h=776&s=103578&e=png&b=1c1c1c)

没啥问题。

这个包用起来很简单，但在匹配文件方面的功能是很强大的。

### inquirer

这个和我们之前用的 [prompts](https://www.npmjs.com/package/prompts "https://www.npmjs.com/package/prompts") 差不多：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2701e3a628214fcdb3f3fd2fb3248750~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=810&h=227&s=36999&e=png&b=1e2127)

我们也先用一下：

```css
npm install --save @inquirer/prompts
```

创建 src/inquirer.mjs

```javascript
import { input, select, password } from '@inquirer/prompts';

const name = await input({ message: '请输入你的名字' });

const job = await select({
    message: '选择你的职业',
    choices: [
      {
        name: '教师',
        value: '教师',
        description: '11111',
      },
      {
        name: '医生',
        value: '医生',
        description: '22222',
      }
    ],
});

const pass = await password({ message: '请输入密码' });

console.log({
    name,
    job,
    pass
})

```

这里我用到了 es module 的 import，并且用了顶层 await。

或者在 package.json 里声明 type 为 module，或者指定这个文件后缀为 .mjs

我们用的后面的方式

跑一下：

```bash
node ./src/inquirer.mjs 
```

![2024-11-01 16.54.31.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41feef0792ee4ae1b08eef90bafa0e5e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1210&h=722&s=119360&e=gif&f=55&b=181818)

和我们之前用 prompts 差不多。

### semver

semver 是 semantic version 语义化版本号的意思，是 npm 包的版本号的规范。

规范是这样的：

1.2.3 分别是 major 主版本号、minor 次版本号、patch 修订版本号

* major：当你做的不兼容的 api 修改时，改这个版本号
* minor：当你做了向下兼容的功能新增时，改这个版本号
* patch：当你做了向下兼容的问题修复时，改这个版本号

而 semver 这个包就是可以根据这个规范来判断 npm 包的版本号是否有效，以及比较大小等。

3.5 亿 的[周下载量](https://www.npmjs.com/package/semver "https://www.npmjs.com/package/semver")：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83d358374be5495097cb6af6fb65a012~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1930&h=1062&s=239598&e=png&b=fcfcfc)

我们用一下：

```css
npm install --save semver
```

创建 src/semver.js

```javascript
const semver = require('semver');

if(semver.valid('1.2.3#')) {
    console.log('版本号有效');
} else {
    console.log('版本号无效');
}

if(semver.gt('2.0.0', '1.0.8')) {
    console.log('有新版本可以安装')
}

if(semver.lte(process.version, '22.0.0')) {
    console.log(`node 版本 ${process.version} 小于 22`)
}
```

跑一下：

```bash
node ./src/semver.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3250efc099b143c7b3b6d78038711018~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=532&h=168&s=28229&e=png&b=191919)

在判断 node 版本是否符合要求，是否有新版本可以安装等场景，都会用到这个包。

## npminstall

我们会从 npm 仓库下载模版到本地的临时目录，这时候就可以用 npminstall 这个包来下载：

```css
npm install --save npminstall
```

创建 src/npminstall.js

```javascript
const npminstall = require('npminstall');

(async () => {
  await npminstall({
    pkgs: [
        { name: 'chalk', version: 'latest' },
    ],
    root: process.cwd() + '/aaa',    
    registry: 'https://registry.npmjs.org',
  });
})().catch(err => {
  console.error(err);
});
```

安装 chalk 最新版本到 aaa 目录下。

跑一下：

```bash
node ./src/npminstall.js
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e74bb3adace748998ccfc6c0ad216a01~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1678&h=966&s=256027&e=png&b=1c1c1c)

可以看到，确实安装好了。

而且看到这个软链标记没，说明 npminstall 实现的是 pnpm 的安装逻辑。

## cli-spinner、ora

网络请求、文件写入等耗时逻辑需要展示一个 loading，而 cli-spinner 就是 cli 里的 loading。

安装下：

```css
npm install --save cli-spinner
```

创建 cli-spinner.js

```javascript
const Spinner = require('cli-spinner').Spinner

console.log('111');
console.log('222');

const spinner = new Spinner(`安装中.. %s`)
spinner.start()

setTimeout(() => {
    spinner.stop(true);
}, 3000);
```

跑一下：

![2024-11-01 18.03.58.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c6943d50b0140929be97f9fb1ac4f99~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1210&h=722&s=93857&e=gif&f=42&b=181818)

类似的还有一个 [ora](https://www.npmjs.com/package/ora "https://www.npmjs.com/package/ora") 的包，更常用一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d70ed12447bb47fdba12bf5828b65189~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1944&h=952&s=130016&e=png&b=fdfdfd)

安装下：

```css
npm install --save ora
```

创建 src/ora.js

```javascript
const ora = require('ora');

console.log(111);
console.log(222);

const spinner = ora('下载中...').start();

setTimeout(() => {
	spinner.color = 'yellow';
	spinner.text = '快了快了...';
}, 2000);

setTimeout(() => {
    spinner.stop();
}, 5000);
```

跑一下：

```bash
node ./src/ora.js
```

![2024-11-01 18.43.59.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f640085111ed4ab8afc8f184c979be5c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1210&h=722&s=68380&e=gif&f=53&b=181818)

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/scaffold-pkg-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/scaffold-pkg-test")

## 总结

我们会用 create-react-app、create-vite、@vue/cli 等脚手架创建项目，但他们创建的只是基础的项目结构。

很多时候，我们会把公司里封装的一些东西整合到项目里，然后封装成一个项目模版，这时候就需要自研脚手架了，大厂内都有这种基建。

我们分析了下脚手架 cli 的实现思路，项目模版通过 npm 包的方式来管理，这样可以方便的切换版本，然后 cli 里选择后下载模版到临时目录，然后复制到目标目录即可。

然后我们过了一遍写脚手架的时候会用到的包，包括 fs-extra、inquirer、glob、npminstall、semver、ora、cli-spinner 这些。

思路理清了，下节我们就正式来开发脚手架 cli。