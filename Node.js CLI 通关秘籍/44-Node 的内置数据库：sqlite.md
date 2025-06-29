用 Node.js 写工具的时候，经常会需要存储一些数据。

这些数据应该存在哪里呢？

简单的情况我们会存在文件里，比如用 json 文件存储省市区的数据。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c717f562c53d462089da81cb8b540345~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=878&h=932&s=150420&e=png&b=fafafa)

但如果数据之间有复杂的关联关系呢？

比如十几个对象之间的关系，一对一、一对多、多对多等复杂关系。

这种就得用关系型数据库来存了。

当然，写工具不需要跑一个 mysql 那种数据库服务。

我们会用 sqlite 这种轻量的数据库。

很多手机里的 APP 都会用 sqlite 来存复杂的数据，比如微信里就有很多用 sqlite 存储的数据。

[Node.js 22](https://nodejs.org/docs/latest/api/sqlite.html "https://nodejs.org/docs/latest/api/sqlite.html") 之后，内置了 sqlite 模块，可以直接用：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed6a6f1069f848f0b17ff682ad09d9da~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1760&h=1086&s=214310&e=png&b=fcfcfc)

我们来试一下：

创建项目：

```bash
mkdir sqlite-test
cd sqlite-test

npm init -y
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5db72add0c648d98522376890d3497e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=856&h=666&s=84487&e=png&b=010101)

进入项目，创建 src/index.mjs

```javascript
import { DatabaseSync } from 'node:sqlite';
const database = new DatabaseSync('data.db');

database.exec(`
  CREATE TABLE student(
    id INTEGER PRIMARY KEY,
    name TEXT,
    age INT
  ) STRICT
`);

const insert = database.prepare('INSERT INTO student (id, name, age) VALUES (?, ?, ?)');
insert.run(1, '张三', 20);
insert.run(2, '李四', 21);
insert.run(3, '王五', 22);

const query = database.prepare('SELECT * FROM student ORDER BY id');
console.log(query.all());
```

创建 DatabaseSync 的实例，指定存储的文件位置。

exec 方法是执行 sql

prepare 方法也是准备 sql，其中 ? 是占位符，后面调用 run 方法传入具体的值才会执行。

我们先创建了 student 表，插入了三条数据，然后查询出来。

跑一下：

```css
node --experimental-sqlite ./src/index.mjs
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c37dcd3ba5e44d3e8ae762e2c91bd4e0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1322&h=248&s=56325&e=png&b=181818)

记得要切换下 node 版本到 22 以上再跑。

现在 node 22.12 已经是 LTS（长期支持）版本了：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25a0a06af1d84775acc2660125b2297b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1066&h=866&s=334798&e=png&b=ffffff)

不过要加上 --experimental-sqlite 才行，现在还是实验性的。

执行后创建了表、并插入了几条数据，然后用 sql 查询了出来。

数据都存放在 data.db 这个文件里，二进制存储的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f126785cea14e1698a0008b540e9351~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2178&h=584&s=102932&e=png&b=1d1d1d)

除了用 node 的 api 外，还可以用一些客户端 GUI 工具来管理。

我们安装 DB Browser for SQLite 这个工具。

从 github 下载[安装包](https://github.com/sqlitebrowser/sqlitebrowser/releases/tag/v3.13.1 "https://github.com/sqlitebrowser/sqlitebrowser/releases/tag/v3.13.1")：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4eeaddc86484af5bfc7f5a3c464d498~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1632&h=936&s=200365&e=png&b=ffffff)

安装后打开：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad8dfe48141540f5ad5926aa6b29a268~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=424&h=302&s=57015&e=png&b=ac4410)

点击 open database，选择刚才的文件，就可以看到 db browser 把其中的表解析了出来：

![2024-12-08 10.12.02.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ba6a131e42c4d41a70e8a52c990dd8c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2062&h=1296&s=1289267&e=gif&f=70&b=eeeded)

可以看到所有的表的数据：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/899a1e0dc36f496e90603d5cf3657155~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=564&h=346&s=30898&e=png&b=f4f4f4)

还可以执行 sql，按 cmd + enter 执行：

```sql
select * from student
select * from student WHERE name = "张三"
```

![2024-12-08 10.14.53.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e639f5ed6c9a455292f8d968c49130b8~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2062&h=1296&s=356461&e=gif&f=40&b=f0efef)

我们手动删除一条数据：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f8562818a26482da21bbb56df0f989e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=920&h=478&s=97041&e=png&b=f2f2f2)

右键单击这一列，点击删除。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02e7386947044d79b74c12a1d0f685d2~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=808&h=496&s=71224&e=png&b=ededed)

点击 write changes 按钮，才会把改动写入文件。

然后在代码里查询下：

创建 src/index2.mjs

```javascript
import { DatabaseSync } from 'node:sqlite';
const database = new DatabaseSync('data.db');

const query = database.prepare('SELECT * FROM student ORDER BY id');
console.log(query.all());
```

跑一下：

