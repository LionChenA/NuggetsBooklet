我们每天都在用 create-vite：

![2024-09-04 09.50.58.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a489c84ddd154deb9c06c5b9b4efb926~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1254&h=808&s=93604&e=gif&f=55&b=020202)

而且也自己实现过它用的 prompts：

![2024-09-04 17.05.02.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62a074cce0ea492ea1641ef6f6cb0529~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=882&h=490&s=61385&e=gif&f=46&b=181818)

这节我们来实现下完整的 create-vite。

其实它的原理很简单：

首先，它有一堆的 template：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01938de7e46e4fa0a42441000af2739e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=760&h=974&s=98757&e=png&b=181818)

当你通过 prompts 选择了某个模版后，会根据模版路径读取所有的文件：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/334f96bbdd584f9fb26e71fd96aa776b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2032&h=1266&s=508729&e=png&b=202020)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b07f309d09c45f48a39d7d8eeef20fb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2568&h=1272&s=532188&e=png&b=202020)

然后依次写入目标目录（注意左侧的 bb 目录）：

![2024-09-27 09.27.55.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d2b9ba4775b4d5dbced48f1d8cfd068~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1980&h=764&s=3629640&e=gif&f=64&b=191919)

也就是说，create-vite 只是做了一个根据选择读取对应的模版，复制到目标目录的操作。

我们也来实现一遍。

```perl
mkdir my-create-vite
cd my-create-vite
npm init -y
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6390368c53942b28c520329717b901a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=822&h=682&s=131150&e=png&b=000000)

进入项目安装用到的包：

```css
npm install --save minimist prompts chalk
npm i --save-dev @types/minimist
npm i --save-dev @types/prompts
```

minimist 是用于命令行解析的，因为 create-vite 只有一个 --template 参数，比较简单，没必要用 commander

prompts 是用于用户输入和选择的

chalk 用于显示颜色

然后安装 typescript

```sql
npm install typescript  @types/node --save-dev
```

创建 tsconfig.json

```csharp
npx tsc --init
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c55f9fb2d59f49428da0cac6b8de4b95~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=692&h=366&s=43074&e=png&b=181818)

改一下：

```json
{
  "compilerOptions": {
    "outDir": "dist",
    "types": [ "node" ],
    "target": "es2016", 
    "module": "NodeNext", 
    "moduleResolution": "NodeNext",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
  }
}
```

在 package.json 设置 type 为 module：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ed141f59aca4d97be4a4925a56e25fd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=590&h=312&s=40849&e=png&b=1f1f1f)

然后写下 src/index.ts

```javascript
import minimist from 'minimist'

const argv = minimist<{
    template?: string
    help?: boolean
}>(process.argv.slice(2), {
    alias: { h: 'help', t: 'template' }
})

console.log(argv);
```

支持 help、template 两个选项，并且有别名 h 和 t

类型在 minimist 的泛型那里声明

编译，然后跑一下：

```bash
npx tsc -w

node ./dist/index.js
```

分别试下不同的选项：

```bash
node ./dist/index.js -h

node ./dist/index.js --help

node ./dist/index.js -t react-ts

node ./dist/index.js --template react-ts

node ./dist/index.js --template react-ts aaa

node ./dist/index.js --template react-ts 111
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5d51bc028c449a3ab3c0bbc276b72bd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=836&h=522&s=126960&e=png&b=181818)

可以看到，--help 和 --template 还有它们的别名 -h、-t 都生效了。

而参数放在 \_ 数组里。

当参数为 111 的时候，会解析为数字。

我们知道这个参数是目标目录，应该是字符串。

改一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/654f15faa96743f4892c24eb6fdeae62~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=920&h=758&s=102754&e=png&b=1e1e1e)

这样就好了。

minimist 不像 commander 会自动生成帮助信息，我们要自己打印。

写一下：

```javascript
import minimist from 'minimist'
import chalk from 'chalk'

