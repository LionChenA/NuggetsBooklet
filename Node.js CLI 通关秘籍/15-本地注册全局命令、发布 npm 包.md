上节我们实现了基于 AST 的代码修改，是直接用 node 跑的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd7a798a35e842cd8fd8a604d3d33db5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=852&h=668&s=91613&e=png&b=1c1c1c)

而实际上 @nestjs/cli 是这样用的：

```java
npx @nestjs/cli g module aaa
```

![2024-09-06 08.41.38.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/488673915f234f8b8cbf911c352e8b6a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=335201&e=gif&f=49&b=191919)

```bash
npx @nestjs/cli g controller aaa
```

![2024-09-06 08.43.53.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/658b9fd098154c03ba37040cc3664330~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=314371&e=gif&f=28&b=191919)

我们需要进一步把它封装成命令。

可以注册为全局命令，这样到处都可以用。

那如何注册为全局命令呢？

有的同学可能会说，把它封装为 npm 包发到 npm 仓库，然后全局 npm install -g 安装就好了。

这样确实可以，但是本地测试的时候，代码还没完成，这时候发包不是很好。

可以用 npm link 来做。

我们先来写个 cli：

```perl
mkdir my-nest-cli
cd my-nest-cli
npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e22662621b814796af1534d3128aa35c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=818&h=666&s=125680&e=png&b=000000)

进入项目，安装 typescript：

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

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08cb6a641a1e4abebc57f1f202c7fb88~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=610&h=320&s=40513&e=png&b=1f1f1f)

创建 src/transform.ts

```javascript
import { PluginObj, transformFromAstSync } from '@babel/core';
import parser from '@babel/parser';
import template from '@babel/template';
import { isObjectExpression } from '@babel/types';
import prettier from 'prettier';
import { readFile } from 'node:fs/promises';

function myPlugin(): PluginObj {

    return {
        visitor: {
            Program(path) {
                let index = 0;
                
                while(path.node.body[index].type === 'ImportDeclaration') {
                    index ++;
                }

                const ast = template.statement("import { AaaController } from './aaa.controller';")()
                path.node.body.splice(index, 0, ast);
            },
            Decorator(path: any) {
                const decoratorName = path.node.expression.callee.name
                if(decoratorName !== 'Module') {
                    return;
                }

                const obj = path.node.expression.arguments[0];
                
                const controllers = obj.properties.find((item: any) => item.key.name === 'controllers');
                if (!controllers) {
                    const expression = template.expression('{controllers: [AaaController]}')();

                    if(isObjectExpression(expression)) {
                        obj.properties.push(expression.properties[0]);
                    }
                } else {
                    const property = template.expression('AaaController')();
                    controllers.value.elements.push(property);
                }
            }
        }
    }
}


export async function transformFile(filePath: string) {
    const sourceCode = await readFile(filePath);

    const ast = parser.parse(sourceCode, {
        sourceType: 'module',
        plugins: ["decorators"]
    });

    const res = transformFromAstSync(ast, sourceCode, {
        plugins: [ myPlugin ],
        retainLines: true
    });
    
    const formatedCode = await prettier.format(res?.code!, {
        filepath: filePath
    });

    return formatedCode;
}
```

其实就是把上节的 src/index.ts 复制了过来封装了个方法：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d4687ecc8a146c6a99608a89cee9366~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1162&h=902&s=169301&e=png&b=1f1f1f)

安装用到的包：

```lua
npm install --save @babel/core
npm install --save @babel/parser
npm install --save @babel/template
npm install --save @babel/types

npm install --save-dev @types/babel__core

npm install --save prettier
```

我们先来测试下：

nest-project/aaa.module.ts

```javascript
//@ts-ignore
import { Module } from '@nestjs/common';

