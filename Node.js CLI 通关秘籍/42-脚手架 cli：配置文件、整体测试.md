上节跑通了 AI 生成代码的流程，这节我们加上配置文件。

babel 的[配置方式](https://babel.dev/docs/config-files "https://babel.dev/docs/config-files")有很多种：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed7a5e5ced2c4531a568dbb960f051ba~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2186&h=826&s=230858&e=png&b=fefefe)

可以在 .babelrc、babel.config.json、babel.config.js 等文件里配置，还可以在 package.json 里配置。

如果我们也要支持这些配置方式，自己实现还是挺麻烦的。

但没必要自己实现，可以用 [cosmiconfig](https://www.npmjs.com/package/cosmiconfig "https://www.npmjs.com/package/cosmiconfig") 这个包。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14812902b49d428785d1c170283fc418~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1908&h=1002&s=242935&e=png&b=fefefe)

6000w+ 的周下载量，用的挺多的。

我们先测试下：

在 genereate 包下创建 test.ts

src/test.ts

```javascript
import { cosmiconfig, cosmiconfigSync } from 'cosmiconfig';
import path from 'node:path';

const explorer = cosmiconfig("xxx");

async function main() {
    const result = await explorer.search(path.join(import.meta.dirname, '../'));

    console.log(result?.config);

}

main();
```

安装下依赖：

```csharp
pnpm --filter generate add cosmiconfig
```

在 generate 下创建 xxx.config.js

```javascript
export default {
    apiKey: '111',
    baseUrl: '222',
    systemContent: 'xxx'    
}
```

跑一下：

```lua
pnpm --filter generate exec npx tsc -w 
pnpm --filter generate exec node ./dist/test.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77ec0d0228a44191a3ebd206b8bfc6a5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=986&h=114&s=27330&e=png&b=191919)

当然，你也可以删掉 xxx.config.js，在 package.json 里配置：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/514ea2ffc9a744fdacea793fc77a4814~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=864&h=732&s=103538&e=png&b=1f1f1f)

再跑下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b83d96171ea24c98bb67ff5afd1694af~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=980&h=106&s=22096&e=png&b=191919)

你还可以试下 .xxxrc、xxx.config.json 等方式，都是能读取出来的。

然后我们再导出个类型：

创建 src/configType.ts

```javascript
export interface ConfigOptions {
    apiKey: string;
    baseUrl: string;
    systemSetting: string;
}
```

然后再配置 xxx.config.js，可以用 jsdoc 的方式来定义配置的类型：

```javascript
/** @type { import('./dist/configType'.ConfigOptions)} */
export default {
    
}
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71d8e59e588a4a4bb67419ce15110eb9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1140&h=328&s=71879&e=png&b=202020)

这样就有类型提示了。

测试完之后，我们在代码里引入下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e049d02f521405bacae63a54fb46e4f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1224&h=1124&s=235024&e=png&b=1f1f1f)

只是把这几个值抽离的配置文件里，其他都不变。

```javascript
import { select, input, confirm } from '@inquirer/prompts';
import os from 'node:os';
import path from 'node:path';
import OpenAI from 'openai';
import fs from 'node:fs';
import fse from 'fs-extra';
import {remark} from 'remark'
import ora from 'ora';
import { cosmiconfig } from 'cosmiconfig';
import { ConfigOptions } from './configType.js';

async function generate() {
    const explorer = cosmiconfig("generate");

    const result = await explorer.search(process.cwd());

    if(!result?.config) {
        console.error('没找到配置文件 generate.config.js');
        process.exit(1);
    }
    
    const config: ConfigOptions = result.config;

    const client = new OpenAI({
        apiKey: config.apiKey,
        baseURL: config.baseUrl
    });
    
    const systemContent = config.systemSetting

    let componentDir = '';
    while(!componentDir) {
        componentDir = await input({ message: '生成组件的目录', default: 'src/components' });
    }

    let componentDesc = '';
    while(!componentDesc) {
        componentDesc = await input({ message: '组件描述', default: '生成一个 Table 的 React 组件，有包含 name、age、email 属性的 data 数组参数' });
    }
    
    const spinner = ora('AI 生成代码中...').start();

    const res = await client.chat.completions.create({
        model: "gpt-4",
        messages: [
          {role: 'system', content: systemContent},
          {role: 'user', content: componentDesc}
        ]
    });

    spinner.stop();

    const markdown = res.choices[0].message.content || '';
    
    await remark().use(function(...args) {
        return function(tree: any) {
            let curPath = '';
    
            for(let i = 0; i< tree.children.length; i++ ) {
                const node = tree.children[i];
                if(node.type === 'heading') {
                    curPath = path.join(componentDir, node.children[0].value);
                } else {
                    try {
                        fse.ensureFileSync(curPath);
                        fse.writeFileSync(curPath, node.value);
        
                        console.log('文件创建成功：', curPath)
                    } catch(e) {
                    }
                }
            }
    
        }
    }).process(markdown);
}

generate();

export default generate;
```

