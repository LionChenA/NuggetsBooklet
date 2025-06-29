这节我们来实现脚手架的 generate 命令。

不知道大家有没有用过 nest 的 cli。

我们先用一下：

```java
npm install -g @nestjs/cli
nest new hello-nest
```

![2024-11-22 11.40.47.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4f03fc030fb46348a458493516bc373~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=942&h=668&s=147306&e=gif&f=32&b=010101)

进入项目，把它跑起来：

```arduino
npm run start:dev
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdbdbb0e0e7d4376a7be02b979ef8959~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1912&h=414&s=145171&e=png&b=181818)

浏览器访问下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5eac3bc6e92441680230d8e2e1700a0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=696&h=182&s=19839&e=png&b=ffffff)

然后我们用下它的 generate 命令：

```
nest g resource aaa
```

![2024-11-22 11.51.31.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af2cfdce9084431fb0a7c3a32307c13f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1334&h=578&s=165127&e=gif&f=37&b=181818)

它就是选择生成的类型，然后就会创建这些文件，并且更新 AppModule

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e9e25bf189c4c3f8052c29d3661d1c0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1598&h=844&s=212743&e=png&b=1d1d1d)

aaa 目录下的文件就是简单的写入文件就行，而修改 AppModule 这个是基于 AST。

因为 AppModule 的格式是规定的，所以通过 AST 能精准修改代码。

安装下依赖，再次跑：

```css
npm install --save @nestjs/mapped-types

npm run start:dev
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a21c750d96e946b3b304ee32cbbc81b5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1882&h=604&s=274168&e=png&b=181818)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/105ebb480e6c48a4828ac1cd1a9b1b2a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=760&h=260&s=24565&e=png&b=ffffff)

这样用 generate 能直接生成 aaa 的 CRUD 代码。

就很方便。

这也是 generate 命令的意义。

我们也可以实现这样一个 generate 命令。

但只是生成组件的基本结构意义不大，我们完全可以结合 AI 来生成符合需求的代码。

我们来写一下，创建 generate 包：

```bash
mkdir packages/generate

cd packages/generate

npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fac61a073b04e3b93380683a0b19886~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1222&h=694&s=106319&e=png&b=181818)

创建 generate 包，然后改下 package.json

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6684e726c0934a78a6f05ab355e6c672~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=826&h=276&s=46102&e=png&b=1f1f1f)

```json
"name": "@guang-cli/generate",
"publishConfig": {
  "access": "public"
},
```

在 generate 包下创建 tsconfig.json

```css
pnpm --filter generate exec npx tsc --init
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
  },
  "include": [
    "src/**/*.ts"
  ]
}
```

修改 package.json 的 type 为 module

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f8106627c54459687c0bc885edfd11e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=718&h=412&s=55914&e=png&b=1f1f1f)

并且注册下代码入口 main 和 ts 类型 types

```json
"type": "module",
"main": "dist/index.js",
"types": "dist/index.d.ts",
```

然后开始写代码。

创建 src/index.ts

```javascript
import { select, input, confirm } from '@inquirer/prompts';
import os from 'node:os';
import path from 'node:path';

async function generate() {

    let componentDir = '';
    while(!componentDir) {
        componentDir = await input({ message: '生成组件的目录', default: 'src/components' });
    }

    let componentDesc = '';
    while(!componentDesc) {
        componentDesc = await input({ message: '组件描述（尽量详细一些）', default: '生成一个 Table 组件，有包含 name、age、email 属性的 data 数组参数' });
    }
    
    console.log(componentDir, componentDesc)
}

generate();

export default generate;
```

安装下依赖：

```css
pnpm --filter generate add @inquirer/prompts
```

跑一下:

```lua
pnpm --filter generate exec npx tsc -w

pnpm --filter generate exec node ./dist/index.js
```

npx tsc -w 会监听文件变化实时编译，就不用每次手动 npx tsc 编译了。

![2024-11-24 10.54.49.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3c896f61ea641869aac2ac137fcf2fe~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1496&h=612&s=149189&e=gif&f=50&b=171717)

然后我们引入 openai 来对接 AI 接口：

```csharp
pnpm --filter generate add openai
```

上节是用的 tools （也就是 function call）实现的代码提取，这次我们通过正则提取。

改下 src/index.ts

```javascript
import { select, input, confirm } from '@inquirer/prompts';
import os from 'node:os';
import path from 'node:path';
import OpenAI from 'openai';
import fs from 'node:fs';

