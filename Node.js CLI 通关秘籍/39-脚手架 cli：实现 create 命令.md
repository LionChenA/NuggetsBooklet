上节封装了 utils 包，这节继续来实现 create 命令。

按照之前分析的流程：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb38b576082a4abf8f3983adbf0a580e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1872&h=974&s=127296&e=png&b=fefdfd)

我们改下 create 包下的 index.ts

```javascript
import { select, input } from '@inquirer/prompts';

async function create() {

    const projectTemplate = await select({
        message: '请选择项目模版',
        choices: [
          {
            name: 'react 项目',
            value: '@guang-cli/template-react'
          },
          {
            name: 'vue 项目',
            value: '@guang-cli/template-vue'
          }
        ],
    });

    let projectName = '';
    while(!projectName) {
        projectName = await input({ message: '请输入项目名' });
    }

    console.log(projectTemplate, projectName);

}

create();

export default create;
```

首先用 inquirer 问两个问题。

为保证 projectName 不能为空，用 while 循环来问。

安装下 inquirer：

```css
pnpm --filter create add @inquirer/prompts
```

编译然后跑一下：

```lua
pnpm --filter create exec npx tsc
pnpm --filter create exec node ./dist/index.js
```

![2024-11-17 17.56.40.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e71a49010944859be9883b6cda86c5a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1106&h=568&s=94549&e=gif&f=61&b=161817)

然后根据下载选择的模版。

这里需要我们封装的 utils 包，安装下：

```css
pnpm --filter create add @guang-cli/utils --workspace
```

\--filter 指定在 create 包下执行 add 命令

加上 --workspace 就是从本地查找

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3230308cc7849f7975e897b90b1d5e5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=858&h=594&s=96705&e=png&b=1f1f1f)

这里我们把它下载到 home 目录的一个临时目录下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd36682bddfa4e25a0542e448a7bd186~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1222&h=750&s=152542&e=png&b=1f1f1f)

```javascript
import { select, input } from '@inquirer/prompts';
import os from 'node:os';
import { NpmPackage } from '@guang-cli/utils';
import path from 'node:path';
import ora from 'ora';

async function create() {

    const projectTemplate = await select({
        message: '请选择项目模版',
        choices: [
          {
            name: 'react 项目',
            value: '@guang-cli/template-react'
          },
          {
            name: 'vue 项目',
            value: '@guang-cli/template-vue'
          }
        ],
    });

    let projectName = '';
    while(!projectName) {
        projectName = await input({ message: '请输入项目名' });
    }

    // console.log(projectTemplate, projectName);

    const pkg = new NpmPackage({
        name: projectTemplate,
        targetPath: path.join(os.homedir(), '.guang-cli-template')
    });

    if (!await pkg.exists()) {
        const spinner = ora('下载模版中...').start();
        await pkg.install();
        await sleep(1000);
        spinner.stop();
    } else {
        const spinner = ora('更新模版中...').start();
        await pkg.update();
        await sleep(1000);
        spinner.stop();
    }
}

function sleep(timeout: number) {
    return new Promise((resolve => {
        setTimeout(resolve, timeout);
    }));
}

create();

export default create;

```

因为我本地下载比较快，我加了个 1s 的延迟，不然看不到下载过程。

安装用到的包：

```csharp
pnpm --filter create add ora
```

跑一下：

```lua
pnpm --filter create exec npx tsc
pnpm --filter create exec node ./dist/index.js
```

第一次是下载模版：

![2024-11-17 18.00.36.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31354730530949c6aa3c6b35a820d799~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1322&h=720&s=109815&e=gif&f=52&b=171717)

第二次就是更新模版了：

![2024-11-17 18.17.16.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34dba9537aec40f99e923ceb540bab99~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1340&h=630&s=141717&e=gif&f=52&b=171717)

然后我们把下载好的模版复制到目标目录：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f8a467b19ac4f2e813c345544db78ae~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1278&h=598&s=132981&e=png&b=1f1f1f)

```javascript
const spinner = ora('创建项目中...').start();
await sleep(1000);

const templatePath = path.join(pkg.npmFilePath, 'template');
const targetPath = path.join(process.cwd(), projectName);

fse.copySync(templatePath, targetPath);

spinner.stop();
```

直接用 fs-extra 包的 copySync 复制即可。

模版包都有个 template 目录，路径要拼接一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/302df369a13941368874e6e220bfb555~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=440&h=430&s=43302&e=png&b=181818)

跑一下：

```lua
pnpm --filter create exec npx tsc
pnpm --filter create exec node ./dist/index.js
```

![2024-11-17 18.27.08.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90afa4e16a8d4fe995e49ae96d5a5711~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1340&h=630&s=114067&e=gif&f=58&b=181818)