const argv = minimist<{
    template?: string
    help?: boolean
}>(process.argv.slice(2), {
    alias: { h: 'help', t: 'template' },
    string: ['_']
})

const helpMessage = `\
Usage: create-vite [OPTION]... [DIRECTORY]

Create a new Vite project in JavaScript or TypeScript.
With no arguments, start the CLI in interactive mode.

Options:
  -t, --template NAME        use a specific template

Available templates:
${chalk.yellow    ('vanilla-ts     vanilla'  )}
${chalk.green     ('vue-ts         vue'      )}
${chalk.cyan      ('react-ts       react'    )}
${chalk.cyan      ('react-swc-ts   react-swc')}
${chalk.magenta   ('preact-ts      preact'   )}
${chalk.redBright ('lit-ts         lit'      )}
${chalk.red       ('svelte-ts      svelte'   )}
${chalk.blue      ('solid-ts       solid'    )}
${chalk.blueBright('qwik-ts        qwik'     )}`

async function init() {

    const help = argv.help
    if (help) {
        console.log(helpMessage)
        return
    }
}

init().catch(e => {
    console.error(e);
})
```

就是如果传入了 -h 选项，就打印帮助信息。

```bash
node ./dist/index.js -h
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e77f16e007d4e1f83537c8f3ab149e1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=870&h=560&s=66752&e=png&b=181818)

然后来处理下其他选项、参数：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3fb3578fa6df4dba98f88bca3bbbe37a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1034&h=774&s=142622&e=png&b=1f1f1f)

如果目录名后面带了 /，那就去掉它，比如 aaa/ 替换成 aaa

然后如果没有传入目标目录，就用默认的 vite-project

```javascript
function formatTargetDir(targetDir: string | undefined) {
    return targetDir?.trim().replace(/\/+$/g, '')
}

const defaultTargetDir = 'vite-project'

async function init() {
    const argTargetDir = formatTargetDir(argv._[0])
    const argTemplate = argv.template || argv.t

    const help = argv.help
    if (help) {
        console.log(helpMessage)
        return
    }

    let targetDir = argTargetDir || defaultTargetDir

    console.log(targetDir)
}
```

试一下：

```bash
node ./dist/index.js aaa/

node ./dist/index.js
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ca460a85cec4a9c9287e293dd2ee984~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=540&h=174&s=28588&e=png&b=181818)

默认目录、目录的路径处理，都没问题。

接下来用 prompts 来做提问：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6be965a88f8647d4bbf93ad5161a881c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1280&h=1198&s=184215&e=png&b=1f1f1f)

```javascript
let result: prompts.Answers<
    'projectName'
>

try {
    result = await prompts(
        [
            {
                type: argTargetDir ? null : 'text',
                name: 'projectName',
                message: chalk.reset('Project name:'),
                initial: defaultTargetDir,
                onState: (state) => {
                    targetDir = formatTargetDir(state.value) || defaultTargetDir
                }
            }
        ],
        {
            onCancel: () => {
                throw new Error(chalk.red('✖') + ' Operation cancelled')
            },
        },
    )
} catch (cancelled: any) {
    console.log(cancelled.message)
    return
}