@Module({})
export class AaaModule {}
```

这个文件不需要编辑，在 tsconfig.json 里排除下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44038d6d48544b999901a1f891360437~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=786&h=612&s=88997&e=png&b=1f1f1f) 创建 src/test.ts

```javascript
import path from "node:path";
import { transformFile } from "./transform.js";

(async function() {    
    const filePath = path.join(process.cwd(), './nest-project/aaa.module.ts');

    const code = await transformFile(filePath);
    console.log(code);
})();

```

跑一下：

```bash
npx tsc -w
node ./dist/test.js
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d05dfdab22fa42a091b781a8df62d17a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1668&h=716&s=178857&e=png&b=1b1b1b)

然后用 commander 封装个 cli

创建 src/cli.ts

```javascript
import { transformFile } from "./transform.js";
import { access, writeFile } from "node:fs/promises";
import path from "node:path";
import { Command } from 'commander';
import chalk from "chalk";

const program = new Command();

program
  .name('my-nest-cli')
  .description('自动添加 controller')
  .version('0.0.1');

program.command('transform')
  .description('修改 module 代码，添加 controller')
  .argument('path', '待转换的文件路径')
  .action(async (filePath: string) => {
    if(!filePath) {
        console.log(chalk.red('文件路径不能为空'))
    }

    const p = path.join(process.cwd(), filePath);

    try {
        await access(p);

        const formattedCode = await transformFile(filePath);
        writeFile(p, formattedCode);

        console.log(`${chalk.bgBlueBright('UPDATE')} ${filePath}`)
    } catch(e) {
        console.log(chalk.red('文件路径不存在'))
    }
  });

program.parse();

```

声明一个 transform 子命令，有一个 path 的参数，可以传入转换的文件路径。

通过 access 访问文件路径，如果抛异常，就说明文件路径不存在。

否则，转换文件后写入该路径。

安装下用到的包：

```css
npm install --save commander
npm install --save chalk
```

跑一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c0be5dcf75645609878df5ad2161750~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=708&h=132&s=26873&e=png&b=191919)

![2024-09-07 11.48.53.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8cbcf61d937644329acae8b1338e3b10~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=230202&e=gif&f=25&b=191919)

是不是有 @nestjs/cli 的感觉了？

![2024-09-06 08.43.53.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/658b9fd098154c03ba37040cc3664330~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=314371&e=gif&f=28&b=191919)

然后我们把它注册为全局命令：

package.json 添加 bin 字段，指定命令名和对应的文件路径：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4cdf7e961ce34ccdbdfb8c92f40e6b3a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=582&h=430&s=52779&e=png&b=202020)

```javascript
"bin": {
    "my-cli": "./dist/cli.js"
},
```

还要在文件开头加上这个：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af63cad902bb4a05982fe5bb45def68d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=928&h=514&s=99619&e=png&b=1f1f1f)

```javascript
#!/usr/bin/env node
```

这行代码是告诉 shell 用 node 去执行这个文件，就和我们 node ./dist/cli.js 一样。

然后在项目根目录执行 npm link

```bash
npm link
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/371b4dd3f9ea4c42bfbd4015c7cd1316~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=638&h=150&s=19002&e=png&b=181818)

之后神奇的事情发生了。

现在你在任何目录都可以执行 my-cli 了：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/723043412bf944d39d75c117896eea9e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1096&h=1104&s=224192&e=png&b=010101)

然后我们创建个 nest 项目：

```java
npx @nestjs/cli new test-nest-app
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1eb2abdeea7a466fbbd4cd84429b3a14~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=882&h=702&s=264385&e=png&b=010101)

然后进入项目，执行 my-cli transform

```css
my-cli transform ./src/app.module.ts
```

![2024-09-07 13.46.02.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebb1575ee1f2431a87164dcbf0ee56cf~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=367216&e=gif&f=35&b=1a1a1a)

可以看到，代码转换成功了。

然后用 nest cli 创建一个模块：

