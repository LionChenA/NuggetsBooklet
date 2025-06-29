我们在找工作的时候，都会用 boss 直聘、拉钩之类的 APP 投简历。

根据职位描述筛选出适合自己的来投。

此外，职位描述也是我们简历优化的方向，甚至是平时学习的方向。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64f41bb318ea4ac0ad747e5d79575265~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1578&h=1028&s=271262&e=png&b=fefefe)

所以我觉得招聘网站的职位描述还是挺有价值的，就想把它们都爬取下来存到数据库里。

今天我们一起来实现下。

爬取数据我们使用 Puppeteer 来做，然后把爬到的数据存到 sqlite 表里。

创建个项目：

```bash
mkdir jd-spider
cd jd-spider
npm init -y
```

进入项目，安装 puppeteer：

```css
npm install --save puppeteer
```

我们要爬取的是 boss 直聘的网站数据。

首先，进入[搜索页面](https://www.zhipin.com/web/geek/job?query=%E5%89%8D%E7%AB%AF&city=100010000 "https://www.zhipin.com/web/geek/job?query=%E5%89%8D%E7%AB%AF&city=100010000")，选择全国范围，搜索前端：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3608e2ed5b014e768043743f7b6f7f00~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1942&h=1268&s=1338255&e=gif&f=40&b=fcfcfc)

然后职位列表的每个点进去查看描述，把这个岗位的信息和描述抓取下来：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25f97e689fb3435d884efbdcd298bf9d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1942&h=1268&s=2453118&e=gif&f=37&b=fdfdfd)

创建 test.js

```javascript
import puppeteer from 'puppeteer';

const browser = await puppeteer.launch({
    headless: false,
    defaultViewport: {
        width: 0,
        height: 0
    }
});

const page = await browser.newPage();

await page.goto('https://www.zhipin.com/web/geek/job');

await page.waitForSelector('.job-list-box');

await page.click('.city-label', {
    delay: 500
});

await page.click('.city-list-hot li:first-child', {
    delay: 500
});

await page.focus('.search-input-box input');

await page.keyboard.type('前端', {
    delay: 200
});

await page.click('.search-btn', {
    delay: 1000
});
```

调用 launch 跑一个浏览器实例，指定 headless 为 false 也就是有界面。

defaultView 设置 width、height 为 0 是网页内容充满整个窗口。

然后就是自动化的流程了：

首先进入职位搜索页面，等 job-list-box 这个元素出现之后，也就是列表加载完成了。

就点击城市选择按钮，选择全国。

然后在输入框输入前端，点击搜索。

然后跑一下。

跑之前在 package.json 设置 type 为 module，也就是支持 es module 的 import：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d84b572a639e4c54b2ab475caa175dfd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=526&h=346&s=47857&e=png&b=202020)

```bash
node ./test.js
```

它会自动打开一个浏览器窗口：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b49f9a105fe4b80b6974a4506253d34~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2224&h=1328&s=963516&e=gif&f=41&b=1a1a1a)

然后会执行自动化脚本：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b5eb0ec636c4aae9bab904cae3dca43~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2224&h=1328&s=3039422&e=gif&f=55&b=fefefe)

这样，下面的列表数据就是可以抓取的了。

不过这里其实没必要这么麻烦，因为只要你 url 里带了 city 和 query 的参数，会自动设置为搜索参数：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb3816d3f7e34a10910630998a969310~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1904&h=1294&s=421979&e=png&b=fefefe)

所以直接打开这个 url 就可以：

```javascript
import puppeteer from 'puppeteer';

const browser = await puppeteer.launch({
    headless: false,
    defaultViewport: {
        width: 0,
        height: 0
    }
});

const page = await browser.newPage();

await page.goto('https://www.zhipin.com/web/geek/job?query=前端&city=100010000');

await page.waitForSelector('.job-list-box');
```

然后我们要拿到页数，用来访问列表的每页数据。

怎么拿到页数呢？

其实就是拿 options-pages 的倒数第二个 a 标签的内容：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f65f7ab734a4cf5ae1c3ef69510d665~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1396&h=708&s=181545&e=png&b=ffffff)

