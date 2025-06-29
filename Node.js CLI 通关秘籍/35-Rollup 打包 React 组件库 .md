上节学了 Rollup，其实不管是 react 还是 vue 组件库都会用 rollup 打包。

这节我们先来做下 react 组件库的打包。

我们先来写两个 react 组件：

```lua
npx create-vite react-comp
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c20534aea0164b1f8441a804b1ecf5d3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=786&h=500&s=59270&e=png&b=010101)

进入项目，改下 main.tsx

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ddb87813c0b940888f6c20c3246c37b2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1072&h=482&s=91263&e=png&b=1f1f1f)

去掉 index.css

创建 src/Button/index.tsx

```javascript
import { CSSProperties, MouseEventHandler, PropsWithChildren } from 'react'
import './index.scss';

export interface ButtonProps extends PropsWithChildren  {
    className?: string;
    style?: CSSProperties;
    type?: 'primary' | 'default';
    onClick?: MouseEventHandler
}

function Button(props: ButtonProps) {
    const {
        className = '',
        style,
        type = 'primary',
        children,
        onClick = () => {}
    } = props

    return <div 
        className={`btn ${className} btn-${type}`}
        style={{...style}}
        onClick={onClick}
    >{children}</div>
}

export default Button;
```

我们写了一个按钮的 React 组件。

有 className、style、type、onClick、children 这 5 个 props。

然后写下样式：

src/Button/index.scss

```scss
.btn {
    padding: 8px 10px;

    display: inline-block;
    border-radius: 4px;
    cursor: pointer;

    &-primary {
        background: skyblue;
        &:hover {
            background: lightblue;
        }
    }
    &-default {
        border: 1px solid skyblue;
        &:hover {
            color: skyblue;
        }
    }
}
```

用 scss 来写样式，分别写下 primary 和 default 的样式和 hover 时的样式。

在 App.tsx 里引入：

```javascript
import Button from "./Button"

function App() {

  return <div>
    <Button type='primary' style={{marginRight: '20px'}}>按钮一</Button>
    <Button type='default' onClick={() => alert(1)}>按钮二</Button>
  </div>
}

export default App
```

安装依赖：

```css
npm install
npm install --save sass
```

跑一下：

```arduino
npm run dev
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f0abc65a03a4603872734623992e9d5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=920&h=330&s=43869&e=png&b=191919)

![2024-12-28 11.05.31.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44fa159b0efc42d3b0e370f84192d597~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1974&h=1060&s=235165&e=gif&f=30&b=fefefe)

没啥问题。

在同一个项目里，引入写好的组件我们都会。

那如果想把这个组件封装成组件库，然后在多个项目里引入呢？

这时候就需要单独的构建过程了。

我们创建个组件库项目：

```bash
mkdir react-comp-lib
cd react-comp-lib
npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08a1a4e9e1b84323b663081d38789d62~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=852&h=674&s=86937&e=png&b=010101)

进入项目，把 Button 组件复制过来：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/948418ef902b4589beacdfdb0c840d92~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1266&h=542&s=119929&e=png&b=1c1c1c)

创建 src/index.ts 把组件和类型导出：

```javascript
import Button, { ButtonProps } from './Button';

export {
    Button
}

export type {
    ButtonProps
}
```

安装下用到的包：

```kotlin
npm install --save react@18 react-dom@18

npm install --save-dev  @types/react-dom
```

然后安装 rollup 来做打包：

```css
npm install --save-dev rollup
```

创建 rollup 配置文件：

rollup.config.mjs

```javascript
import postcss from 'rollup-plugin-postcss';
import typescript from '@rollup/plugin-typescript';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import replace from '@rollup/plugin-replace';

