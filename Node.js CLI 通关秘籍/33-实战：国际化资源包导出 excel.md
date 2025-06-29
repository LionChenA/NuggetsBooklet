前面我们学过国际化，我们会把文案抽离出来，放在不同的资源包里维护。

比如 zh-CN.json：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41856af826c44430b6e9a752508f528d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=796&h=370&s=64828&e=png&b=1f1f1f)

en-US.json：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a402c7cfe4da42f3bfe23f3462a312bd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=920&h=346&s=66843&e=png&b=1f1f1f)

而这个文案的翻译一般是产品经理做的。

那怎么把这个资源包给产品经理编辑呢？

直接给他 json 文件么？

这样并不好。

一般我们都是导出 excel。

上节我们学了 excel 如何生成，这节我们就来实战一下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/250767bae1a647149287af233f408be6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=854&h=684&s=86813&e=png&b=000000)

```arduino
mkdir excel-export
cd excel-export
npm init -y
```

进入项目，安装 exceljs：

```css
npm install --save exceljs
```

写下 index.js

```javascript
const { Workbook } = require('exceljs');

async function main(){
    const workbook = new Workbook();

    const worksheet = workbook.addWorksheet('guang111');

    worksheet.columns = [
        { header: 'ID', key: 'id', width: 20 },
        { header: '姓名', key: 'name', width: 30 },
        { header: '出生日期', key: 'birthday', width: 30},
        { header: '手机号', key: 'phone', width: 50 }
    ];

    const data = [
        { id: 1, name: '光光', birthday: new Date('1994-07-07'), phone: '13255555555' },
        { id: 2, name: '东东', birthday: new Date('1994-04-14'), phone: '13222222222' },
        { id: 3, name: '小刚', birthday: new Date('1995-08-08'), phone: '13211111111' }
    ]
    worksheet.addRows(data);

    workbook.xlsx.writeFile('./data.xlsx');    
}

main();
```

就是按照 workbook（工作簿） > worksheet（工作表）> row （行）的层次来添加数据。

跑一下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b067693c3e9c4212aa08ad0e4b1efae0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1702&h=1062&s=231109&e=gif&f=32&b=191919)

生成了 excel 文件。

打开看下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a196e53f16f043d8bde88948f9e4b233~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1446&h=1160&s=147634&e=png&b=fefefe)

可以看到 worksheet 的名字，还有每行的数据都是对的。

这样，就完成了 excel 的生成。

那我们就可以把 zh-CN.json、en-US.json 等的内容读取出来，然后生成 excel 文件：

把 zh-CN.json 和 en-US.json 复制过来：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f08f285b80ce42d596cb69848fc32aa3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1320&h=448&s=108106&e=png&b=1d1d1d) zh-CN.json

```json
{
    "username": "用户名 <bbb>{name}</bbb>",
    "password": "密码",
    "rememberMe": "记住我",
    "submit": "提交",
    "inputYourUsername": "请输入你的用户名！",
    "inputYourPassword": "请输入你的密码！"
}
```

en-US.json

```json
{
    "username": "Username <bbb>{name}</bbb>",
    "password": "Password",
    "rememberMe": "Remember Me",
    "submit": "Submit",
    "inputYourUsername": "Please input your username!",
    "inputYourPassword": "Please input your password!"
}
```

然后写下 index2.js

```javascript
const { Workbook } = require('exceljs');
const fs = require('node:fs');

const languages = ['zh-CN', 'en-US'];

async function main(){
    const workbook = new Workbook();

    const worksheet = workbook.addWorksheet('test');

    const bundleData = languages.map(item => {
        return JSON.parse(fs.readFileSync(`./${item}.json`));
    })

    const data = [];

    bundleData.forEach((item, index) => {
        for(let key in item) {
            const foundItem = data.find(item => item.id === key);
            if(foundItem) {
                foundItem[languages[index]] = item[key]
            } else {
                data.push({
                    id: key,
                    [languages[index]]: item[key]
                })
            }
        }
    })

    console.log(data);

    worksheet.columns = [
        { header: 'ID', key: 'id', width: 30 },
        ...languages.map(item => {
            return {
                header: item,
                key: item,
                width: 30
            }
        })
    ];

    worksheet.addRows(data);

    workbook.xlsx.writeFile('./bundle.xlsx');    
}

main();
```

这里我们读取了 en-US.json 和 zh-CN.json 的内容，然后按照 id、en-US、zh-CN 的 column 来写入 excel。

跑一下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4ee1bd79a014805828c7e177cdfcd03~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1668&h=1216&s=269749&e=png&b=1a1a1a)

看下生成的 excel：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a7a711d583940998decddf89a5fd58e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1328&h=758&s=107859&e=png&b=fafafa)

随便改一点内容：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9234095170354659b79f9f4028fed1fe~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1272&h=422&s=75252&e=png&b=fefcfc)

然后改完之后要用这个生成 en-US.json 和 zh-CN.json，之后在项目里引入用。

写一下解析 excel 的脚本：

创建 index3.js

