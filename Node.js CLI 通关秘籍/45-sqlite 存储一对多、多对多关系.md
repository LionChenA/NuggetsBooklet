上节学了 sqlite，做工具的时候需要存储复杂的关系数据，就可以用它。

不过我们上节只是做了单表的增删改查，比较简单。

一般用到 sqlite 的场景都是多表的关联，比如一对多、多对多的关系。

一对多关系在生活中随处可见：

一个作者可以写多篇文章，而每篇文章只属于一个作者。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/652efb6bbd7d4af1945b0fe912bd2d91~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=740&h=576&s=34725&e=png&b=ffffff)

一个订单有多个商品，而商品只属于一个订单。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37e82479d7ae4e7c88c4df4cf040c2d5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=806&h=568&s=33102&e=png&b=ffffff)

一个部门有多个员工，员工只属于一个部门。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f79387f8934e468ca0406b7aca4ebfb6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=814&h=596&s=32816&e=png&b=ffffff)

多对多的关系也是随处可见：

一篇文章可以有多个标签，一个标签可以多篇文章都有。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81b2e6cef1fc4480b98a760bd61ad21a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=732&h=550&s=39004&e=png&b=ffffff)

一个学生可以选修多门课程，一门课程可以被多个学生选修。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a409a01fd32a431c9b378f4ebc7a12f6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=768&h=580&s=44439&e=png&b=ffffff)

一个用户可以有多个角色，一个角色可能多个用户都有。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a70416ef1fbb49c393791a640417a217~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=772&h=570&s=43090&e=png&b=ffffff)

这种就叫做复杂的关系。

当然，如果只是两个表之间的关系，你可能觉得不复杂，如果是有多个表、每个表之间都是一对多、多对多的关系呢？

这种错综复杂的关系，再用 json 存储显然就不合适了。

那在数据库里如何存储这种关系呢？

我们分别来看一下：

一对多的关系，比如一个部门有多个员工。

我们会有一个部门表和一个员工表：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9f5d802c0ba43edb12f9f42b2b1a164~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=838&h=432&s=23460&e=png&b=ffffff)

在员工表添加外键 department\_id 来表明这种多对一关系：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64756ca65ed24454a697dd63ad464665~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=892&h=536&s=37993&e=png&b=ffffff)

其实和一对一关系的数据表设计是一样的。

我们在 DB Browser 添加这两个表。

点击 create database，把数据存在 sqlite-test 的 1-many.db 文件里：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9166d683097d4730adb23b1624e57de0~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1532&h=718&s=282987&e=png&b=f3f2f2)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5bdb546a849244028c3d77482a26c5ee~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1388&h=986&s=136638&e=png&b=f1f0f0)

填入表名 department 和两个列 id、name

指定 id 是 INTEGER 类型，约束为 primary key（主键）、not null（非空）、 auto increment（自动递增）。

name 是 TEXT 类型。

点击 ok，可以看到表已经创建好了：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5deebc719a9f412289c7bb1d058233ad~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=850&h=346&s=56543&e=png&b=eeeeee)

同样的方式创建 employee 表：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad443864c74d442a92fa10ec367d7f7b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1326&h=936&s=135716&e=png&b=f2f2f2)

添加 id、name、department\_id 这 3 列。

然后添加一个外键约束，department\_id 列引用 department 的 id 列。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b3710167ec14b52982936c8a5a15bc5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1530&h=1014&s=142705&e=png&b=f3f3f3)

点击 ok。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b22389a00d19459a86ad53dd0a9984fd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=700&h=354&s=47613&e=png&b=f0f0f0)

employee 表也创建成功了。

点击 write changes 把改动写入文件。

然后我们在代码里跑下插入数据的 sql：

创建 src/1-many-insert.mjs