/** @type {import("rollup").RollupOptions} */
export default {
    input: 'src/index.ts',
    external: [ 'react', 'react-dom' ],
    output: [
        {
            file: 'dist/esm.js',
            format: 'esm'
        },
        {
            file: 'dist/cjs.js',
            format: "cjs"
        },
        {
            file: 'dist/umd.js',
            name: 'Guang',
            format: "umd",
            globals: {
                'react': 'React',
                'react-dom': 'ReactDOM'
            }
        }
    ],
    plugins: [
        resolve(),
        commonjs(),
        typescript({
            tsconfig: 'tsconfig.json'
        }),
        postcss({
            extract: true,
            extract: 'index.css'
        }),
        replace({
            'process.env.NODE_ENV': '"production"'
        })
    ]
};
```

input 指定入口模块。

output 指定产物的格式，分别打包 esm、cjs、umd 模块规范的产物。

其中 umd 格式里通过 name 指定全局变量的名字，通过 globals 指定引入的其他 umd 模块用什么变量名。

external 是指定不打包到产物里的 npm 包。

plugins 里的 typescript 插件是做 ts 代码编译的，postcss 插件是做 css 代码的编译和合并的。

[node-resolve 插件](https://www.npmjs.com/package/@rollup/plugin-node-resolve "https://www.npmjs.com/package/@rollup/plugin-node-resolve")是解析 node\_modules 下的包的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a547a97a85a4f8eb9bd86c483d64bd6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1240&h=260&s=49456&e=png&b=fefefe)

而 [commonjs 插件](https://www.npmjs.com/package/@rollup/plugin-commonjs "https://www.npmjs.com/package/@rollup/plugin-commonjs")是转换 commonjs 到 es module 的（因为 rollup 只支持 es module 模块）:

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44776e9204544af48704ef19cbbf1341~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1206&h=286&s=40539&e=png&b=fefefe)

这俩插件基本都是必用的。

创建 tsconfig.json

```javascript
{
    "compilerOptions": {
      "outDir": "dist",
      "declaration": true,
      "allowSyntheticDefaultImports": true,
      "target": "es2015",
      "lib": ["ES2020", "DOM", "DOM.Iterable"],
      "module": "ESNext",
      "skipLibCheck": true,
      "moduleResolution": "bundler",
      "allowImportingTsExtensions": true,
      "resolveJsonModule": true,
      "isolatedModules": true,
      "noEmit": true,      
      "jsx": "react-jsx", 
      "strict": true,
    },
    "include": [
      "src"
    ],
    "exclude": [
      "src/**/*.test.tsx",
      "src/**/*.stories.tsx"
    ]
 }
```

安装下这些依赖（还要安装下 sass）：

```scss
npm install --save-dev sass @rollup/plugin-commonjs @rollup/plugin-node-resolve @rollup/plugin-replace @rollup/plugin-typescript rollup-plugin-postcss tslib
```

然后打包下：

```arduino
npx rollup -c rollup.config.mjs
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b6e14d0e6384973b30e6f52f305eef3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1162&h=478&s=109870&e=png&b=191919)

看下产物：

umd：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a06c0d805474ad29dec28f84d8e2a6b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2096&h=524&s=186584&e=png&b=1d1d1d)

esm：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40db399d1eea4493b3d6ac96ce4350e3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1548&h=662&s=166005&e=png&b=1c1c1c)

cjs：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6aa0421fd3994bb98b80053e64183932~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1524&h=536&s=137804&e=png&b=1d1d1d)

css：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8ea8fda65ac48ca94d8d10444ed05fc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1282&h=736&s=150599&e=png&b=1c1c1c)

dts：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/650832d8c4694652b3ec45be6d4e1c6c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1954&h=498&s=156931&e=png&b=1d1d1d)

都没问题。

rollup 抽离出 css 代码后打包到了一个文件里，这样只要引入这个 index.css 就好了。

而且 rollup 支持 tree shaking，会去掉 js 产物里的引入的 css 模块。

所以源码里引入的 scss 就不用去掉了：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06ed267a25654e82be875ad1733caf60~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1348&h=700&s=170774&e=png&b=1c1c1c)

我们改下 package.json

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/828f180e2335424db084630df605bde1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=722&h=516&s=75502&e=png&b=202020)

```javascript
"main": "dist/cjs.js",
"module": "dist/esm.js",
"types": "dist/index.d.ts",
"unpkg": "/dist/umd.js",
"files": [
    "dist",
    "package.json",
    "README.md"
],
```

main、module、unpkg 分别是 commonjs、es module、umd 的入口，types 是 dts 的入口。

