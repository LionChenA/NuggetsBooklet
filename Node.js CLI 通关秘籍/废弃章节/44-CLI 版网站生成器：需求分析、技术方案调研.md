上节我们用了一下 [bolt.new](https://bolt.new/ "https://bolt.new/")

在输入框输入需求：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa0d36834d7f48bb9892af24ce5d5b02~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2308&h=1162&s=440815&e=png&b=0c0c0c)

就会自动生成代码，并在右侧编辑器展示：

![2024-11-19 17.51.24.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f95ae6675c0e45dabb753d1ee09d5f00~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2332&h=1256&s=2754167&e=gif&f=70&b=111111)

然后把开发服务器跑起来，可以在右侧预览：

![2024-11-19 18.01.24.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fda3111166e499492e5df74443c70cb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2602&h=1456&s=5567385&e=gif&f=42&b=171717)

之后还会自动部署，给你一个 url。

它实现浏览器里 npm install、npm run dev 是通过 webcontainer 实现的，也就是基于 wasm 实现了一套浏览器里的 node api。

而 cli 天然就可以跑 node，那我们是不是可以在 cli 实现这个呢？

对接 AI 接口生成代码的部分前面学过了，这节我们主要来调研下 bolt.new 那种 UI 界面在 cli 里能不能实现。

创建个项目：

```arduino
mkdir cli-bolt-new-test
cd cli-bolt-new-test

npm init -y
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b92937ae9d844ac9876bd137d093a10~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=856&h=658&s=89026&e=png&b=010101)

我们首先来做下输入 API KEY 和 BASE URL 的输入框：

安装依赖：

```css
npm install --save @clack/prompts chalk
```

创建 src/index.mjs

```javascript
import * as prompts from '@clack/prompts';
import chalk from 'chalk';

async function main() {
    console.clear();

	prompts.intro(`${chalk.bgCyan.black(' cli 网站生成器 ')}`);

	const info = await prompts.group(
		{
			path: () =>
				prompts.password({
					message: '请输入你的 API KEY',
					validate: (value) => {
						if (!value) return 'API KEY 不能为空';
					},
				}),
			password: () =>
				prompts.text({
					message: '请输入 BASE URL',
                    placeholder: 'https://api.openai.com/v1',
					validate: (value) => {
						if (!value) return 'BASE URL 不能为空';
					}
				})
		},
		{
			onCancel: () => {
				prompts.cancel('下次再见~');
				process.exit(0);
			},
		}
	);

    console.log(info);
}

main();
```

之前我们用过 prompts、inquirer 来做 cli 里的表单输入，这次用 clack 来做，它的 UI 更好看一些。

原理都一样，都是控制光标、颜色来实现的，我们还自己实现过 prompts。

这里的 console.clear() 是清空上面的内容的。

创建 src/test.js

```javascript
console.clear()
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14641915040f4590adee51f3598cff6d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1718&h=646&s=112467&e=gif&f=21&b=171717)

先跑下再说：

```bash
node src/index.mjs
```

![2024-11-25 10.54.46.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dcaaa2c505aa4bc0bd396dcf7a08d60b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1718&h=646&s=131713&e=gif&f=43&b=181816)

![2024-11-25 10.55.48.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c85cacadd6d4c4899db4c3e95ba5116~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1718&h=646&s=99712&e=gif&f=36&b=171717)

上面的 prompts.intro、prompts.password、prompts.text、prompts.cancel 就是分别展示上面的内容的

而左边的竖线是通过 prompts.group 来组织的。

相比 inquirer，确实好看不少。

输入 API KEY、BASE URL 之后就可以进入网站生成器界面了：

我们在下面画个输入框：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa0d36834d7f48bb9892af24ce5d5b02~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2308&h=1162&s=440815&e=png&b=0c0c0c)

这个用到我们之前学的 blessed、blessed-contrib 来画：

```css
npm install --save blessed blessed-contrib
```

pm2 monit 就是用它画的：

![2024-09-03 11.31.02.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/456291554d1140fcbd4e32a4cdb450f6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1972&h=942&s=6117882&e=gif&f=70&b=171717)

创建 src/input.mjs