```javascript
import puppeteer from 'puppeteer';

const browser = await puppeteer.launch({
    headless: false,
    defaultViewport: {
        width: 0,
        height: 0
    }
});

const page = await browser.newPage();

await page.goto('https://www.zhipin.com/web/geek/job?query=前端&city=100010000');

await page.waitForSelector('.job-list-box');

const res = await page.$eval('.options-pages a:nth-last-child(2)', el => {
    return parseInt(el.textContent)
});

console.log(res);
```

$eval 第一个参数是选择器，第二个参数是对选择出的元素做一些处理后返回。

跑一下：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4551f481f16941ec9d9030172d8edd10~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=740&h=166&s=19530&e=png&b=191919)

页数没问题。

然后接下来就是访问每页的列表数据了。

就是在 url 后再带一个 page 的参数：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca2993be235b46f991bcfc4cc613d4e1~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1360&h=876&s=203355&e=png&b=fefefe)

然后，我们遍历访问每页数据，拿到每个职位的信息：

```javascript
import puppeteer from 'puppeteer';

const browser = await puppeteer.launch({
    headless: false,
    defaultViewport: {
        width: 0,
        height: 0
    }
});

const page = await browser.newPage();

await page.goto('https://www.zhipin.com/web/geek/job?query=前端&city=100010000');

await page.waitForSelector('.job-list-box');

const totalPage = await page.$eval('.options-pages a:nth-last-child(2)', e => {
    return parseInt(e.textContent)
});

const allJobs = [];
for(let i = 1; i <= totalPage; i ++) {
    await page.goto('https://www.zhipin.com/web/geek/job?query=前端&city=100010000&page=' + i);

    await page.waitForSelector('.job-list-box');

    const jobs = await page.$eval('.job-list-box', el => {
        return [...el.querySelectorAll('.job-card-wrapper')].map(item => {
            return {
                job: {
                    name: item.querySelector('.job-name').textContent,
                    area: item.querySelector('.job-area').textContent,
                    salary: item.querySelector('.salary').textContent
                },
                link: item.querySelector('a').href,
                company: {
                    name: item.querySelector('.company-name').textContent,
                }
            }
        })
    });
    allJobs.push(...jobs);
}

console.log(allJobs);
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9fc855414f844db4b4a1d312fd8e9e55~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1520&h=1144&s=267097&e=png&b=1f1f1f)

具体的信息都是从 dom 去拿的：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59e84b44db2845ceba1383ae9bf99a48~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1632&h=866&s=315365&e=png&b=fefefe)

跑一下试试：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff27d24e98bf48b5b3308f9983b59b72~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2224&h=1124&s=2368796&e=gif&f=32&b=fcfcfc)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5234507b1ff644e0932b9b843ed2c1bb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1464&h=1004&s=265647&e=png&b=181818)

可以看到，它会依次打开每一页，然后把职位数据爬取下来。

做到这一步还不够，我们要点进去这个链接，拿到 jd 的描述。

```javascript
for(let i = 0; i< allJobs.length; i ++) {
    await page.goto(allJobs[i].link);

    try{
        await page.waitForSelector('.job-sec-text');

        const jd= await page.$eval('.job-sec-text', el => {
            return el.textContent
        });
        allJobs[i].desc = jd;

        console.log(allJobs[i]);
    } catch(e) {}
}

```

try catch 是因为有的页面可能打开会超时导致中止，这种就直接跳过好了。

跑一下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba3dcdb20d1742b8a40ecd6a0157b748~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2224&h=1124&s=779848&e=gif&f=36&b=2f495d)

它同样会自动打开每个岗位详情页，拿到职位描述的内容，并打印在控制台。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3644003abe114974a64c1e1db1d96f0a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1686&h=910&s=10214544&e=gif&f=43&b=181818)

接下来只要把这些存入数据库就好了。

安装 sqlite：

```css
npm install sqlite sqlite3 --save
```

在 DB Browser 创建个数据库文件：

点击 New Database，在目标位置创建 data.db 文件：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b70ca09304f045e98d53a82bf5bf0e15~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1698&h=1058&s=384139&e=png&b=f4f2f2)

然后填入表结构：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bbd471bfbe64d8abf9756d0b3be63fd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1574&h=1552&s=217398&e=png&b=f3f3f3)

把这个 sql 复制出来：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/587fa697424f4eacba3d331a46552e9c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1574&h=1552&s=217398&e=png&b=f3f3f3)

点击 cancel，我们用代码创建表：

```sql
CREATE TABLE "job" (
	"id"	INTEGER,
	"name"	TEXT,
	"area"	TEXT,
	"salary"	TEXT,
	"link"	TEXT,
	"company"	TEXT,
	"desc"	TEXT,
	PRIMARY KEY("id" AUTOINCREMENT)
);
```

创建 create-table.js

```javascript
import sqlite3 from 'sqlite3'
import { open } from 'sqlite'