files 是指定哪些文件、目录发到 npm 包。

然后在 [npm 网站](https://www.npmjs.com/ "https://www.npmjs.com/")创建这个组织：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/882cfa60a98b486c97261d579e95707b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=584&h=766&s=53006&e=png&b=fcfcfc)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ee9f8bf808c4a0d916b533cb9943c3a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1110&h=958&s=106298&e=png&b=fbfbfb)

加上发布配置，发布公开的包：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30f5b4eec4e84957a2cf1f9fa333fb38~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=736&h=618&s=90375&e=png&b=1f1f1f)

```json
"publishConfig": {
    "access": "public"
},
```

发布下：

```
npm adduser

npm publish
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c2eb049c5464a5c885874ac5db111ea~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1420&h=264&s=54425&e=png&b=181818)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec2516fe3df445ca909b0e54ecdda131~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1274&h=688&s=175658&e=png&b=181818)

发布成功。

去 [npm 网站](https://www.npmjs.com/package/@guang-comp/react-comp-lib "https://www.npmjs.com/package/@guang-comp/react-comp-lib")搜一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27fb308b61df4c15bcb4635697dd8e40~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1790&h=1048&s=159651&e=png&b=fefefe)

然后我们创建个项目引入下试试：

```lua
npx create-vite my-comp-lib-test
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9bf1ca238da8495e96152a7aab9f95e5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=936&h=386&s=48696&e=png&b=010101)

进入项目，安装我们的组件库：

```css
npm install

npm install --save @guang-comp/react-comp-lib
```

去掉 main.tsx 里的样式：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c96ed026ee04bd8accde076dfba2db7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=984&h=398&s=75463&e=png&b=1f1f1f)

然后改下 App.tsx

```javascript
import { Button } from "@guang-comp/react-comp-lib"
import '@guang-comp/react-comp-lib/dist/index.css';

function App() {

  return <div>
    <Button type='primary' style={{marginRight: '20px'}}>按钮一</Button>
    <Button type='default' onClick={() => alert(1)}>按钮二</Button>
  </div>
}

export default App
```

从组件库引入 Button 组件和样式。

跑一下:

```arduino
npm run dev
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f72e9d6e81764307b8ed100b83517103~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=838&h=304&s=38179&e=png&b=191919)

![2024-12-28 11.49.30.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9b71f9b09384d68a88900845896691f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1974&h=1060&s=181774&e=gif&f=28&b=fefefe)

样式、逻辑都没问题。

然后再来试下 umd 产物：

创建 test.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://unpkg.com/react@18.3.1/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@guang-comp/react-comp-lib@1.0.1/dist/umd.js"></script>
    
    <link rel="stylesheet" href="https://unpkg.com/@guang-comp/react-comp-lib@1.0.1/dist/index.css" />

</head>
<body>
    <div id="root"></div>
    <script>
        const container = document.getElementById('root');
        const root = ReactDOM.createRoot(container);

        root.render(React.createElement(Guang.Button, { type: 'primary', onClick: () => alert(666), children: '按钮' }));
    </script>
</body>
</html>

```

跑个静态服务器：

```vbscript
npx http-server .
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5c1459f49b7454da2a02b40f157f1e9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=776&h=560&s=87089&e=png&b=181818)

浏览器访问下 [http://localhost:8080/test.html](http://localhost:8080/test.html "http://localhost:8080/test.html")

![2024-12-28 12.44.05.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c12f85a40f694a2ba405b7f293443478~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1974&h=1060&s=219283&e=gif&f=26&b=fefefe)

umd 产物也没问题。

这样，我们通过 rollup 打包 cjs、esm、umd 产物就都成功了。

> 代码上传了小册仓库：
>
> [react 组件](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/react-comp "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/react-comp")
>
> [rollup 打包 react 组件库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/react-comp-lib "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/react-comp-lib")
>
> [测试项目](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/my-comp-lib-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/my-comp-lib-test")

## 总结

这节我们通过 rollup 做了组件库的打包。

分别支持了 esm、cjs、umd 还有 css 的打包。

一般自己写组件库，都是用 rollup 来打包的。