```javascript
const { Workbook } = require('exceljs');

async function main(){
    const workbook = new Workbook();

    const workbook2 = await workbook.xlsx.readFile('./bundle.xlsx');

    workbook2.eachSheet((sheet, index1) => {
        console.log('工作表' + index1);

        sheet.eachRow((row, index2) => {
            const rowData = [];
    
            row.eachCell((cell, index3) => {
                rowData.push(cell.value);
            });

            console.log('行' + index2, rowData);
        })
    })
}

main();
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e846baca782e44e7a030edff7f5aa8c7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1426&h=376&s=93770&e=png&b=181818)

解析也是按照 workbook（工作簿） > worksheet（工作表）> row （行）的层次，调用 eachSheet、eachRow、eachCell 就好了。

然后生成 json：

改下 index3.js

```javascript
const { Workbook } = require('exceljs');
const fs = require('node:fs');

async function main(){
    const workbook = new Workbook();

    const workbook2 = await workbook.xlsx.readFile('./bundle.xlsx');

    const zhCNBundle = {};
    const enUSBundle = {};

    workbook2.eachSheet((sheet) => {

        sheet.eachRow((row, index) => {
            if(index === 1) {
                return;
            }
            const key = row.getCell(1).value;
            const zhCNValue = row.getCell(2).value;
            const enUSValue = row.getCell(3).value;

            zhCNBundle[key] = zhCNValue;
            enUSBundle[key] = enUSValue;
        })
    });

    console.log(zhCNBundle);
    console.log(enUSBundle);
    fs.writeFileSync('zh-CN.json', JSON.stringify(zhCNBundle, null, 2));
    fs.writeFileSync('en-US.json', JSON.stringify(enUSBundle, null, 2));
}

main();
```

读取每一行的每一列，放到一个 js 对象里，最后写入文件。

跑一下：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37d6488bb365461c9c2c007d1c830a53~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=940&h=612&s=112543&e=png&b=181818)

这样就把产品经理编辑后的 excel 生成了国际化资源包：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/058c71d07a824309b6cb00f0208f75b0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=918&h=402&s=77614&e=png&b=1f1f1f)

项目里直接用这个资源包就好了。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97d36f01e6f34c35b99abb8fb942215e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1088&h=1014&s=198329&e=png&b=1f1f1f)

现在这样的工作流是可以的，但是不能协同编辑。

如果能够像在线文档一样协同编辑这个 excel 就好了。

可以的，用 google sheets.

打开 google sheets（需要科学上网）： [docs.google.com/spreadsheet…](https://docs.google.com/spreadsheets/ "https://docs.google.com/spreadsheets/")

登录之后创建一个新的 sheet：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3bf083af6064e23a5d76605ca0a7be4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1484&h=828&s=124086&e=png&b=f0f3f4)

它可以导入 csv 格式的文件：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae2e6e14a7324fd1907729ed629ef491~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2708&h=1668&s=1363345&e=gif&f=50&b=fcfcfc)

选择 replace 替换当前工作表：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec73fef7caee45d7ae2151e83950551c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2660&h=1526&s=656029&e=gif&f=34&b=5a5a5a)

这样，就导入了 csv 的数据：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d53d85ce1d2647f89ae730b5a346c403~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1812&h=916&s=278982&e=png&b=fcfcfc)

可以在线编辑了。

把这个 url 分享出去就行。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e0b373daf2c47c6a951fb54d76adf83~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=402&h=182&s=12878&e=png&b=e9f2fb)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0de11b20c6d476d89fe18fcb545be73~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1172&h=942&s=131549&e=png&b=fcfcfc)

比如这个 url：

[docs.google.com/spreadsheet…](https://docs.google.com/spreadsheets/d/1FgCNmoTz9FWuR6Jv1SJ9ioWd2bBfrtRAeoi5CYpmXBA/edit?usp=sharing "https://docs.google.com/spreadsheets/d/1FgCNmoTz9FWuR6Jv1SJ9ioWd2bBfrtRAeoi5CYpmXBA/edit?usp=sharing")

接下来的问题就变成了如何用 node 生成和解析 csv 文件。

这个可以用 csv-parse 和 csv-stringify 来做。

安装 csv-stringify：

```css
npm install --save csv-stringify
```

然后写下 index4.js

```javascript
const { stringify } = require("csv-stringify");
const fs = require('node:fs');

const languages = ['zh-CN', 'en-US'];

async function main(){
    const bundleData = languages.map(item => {
        return JSON.parse(fs.readFileSync(`./${item}.json`));
    })

    const data = [];
    bundleData.forEach((obj, index) => {
        const keys = Object.keys(obj);

        for(let i = 0; i< keys.length; i++) {
            const key = keys[i];

            const foundItem = data.find(item => item.id === key);
            if(foundItem) {
                foundItem[languages[index]] = obj[key]
            } else {
                data.push({
                    id: key,
                    [languages[index]]: obj[key]
                });
            }
        }        
    });

    console.log(data);

    const columns = {
        id: "Message ID",
        'zh-CN': "zh-CN",
        'en-US': "en-US"
    };
      
    stringify(data, { header: true, columns }, function (err, output) {
        fs.writeFileSync("./messages.csv", output);
    });
}