```sql
npx @nestjs/cli g module user
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17f9c4a281444ac290026c05a6ba988c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1596&h=904&s=251237&e=png&b=1c1c1c)

然后再用我们写的工具去修改下这个 module：

![2024-09-07 13.59.16.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/831ebb30f0244f3f8022d5cac4afc7e3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=250673&e=gif&f=40&b=1a1a1a)

```css
my-cli transform ./src/user/user.module.ts
```

可以看到，代码依然被正确的修改了。

这就是通过 AST 修改代码的魅力，可以精准的修改。

当然，通过 npm link 注册的全局包只是本地测试用。

最终还是要发到 npm 仓库，然后 npm install -g 来用的。

删除刚才注册的本地命令：

```perl
 npm uninstall -g my-nest-cli
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ae4cfd4dde746908be2fdc58fab461d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=452&h=80&s=18131&e=png&b=010101)

然后把这个包发到 npm 仓库：

改下 package.json

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aeed60072b8446948aa011621f6b2f04~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=742&h=606&s=77595&e=png&b=1f1f1f)

bin 是配置命令的入口，main 和 module 分别是 commonjs 和 es module 的入口。

files 是要发布到 npm 仓库的文件。

然后先登录下：

```
npm adduser
```

执行 npm adduser 命令，会让你输入用户名、密码、邮箱、验证码，然后就可以登陆了：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/326f90a67bdb41158dc75adfc490abf7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=988&h=244&s=55915&e=png&b=181818)

如果你还没账号，就先去 [www.npmjs.com/](https://www.npmjs.com/ "https://www.npmjs.com/") 注册一个。

执行 publish：

```
npm publish
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6649dfd66e6e4abba860499944719415~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=988&h=590&s=138263&e=png&b=161616)

这样我们就把他发到了 npm 仓库上。

（你发布的时候改一下 package.json 里的 name，不然会重名）

去搜一下：

[www.npmjs.com/package/my-…](https://www.npmjs.com/package/my-nest-cli "https://www.npmjs.com/package/my-nest-cli")

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53317b140fcb43bfbf27d2aa3c12b4ee~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2388&h=1232&s=195351&e=png&b=fdfdfd)

现在可以在 npm 仓库搜到了，还没有 README.md

我们加一下：

```markdown
# my nest cli

掘金小册 《Node.js 工具链通关秘籍》案例代码

第一步，全局安装这个命令：

npm install -g my-nest-cli

第二步，进入 Nest 项目目录，执行 my-cli transform

my-cli transform ./src/app.module.ts

```

改下 package.json 里的版本号，再次发布：

```
npm publish
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fba3bba2ec7c49948741826d7f1bf773~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1014&h=1114&s=216571&e=png&b=1a1a1a)

可以看到，README 有了，版本号也改了：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/333780dcbbce41e4b36212d8c5bd2b73~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2404&h=1402&s=246993&e=png&b=fefefe)

然后我们全局安装下：

```perl
npm install -g my-nest-cli
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39bbdcdb82bd4093b13effab905b0567~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=856&h=440&s=81210&e=png&b=010101)

然后再次执行 my-cli：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db476da3d0ce4cb4808072fe37e66dda~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=784&h=444&s=54848&e=png&b=010101)

现在就不是跑的本地那个了，而是从 npm 仓库下载的。

再试下：

![2024-09-07 15.50.44.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5a0b472f1d244cb8cb8b741109a3afe~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=237554&e=gif&f=22&b=191919)

功能正常。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/my-nest-cli "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/my-nest-cli")

## 总结

这节我们把上节实现的 AST 修改功能封装成了 cli，并 link 到全局在 Nest 项目里测试了一下。

和 nest cli 添加 controller 的功能一样。

然后把它发布到了 npm 仓库，全局安装这个命令，之后又测试了一遍。

我们开发一些 cli 命令的时候，就是这样的方式，本地用 npm link，然后发布 npm 仓库之后，就可以 npm install 来用了。