console.log(result);
```

type 是指定类型，比如 text、select，当指定为 null 的时候就会忽略这个问题。

如果参数传入了 projectName，那自然就不用提问了，这时候就设置 type 为 null。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b94dc63911be4be8a7d854192b46d8e7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=534&h=164&s=16395&e=png&b=010101)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7eba334096b42b2b82f876869b63b88~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=522&h=294&s=29517&e=png&b=000000)

chalk.reset 是重置颜色的意思，不受之前设置的颜色的影响：

initial 是初始值

onState 是输入的值变化的时候，对输入的目录也做一下格式化，也就是 aaa/ 变成 aa，或者用默认的目录

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8512bb8453e4c0bab39377cdd63be3d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1212&h=836&s=123650&e=png&b=1f1f1f)

当用户通过 ctrl + c 取消的时候，抛一个 error，然后在 ctach 里打印下。

测试下：

![2024-10-10 09.23.53.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac47d23a46764806a9978e7a7bcb4c50~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1030&h=714&s=171536&e=gif&f=70&b=171717)

默认值、输入目录名、取消输入，都没问题。

并且当传入了参数的时候，不会显示这个问题：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40c6c4463e51477b8722d001a3150122~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=676&h=432&s=82683&e=png&b=181818)

然后我们加一下剩下的问题，也就是 framework、variant 的选择：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89abbfa190aa4acfafac562fa125b6dc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=590&h=356&s=59090&e=png&b=010101)

framework 有哪些、显示什么颜色，明显是要配置的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1820202724f43c7b81d069c8db41e42~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=584&h=426&s=53069&e=png&b=000000)

variant 也是：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99bacc831e8d4712870215b463c72717~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=680&h=334&s=64523&e=png&b=000000)

我们先加一下配置：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51ca140027994c5294ff01a347d3c5fa~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1204&h=1376&s=208443&e=png&b=1f1f1f)

```javascript
type Framework = {
    name: string
    display: string
    color: Function
    variants: FrameworkVariant[]
}

type FrameworkVariant = {
    name: string
    display: string
    color: Function
    customCommand?: string
}
  
const FRAMEWORKS: Framework[] = [
    {
        name: 'vue',
        display: 'Vue',
        color: chalk.green,
        variants: [
            {
                name: 'vue-ts',
                display: 'TypeScript',
                color: chalk.blue,
            },
            {
                name: 'vue',
                display: 'JavaScript',
                color: chalk.yellow,
            }
        ],
    },
    {
        name: 'react',
        display: 'React',
        color: chalk.cyan,
        variants: [
            {
                name: 'react-ts',
                display: 'TypeScript',
                color: chalk.blue,
            },
            {
                name: 'react-swc-ts',
                display: 'TypeScript + SWC',
                color: chalk.blue,
            },
            {
                name: 'react',
                display: 'JavaScript',
                color: chalk.yellow,
            },
            {
                name: 'react-swc',
                display: 'JavaScript + SWC',
                color: chalk.yellow,
            }
        ],
    }
]
  
const TEMPLATES = FRAMEWORKS.map((f) => {
    return f.variants?.map((v) => v.name)
}).reduce((a, b) => {
    return a.concat(b)
}, [])

console.log(FRAMEWORKS, TEMPLATES)
```

我们先写了下类型，配置 framework 的 name、display（现实的名字）、color（颜色函数）、variants

variants 数组里就是这个 framework 对应的那个数组

因为当你选择 react、vue 的时候，这个数组是不同的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e45d141b87544331926492463214557f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=506&h=336&s=60251&e=png&b=010101)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48f62a87fa0f4538befcdfa5360a8ad0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=482&h=300&s=56123&e=png&b=010101)

最后用 map、reduce 方法来把 varaints 数组取出来合并到一个数组里。

跑一下：

```bash
node ./dist/index.js
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a262aef628894be4ac8b988511e8d6a1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1090&h=674&s=106288&e=png&b=181818)

可以看到，这两个数组打印的都没问题。

然后去掉 console.log，我们继续写剩下的两个 prompts

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/beee9e69121f450dbd162a97387b88f8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1386&h=1068&s=204477&e=png&b=1f1f1f)

第一个 type 根据输入的参数是否在 template 数组里来决定显不显示。

第二个 type 是个函数，当 type 为函数的时候，参数为上个问题的答案，所以这里可以根据 framework.variants 是否存在来决定是否显示。

choices 部分就是分别指定 title、value 就好了，颜色用之前的配置。

