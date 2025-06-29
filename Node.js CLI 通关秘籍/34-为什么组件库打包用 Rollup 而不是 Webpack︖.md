这节我们来学习 Rollup。

它是一个打包工具，类似 Webpack。

组件库打包基本都是用 Rollup。

那 Webpack 和 Rollup 有什么区别呢？为什么组件库打包都用 Rollup 呢？

我们来试一下：

```bash
mkdir rollup-test
cd rollup-test
npm init -y
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fdd213621e04e648bf67e0f7a3a4623~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=816&h=674&s=127704&e=png&b=000000)

我们创建两个模块：

src/index.js

```javascript
import { add } from './utils';

function main() {
    console.log(add(1, 2))
}

export default main;
```

src/utils.js

```javascript
function add(a, b) {
    return a + b;
}

export {
    add
}
```

很简单的两个模块，我们分别用 rollup 和 webpack 来打包下：

安装 rollup：

```css
npm install --save-dev rollup
```

创建 rollup.config.js

```javascript
/** @type {import("rollup").RollupOptions} */
export default {
    input: 'src/index.js',
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
            format: "umd"
        }
    ]
};
```

配置入口模块，打包产物的位置、模块规范。

在 webpack 里叫做 entry、output，而在 rollup 里叫做 input、output。

我们指定产物的模块规范有 es module、commonjs、umd 三种。

umd 是挂在全局变量上，还要指定一个全局变量的 name。

上面的 @type 是 jsdoc 的语法，也就是 ts 支持的在 js 里声明类型的方式。

效果就是写配置时会有类型提示：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02c1d9f3708e4c92863308439e872e88~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1072&h=622&s=98796&e=png&b=1f1f1f)

不引入的话，啥提示都没有：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a61d5ec36a56485489e7521300792893~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1028&h=674&s=86054&e=png&b=1f1f1f)

关于 jsdoc，这里不展开了，感兴趣可以看下[这篇文章](https://juejin.cn/post/7292437487011856394 "https://juejin.cn/post/7292437487011856394")。

这里我们用了 export，把 rollup.config.js 改名为 rollup.config.mjs，告诉 node 这个模块是 es module 的。（之前我们是直接在 package.json 里添加了 type: module，效果一样）。

配置好后，我们打包下：

```arduino
npx rollup -c rollup.config.mjs
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f3437dd655043daad8892a571f1c8be~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=994&h=210&s=42882&e=png&b=181818)

看下产物：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b784d586e224e92918fab031d85d788~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1188&h=568&s=98687&e=png&b=1d1d1d)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/231502204a4d4558a346e3429819a9e9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1182&h=490&s=86544&e=png&b=1d1d1d)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46081a2a46894708813368829ca1f223~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2418&h=902&s=231173&e=png&b=1e1e1e)

三种模块规范的产物都没问题。

那用 webpack 打包，产物是什么样呢？

我们试一下：

```css
npm install --save-dev webpack-cli webpack
```

创建 webpack.config.mjs

```javascript
import path from 'node:path';

/** @type {import("webpack").Configuration} */
export default {
    entry: './src/index.js',
    mode: 'development',
    devtool: false,
    output: {
        path: path.resolve(import.meta.dirname, 'dist2'),
        filename: 'bundle.js',
        libraryTarget: 'commonjs2'
    }
};
```

指定 libraryTarget 为 commonjs2

打包下：

```arduino
npx webpack-cli -c webpack.config.mjs
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be19fa18b3c94d6d94e94f66c49a2003~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1002&h=272&s=68675&e=png&b=191919)

可以看到，webpack 的打包产物有 100 行代码：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97b90d75f1d54a7ca90968c0ee5c6d10~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1562&h=878&s=581709&e=png&b=1d1d1d)

再来试试 umd 的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de14e40cc84946a0aaeb96af51fce6d3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1134&h=640&s=311409&e=png&b=1f1f1f)

umd 要指定全局变量的名字。

打包下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e25ac4ff9b8455a852fff6e08e1226e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=904&h=256&s=198655&e=png&b=1a1a1a)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/511cc50a33434628a556edee6755f0d0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1328&h=916&s=432654&e=png&b=1f1f1f)

也是 100 多行。

最后再试下 es module 的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12515b11d5744461ac43cbce8e22cabc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1136&h=700&s=344374&e=png&b=1f1f1f)

libraryTarget 为 module 的时候，还要指定 experiments.outputModule 为 true。

```javascript
import path from 'node:path';

