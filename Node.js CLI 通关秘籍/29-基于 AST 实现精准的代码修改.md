很多 Node.js 工具可以修改代码。

比如你用 @nestjs/cli 创建模块的时候：

首先创建个项目：

```java
npx @nestjs/cli new test-app
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b82c7ba68be74a95aacc68010835c6e4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=832&h=688&s=251054&e=png&b=010101)

创建 aaa 模块：

```java
npx @nestjs/cli g module aaa
```

![2024-09-06 08.41.38.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/488673915f234f8b8cbf911c352e8b6a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=335201&e=gif&f=49&b=191919)

注意到代码的变化了没

AppModule 多了两行代码： ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9aecfa86788042dda96a6b51b939b04d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=820&h=734&s=137271&e=png&b=1d1d1d)

插入的很精准。

再来试下：

创建一个 Controller

```bash
npx @nestjs/cli g controller aaa
```

![2024-09-06 08.43.53.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/658b9fd098154c03ba37040cc3664330~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=314371&e=gif&f=28&b=191919)

AaaModule 的代码也插入了两行代码：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d04e6ecee95f42219ed1abcb537eca0b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1000&h=730&s=107041&e=png&b=1d1d1d)

这么精准的代码修改可太方便了，不然我们要创建完手动在这里引入。

那这种工具是怎么实现精准的代码修改的呢？

答案就是 AST。

我们写的代码不管是 Node 还是网页的，都是需要编译后再跑。

编译的过程是这样的：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee9eaa1f265c4c49ad156f2c691748d9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=2386&h=666&s=209602&e=png&b=feffff)

源码首先会被解析成 AST（ Abstract Syntax Tree 抽象语法树），然后对 AST 做转换，也就是对这棵树的节点做增删改之后，递归打印，生成新的代码。

也就是 parse、transform、generate 这三个阶段。

我们只要在 transform 阶段对 AST 做一些自定义的修改，就能达到上面的效果。

我们来写一下：

```bash
mkdir ast-transform-test
cd ast-transform-test
npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9cc5e31fc7fb48ce9424fde66a6b0f3e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=844&h=674&s=135017&e=png&b=000000)

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

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/73a929b7fb1d494b89575f633d176374~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=578&h=286&s=38595&e=png&b=1f1f1f)

我们来实现这个自动插入 controllers 的效果：

![2024-09-06 08.43.53.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/658b9fd098154c03ba37040cc3664330~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=314371&e=gif&f=28&b=191919)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d04e6ecee95f42219ed1abcb537eca0b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1000&h=730&s=107041&e=png&b=1d1d1d)

写下 src/index.ts

```javascript
import { PluginObj, transformFromAstSync } from '@babel/core';
import parser from '@babel/parser';
import template from '@babel/template';
import { isObjectExpression } from '@babel/types';

const sourceCode = `
import { Module } from '@nestjs/common';

@Module({})
export class AaaModule {}
`;


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

const ast = parser.parse(sourceCode, {
    sourceType: 'module',
    plugins: ["decorators"]
});

const res = transformFromAstSync(ast, sourceCode, {
    plugins: [ myPlugin ]
});

console.log(res?.code);
```

首先用 parser.parse 把源码转为 AST：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03ee7582ddec49bcba34f41047fd1564~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=902&h=456&s=76620&e=png&b=1f1f1f)

设置 sourceType 为 module 就是按照 es module 来解析，这里要指定 decoratos 的装饰器 语法插件，不然解析不了装饰器。

然后调用 transformFromAstSync 对 AST 进行转换，这个过程中会调用 babel 插件。

之后就可以拿到生成的代码，然后打印。

目标是实现这个转换：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/349d2732c6ac4d8db7d055a8693feef9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=994&h=944&s=475123&e=png&b=1e1e1e)

插件写法如下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d30370037c714825b93982ddb5ff7549~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1126&h=878&s=143082&e=png&b=1f1f1f)

在 visitor 里声明要处理的节点，然后在回调函数里对节点做修改。

怎么知道修改啥节点呢？

用 [astexplorer.net](https://astexplorer.net/#/gist/ff98df80ff82521e7867662083bbf943/9cb1f591e6120615369241da70bc827af777f3be "https://astexplorer.net/#/gist/ff98df80ff82521e7867662083bbf943/9cb1f591e6120615369241da70bc827af777f3be") 看下就知道了：

设置下 babel parser，勾选 decorators 这两个选项：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5459fd698181485d8070b81a26ba30ce~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1708&h=1454&s=253420&e=png&b=fdfdfd)

这样就能 parse 刚才这段代码了：

```javascript
import { Module } from '@nestjs/common';

@Module({})
export class AaaModule {}
```

最外层是 Program 节点：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7befe3c19e154404b780e8e264699e37~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2348&h=1348&s=293241&e=png&b=f8f7f7)