```javascript
let result: prompts.Answers<
    'projectName' | 'framework' | 'variant'
>

try {
    result = await prompts(
        [
            {
                type: argTargetDir ? null : 'text',
                name: 'projectName',
                message: chalk.reset('Project name:'),
                initial: defaultTargetDir,
                onState: (state) => {
                    targetDir = formatTargetDir(state.value) || defaultTargetDir
                }
            },
            {
                type:
                  argTemplate && TEMPLATES.includes(argTemplate) ? null : 'select',
                name: 'framework',
                message: chalk.reset('Select a framework:'),
                initial: 0,
                choices: FRAMEWORKS.map((framework) => {
                  const frameworkColor = framework.color
                  return {
                    title: frameworkColor(framework.display || framework.name),
                    value: framework,
                  }
                }),
              },
              {
                type: (framework: Framework) =>
                  framework && framework.variants ? 'select' : null,
                name: 'variant',
                message: chalk.reset('Select a variant:'),
                choices: (framework: Framework) =>
                  framework.variants.map((variant) => {
                    const variantColor = variant.color
                    return {
                      title: variantColor(variant.display || variant.name),
                      value: variant.name,
                    }
                  }),
              },
        ],
        {
            onCancel: () => {
                throw new Error(chalk.red('✖') + ' Operation cancelled')
            },
        },
    )
} catch (cancelled: any) {
    console.log(cancelled.message)
    return
}
```

跑一下：

```bash
node ./dist/index.js

node ./dist/index.js --template react-ts
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89863d80d1a44bd59235ff405293fb43~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=626&h=266&s=25384&e=png&b=181818)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bf3fe456139420b8aafb3f8350ee5d2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=764&h=290&s=42543&e=png&b=181818)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6fe3526ff3db4533a0ff599cb7fa2d84~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=590&h=194&s=30264&e=png&b=191919)

可以看到，vue、react 模版的 variant 是不同的。

当输入 template 的时候，会跳过提问：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c69382d31f6a488aaa04ddaf7440493f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=704&h=128&s=21724&e=png&b=181818)

至此，知道了用什么模版，那就可以读取对应的模版写入目标目录了。

然后我们拼接一下 template 目录：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dedb8f6ba4a64ba4a77c56076fdb66fb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=946&h=794&s=144935&e=png&b=1f1f1f)

```javascript
const { framework, variant } = result

const root = path.join(process.cwd(), targetDir)

let template: string = variant || argTemplate

console.log(`
Scaffolding project in ${root}...`)

const templateDir = path.resolve(
    fileURLToPath(import.meta.url),
    '../..',
    `template-${template}`,
)

console.log(templateDir)
```

process.cwd() 是执行命令的目录，然后拼上 targetDir 就是目标目录。

根据 template 拼接要读取的 template 目录。

import.meta.url 就是当前文件的路径、不过是 file:/// 开头的，可以用 fileURLToPath 转为文件路径。

这个 api 是 url 包里的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aed09ff6180d4af29007f35c540cf5d4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=722&h=266&s=47528&e=png&b=1f1f1f)

```javascript
import { fileURLToPath } from 'node:url'
import fs from 'node:fs';
```

作用就是去掉前面的 file:///

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ca2d4c7e6cd4c0c92d7ea5e181e78b3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1520&h=594&s=130819&e=png&b=1c1c1c)

然后跑一下：

```bash
node ./dist/index.js
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd30b66a15cd481ebb6f04f4c6f71d33~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=940&h=446&s=90925&e=png&b=191919)

可以看到，选择 vue、react 不同的模版的时候，生成的模版路径都是对的。

接下来就是读取模版文件，写入目标目录了。

我们复制几个 [template 目录](https://github.com/vitejs/vite/tree/main/packages/create-vite "https://github.com/vitejs/vite/tree/main/packages/create-vite")过来：

可以 git clone 下代码来然后复制：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a971c454d4114a1c90435f8d338efe0f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1674&h=1420&s=257832&e=png&b=ffffff)

或者在本地安装这个包之后复制：