/** @type {import("webpack").Configuration} */
export default {
    entry: './src/index.js',
    mode: 'development',
    devtool: false,
    experiments: {
        outputModule: true
    },
    output: {
        path: path.resolve(import.meta.dirname, 'dist2'),
        filename: 'bundle.js',
        libraryTarget: 'module'
    }
};
```

打包下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b33eb0882d1b43cf8beff81b16c9a223~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1370&h=670&s=339127&e=png&b=1f1f1f)

产物也同样是 100 多行。

相比之下，rollup 的产物就非常干净，没任何 runtime 代码：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a676027754443f5a7fcd2d884703a1b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1168&h=582&s=88681&e=png&b=1d1d1d)

更重要的是 **webpack 目前打包出 es module 产物还是实验性的，并不稳定**。

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/b953147e654044cc93db4271ab62f598~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgenhnX-elnuivtOimgeacieWFiQ==:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMjc4ODAxNzIxNjY4NTExOCJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1732530581&x-orig-sign=E0iwliuBxjhggMr9BOeigpnm%2ByE%3D)

webpack 打 cjs 和 umd 的 library 还行。

但 js 库一般不都要提供 es module 版本么，支持的不好怎么行？

所以**我们一般用 rollup 来做 js 库的打包，用 webpack 做浏览器环境的打包。**

前面说组件库打包一般都用 rollup，我们来看下各大组件库的打包需求。

安装 antd:

```css
npm install --no-save antd
```

在 node\_modules 下可以看到它分了 dist、es、lib 三个目录：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9d31fa31d0148ff8d2598bb3c14d5a8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=414&h=360&s=25952&e=png&b=191919)

分别看下这三个目录的组件代码：

lib 下的组件是 commonjs 的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f56b42587074655afebf0a555b33981~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1764&h=910&s=415186&e=png&b=1d1d1d)

es 下的组件是 es module 的：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9131c1971860413a8780a8f4f125a348~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1612&h=834&s=313225&e=png&b=1d1d1d)

dist 下的组件是 umd 的：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d1c9f6af12d4d0fb72ea940c1a2fca3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1780&h=578&s=252241&e=png&b=1e1e1e)

然后在 package.json 里分别声明了 commonjs、esm、umd 还有类型的入口：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd5451ec3cec426abe12b886d54f2072~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=728&h=250&s=45287&e=png&b=1f1f1f)

这样，当你用 require 引入的就是 lib 下的组件，用 import 引入的就是 es 下的组件。

而直接 script 标签引入的就是 unpkg 下的组件。

再来看一下 semi design 的：

```css
npm install --no-save @douyinfe/semi-ui
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/157ebd3a769d4f72a33047fbd695ae1a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=494&h=638&s=50362&e=png&b=1b1b1b)

也是一样：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7715e7c48a944cf4a845ad77010ba470~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=882&h=472&s=84118&e=png&b=1f1f1f)

只不过多了个 css 目录。

所以说，**组件库的打包需求就是组件分别提供 esm、commonjs、umd 三种模块规范的代码，并且还有单独打包出的 css。**

那 rollup 如何打包 css 呢？

我们试一下：

创建 src/index.css

```css
.aaa {
    background: blue;
}
```

创建 src/utils.css

```css
.bbb {
    background: red;
}
```

然后分别在 index.js 和 utils.js 里引入下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb247b10d94a40009ad49a79e12ccf9c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1366&h=512&s=99356&e=png&b=1d1d1d)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b040aeabbacd465685b3e7e4e6d22a14~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1298&h=502&s=80755&e=png&b=1c1c1c)

安装 rollup 处理 css 的插件：

```css
npm install --save-dev rollup-plugin-postcss
```

引入下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be5b759c43b7450aaacfe82e988a8f0b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1096&h=1128&s=153020&e=png&b=1f1f1f)

```javascript
import postcss from 'rollup-plugin-postcss';

/** @type {import("rollup").RollupOptions} */
export default {
    input: 'src/index.js',
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
            format: "umd"
        }
    ],
    plugins: [
        postcss({
            extract: true,
            extract: 'index.css'
        }),
    ]
};
```

然后跑一下：