可以看到，项目创建成功：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e251084cebe4130a406b7b9150d7aee~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1372&h=772&s=202039&e=png&b=1a1a1a)

进入项目目录，跑一下：

```bash
cd ./packages/create/hello

npm install

npm run dev
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de4e871a55894ea1abed5ceacd31fd8a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=942&h=340&s=42991&e=png&b=181818)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57df124139d649a69bbc4da9f2e2dc24~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2370&h=1358&s=166743&e=png&b=ffffff)

我们用自己的脚手架、自己的项目模版，创建的项目成功跑起来了。

接下来再优化下细节就好了：

填完项目名之后，如果该目录不为空，我们可以让用户选择是否清空：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8a8a1f5ea0747378a42112d9631736a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1346&h=550&s=125558&e=png&b=1f1f1f)

```javascript
const targetPath = path.join(process.cwd(), projectName);

if(fse.existsSync(targetPath)) {
    const empty = await confirm({ message: '该目录不为空，是否清空'});
    if(empty) {
        fse.emptyDirSync(targetPath);
    } else {
        process.exit(0);
    }
}
```

因为现在项目下多一些别的目录，tsc 编译会报错

我们改下 tsc 编译的范围：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/762cc94bc4784f04bbcbb69c5cdbe5e1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=798&h=512&s=85275&e=png&b=1f1f1f)

```javascript
"include": [
    "src/**/*"
]
```

跑一下：

```lua
pnpm --filter create exec npx tsc
pnpm --filter create exec node ./dist/index.js
```

选择不清空：

![2024-11-18 08.23.55.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55517187b2b440408293ba80f96a7a48~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1340&h=630&s=94990&e=gif&f=57&b=171717)

选择清空：

![2024-11-18 08.24.22.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0590a675561049ecb833e7703c01747c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1340&h=630&s=148667&e=gif&f=70&b=171717)

清空了 hello 目录，并且创建了 vue 项目。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/823cd6da6b15492c9c7d3f3147e1faf6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=594&h=544&s=46204&e=png&b=191919)

跑下创建的项目看看：

```bash
cd ./packages/create/hello

npm install

npm run dev
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5bce21e1621748e3954865fc995922ce~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=854&h=318&s=41409&e=png&b=181818)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4315abcc1fd24d2892f72a75309e634c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1954&h=1374&s=202407&e=png&b=ffffff)

还有，现在 package.json 没改：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bbcdf2903fcc4712992200ba5901cc5d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1580&h=712&s=167529&e=png&b=1c1c1c)

我们可以创建好项目下手动修改下 package.json，但是如果很多处都要用到项目名，这样就比较麻烦了。

更好的方式是通过模版引擎渲染的方式。

我们用 ejs 模版引擎

安装下：

```css
pnpm --filter create add ejs glob
pnpm --filter create add --save-dev @types/ejs
```

顺便安装下 glob 用来查找文件。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c537d0c3e33341e7a2c7b7015cd6bf18~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1184&h=636&s=136609&e=png&b=1f1f1f)

依次渲染模版，并把渲染结果写到文件里。

```javascript
const files = await glob('**', {
    cwd: targetPath,
    nodir: true,
    ignore: 'node_modules/**'
})

for (let i = 0; i< files.length; i++) {
    const filePath = path.join(targetPath, files[i]);
    const renderResult = await ejs.renderFile(filePath, {
        projectName
    })
    fse.writeFileSync(filePath, renderResult);
}
```

现在 template 里并没有 ejs 模版的语法，我们改下 template-vue、template-react 的 package.json

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf275c7bbcb24d68a36c1666e3fcb4bb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1640&h=626&s=178609&e=png&b=1d1d1d)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/324d51b99ae640afb6c72573d55f98ca~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1644&h=550&s=165671&e=png&b=1d1d1d)

这里 <%= %> 就是 ejs 读取变量的语法。

然后发个新版本：

```csharp
npx changeset add
```

选择发这两个包：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/905f75be1bf74391bea7266a8fec9f09~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=926&h=322&s=54561&e=png&b=191919)

改下 minor 版本号：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54ac06a114e04ddb92139311475bbd60~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1206&h=276&s=73761&e=png&b=191919)

修改版本号，生成 changelog：

```
npx changeset version
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ae23d3bce354fa99e2120112031df5f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1254&h=104&s=31523&e=png&b=191919)

然后可以把这些改动创建个 commit，后面好打 tag：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11343adf39a141bc8b91734b3544a178~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1684&h=850&s=223026&e=png&b=1d1d1d)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74afbbc47884434eb3852b13227ced5c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1406&h=724&s=143746&e=png&b=1b1b1b)

```sql
git add .
git commit -m 'feat: 支持 ejs template'