本地测试的话先在 generate 下创建 generate.config.js

```javascript
const systemSetting = `
# Role: 前端工程师

## Profile

- author: 神光
- language: 中文
- description: 你非常擅长写 React 组件

## Goals

- 根据用户需求生成组件代码

## Skills

- 熟练掌握 typescript

- 会写高质量的 React 组件

## Constraints

- 用到的组件来源于 antd

- 样式用 scss 写

## Workflows

根据用户描述生成的组件，规范如下：

组件包含 4 类文件:

    1、index.ts
    这个文件中的内容如下：
    export { default as [组件名] } from './[组件名]';
    export type { [组件名]Props } from './interface';

    2、interface.ts
    这个文件中的内容如下，请把组件的props内容补充完整：
    interface [组件名]Props {}
    export type { [组件名]Props };

    4、[组件名].tsx
    这个文件中存放组件的真正业务逻辑，不能编写内联样式，如果需要样式必须在 5、styles.ts 中编写样式再导出给本文件用

    5、styles.scss
    这个文件中必须用 scss 给组件写样式，导出提供给 4、[组件名].tsx

    每个文件之间通过这样的方式分隔：

    # [目录名]/[文件名]

    目录名是用户给出的组件名

## Initialization

作为前端工程师，你知道你的[Goals]，掌握技能[Skills]，记住[Constraints], 与用户对话，并按照[Workflows]进行回答，提供组件生成服务
`
/** @type { import('./dist/configType'.ConfigOptions)} */
export default {
    apiKey: '你的 API KEY',
    baseUrl: '你用的代理商的 BASE URL',
    systemSetting: systemSetting
}
```

跑一下：

![2024-11-25 19.53.39.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/effcd2b70b3846b7858940108b38dac4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1830&h=562&s=151749&e=gif&f=43&b=191919)

![2024-11-25 19.53.49.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d987c62f916942eb9b61d083477db90c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1830&h=562&s=552833&e=gif&f=70&b=191919)

没啥问题。

不过这个包不是直接调用的，把这行调用去掉：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e4e3566e3ea4c669caf8f89deca002c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=948&h=516&s=86891&e=png&b=1f1f1f)

然后我们在 cli 包里调用下：

```css
pnpm --filter cli add @guang-cli/generate --workspace
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fc20d42495b4eb1ac9651544b857b34~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=904&h=392&s=76091&e=png&b=1f1f1f)

首先安装这个包，然后添加个命令：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b33ebe5ced14470dbc824ff377415bfa~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1032&h=1190&s=230714&e=png&b=1f1f1f)

```javascript
program.command('generate')
    .description('生成组件（基于 AI）')
    .action(async () => {
        generate();
    });
```

跑一下：

```lua
pnpm --filter cli exec npx tsc
pnpm --filter cli exec node ./dist/index.js generate
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b9dd4d3d6464c6a9e33e2704e9c9e01~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1056&h=158&s=48271&e=png&b=181818)

通了。

这样，我们就可以发包了。

顺便我们改下项目模版，在 react 项目和 vue 项目里都内置一个配置文件。

template-react 里就把刚才的配置文件复制过来就行：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dea9bb26e7af4f66888df51172c9504a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1390&h=514&s=150813&e=png&b=1d1d1d)

template-vue 里也是，不过要改下 system 设置：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83f95178563942cb948c3a417b97ff52~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2014&h=872&s=309917&e=png&b=1e1e1e)

我对 vue 不熟，这里就不写内置的 system 配置了，大家可以自己根据公司代码规范来规定 AI 返回的代码格式。

然后发个包：

你可以先把本地代码 commit 一下，然后用 changeset add 发包：

```csharp
npx changeset add
```

发这四个包：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9fb8271b099d44aeb4c1b1ae5a928769~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1036&h=344&s=62231&e=png&b=181818)