import 语句对应 ImportDeclaration 节点：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a2bd7b8a19b4805bef0ec462a77d64f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2486&h=1090&s=274001&e=png&b=f9f0ee)

@Module 装饰器对应 Decorator 节点：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d8f72732087444dbcb9d60879335a46~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2420&h=1270&s=275760&e=png&b=faece6)

而我们要修改的就是这个：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a76a013bacfa4332b48baeca060dc79f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2528&h=1364&s=327592&e=png&b=faebe4)

一层层找到 @Module 里的 controllers 数组，向其中添加一个元素就好了。

对应的代码就是这样的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a706c599eaf4e96ac33698c3a0f2f93~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1508&h=496&s=89793&e=png&b=1f1f1f)

首先，从上到下找 ImportDeclaration，直到最后一个，然后在最后面插入一个 import 语句。

这里用 @babel/template 包的 api 来创建这个 ast。

然后就是在 controllers 数组插入一个元素。

这里层层找到 controllers 数组的 ast，找到之后在其中加入一个 AaaController 的 ast。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71bbab7ccf6a4239b66906957f816a0b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1562&h=862&s=184155&e=png&b=1f1f1f)

如果没有 controllers 数组，那就在对象里出入这个属性。

安装用到的包：

```scss
npm install --save @babel/core
npm install --save @babel/parser
npm install --save @babel/template
npm install --save @babel/types

npm install --save-dev @types/babel__core
```

测试下：

```bash
npx tsc -w

node ./dist/index.js
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97b073767a424e728f6be5509a12984b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=942&h=538&s=83049&e=png&b=1c1c1c)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55c3aaf555c24fb8b9b8d4ed84d7dee8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=842&h=616&s=84830&e=png&b=1d1d1d)

可以看到，代码被正确的修改了。

这就是基于 AST 来修改代码的好处，很精准。

我们用 @nestjs/cli 等工具生成代码的时候，为什么能那么精准呢？

![2024-09-06 08.41.38.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/488673915f234f8b8cbf911c352e8b6a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=335201&e=gif&f=49&b=191919)

![2024-09-06 08.43.53.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/658b9fd098154c03ba37040cc3664330~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=314371&e=gif&f=28&b=191919)

就是基于 AST 做的。

当然，现在我们的格式还不太对。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2899c73b9c7947b4b1cfa6f97d0c9c18~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=842&h=610&s=90391&e=png&b=1c1c1c)

转换完以后行数变了。

这时候加一个 retianLines 就好了：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10d225229ac84781b9d5610a1878ebe7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1284&h=614&s=95817&e=png&b=1c1c1c)

它会保留原本的行列号。

但这样格式依然不对，再用 prettier 格式化就好了。

安装 prettier：

```css
npm install --save prettier
```

用它格式化下编译后的代码：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e09d7ce3e48146e8b12fbbdccaac40ef~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1012&h=482&s=90145&e=png&b=1f1f1f)

这里指定文件名是为了让 prettier 自动推断用啥 parser。

```javascript
(async function() {
    const formatedCode = await prettier.format(res?.code!, {
        filepath: 'aaa.ts'
    });
    console.log(formatedCode);
})();
```

跑一下：

```bash
node ./dist/index.js
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/15de51f836dd407e8f57b7ff7a9a3d8d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1072&h=592&s=96528&e=png&b=1c1c1c)

这样格式就对了。

然后封装个 cli，就是我们每天在用的这种效果了：

```java
npx @nestjs/cli g module aaa
```

![2024-09-06 08.41.38.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/488673915f234f8b8cbf911c352e8b6a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=335201&e=gif&f=49&b=191919)

```bash
npx @nestjs/cli g controller aaa
```

![2024-09-06 08.43.53.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/658b9fd098154c03ba37040cc3664330~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1996&h=1034&s=314371&e=gif&f=28&b=191919)

最后，说明下为什么用了 any：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82b4e31309944fc4ba4d432e35743686~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1552&h=604&s=143727&e=png&b=1f1f1f)

因为 AST 一层套一层的，并不能直接确定下一层的类型，需要你每层做下判断才能确定类型

比如这样：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/434a632324ac422c9cea922096053932~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1226&h=240&s=57980&e=png&b=1f1f1f)

但这样太麻烦了，any 就可以直接一层层取了：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0134d9a30b9541feba74dbfedd4104ed~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2644&h=1404&s=290775&e=png&b=f9e9e2)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6514a0fbe3e74d01ba20083f03718aa7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1414&h=436&s=102357&e=png&b=1f1f1f)

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/ast-transform-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/ast-transform-test")

## 总结

这节我们实现了基于 AST 的精准代码修改。

我们用的很多 cli 为什么那么方便，可以精准的知道在哪里改？

就是基于 AST 做的。

我们基于 babel 插件实现了下 @nestjs/cli 生成 controller 时的代码修改功能。

然后用 prettier 做了下格式化。

如果你要做一个分析、修改代码的 CLI，那 AST 知识是必不可少的。