npx changeset publish
```

用 changeset publish 发布

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d7158a16e3647cba2356ce4be301a2b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1276&h=746&s=218900&e=png&b=191919)

发布成功。

然后再跑下 create 命令：

```lua
pnpm --filter create exec npx tsc
pnpm --filter create exec node ./dist/index.js
```

![2024-11-18 09.23.31.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/764d2736b3234406b67be45b66748d14~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1340&h=630&s=126551&e=gif&f=67&b=161616)

可以看到，现在创建的项目里的 package.json 就是改了 projectName 之后的了：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/382e1cf1184e425f92ccad9039bc0bba~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1472&h=612&s=149663&e=png&b=1d1d1d)

此外，我们还可以把一些工具作为可选的，比如 eslint

如果需要 eslint，那就有这三部分：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc63b6751b994c70ad80c396006d3ab1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1512&h=842&s=249267&e=png&b=1d1d1d)

不需要的话，那就没有。

package.json 内容可以通过 ejs 的 if、else 来渲染。

而 eslint.congif.js 文件也是通过 if、else 来创建、删除。

我们改下 template-react 的模版代码：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5cfdf5e53be6400c8fb0e6c89369cd0e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1584&h=1016&s=291906&e=png&b=1e1e1e)

```javascript
<% if (eslint) { %>
"lint": "eslint .",
<% } %>
```

```javascript
<% if (eslint) { %>
"eslint": "^9.13.0",
"eslint-plugin-react-hooks": "^5.0.0",
"eslint-plugin-react-refresh": "^0.4.14",
<% } %>
```

用 ejs 的 if 把这两段代码包裹起来，只有当选择了启用 eslint 的时候才会渲染。

然后在 template-react 根目录加一个 questions.json

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18c09a394e9b4b30b0357938e90203ac~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1228&h=490&s=81659&e=png&b=1d1d1d)

也就是当询问是否启用 eslint 的时候，涉及到哪些文件。

```json
{
    "eslint": {
        "files": [
            "eslint.config.js"
        ]
    }
}
```

发个版本：

```csharp
npx changeset add
npx changeset version
npx changeset publish
```

还是改 minor 版本号：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e3a92def07645549cd383ef61b2af60~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1288&h=364&s=104706&e=png&b=191919)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7c3da2e21594efb810a1d5149488c5d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1046&h=564&s=165034&e=png&b=191919)

然后改下 create 命令：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a354ff721f304d52853228e5e5865b21~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1438&h=816&s=182899&e=png&b=1f1f1f)

读取 questions.json，根据询问结果来设置 renderData 和 deleteFiles。

之后用这些数据渲染，如果没开启 eslint 的话并删掉对应的文件：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91fb44821e25495d9c63c1cf2624bd4f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1394&h=832&s=167628&e=png&b=1f1f1f)

```javascript
import { select, input, confirm } from '@inquirer/prompts';
import os from 'node:os';
import { NpmPackage } from '@guang-cli/utils';
import path from 'node:path';
import ora from 'ora';
import fse from 'fs-extra';
import ejs from 'ejs';
import { glob } from 'glob';

async function create() {

    const projectTemplate = await select({
        message: '请选择项目模版',
        choices: [
          {
            name: 'react 项目',
            value: '@guang-cli/template-react'
          },
          {
            name: 'vue 项目',
            value: '@guang-cli/template-vue'
          }
        ],
    });

    let projectName = '';
    while(!projectName) {
        projectName = await input({ message: '请输入项目名' });
    }

    const targetPath = path.join(process.cwd(), projectName);

    if(fse.existsSync(targetPath)) {
        const empty = await confirm({ message: '该目录不为空，是否清空'});
        if(empty) {
            fse.emptyDirSync(targetPath);
        } else {
            process.exit(0);
        }
    }

    const pkg = new NpmPackage({
        name: projectTemplate,
        targetPath: path.join(os.homedir(), '.guang-cli-template')
    });

    if (!await pkg.exists()) {
        const spinner = ora('下载模版中...').start();
        await sleep(1000);
        await pkg.install();
        spinner.stop();
    } else {
        const spinner = ora('更新模版中...').start();
        await sleep(1000);
        await pkg.update();
        spinner.stop();
    }

    const spinner = ora('创建项目中...').start();
    await sleep(1000);

    const templatePath = path.join(pkg.npmFilePath, 'template');

    fse.copySync(templatePath, targetPath);

    spinner.stop();

    const renderData: Record<string, any> = { projectName };
    const deleteFiles: string[] = [];

    const questionConfigPath = path.join(pkg.npmFilePath, 'questions.json');

    if(fse.existsSync(questionConfigPath)) {
        const config = fse.readJSONSync(questionConfigPath);
        
        for (let key in config) {
            const res = await confirm({ message: '是否启用 ' + key});
            renderData[key] = res;

            if (!res) {
                deleteFiles.push(...config[key].files)
            }
        }
    }

    const files = await glob('**', {
        cwd: targetPath,
        nodir: true,
        ignore: 'node_modules/**'
    })

    for (let i = 0; i< files.length; i++) {
        const filePath = path.join(targetPath, files[i]);
        const renderResult = await ejs.renderFile(filePath, renderData)

        fse.writeFileSync(filePath, renderResult);
    }

    deleteFiles.forEach(item => {
        fse.removeSync(path.join(targetPath, item));
    })

}