```css
node --experimental-sqlite ./src/index2.mjs
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/740075ffd0ed4378a7121dd702b2b05c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1376&h=368&s=71290&e=png&b=181818)

可以看到，id 为 2 的数据确实被删除了。

通过 GUI 工具来可视化的管理数据更方便一些。

但最终还是要在代码里读写的。

此外，sqlite 支持在内存中存储数据，指定存储位置为 :memory:

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8714779652d64999ac5e0e3ca676a355~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1238&h=342&s=79877&e=png&b=1f1f1f)

这样表的数据都存在内存里，重新跑就没有了。

考虑到 node:sqlite 还不稳定，我们还是用 sqlite 的 npm 包来写。

首先用下 sqlite3 这个包：

```css
npm install --save sqlite3
```

创建 src/index3.mjs

```javascript
import sqlite3 from 'sqlite3';

const db = new sqlite3.Database(':memory:');

db.serialize(() => {
    db.run(`
    CREATE TABLE student(
        id INTEGER PRIMARY KEY,
        name TEXT,
        age INT
    ) STRICT
    `);

    const stmt = db.prepare('INSERT INTO student (id, name, age) VALUES (?, ?, ?)');
    stmt.run(1, '张三', 20);
    stmt.run(2, '李四', 21);
    stmt.run(3, '王五', 22);
    stmt.finalize();

    db.each('SELECT * FROM student ORDER BY id', (err, row) => {
        console.log(row.id, row.name, row.age);
    });
});

db.close();
```

run 方法是执行 sql

prepare 方法是准备 sql，其中 ? 是占位符，后面调用 run 方法传入具体的值才会执行。

each 方法是查询列表数据，然后依次调用回调函数。

和 node 内置的 sqlite 模块的 api 差不多。

我们也是先创建了 student 表，插入了三条数据，然后查询出来。

数据放在 :memory: 也就是内存里。

跑一下：

```bash
node ./src/index3.mjs
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cdf851a92fd44033a20656499fd45a0e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=716&h=222&s=24672&e=png&b=181818)

api 确实都差不多，可能 node 的 sqlite 模块的 api 也是参考这些包设计的。

再来试下 [sqlite](https://www.npmjs.com/package/sqlite "https://www.npmjs.com/package/sqlite") 这个包：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/007c6675331244afb4822ce69da4b1ee~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1592&h=808&s=171617&e=png&b=fdfcfc)

它是对 sqlite3 包的封装。

因为 sqlite3 这个包的 api 都是回调函数的方式。

而 sqlite 包封装了 promise 版本，也就是可以用 async、await 的方式调用 api。

安装下：

```css
npm install sqlite --save
```

创建 src/index4.mjs

```javascript
import sqlite3 from 'sqlite3'
import { open } from 'sqlite'

async function main() {
    const db = await open({
        filename: 'data.db',
        driver: sqlite3.Database
    });

    const person = await db.get('select * from student WHERE name = :name', {
        ':name': '张三'
    })
    console.log(person);

    const allData = await db.all('SELECT * FROM student');
    console.log(allData);
} 

main();

```

open 是打开数据库存储的文件，指定刚才的 filename，以及用 sqlite3 作为驱动。

查询下 name 为张三的记录，以及全部记录。

跑一下：

```bash
node src/index4.mjs
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca5ae79625cd4a4195448af540f040bc~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1462&h=154&s=40684&e=png&b=181818)

然后再来试下增删改：

创建 src/index5.mjs

```javascript
import sqlite3 from 'sqlite3'
import { open } from 'sqlite'

async function main() {
    const db = await open({
        filename: 'data.db',
        driver: sqlite3.Database
    });

    const insert = await db.prepare('INSERT INTO student (id, name, age) VALUES (?, ?, ?)');
    insert.run(4, '东东', 20);
    insert.run(5, '光光', 21);
    insert.finalize()

    const update = await db.prepare('UPDATE student SET name = ? WHERE id = ?');
    update.run('张三222', 1);
    update.finalize();

    const del = await db.prepare('DELETE FROM student WHERE id = ? ');
    del.run(4);
    del.finalize();

    const allData = await db.all('SELECT * FROM student');
    console.log(allData);
} 

main();

```

跑一下：

```bash
node src/index4.mjs
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f9f66bc92f94c9caa8282eec9e50138~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=850&h=262&s=39111&e=png&b=181818)

可以看到，新增、更改、删除的 sql 都执行成功了。

综上，不管是内置的 sqlite 模块，还是三方的 sqlite3 这个包，或者是封装了一层的 sqlite 包，api 用起来都比较简单。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/sqlite-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/sqlite-test")

## 总结

写 Node.js 工具难免要存储一些数据，简单的数据直接存在 json 文件里就行，但存一些复杂的关系数据的时候就需要用到关系型数据库了。

sqlite 就是一个小型的关系型数据库，可以在内存、单个文件里存储所有的数据，在手机 APP 里用的非常多。

我们分别用了下 node 内置的 sqlite 模块、第三方的 sqlite3、sqlite 包，还有 GUI 工具 DB Browser。

内置的 node:sqlite 模块是 node 22 加入的，还在实验阶段，不够稳定，需要 node --experimental-sqlite 才能跑，所以可以直接用三方的包。

sqlite3 是驱动包，而 sqlite 是封装了一层的包，是 promise 版本的 sqlite3。

当然，这些 api 用起来都很简单。

下节我们用 sqlite 来做一些复杂的关系型数据的存储和管理。