```javascript
import sqlite3 from 'sqlite3'
import { open } from 'sqlite'

async function main() {
    const db = await open({
        filename: '1-many.db',
        driver: sqlite3.Database
    });

    const insert = await db.prepare('INSERT INTO department (id, name) VALUES (?, ?)');
    insert.run(1, '人事部');
    insert.run(2, '财务部'),
    insert.run(3, '市场部'),
    insert.run(4, '技术部'),
    insert.run(5, '销售部'),
    insert.run(6, '客服部'),
    insert.run(7, '采购部'),
    insert.run(8, '行政部'),
    insert.run(9, '品控部'),
    insert.run(10, '研发部');
    insert.finalize()

    const insert2 = await db.prepare('INSERT INTO employee(id, name, department_id) VALUES (?, ?, ?)');
    insert2.run(1, '张三', 1);
    insert2.run(2, '李四', 2); 
    insert2.run(3, '王五', 3);
    insert2.run(4, '赵六', 4);
    insert2.run(5, '钱七', 5);
    insert2.run(6, '孙八', 5);
    insert2.run(7, '周九', 5);
    insert2.run(8, '吴十', 8);
    insert2.run(9, '郑十一', 9);
    insert2.run(10, '王十二', 10);
    insert2.finalize();
} 

main();
```

分别往 department 和 employee 表插入了一些数据。

跑一下：

```bash
node ./src/1-many-insert.mjs
```

之后去 DB Browser 里看下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce50cda1b60b4e92b3942208831f391c~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=730&h=374&s=98091&e=png&b=ececec)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1649d6f5a44043d98ed20c60803437b9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=922&h=790&s=98018&e=png&b=f6f6f6)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df94e702a70541e4a27a4bf4634311b4~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=924&h=810&s=93141&e=png&b=f7f7f7)

两个表的数据都插入成功了。

那如果要查询 id 为 5 的部门的所有员工呢？

这种就涉及到关联查询了：

用 JOIN ON 来关联查询下：

```sql
select * from department
    join employee on department.id = employee.department_id
    where department.id = 5
```

可以看到，正确查找出了销售部的 3 个员工：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/398049c1ca9c44f5a582968e2d313837~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=920&h=750&s=116595&e=png&b=f2f2f2)

这就是一对多。

当然，从创建表到执行这些 sql 都是可以在代码里做的，可以在这里复制建表语句：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce615181df2c45be90466de60ad9cb18~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=756&h=390&s=90589&e=png&b=f0f0f0)

接下来我们来看多对多。

比如文章和标签：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81b2e6cef1fc4480b98a760bd61ad21a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=732&h=550&s=39004&e=png&b=ffffff)

之前一对多关系是通过在多的一方添加外键来引用一的一方的 id。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64756ca65ed24454a697dd63ad464665~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=892&h=536&s=37993&e=png&b=ffffff)

但是现在是多对多了，每一方都是多的一方。这时候是不是双方都要添加外键呢？

一般我们是这样设计：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d79e2c5f5ec48a8952acbfb8803f986~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.image#?w=1098&h=516&s=45503&e=png&b=ffffff)

文章一个表、标签一个表，这两个表都不保存外键，然后添加一个中间表来保存双方的外键。

这样文章和标签的关联关系就都被保存到了这个中间表里。

先创建文章表：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff28bc768510424288e1a5fc005d8a7d~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1346&h=1016&s=134746&e=png&b=f1f1f1)

看下创建的表：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a81035262f744720b6250079ce66f9fa~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=668&h=334&s=40734&e=png&b=f1f1f1)

然后创建标签表：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c735bd8bc51b4dcfb3d33cc0c7f8bf7e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1168&h=930&s=108510&e=png&b=f0f0f0)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f661f67b70874e98a33193cf80cd214a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=406&h=352&s=33305&e=png&b=fdfdfd)

之后加一个中间表：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0e0f206415d4968b0a5c60fb1d8b6f3~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1418&h=950&s=130261&e=png&b=f1f1f1)

这里同时指定这两列为 primary key，也就是复合主键。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4d438866a6641838028daac37f24ced~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1432&h=900&s=140322&e=png&b=f2f2f2)

添加 article\_id 和 tag\_id 的外键引用：

article\_id 引用 article 表的 id、tag\_id 引用 tag 表的 id。

点击 ok 创建表。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/739305c2828d4cb183f670210b89b349~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=562&h=388&s=40266&e=png&b=fbfbfb)