function sleep(timeout: number) {
    return new Promise((resolve => {
        setTimeout(resolve, timeout);
    }));
}

create();

export default create;
```

跑一下：

```lua
pnpm --filter create exec npx tsc
pnpm --filter create exec node ./dist/index.js
```

当启用 eslint 时：

![2024-11-18 10.44.21.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9aa82f1ee14c4119af1eb0d14ff2491a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1340&h=630&s=201416&e=gif&f=70&b=181818)

可以看到 package.json 中有对应的依赖，并且也有 eslint.config.js

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bae0b5c490504002a00e57022562ee89~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1558&h=1026&s=260024&e=png&b=1d1d1d)

（这里多了一些空行，不过不影响）

当不启用 eslint 时：

![2024-11-18 10.46.16.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/387a30168e28409693fcebb49f54c28f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1340&h=630&s=163937&e=gif&f=70&b=181818)

就没有 eslint 的东西了：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cbe3b8b7efa4be590d705d01b87738b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1384&h=854&s=218177&e=png&b=1d1d1d)

当然，只是 template-react 有这些选项，template-vue 就没有：

![2024-11-18 10.47.59.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/024658a7b7a843caa9e04289e0c83d9a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1340&h=630&s=208689&e=gif&f=69&b=171717)

这样，我们的脚手架就完成了。

最后打印下创建成功的消息：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3844a847956345068164cc274711b705~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1330&h=1130&s=270589&e=png&b=1c1c1c)

```javascript
console.log(`项目创建成功： ${targetPath}`)
```

把这个调用删掉：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d74dd772e6214b269b60bc94ebc8de35~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=892&h=606&s=95248&e=png&b=1f1f1f)

我们还是通过 cli 来调。

先本地跑下：

```lua
pnpm --filter create exec npx tsc
pnpm --filter cli exec node ./dist/index.js
```

没啥问题：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09a6d055a85e4a09b408d26bb53b845f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1062&h=182&s=31529&e=png&b=181818)

然后我们把 create 包也发个新版本：

```sql
npx changeset add
npx changeset version

git add .
git commit -m 'feat: 实现 create 命令'
npx changeset publish
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3572c17cfc0474984d2bafd25caba45~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1266&h=224&s=75482&e=png&b=191919)

还是改 minor 版本号。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3f441845dae4405ab983cef7b372ac2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1150&h=686&s=196351&e=png&b=191919)

因为 cli 包依赖了 create 包，所以也会发一个 patch 版本。

测试下：

```sql
npx @guang-cli/cli@latest create
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5d21434f6f14115b048426bf2a744b3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=776&h=270&s=62664&e=png&b=010101)

![2024-11-18 11.29.22.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5bd73b64cc9d431da752118fad60db77~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=954&h=684&s=151155&e=gif&f=61&b=020202)

创建成功。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5db1f3300c384257b0bfd89afc220427~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1480&h=1108&s=246264&e=png&b=1d1d1d)

项目名改了，eslint 也启用了。

跑一下：

```arduino
npm install

npm run dev
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/050153c0054f4c2bb8c8e5adee479a75~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2296&h=1390&s=165465&e=png&b=ffffff)

也能正常跑起来。

这样，我们的脚手架就完成了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/guang-cli "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/guang-cli")

## 总结

这节我们实现了 create 命令。

用户选择一个模版，填入项目名后，从 npm 仓库下载对应的模版到 home 下的一个目录，然后复制模版到目标目录。

通过 ejs 模版引擎的渲染，来填充数据到 package.json 中的一些地方。

我们还支持了 eslint 等的是否启用，不启用的话就不渲染对应的内容，以及删除 eslint.config.js。

当公司项目比较多的时候，都会沉淀一些自己的项目模版，所以每个大公司都有这样的脚手架 cli 工具。