```javascript
import blessed from 'blessed';
import contrib from 'blessed-contrib';

async function main() {
    const screen = blessed.screen({
        fullUnicode: true
    });

    const grid = new contrib.grid({ rows: 12, cols: 12, screen: screen });
    
    const textarea = grid.set(8, 2, 4, 8, blessed.textarea, {
        label: '描述下你想生成什么样的网站', 
        fg: 'green',
        mouse: true
    });

    textarea.readInput((err, value) => {
        screen.destroy();
        console.log(textarea.value);
    } );

    screen.key('C-c', function() {
        screen.destroy();
    });
    
    screen.render();
}

main();
```

grid 布局的参数是这样的：

```scss
grid.set(row, col, rowSpan, colSpan, obj, opts)
```

前两个参数是位置，也就是行号、列号

后两个参数是大小

最后的就是渲染的组件和参数了。

我们设置了行、列都分为 12 份，那就是按比例来计算。

跑一下：

![2024-11-25 11.16.31.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9925c9eca07c4ce39a87d2426e029828~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2110&h=782&s=285797&e=gif&f=64&b=171717)

在输入框内输入内容后，按 esc 确认，然后在 readInput 的回调里就可以拿到输入的内容了。

我们还在上面展示下 logo 和提示信息。

此外， bolt.new 还支持传入一张图片来生成网站代码：

我上传这张草图：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e28df380886b4f44b305c22ac6bca56b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=620&h=578&s=84621&e=png&b=fdfdfd)

让 bolt.new 感觉这个草图的布局来生成网站：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3e033e2c5254373ae046fbcdbb3ea6c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1426&h=964&s=133771&e=png&b=0d0d0d)

![2024-11-25 11.47.25.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e098e63541cb43639ab669be15af65d0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2672&h=1406&s=630757&e=gif&f=42&b=161616)

预览没问题：

![2024-11-25 11.50.00.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e35d69145a2a4126af2f48fb50d99416~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2672&h=1406&s=5902909&e=gif&f=40&b=1a1919)

点击部署：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02a42d5b21934a39a01fe1c566efc8f6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2160&h=1190&s=991122&e=png&b=1e1e1e)

打开这个链接看下：

[stately-kitten-7f0362.netlify.app/](https://stately-kitten-7f0362.netlify.app/ "https://stately-kitten-7f0362.netlify.app/")

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aec2e70d170543c0a0b53c2fcc4c2ef3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2614&h=1452&s=2572619&e=png&b=e6eaee)

完全是按照我们的草图的布局来的。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e28df380886b4f44b305c22ac6bca56b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=620&h=578&s=84621&e=png&b=fdfdfd)

我们也可以实现这个功能。

当然，前提是在 cli 里实现文件选择器。

这个组件 blessed 也有：

创建 src/fileSelector.mjs

```javascript
import blessed from 'blessed';

const screen = blessed.screen({
    fullUnicode: true
});

const fm = blessed.filemanager({
    parent: screen,
    border: 'line',
    height: 'half',
    width: 'half',
    top: 'center',
    left: 'center',
    label: ' {blue-fg}%path{/blue-fg} ',
    cwd: process.cwd(),
    keys: true,
    style: {
        selected: {
            bg: 'blue'
        }
    },
    scrollbar: {
        bg: 'white'
    }
});
  
fm.on('file', (file)=> {
    screen.destroy();

    console.log(file);
})

screen.key('C-c', function() {
    screen.destroy();
});

fm.refresh();

screen.render();
```

跑一下：

![2024-11-25 12.00.17.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06ce9f3ccd624667859b2695247c07e2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1898&h=616&s=151185&e=gif&f=35&b=181818)

选中文件，拿到路径之后，就是读取文件内容，调用 AI 接口来生成代码就好了。

生成代码后怎么展示呢？

其实布局和 pm2-monit 这个很类似：

![2024-09-03 11.31.02.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/456291554d1140fcbd4e32a4cdb450f6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1972&h=942&s=6117882&e=gif&f=70&b=171717)

我们来写一下：

创建 src/editor.mjs