```css
npm install --no-save create-vite
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c3d5fcd8118497fbf07838a70276e5b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=590&h=858&s=84453&e=png&b=191919)

我们本地安装下 create-vite，然后把 template-react、template-react-ts、template-vue、template-vue-ts 复制出来。

有的同学可能会问，不是还有 template-react-swc 么，为什么没找到对应模版？

因为那个也是 template-react，只是 package.json 的依赖有些不同，后面我们单独处理。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e61aad56eba94597a7f6806b66c95359~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=588&h=500&s=48321&e=png&b=191919)

复制过来会导致 ts 报错，在 tsconfig.json 忽略这些目录就好了：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b037807eafa4bee95e8678ad8ce8015~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=836&h=634&s=89344&e=png&b=1f1f1f)

```json
"exclude": [
    "template-**"
]
```

模版复制过来之后，我们继续写代码：

```javascript
const renameFiles: Record<string, any> = {
    _gitignore: '.gitignore',
}

const write = (file: string, content?: string) => {
    const targetPath = path.join(root, renameFiles[file] ?? file)
    if (content) {
        fs.writeFileSync(targetPath, content)
    } else {
        copy(path.join(templateDir, file), targetPath)
    }
}

function copyDir(srcDir: string, destDir: string) {
    fs.mkdirSync(destDir, { recursive: true })
    for (const file of fs.readdirSync(srcDir)) {
      const srcFile = path.resolve(srcDir, file)
      const destFile = path.resolve(destDir, file)
      copy(srcFile, destFile)
    }
}

function copy(src: string, dest: string) {
    const stat = fs.statSync(src)
    if (stat.isDirectory()) {
        copyDir(src, dest)
    } else {
        fs.copyFileSync(src, dest)
    }
}
```

fs 模版有 copyFileSync 的方法，但没有 copyDir 的方法，所以我们要自己实现目录的复制。

write 方法如果传入了 content，就是用 fs.writeFileSync 写入文件。

否则就是复制，用 fs.stateSync 拿到这个路径是文件还是目录，如果是目录，就先递归创建这个目标目录，然后依次复制源目录下的文件。

然后用这个 write 方法来实现复制：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d7f2e0f01554db6b28268545c0a953d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1284&h=930&s=195646&e=png&b=1f1f1f)

```javascript
if (!fs.existsSync(root)) {
    fs.mkdirSync(root, { recursive: true })
}

const files = fs.readdirSync(templateDir)
for (const file of files) {
    write(file)
}

const cdProjectName = path.relative(process.cwd(), root)
    console.log(`
Done. Now run:
`)
if (root !== process.cwd()) {
    console.log(
        `  cd ${
        cdProjectName.includes(' ') ? `"${cdProjectName}"` : cdProjectName
        }`,
    )
}
console.log(`  npm install`)
console.log(`  npm run dev`)
console.log()
```

如果目标目录不存在，就用 mkdirSync 创建。

用 readdirSync 读取模版目录下的文件，依次写入目标目录。

最后打印一些信息。

path.relative 是拿到从 a 目录到 b 目录的相对路径。

跑下试试：

```bash
node ./dist/index.js
```

![2024-10-10 19.00.41.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de123a0372a54846a65241b555cbe720~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1618&h=1122&s=387010&e=gif&f=53&b=181818)

项目创建成功！

我们进入项目跑一下看看：

```arduino
cd aaa

npm install

npm run dev
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e202cb4653746bc84651eaeac5bbf12~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=764&h=286&s=36342&e=png&b=191919)

浏览器访问下：

![2024-10-10 19.02.39.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e231df25ff441d1bca4769b42f87acc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2156&h=1330&s=433931&e=gif&f=20&b=fefefe)

没啥问题。

这样我们就实现了 create-vite。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/my-create-vite "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/my-create-vite")

## 总结

这节我们实现了 create-vite。

它的逻辑很简单，就是根据用户输入选择对应的模版，然后复制模版文件到目标目录。

用户输入一个是命令行参数，通过 minimist 来解析。

另一个是交互式选择的，通过 prompts 来做提示。

然后复制对应模版目录下的文件到目标目录就好了。