generate 和 cli 改 minor 版本号，两个 template 包改 patch 版本号：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92e2925c2c314125b5ec863d5662bae6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1654&h=446&s=128904&e=png&b=181818)

在 .changeset 下多了这个临时文件：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0dd24d0e181466a8b291e2b9cedfcb2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1310&h=426&s=100447&e=png&b=1d1d1d)

然后生成版本号和 CHANGELOG.md

```
npx changeset version
```

可以看到，两个 template 都是改了 patch 版本号：

template-vue：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57ce1b2cacd2417f8491153db266d727~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2134&h=912&s=235228&e=png&b=1e1e1e)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b80c42a88e748368abd14f6e2efe70e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2196&h=958&s=308546&e=png&b=1f1f1f)

template-react：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/644d940ca4f24cf69d30fb6b07537cdf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2362&h=896&s=253996&e=png&b=1e1e1e)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c948d895b0984b079e7fe0191acd7944~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2256&h=886&s=315929&e=png&b=1f1f1f)

generate 和 cli 改的是 minor 版本号：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/692bf01fd62044478d7ae74aebe38173~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2074&h=782&s=258876&e=png&b=1e1e1e)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9178506328414621898a3070456fdffa~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2074&h=716&s=244936&e=png&b=1f1f1f)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fedadf241654e5cb5b3efc7be89fa02~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2226&h=572&s=182071&e=png&b=1e1e1e)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7c3423184af4879bdeb64eadcc0a65e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2266&h=630&s=289347&e=png&b=212121)

同时 cli 包依赖的 generate 包版本变化也会更新一个 patch 版本。

都没啥问题之后，就可以发包了：

```sql
git add .

git commit -m 'feat: 实现 generate 命令'

npx changeset publish
```

commit 一下是为了自动打 tag 的时候打在这个 commit 上。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8516badd4d4a49789415811ef78f33f8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1226&h=970&s=297869&e=png&b=191919)

发包成功！

我们试一下：

```css
npx @guang-cli/cli@latest
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db8f9d42aea94459ba31745e79b820f5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=650&h=564&s=92780&e=png&b=010101)

可以看到，多了这个命令。

我们用脚手架创建个 react 项目：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3a1770199264595914d41fe342e8371~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=792&h=516&s=182648&e=png&b=010101)

vscode 打开看看：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9f5de19f1004c13bde68a268e874e21~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1582&h=918&s=245274&e=png&b=1c1c1c)

是有 generate.config.js 配置文件的。

（其实这里 jsdoc 的路径应该改一下，改成 @guang-cli/generate 下的类型，不过不重要）

填入 apiKey 和 baseUrl

执行下 generate 命令：

```css
npx @guang-cli/cli@latest generate
```

![2024-11-25 20.36.19.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce34e7a610064185bf5e08883dd1537b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2002&h=902&s=759487&e=gif&f=61&b=191919)

我们跑一下：

```css
npm install
npm install --save antd
npm install --save-save sass
```

在 App.tsx 引入下：

```javascript
import { UserTable } from "./components/UserTable"

function App() {

  return (
    <UserTable data={[
      {
        name: 'guang',
        age: 20,
        email: 'guang@guang.com'
      },
      {
        name: 'guang222',
        age: 21,
        email: 'guang@guang.com'
      },
      {
        name: 'guang333',
        age: 23,
        email: 'guang22@guang.com'
      },
    ]}/>
  )
}

export default App
```

去掉 main.tsx 里的 index.css

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9a0cfd077d84d189c35f41e5b9575ee~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=906&h=422&s=75047&e=png&b=1f1f1f)

跑起来：

```arduino
npm run dev
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb3fb04680c14f75b8d2fff426bc53c6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=712&h=366&s=43999&e=png&b=181818)

浏览器看一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1ddae34c427431584750b5d4673750d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1794&h=626&s=64669&e=png&b=fefefe)

没啥问题。

这样，我们的 generate 命令就完成了。

相比 @nestjs/cli 的 generate 命令，用 AI 生成的代码显然更有价值一点。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/guang-cli "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/guang-cli")

## 总结

这节我们加上了配置文件，用 cosmiconfig，它支持从 js、json 等文件里读取配置。

然后改了在 cli 里引入了 generate 命令，用 changeset 发了 npm 包的新版本。

之后在本地整体测试了下 create、generate 命令，都没啥问题。

这样，我们的脚手架就开发完成了。