```arduino
npx rollup -c rollup.config.mjs
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ca4dc36e19c4f11a6e47c0600c24cac~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1000&h=164&s=29262&e=png&b=181818)

可以看到，产物多了 index.css

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cc1c30fced54a9e9dcd3cd7b242c69f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1252&h=394&s=65372&e=png&b=1c1c1c)

而 js 中没有引入 css 了：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7889a622c0e4a8bb2134b70ab67736c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1324&h=548&s=98826&e=png&b=1c1c1c)

被 tree shaking 掉了，rollup 默认开启 tree shaking。

这样我们就可以单独打包组件库的 js 和 css。

删掉 dist，我们试下不抽离是什么样的：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89ec284fa28e4d5aa35c977385cc123b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=800&h=532&s=58170&e=png&b=1f1f1f)

```arduino
npx rollup -c rollup.config.mjs
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec9eae9552c24ee3acc8e02a8e27dce1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=976&h=154&s=30583&e=png&b=181818)

可以看到，代码里多了 styleInject 的方法：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80d08fdfb322454c9bf32e1abadf32aa~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1968&h=1222&s=318315&e=png&b=1d1d1d)

用于往 head 里注入 style

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88533542fa16400da547884a37532b22~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1640&h=830&s=214061&e=png&b=1d1d1d)

一般打包组件库产物，我们都会分离出来。

然后我们再用 webpack 打包试试：

安装用到的 loader：

```css
npm install --save-dev css-loader style-loader
```

css-loader 是读取 css 内容为 js

style-loader 是往页面 head 下添加 style 标签，填入 css

这俩结合起来和 rollup 那个插件功能一样。

配置 loader：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/096469a075e84593aed347a0d4862fc2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1162&h=828&s=137620&e=png&b=1f1f1f)

```javascript
module: {
    rules: [{
        test: /\.css$/i,
        use: ["style-loader", "css-loader"],
    }],
}
```

用 webpack 打包下：

```arduino
npx webpack-cli -c webpack.config.mjs
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae242ad4267f4327affc6fba29c07905~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1626&h=598&s=181957&e=png&b=191919)

可以看到 css 变成 js 模块引入了：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/044c92d8b7d5417b9a0a41ce355a20c8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1898&h=596&s=193335&e=png&b=1d1d1d)

这是 css-loader 做的。

而插入到 style 标签的 injectStylesIntoStyleTag 方法则是 style-loader 做的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a49b8cb85624d2fa4dbcff6470cda43~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2432&h=564&s=232054&e=png&b=1e1e1e)

然后再试下分离 css，这用到一个单独的插件：

```css
npm install --save-dev mini-css-extract-plugin
```

配一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/206470bf17b64eb18687f48d11bc8d69~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1226&h=1022&s=192027&e=png&b=1f1f1f)

```javascript
import path from 'node:path';
import MiniCssExtractPlugin from "mini-css-extract-plugin";

/** @type {import("webpack").Configuration} */
export default {
    entry: './src/index.js',
    mode: 'development',
    devtool: false,
    output: {
        path: path.resolve(import.meta.dirname, 'dist2'),
        filename: 'bundle.js',
    },
    module: {
        rules: [{
            test: /\.css$/i,
            use: [MiniCssExtractPlugin.loader, "css-loader"],
        }],
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'index.css'
        })
    ]
};

```

指定抽离的 filename 为 index.css

抽离用的 loader 要紧放在 css-loader 之前。

样式抽离到了 css 中，这时候 style-loader 也就不需要了。

打包下：

```arduino
npx webpack-cli -c webpack.config.mjs
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69f93ee9bfd9483ab6eb20dfcc9435a7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1460&h=524&s=149259&e=png&b=191919)

样式抽离到了 css 中：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1010213d837448d2b44990f5760a8add~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1840&h=560&s=131448&e=png&b=1d1d1d)

而 js 里的这个模块变为了空实现：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e68a78f4d454d3ca55440b8d80cdc5f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1880&h=574&s=159887&e=png&b=1d1d1d)

所以 webpack 的 style-loader + css-loader + mini-css-extract-plugin 就相当于 rollup 的 rollup-plugin-postcss 插件。

为什么 rollup 没有 loader 呢？

因为 rollup 的 plugin 有 transform 方法，也就相当于 loader 的功能了。

我们自己写一下抽离 css 的 rollup 插件：

创建 my-extract-css-rollup-plugin.mjs（注意这里用 es module 需要指定后缀为 .mjs）：

```javascript
const extractArr = [];

export default function myExtractCssRollupPlugin (opts) {
    return {
      name: 'my-extract-css-rollup-plugin',
      transform(code, id) {
        if(!id.endsWith('.css')) {
          return null;
        }

        extractArr.push(code);

        return {
          code: 'export default undefined',
          map: { mappings: '' }
        }
      },
      generateBundle(options, bundle) {

        this.emitFile({
          fileName: opts.filename || 'guang.css',
          type: 'asset',
          source: extractArr.join('
/*光光666*/
')
        })
      }
    };
  }

```