const client = new OpenAI({
    apiKey: '你的 API KEY',
    baseURL: '你的代理商的 BASE URL'
});

const systemContent = `
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


async function generate() {

    let componentDir = '';
    while(!componentDir) {
        componentDir = await input({ message: '生成组件的目录', default: 'src/components' });
    }

    let componentDesc = '';
    while(!componentDesc) {
        componentDesc = await input({ message: '组件描述', default: '生成一个 Table 的 React 组件，有包含 name、age、email 属性的 data 数组参数' });
    }
    
    const res = await client.chat.completions.create({
        model: "gpt-4",
        messages: [
          {role: 'system', content: systemContent},
          {role: 'user', content: componentDesc}
        ]
    });
    
    console.log(res.choices[0].message.content || '')
}

generate();

export default generate;
```

在 system 的内容里指定返回内容的格式

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c4f441ad6e34e9fa634ef88f0c4287c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1192&h=690&s=132454&e=png&b=1f1f1f)

```css
每个文件之间通过这样的方式分隔：

# [目录名]/[文件名]

目录名是用户给出的组件名
```

跑一下：

```bash
pnpm --filter generate exec node ./dist/index.js
```

可以看到，返回的格式就是我们指定的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cbe4e72d982402ebe3675b5998171a9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1530&h=914&s=160036&e=png&b=181818)

这样解析起来不就简单了么？

我们在 [astexplorer.net](https://astexplorer.net/#/gist/0c4fdbe3859ed0a78bd1de0eccf6b662/af7caa7a59502a2ce66cee2832d4f67bccaaf9bf "https://astexplorer.net/#/gist/0c4fdbe3859ed0a78bd1de0eccf6b662/af7caa7a59502a2ce66cee2832d4f67bccaaf9bf") 里看下：

切换到 Marddownd 的 remark 编译器，可以看到 parse 出的 AST：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27e05e0f7e6540ecbffd56e7a9417c26~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2234&h=650&s=231976&e=png&b=f8f7f7)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b108c9a988104f998f070bc3847bc632~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2350&h=1282&s=440957&e=png&b=f8f7f7)

通过 AST 非常容易就能拿到文件名和文件内容。

我们安装下 remark 还有 fs-extra

```css
pnpm --filter generate add fs-extra remark      

pnpm --filter generate add --save-dev @types/fs-extra
```

调用 remark 来解析 markdown，拿到文件路径和内容，写入即可：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e384c43f0f41463c945afd8d36bd2dba~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1492&h=950&s=202898&e=png&b=1f1f1f)

这里要拼接上之前用户填入的组件目录，加上解析出的文件路径和文件内容，写入磁盘即可。

这里的 ast 结构就是从 astexplorer.net 上查看的：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af00ee40d608436c8c60eacc0643516b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1130&h=910&s=109112&e=png&b=efefef)

```javascript
import fse from 'fs-extra';
import {remark} from 'remark'
```

```javascript
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
```

跑一下：

```bash
pnpm --filter generate exec node ./dist/index.js
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0b09dc8a9e14b78a630e7e35f76633e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1984&h=618&s=209729&e=png&b=1b1b1b)

创建成功。

我们再加上 ora 来做下 loading：

```csharp
pnpm --filter generate add ora
```

![2024-11-24 19.15.15.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a31930305764a80b13aef4fb874111b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1718&h=646&s=164001&e=gif&f=70&b=181818)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54914d6f5caa426cb61b8a55f2d63d44~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1548&h=466&s=160231&e=png&b=1a1a1a)

这样，我们的 AI 版 generate 命令就跑通了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/guang-cli "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/guang-cli")

## 总结

这节我们实现了 AI 版的 generate 命令。

输入生成组件的目录、组件描述，就会调用 AI 来生成代码，然后写入组件目录。

我们通过在 system 里设置组件规范来保证返回的代码是符合规范的。

并且通过 system 设置规定了返回内容的格式，然后用 remark 通过 AST 解析出文件名和文件内容、写入磁盘。

这样，我们就可以通过 AI 生成符合规范的代码了。

当然，现在的很多内容都是写死的，比如 system 设置、API KEY、BASE URL,这些都应该是在配置文件里配置的，下节我们加上配置文件。