main();
```

定义 columns，准备对应的 data 数组，调用 stringify 来转成 csv 文件。

跑一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ce399d681944835ba0ea447f5f246b8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1866&h=842&s=178242&e=png&b=181818)

可以看到，生成了 message.csv 文件。

然后在 google sheet 里导入：

![2025-01-16 18.10.24.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1ba4802fe1b4bbc92b96aa7bebdc11f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2538&h=1260&s=1738450&e=gif&f=70&b=e6e5e5)

你可以点开这个链接看一下：

[docs.google.com/spreadsheet…](https://docs.google.com/spreadsheets/d/15tYKwXyhKVfe2dm2G28ESjEhd_kuo2-9VMO9HPb6Zfo/edit?usp=sharing "https://docs.google.com/spreadsheets/d/15tYKwXyhKVfe2dm2G28ESjEhd_kuo2-9VMO9HPb6Zfo/edit?usp=sharing")

改一下这个文案：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8eb242503844b3eb416f8fc3bfa68f5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1138&h=680&s=117316&e=png&b=fbfbfb)

然后导出到本地再转成 json 就好了。

怎么导出呢？

在现在的 url 后加一个 export?format=csv 就好了：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12c8a139045e4bd986b656654b646ba3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2570&h=1062&s=301371&e=gif&f=28&b=f8fdfd)

比如这个链接：

[docs.google.com/spreadsheet…](https://docs.google.com/spreadsheets/d/15tYKwXyhKVfe2dm2G28ESjEhd_kuo2-9VMO9HPb6Zfo/export?format=csv "https://docs.google.com/spreadsheets/d/15tYKwXyhKVfe2dm2G28ESjEhd_kuo2-9VMO9HPb6Zfo/export?format=csv")

然后在代码里下载下导出的 csv：

创建 index5.js

```javascript
const { execSync } = require('node:child_process');
const { parse } = require("csv-parse/sync");
const fs = require('node:fs');

const sheetUrl = "https://docs.google.com/spreadsheets/d/15tYKwXyhKVfe2dm2G28ESjEhd_kuo2-9VMO9HPb6Zfo";

execSync(`curl -L ${sheetUrl}/export?format=csv -o ./message2.csv`, {
    stdio: 'ignore'
});

const input = fs.readFileSync("./message2.csv");

const records = parse(input, { columns: true });

console.log(records);
```

这里用 curl 命令来下载，-L 是自动跳转的意思，因为访问这个 url 会跳转一个新的地址。

安装用到的包：

```css
npm install --save-dev csv-parse
```

跑一下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cf3f05d7059420c835d0022dbec8719~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1850&h=934&s=212299&e=png&b=181818)

可以看到，message2.csv 下载了下来，并且还解析出了其中的数据。

接下来用这个生成 zh-CN.json 和 en-US.json，然后在项目里用就好了。

改下 index5.js

```javascript
const { execSync } = require('node:child_process');
const { parse } = require("csv-parse/sync");
const fs = require('node:fs');

const sheetUrl = "https://docs.google.com/spreadsheets/d/15tYKwXyhKVfe2dm2G28ESjEhd_kuo2-9VMO9HPb6Zfo";

execSync(`curl -L ${sheetUrl}/export?format=csv -o ./message2.csv`, {
    stdio: 'ignore'
});

const input = fs.readFileSync("./message2.csv");

const data = parse(input, { columns: true });

const zhCNBundle = {};
const enUSBundle = {};

data.forEach(item => {
    const keys = Object.keys(item);
    const key = item[keys[0]];
    const valueZhCN = item[keys[1]];
    const valueEnUS = item[keys[2]];

    zhCNBundle[key] = valueZhCN;
    enUSBundle[key] = valueEnUS;
})

console.log(zhCNBundle);
console.log(enUSBundle);

fs.writeFileSync('zh-CN.json', JSON.stringify(zhCNBundle, null, 2));
fs.writeFileSync('en-US.json', JSON.stringify(enUSBundle, null, 2));
```

跑一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e815738f3a15412393ac064b96606162~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=990&h=620&s=114788&e=png&b=181818)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e954fedfd47b43a7b9973577b182033c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=916&h=442&s=79894&e=png&b=202020)

这样，就完成了资源包在 google sheet 的在线编辑，以及编辑完以后下载并解析生成资源包的功能。

相比用 exceljs 生成 excel 文件的方式，google sheet 可以把 url 分享出去，可以协同编辑，更方便一点。

> 案例代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/excel-export "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/excel-export")

## 总结

国际化资源包需要交给产品经理去翻译，我们会把 json 转成 excel 交给他。

我们先用 exceljs 实现了 excel 的解析和生成，编辑完之后再转成 en-US.json、zh-CN.json 的资源包。

然后用 google sheet 实现了在线编辑和分享，编辑完之后下载并解析 csv，然后转成 en-US.json、zh-CN.json 的资源包。

用到了 csv-parse、csv-stingify。

这两种方案都可以，确定好方案之后把这些脚本内置到项目里就可以了。