在 transform 里对代码做转换，这就相当于 webpack 的 loader 了。

我们在 transform 里只处理 css 文件，保存 css 代码，返回一个空的 js 文件。

然后 generateBundle 里调用 emitFile 生成一个合并后的 css 文件。

用一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/beb2a640b9c0425bbde8e79d0744746d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1436&h=688&s=137767&e=png&b=1f1f1f)

```javascript
import myExtractCssRollupPlugin from './my-extract-css-rollup-plugin.mjs';
```

```javascript
myExtractCssRollupPlugin({
    filename: '666.css'
})
```

删掉之前的 dist 目录，重新打包：

```arduino
npx rollup -c rollup.config.mjs
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8025dbab4e7745b0b8b4ef163697e541~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1022&h=164&s=32586&e=png&b=181818)

看下产物： ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/347bab3b07354c4798c96861180109ca~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1164&h=476&s=79795&e=png&b=1c1c1c)

可以看到，抽离出了 css，内容是合并后的所有 css。

而 cjs 也没有 css 的引入：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b66d89bac8a44cc9bb5b49cd5af9a2ca~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1232&h=618&s=104212&e=png&b=1d1d1d)

也是被 tree shaking 掉了。

我们把 tree shaking 关掉试试：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6676d20a8ff5456bb0980ff3a10def7f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=922&h=512&s=97514&e=png&b=1f1f1f)

再次打包：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14c2614fc10549e08afb42b9a47e9f72~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=974&h=170&s=32293&e=png&b=181818)

可以看到，两个 css 模块转换后的 js 模块依然被引入了：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d566200441fd4e33956e398fd7c86b6e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1220&h=746&s=136670&e=png&b=1d1d1d)

我们改下插件 transform 的内容：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9decd6b3fd24df3acb61099e96bf211~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1168&h=560&s=99917&e=png&b=1f1f1f)

再次打包：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecb305ed80b843538375f1380fda1bb3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=964&h=148&s=27459&e=png&b=181818)

可以看到引入的也是我们转后后的 css 模块的内容：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fb469a1a0824b8987a553737a85c894~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1306&h=662&s=113482&e=png&b=1c1c1c)

因为没用到，同样会被 tree shaking 掉。

所以说 **rollup 的插件的 transform 就相当于 webpack loader 的功能。**

前面说 webpack 用来做浏览器的打包，而 rollup 一般做 js 库的打包。

这也不全对，vite 就是用 rollup 来做的生产环境的打包。

因为它开发环境下不打包，而是跑了一个开发服务器，对代码做了下转换，不需要 webpack 那些 dev server 的功能。

而生产环境又需要打包，所以 rollup 就很合适。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/888c6bdfaed64e50a27e30b18a8b9e28~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1660&h=992&s=190370&e=png&b=fefcfc)

开发环境下，浏览器里用 type 为 module 的 script 引入，会请求 vite 的开发服务器。

vite 开发服务器会调用 rollup 插件的 transform 方法来做转换。

而生产环境下，用 rollup 打包，也是用同样的 rollup 插件。

当然，vite 还会用 esbuild 来做下依赖的与构建，比如把 cjs 转换成 esm、把小模块打包成一个大的模块。

用 esbuild 是因为它更快。

所以说，vite 是基于 rollup 来实现的，包括开发服务器的 transform，以及生产环境的打包。

但是为了性能考虑，又用了 esbuild 做依赖预构建。

现在 vite 团队在开发 rust 版 rollup 也就是 rolldown 了，有了它之后，就可以完全替代掉 rollup + esbuild 了。

综上，除了 webpack、vite 外，rollup 也是非常常用的一个打包工具。

> 案例代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/rollup-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/rollup-test")

## 总结

这节我们学习了 rollup，虽然它不如 webpack、vite 提到的多，但也是一个常用的打包工具。

它打包产物没有 runtime 代码，更简洁纯粹，能打包出 esm、cjs、umd 的产物，常用来做 js 库、组件库的打包。相比之下，webpack 目前对 esm 产物的支持还是实验性的，不稳定。

rollup 只有 plugin，没有 loader，因为它的 transform 方法就相当于 webpack 插件的 loader。

vite 就是基于 rollup 来实现的，开发环境用 rollup 插件的 transform 来做代码转换，生产环境用 rollup 打包。

不管你是想做组件库、js 库的打包，还是想深入学习 vite，都离不开 rollup。