三个表都创建好了，可以插入数据了。

点击 write changes，把改动写入文件：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33fa047a7589496a936bcf9629406d0e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1312&h=702&s=156796&e=png&b=f3f2f2)

我们还是用代码来插入数据：

创建 src/many-many-insert.mjs

```javascript
import sqlite3 from 'sqlite3'
import { open } from 'sqlite'

async function main() {
    const db = await open({
        filename: '1-many.db',
        driver: sqlite3.Database
    });

    const insert = await db.prepare('INSERT INTO article (id, title, content) VALUES (?, ?, ?)');
    insert.run(1, '文章1', '这是文章1的内容。');
    insert.run(2, '文章2', '这是文章2的内容。');
    insert.run(3, '文章3', '这是文章3的内容。');
    insert.run(4, '文章4', '这是文章4的内容。');
    insert.run(5, '文章5', '这是文章5的内容。');
    insert.finalize();

    const insert2 = await db.prepare('INSERT INTO tag (id, name) VALUES (?, ?)');
    insert2.run(1, '标签一');
    insert2.run(2, '标签二'),
    insert2.run(3, '标签三'),
    insert2.run(4, '标签四'),
    insert2.run(5, '标签五'),
    insert2.finalize()

    const insert3 = await db.prepare('INSERT INTO article_tag(article_id, tag_id) VALUES (?, ?)');
    [
        [1,1], [1,2], [1,3],
        [2,2], [2,3], [2,4],
        [3,3], [3,4], [3,5],
        [4,4], [4,5], [4,1],
        [5,5], [5,1], [5,2]
    ].forEach(item => {
        insert3.run(item[0], item[1]);
    })
    insert3.finalize();
} 

main();
```

跑一下：

```bash
node ./src/many-many-insert.mjs
```

在 DB Browser 里看下：

![2024-12-16 11.32.17.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a17edb6d27df4fb981154dd64d2e48a6~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=1426&h=786&s=552628&e=gif&f=40&b=f5f5f5)

都插入成功了。

那现在有了 article、tag、article\_tag 3 个表了，怎么关联查询呢？

JOIN 3 个表呀！

```sql
SELECT * FROM article a 
    JOIN article_tag at ON a.id = at.article_id
    JOIN tag t ON t.id = at.tag_id
    WHERE a.id = 1
```

这样查询出的就是 id 为 1 的 article 的所有标签。

创建 src/many-many-query.mjs

```javascript
import sqlite3 from 'sqlite3'
import { open } from 'sqlite'

async function main() {
    const db = await open({
        filename: '1-many.db',
        driver: sqlite3.Database
    });

    const allData = await db.all(`
    SELECT * FROM article a 
    JOIN article_tag at ON a.id = at.article_id
    JOIN tag t ON t.id = at.tag_id
    WHERE a.id = 1    
    `);
    console.log(allData);

} 

main();
```

跑一下：

```bash
node ./src/many-many-query.mjs
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/166870d786c145bfb4c751374afff71b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1226&h=1136&s=128593&e=png&b=181818)

这样，一对多、多对多这种复杂关系的保存、新增、查询就完成了。

修改、删除和上节的单表 CRUD 一样，就不测试了。

> 代码上传了[小册仓库](https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/sqlite-test "https://github.com/QuarkGluonPlasma/nodejs-course-code/tree/main/sqlite-test")

## 总结

这节我们学了用 sqlite 存储复杂关系，也就是一对多、多对多关系。

我们创建了部门、员工表，并在员工表添加了引用部门 id 的外键 department\_id 来保存这种一对多关系。

创建了文章表、标签表、文章标签表来保存多对多关系，多对多不需要在双方保存彼此的外键，只要在中间表里维护这种关系即可。

关联多个表的查询需要用 join on，多对多的 join 需要连接 3 个表来查询。

当你用 sqlite 存储复杂的关系数据的时候，就可以用 sql 来做 CRUD 了。

等之后 node:sqlite 这个内置模块稳定了，就可以不用三方包来写了，但用法一样。