async function main() {
    const db = await open({
        filename: 'data.db',
        driver: sqlite3.Database
    });

     await db.exec(`
    CREATE TABLE "job" (
        "id"	INTEGER,
        "name"	TEXT,
        "area"	TEXT,
        "salary"	TEXT,
        "link"	TEXT,
        "company"	TEXT,
        "desc"	TEXT,
        PRIMARY KEY("id" AUTOINCREMENT)
    );    
    `);
} 

main();

```

跑一下:

```lua
node ./create-table.js
```

在 DB Browser 点击 open database，就可以看到这个 job 表：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d26e3a2b8734a8c9f1031da73f80130~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=836&h=620&s=96152&e=png&b=f1f1f1)

然后我们把数据存到数据库里：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/949651f928af4227bd949fa6383b846f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1756&h=1078&s=216527&e=png&b=1f1f1f)

```javascript
import sqlite3 from 'sqlite3'
import { open } from 'sqlite'
```

```javascript
const db = await open({
    filename: 'data.db',
    driver: sqlite3.Database
});

const insert = await db.prepare('INSERT INTO job (name, area, salary,link,company,desc) VALUES (?, ?, ?,?,?,?)');

for(let i = 0; i< allJobs.length; i ++) {
    await page.goto(allJobs[i].link);

    try{
        await page.waitForSelector('.job-sec-text');

        const jd= await page.$eval('.job-sec-text', el => {
            return el.textContent
        });
        allJobs[i].desc = jd;

        console.log(allJobs[i]);
        insert.run(
            allJobs[i].job.name,
            allJobs[i].job.area,
            allJobs[i].job.salary,
            allJobs[i].link,
            allJobs[i].company.name,
            allJobs[i].desc
        );

    } catch(e) {}
}
```

再跑下：

```bash
node ./test.js
```

![2024-12-23 12.54.45.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf0bb3a24cf84cbda35eb5d0c1cb2eae~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2276&h=1358&s=4160830&e=gif&f=70&b=191919)

![2024-12-23 12.55.36.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b18f23d6960848f69f2b13e5b85f0370~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2276&h=1358&s=1867056&e=gif&f=37&b=fdfdfd)

去数据库里看下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/073e39c337fa47289033592422b37493~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1354&h=1038&s=321556&e=png&b=f8f8f8)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1950de92b65644d690f9ba3b7adb6277~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1306&h=972&s=347954&e=png&b=fafafa)

这样，你就可以对这些职位描述做一些搜索，分析之类的了。

比如搜索职位描述中包含 react 的岗位：

```sql
SELECT * FROM `boss-spider`.job where `desc` like "%React%";
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5b3f8476c574578bb62595bb248bed9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1302&h=796&s=159658&e=png&b=f7f7f7)

这样，爬虫就做完了。

不过这个过程中 boss 可能会检测到你访问频率过高，会让你做下是不是真人的验证：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e8590f9018b478eb63031c00a22d30f~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1780&h=1102&s=155397&e=png&b=eef0f5)

这个就是验证码点点就好了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/jd-spider "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/jd-spider")

## 总结

我们通过 puppeteer 实现了对 BOSS 直聘网站的前端职位的爬取，并用 sqlite 把数据保存到了数据库里。

当你想做爬虫或者自动化工具的时候，都可以用 puppeteer 来做。

结合 sqlite 来存储结构化的数据。

可以结合起来做一些工具。