```javascript
import blessed from 'blessed';
import contrib from 'blessed-contrib';
import fs from 'node:fs';
import { highlight } from 'cli-highlight';

async function main() {

    const screen = blessed.screen({
        fullUnicode: true
    });

    const grid = new contrib.grid({rows: 12, cols: 12, screen: screen});
    
    const tree = grid.set(0, 0, 12, 3, contrib.tree, {
        label: '目录', 
        fg: 'green',
        mouse: true
    })

    tree.setData({
        extended: true,
        children: {
            src: { 
                children:
                { 
                    'aaa.ts': {},
                    'bbb.ts': {},
                    components: {
                        children: {
                            'xxx.tsx': {},
                            'yyy.tsx': {}
                        }
                    }
                }
            }, 
            'index.js': {},
        }
    });
    
    const code = grid.set(0, 3, 12, 9, blessed.box, {
        label: '文件内容',
        fg: "green",
        selectedFg: "white",
        fg: 'white',
        mouse: true,
        keys: true,
        scrollable: true
    })
    
    const fileContent = {
        'index.js': fs.readFileSync('./src/index.mjs', 'utf-8'),
        'src/aaa.ts': fs.readFileSync('./src/fileSelector.mjs', 'utf-8'),
        'src/bbb.ts': fs.readFileSync('./src/test.js', 'utf-8'),
        'src/components/xxx.tsx': fs.readFileSync('./package.json', 'utf-8'),
        'src/components/yyy.tsx': fs.readFileSync('./src/editor.mjs', 'utf-8')
    };

    tree.on('select',function(node){
        let filePathArr = [node.name];
        let curNode = node;
        while(curNode.parent !== null) {
            curNode = curNode.parent;

            filePathArr.unshift(curNode.name);
        }

        const filePath = filePathArr.filter(Boolean).join('/')

        code.content = highlight(fileContent[filePath] || '', {
            language: 'javascript',
            ignoreIllegals: true
        });

        screen.render();
    });
    
    screen.key('C-c', function() {
        screen.destroy();
    });
    
    screen.render();
    
    tree.focus();   
}

main();
```

主要就是两个组件 tree、box：

tree 用来展示目录的树形结构

box 用来展示代码，设置 scrollable，然后内容用 cli-hightlight 高亮下

选中 tree 的节点的时候，拼接下路径，然后在右侧展示对应的文件内容。

安装依赖：

```css
npm install --save cli-highlight
```

跑一下：

```bash
node ./src/editor.mjs
```

![2024-11-25 12.41.45.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb33d0307a7141a187d1910d2d199020~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2198&h=884&s=799831&e=gif&f=70&b=171717)

可以看到，目录和代码的展示都没问题。

那生成代码后的构建日志怎么展示呢？

这样：

创建 src/log.mjs

```javascript

import blessed from 'blessed';
import contrib from 'blessed-contrib';
import { spawn } from 'child_process';

async function main() {
    const screen = blessed.screen({
        fullUnicode: true
    });

    const grid = new contrib.grid({ rows: 12, cols: 12, screen: screen });

    const box = grid.set(4, 3, 6, 9, blessed.box, {
        label: '构建日志', 
        mouse: true,
        scrollable: true
    });

    const server = spawn('npx', ['http-server', '.'], {
        env: { ...process.env, FORCE_COLOR: true },
    });

    let content = ''
    server.stdout.on('data', (data) => {
        content += data.toString();
        box.content = content;
        screen.render();
    });

    screen.key('C-c', function() {
        screen.destroy();
    });

    screen.render();
}

main();
```

这里要注意的是用 child\_process 的 spawn 跑的进程，默认的日志是没有颜色的，需要加上 FORCE\_COLOR 环境变量才可以。

跑一下：

```bash
node ./src/log.mjs
```

![2024-11-25 13.05.40.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/388f0fdc380b4714969cc6339c91f94e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2176&h=562&s=200826&e=gif&f=58&b=171717)

这样，生成的代码就可以构建、展示构建日志，然后在浏览器里预览了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/cli-bolt-new-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/cli-bolt-new-test")

## 总结

这节我们单独画了下 cli 版 bolt.new 网站生成器用到的 UI。

用 clack 做的表单，比 prompts、inquirer 好看一些。

用 blessed、blessed-contrib 做的弹性布局，画的输入框、文件树

用 cli-highlight 做的代码高亮。

最后还做了构建日志的打印。

下节开始我们就接入 AI，来一步步实现 CLI 版的网站生成器。