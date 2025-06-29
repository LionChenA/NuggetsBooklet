我们经常会用 vite 的脚手架创建项目：

![2024-09-04 09.50.58.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a489c84ddd154deb9c06c5b9b4efb926~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1254&h=808&s=93604&e=gif&f=55&b=020202)

它会一步步提示你输入和选择一些东西，然后来生成项目。

看下[源码](https://github.com/vitejs/vite/blob/main/packages/create-vite/src/index.ts#L304 "https://github.com/vitejs/vite/blob/main/packages/create-vite/src/index.ts#L304")：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e15ddf7ad05945beb46568459e0af844~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=900&h=632&s=108545&e=png&b=ffffff)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e94a70014dc468db9992569ef3da8d6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=962&h=694&s=113737&e=png&b=fffdfd)

用的是 prompts 这个包。

这节我们就来实现一下这个 prompts。

```perl
mkdir my-prompts
cd my-prompts
npm init -y
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa94f9065cf84ed1b195c445d8fc7a68~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=830&h=674&s=127501&e=png&b=010101)

创建项目，安装 typescript：

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

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01ba4c4c594648819495343fa852a894~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=552&h=290&s=38494&e=png&b=1f1f1f)

安装 prompts

```css
npm install --save prompts
npm install --save-dev @types/prompts
```

创建 src/test.ts

```javascript
import prompt, { PromptObject } from 'prompts'

(async function(){
    const questions: PromptObject[] = [
        {
            type: 'text',
            name: 'name',
            message: `你的名字`,
            initial: `guang`
        },
        {
            type: 'number',
            name: 'age',
            message: '你的年龄?',
            validate: value => value < 18 ? `未满 18 岁不能使用` : true
        },
        {
            type: 'password',
            name: 'secret',
            message: '设置下密码'
        },
        {
            type: 'confirm',
            name: 'confirmed',
            message: '确认么?'
        },
        {
            type: 'toggle',
            name: 'confirmtoggle',
            message: '性别?',
            active: '男',
            inactive: '女'
        },
        {
            type: 'select',
            name: 'color',
            message: '喜欢的颜色？',
            choices: [
                { title: 'Red', description: '这是红色', value: '#ff0000' },
                { title: 'Green', description: '这是绿色',value: '#00ff00' },
                { title: 'Yellow', value: '#ffff00'},
                { title: 'Blue', value: '#0000ff' }
            ]
        },
        {
            type: 'multiselect',
            name: 'multicolor',
            message: '选择不喜欢的颜色（多选）',
            choices: [
                { title: 'Red', description: '这是红色', value: '#ff0000' },
                { title: 'Green', value: '#00ff00' },
                { title: 'Yellow', value: '#ffff00'},
                { title: 'Blue', value: '#0000ff' }
            ]
        },
        {
            type: 'date',
            name: 'birthday',
            message: `你的生日？`,
            validate: date => date > Date.now() ? `不能设置未来的日期` : true
        }
    ];

    const answers = await prompt(questions);
    console.log(answers);
})();
```

我们测试了 text（文本）、number（数字）、password（密码）、confirm（确认）、toggle（切换）、select（选择）、multiselect（多选）、date（日期）这几种类型

其实和网页里的表单很类似。

跑一下：

```bash
npx tsc -w

node ./dist/test.js
```

![2024-09-04 10.37.38.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da898e41bb6d4e82ba6e8d6b74a8d487~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1254&h=808&s=104953&e=gif&f=69&b=171717)

![2024-09-04 10.37.59.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4aed88a603cf4a9bb4f0a2bc0910eac5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1254&h=808&s=152768&e=gif&f=70&b=181818)

填写完后，会返回用户输入的内容：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1b61a524b0a4cd6bffe0899c3e21e75~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=810&h=600&s=94255&e=png&b=191919)

我们每天都在用这种东西：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e518a66fd214747818e8eb86bbc3675~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=616&h=420&s=40766&e=png&b=000000)

今天我们来实现一下。

首先，这种格式是怎么渲染的呢？

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ba77c7644874edd80877094a5c71fff~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=466&h=150&s=23630&e=png&b=191919)

其实学完光标控制、键盘输入就应该有思路了。

我们来写下：

首先写个基础类：

src/Prompt.ts

```javascript
import ansiEscapes from 'ansi-escapes';
import EventEmitter from "events";
import readline from 'node:readline';

export interface Key {
    name: string;
    sequence: string;
}

let onKeypress: (str: string, key: Key) => void

export abstract class Prompt extends EventEmitter{
    value = ''
    rl: readline.Interface

    constructor() {
        super();

        readline.emitKeypressEvents(process.stdin);
        this.rl = readline.createInterface({ input: process.stdin});

        process.stdin.setRawMode(true);

        onKeypress = this.onKeypress.bind(this);
        process.stdin.on('keypress', onKeypress);
    }

    abstract onKeyInput(str: string, key: Key): void;

    private onKeypress(str: string, key: Key) {
        if(key.sequence === '\u0003') {
            process.exit();
        }
        
        if(key.name === 'return') {
            this.close();
            return;
        }
        this?.onKeyInput(str, key)
    }

    close() {        
        process.stdout.write('
');

        process.stdin.removeListener('keypress', onKeypress);
        process.stdin.setRawMode(false);


        this.rl.close();
        this.emit('submit', this.value);

    }

}
```

就是绑定键盘事件的基础代码。

keypress 的时候调用 this.onKeypress，处理下 ctrl +c 和回车键。

按回车键的时候算是输入结束，调用 close 方法。

close 方法里打印一个换行，然后通过 EventEmitter 把值传出去。

并且去掉 keypress 的事件绑定。

然后实现具体的 prompt 渲染逻辑：

src/TextPrompt.ts

```javascript
import ansiEscapes from 'ansi-escapes';
import { Key, Prompt } from "./Prompt.js";
import chalk from 'chalk';

export interface TextPromptOptions  {
    type: 'text'
    name: string
    message: string
}

function isNonPrintableChar(char: string) {
    return /^[\x00-\x1F\x7F]$/.test(char);
}

export class TextPrompt extends Prompt {
    out = process.stdout
    cursor = 0

    constructor(private options: TextPromptOptions) {
        super();
    }

    onKeyInput(str: string, key: Key) {

        if (key.name === 'backspace') {
            this.cursor --;
            this.value = this.value.slice(0, this.cursor);
        }

        if(!isNonPrintableChar(str)) {
            this.value += str;
            this.cursor ++;
        }

        this.render();
    }

    render() {
        this.out.write(ansiEscapes.eraseLine);

        this.out.write(ansiEscapes.cursorTo(0));

        this.out.write([
            chalk.bold(this.options.message),
            chalk.gray('›'),
            ' ',
            chalk.blue(this.value)
        ].join(''))

        this.out.write(ansiEscapes.cursorSavePosition)

        this.out.write(ansiEscapes.cursorDown(1) + ansiEscapes.cursorTo(0))

        if(this.value === '') {
            this.out.write(chalk.red('请输入名字'))
        } else {
            this.out.write(ansiEscapes.eraseLine)
        }

        this.out.write(ansiEscapes.cursorRestorePosition)
    }
}
```

先来看渲染部分：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ba77c7644874edd80877094a5c71fff~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=466&h=150&s=23630&e=png&b=191919)

我们的目标是渲染这种格式。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1af76d2e664c4955adb164fc3271637d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1164&h=878&s=177556&e=png&b=1f1f1f)

首先清除当前行的内容，然后打印问题，如果有错误，去下一行打印错误。

最核心的逻辑是这个：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1099e4994f7c44c08ed1e26252caf2f3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1110&h=856&s=148164&e=png&b=1f1f1f)

去打印错误之前，我们先把光标位置 save 了，然后光标下移一行、回到开头，打印完之后再 restore

然后是处理输入：

![2024-09-04 15.30.55.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/244ccb041b084165a32bbf672fb387f0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=882&h=490&s=26852&e=gif&f=32&b=181818)

在按删除键的时候，光标左移，输入内容的时候光标右移。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/231516481f914de18d58fa87ed3e5e62~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1064&h=990&s=162518&e=png&b=1f1f1f)

我们通过 cursor 记录光标位置，删除的时候用 slice 截取字符串。

这里要要判断下，如果输入的是一些控制字符，比如 ESC 键等，就不做处理。

之后来用一下：

src/index.ts

```javascript
import { TextPromptOptions, TextPrompt } from "./TextPrompt.js";

export type PromptOptions = TextPromptOptions;

const map: Record<string, any> = {
    text: TextPrompt
}

async function runPrompt(question: PromptOptions) {
    const promptClass = map[question.type];

    if(!promptClass) {
        return null;
    }

    return new Promise((resolve) => {
        const prompt = new promptClass(question);

        prompt.render();

        prompt.on('submit', (answer: string) => {
            resolve(answer)
        })
    });
}

export async function prompt(questions: PromptOptions[]) {
    const answers: Record<string, any> = {};

    for(let i = 0; i< questions.length; i++) {
        const name = questions[i].name;

        answers[name] = await runPrompt(questions[i]);
    }

    return answers;
}

```

循环把传入的问题渲染成 Prompt 实例，当触发 sumit 事件的时候拿到 answer 返回。

src/test2.ts

```javascript
import { prompt, PromptOptions } from "./index.js";

const questions: PromptOptions[] = [
    {
        message: '你的名字?',
        type: 'text',
        name: 'name'
    },
    {
        message: '年龄?',
        type: 'text',
        name: 'age'
    }
];

(async function() {
    const answers = await prompt(questions);
    console.log(answers);
})();
```

试一下：

![2024-09-04 15.34.14.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5a978bbb3f544ed97eeba1cc18e884d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=882&h=490&s=56854&e=gif&f=64&b=181818)

我们的 prompts 实现了！

但现在输入完没有退出，我们处理下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd750176f9a54ec3ac9805eecfcb5e0e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1184&h=1112&s=211696&e=png&b=1f1f1f)

用 readline 的 createInterface 来开启键盘控制，然后 close 关闭键盘控制。

```javascript
import EventEmitter from "events";
import readline from 'node:readline';

export interface Key {
    name: string;
    sequence: string;
}

export abstract class Prompt extends EventEmitter{
    value = ''
    rl: readline.Interface

    constructor() {
        super();

        readline.emitKeypressEvents(process.stdin);
        this.rl = readline.createInterface({ input: process.stdin});

        process.stdin.setRawMode(true);

        process.stdin.on('keypress', this.onKeypress.bind(this));
    }

    abstract onKeyInput(str: string, key: Key): void;

    private onKeypress(str: string, key: Key) {
        if(key.sequence === '\u0003') {
            process.exit();
        }
        
        if(key.name === 'return') {
            this.close();
            return;
        }
        this?.onKeyInput(str, key)
    }

    close() {
        this.emit('submit', this.value);
        
        process.stdout.write('
');

        process.stdin.removeListener('keypress', this.onKeypress);
        process.stdin.setRawMode(false);
        this.rl.close();
    }

}
```

![2024-09-04 15.38.21.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5987f154dcd4f12bc8f30afe515c6a6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=882&h=490&s=61610&e=gif&f=41&b=181818)

这样，当输入完之后，就会退出键盘控制。

然后我们再实现下 Select：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ee2a43e1bea4143aaebe8ee0d2cb6db~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=610&h=420&s=41876&e=png&b=000000)

创建 src/SelectPrompt.ts

```javascript
import ansiEscapes from 'ansi-escapes';
import { Key, Prompt } from "./Prompt.js";
import chalk from 'chalk';

export interface SelectPromptOptions  {
    type: 'select'
    name: string
    message: string
    choices: Array<string>
}

export class SelectPrompt extends Prompt {
    out = process.stdout
    index = 0

    constructor(private options: SelectPromptOptions) {
        super();
        this.value = options.choices[0];
    }

    onKeyInput(str: string, key: Key) {
        if(key.name !== 'up' && key.name !== 'down') {
            return;
        }
        if(key.name === 'down') {
            this.index += 1;
            if(this.index > this.options.choices.length - 1) {
                this.index = 0;
            }
        }

        if(key.name === 'up') {
            this.index -= 1;
            if(this.index < 0) {
                this.index = this.options.choices.length - 1
            }
        }

        this.value = this.options.choices[this.index];

        this.render();
    }

    render() {
        this.out.write(ansiEscapes.eraseLine);

        this.out.write(ansiEscapes.cursorSavePosition);

        this.out.write(ansiEscapes.cursorTo(0));

        this.out.write([
            chalk.bold(this.options.message),
            chalk.gray('›'),
            ' ',
            chalk.blue(this.value)
        ].join(''))

        for(let i = 0; i< this.options.choices.length; i++) {
            const choice = this.options.choices[i];

            this.out.write(ansiEscapes.cursorDown(1))
            this.out.write(ansiEscapes.cursorTo(2))
            
            if(this.value === choice) {
                this.out.write(chalk.blue('❯')+ ' ' + choice )
            } else {
                this.out.write('  ' + choice )
            }
        }

        this.out.write(ansiEscapes.cursorRestorePosition);
    }
}
```

首先看渲染部分：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee8d6bcda54e4745a7bb1382d615b9cd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1008&h=1084&s=185221&e=png&b=1f1f1f)

渲染问题这部分和之前一样。

选项就是依次 cursorDown、cursorTo 来渲染：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7cc24a3db2643eeba18cc5641d19f89~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1038&h=1084&s=187231&e=png&b=1f1f1f)

最后把光标恢复到第一行，这样重绘的时候才能正常清除。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36cf624fd7484dcfbcca6ed6631e5a01~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1036&h=1064&s=175648&e=png&b=202020)

然后处理键盘事件：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae6eb4b49b9549759d355c25178a953d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1002&h=1110&s=168784&e=png&b=1f1f1f)

只处理 up 和 down，修改 index 和对应的 value，然后重新渲染。

在 index.ts 支持下这种类型的 prompt 的渲染：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4761558cd6b4a62a460288e149e5025~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1190&h=468&s=112331&e=png&b=1f1f1f)

```javascript
import { SelectPrompt, SelectPromptOptions } from "./SelectPrompt.js";
import { TextPromptOptions, TextPrompt } from "./TextPrompt.js";

export type PromptOptions = TextPromptOptions | SelectPromptOptions;

const map: Record<string, any> = {
    text: TextPrompt,
    select: SelectPrompt
}
```

然后用一下：

src/test2.ts

```javascript
import { prompt, PromptOptions } from "./index.js";

const questions: PromptOptions[] = [
    {
        message: '你的名字?',
        type: 'text',
        name: 'name'
    },
    {
        message: '年龄?',
        type: 'text',
        name: 'age'
    },
    {
        message: '你的班级？',
        type: 'select',
        name: 'class',
        choices: [
            '一班',
            '二班',
            '三班'
        ]
    }
];

(async function() {
    const answers = await prompt(questions);
    console.log(answers);
})();
```

![2024-09-04 17.05.02.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62a074cce0ea492ea1641ef6f6cb0529~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=882&h=490&s=61385&e=gif&f=46&b=181818)

这样，我们的 prompts 就完成了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/my-prompts "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/my-prompts")

## 总结

这节我们实现了下 prompts 包，create-vite 的用户输入就是用这个包做的。

界面的绘制就是之前学的光标控制的知识，主要是 cursorDown、cursorSavePosition、cursorRestorePosition 这些。

然后键盘控制就是分别处理 up、down、enter 等键，做不同的处理，比如修改 this.value 然后重新渲染

这个和我们在 react 里改了 state 重新渲染一样

此外，想退出键盘控制，可以用 readline.createInterface 然后 close 就可以了。

自己写了一遍后，每天在用的这些工具就不再神秘了。