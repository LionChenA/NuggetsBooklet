最近一些 AI 生成前端代码的应用挺火的。

比如 [bolt.new](https://bolt.new/ "https://bolt.new/")

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa0d36834d7f48bb9892af24ce5d5b02~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2308&h=1162&s=440815&e=png&b=0c0c0c)

我告诉他写个菜谱大全网站，用 react、ts 写。

它会依次生成这个项目的全部代码：

![2024-11-19 17.51.24.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f95ae6675c0e45dabb753d1ee09d5f00~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2332&h=1256&s=2754167&e=gif&f=70&b=111111)

可以看到，它的流程是这样的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8c3584afa074f929fb7be5625e8952e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2218&h=1140&s=466453&e=png&b=161616)

先生成 package.json，然后执行 npm install。

依次生成所有的文件。

然后跑下 npm run dev：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/345b55decc5440dcafb4f5b3fe5a8321~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2312&h=1188&s=1279930&e=png&b=1b1b1b)

这样就可以在右侧看到生成的代码和预览效果了：

![2024-11-19 18.01.24.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fda3111166e499492e5df74443c70cb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2602&h=1456&s=5567385&e=gif&f=42&b=171717)

交互啥的都没问题。

之后点击部署：

![2024-11-19 17.56.30.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76313465a8e844a59ec8af8509376b39~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2332&h=1256&s=2539495&e=gif&f=70&b=191919)

它会部署到服务器，然后给你一个 url。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d530569235584fc996722a69de004deb~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2020&h=1236&s=952302&e=png&b=1b1b1b)

打开看下：

[gleeful-sundae-df68f2.netlify.app/](https://gleeful-sundae-df68f2.netlify.app/ "https://gleeful-sundae-df68f2.netlify.app/")

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1029e8aa6f82483ba2dde87c2f3423fe~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2602&h=1456&s=10914305&e=gif&f=63&b=f4f6f8)

这样一个菜谱网站就给写完并部署好了。

是不是很震撼！

要一个月薪 2 万的前端工程师写，也得要至少一天吧。

但是 AI 只要一分钟就给生成然后部署好了。

当然，我的需求比较简单，你完全可以描述更细致的需求，让 AI 给你生成。

生成后也可以再提需求，让它继续完善。

或者你也可以基于这些代码自己改：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14385b1634f74d54bb4c538931ec2910~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1644&h=1154&s=275902&e=png&b=161616)

这对于工作效率的提高，比任何基建都好用。

那我们能不能在 CLI 里实现一套呢？

可以的，具体方案下节再说，这节我们先来看看 bolt.new 是怎么做到的：

bolt.new 这种 AI 生成前端代码并在浏览器里编译、运行的功能，实现原理是什么呢？

我们简单分析下。

首先我们需要调用 AI 接口，告诉它需求，让它给生成代码。

这里看下 cursor 用的模型：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a8e64efe4d44ac2a42b190439cd1459~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1374&h=610&s=100258&e=png&b=191919)

默认是 claude-3.5-sonnet 这个。

国内调用国外的 ai 接口有重重阻碍。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/085c6eef1bf74862a0f788657f724e65~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1208&h=650&s=119890&e=png&b=f5f5f5)

所以我们一般都是找个国内的代理商的 api 来用。

这种代理挺多的，我这里用的 [302.ai](https://gpt302.saaslink.net/bzgo6g "https://gpt302.saaslink.net/bzgo6g")，你也可以用别的

这种代理都是给你对接好了国内、国外各种模型，你可以直接用：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e9fb7a94c214369a1448abca1c39957~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1820&h=660&s=131831&e=png&b=fcfcfc)

刚才 cursor 用的那个模型也有。

我这里充了 5 美元，也就是 40 人民币，可以用支付宝支付，这点确实方便：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e10060253384a34b2658610b566a62a~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1868&h=920&s=142264&e=png&b=77797d)

国外的那些模型，你支付都是个问题。

所以还是调用 AI 接口，还是直接用国内的代理香。

创建个聊天机器人，指定 cursor 用的那个模型：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef8f6b8c795d4a4ba7e01bfe1f239113~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1762&h=1378&s=257453&e=png&b=fffefe)

让它给生成一个菜谱网站：

生成一个菜谱大全网站，用 react、ts 写，先用 json 形式给我项目目录，然后依次给我每个文件的内容

首先，目录是这样的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93b92da9ad034bd8bbffd267b72ba2fd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1754&h=962&s=181933&e=png&b=fdfdfd)

然后依次给我了每个文件的内容：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da37bc8e46e1489786080839ae1d5a25~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1742&h=976&s=160368&e=png&b=fdfdfd)

你也可以分别询问每个文件的内容：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad74f06ba9704af3ab7b0f9fbed40f07~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1772&h=1028&s=199463&e=png&b=fdfdfd)

bolt.new 就是这样新生成了目录，然后依次生成的每个文件：

![2024-11-19 17.51.24.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f95ae6675c0e45dabb753d1ee09d5f00~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.gif#?w=2332&h=1256&s=2754167&e=gif&f=70&b=111111)

我们在项目里怎么调用这个 AI 接口呢？

这样：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0f21134da794007971d552e35e8bdcd~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2580&h=1340&s=274617&e=png&b=ffffff)

创建一个 api key。

然后看下调用接口的文档：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5aaf58327df4263b8d373806f81956e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1744&h=1242&s=237785&e=png&b=ffffff)

把示例代码复制下来：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d35845cb689d4c4aa51df97fea6f9048~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1646&h=1434&s=346813&e=png&b=20293a)

我们创建个测试项目：

```bash
mkdir ai-test
cd ai-test
npm init -y
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52dace1579cf4e4fbbb040710270cb2e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=838&h=664&s=120473&e=png&b=000000)

创建 index.js

```javascript
var axios = require('axios');
var data = JSON.stringify({
   "model": "claude-3-5-sonnet-20241022",
   "messages": [
      {
         "role": "user",
         "content": "生成一个菜谱大全网站，用 react、ts 写，用 json 形式返回，key 是文件目录，value 是文件内容"
      }
   ]
});

var config = {
   method: 'post',
   url: 'https://api.302.ai/v1/chat/completions',
   headers: { 
      'Accept': 'application/json', 
      'Authorization': 'Bearer API_KEY放这里', 
      'Content-Type': 'application/json'
   },
   data : data
};

axios(config)
.then(function (response) {
   console.log(JSON.stringify(response.data, null, 4));
})
.catch(function (error) {
   console.log(error);
});
```

填入刚才的 api key，content 那里输入内容。

安装依赖，然后跑一下：

```bash
npm install --save axios
node ./index.js
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b93ae3df862f4c58bbae7804d1d3916b~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1894&h=842&s=225498&e=png&b=191919)

我们打印下这部分：

```javascript
console.log(response.data.choices[0].message.content)
```

和我们在网页那里问的结果一样：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d3a80e5fe2742e592910e2bc7c9e526~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1388&h=1172&s=142545&e=png&b=181818)

我们可以用 remark 来 parse 出 AST，拿到这部分代码：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db0ea3bec8b74e86ad5fc917d043a556~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2296&h=1360&s=496341&e=png&b=f8f3f2)

在 [astexpoler.net](https://astexplorer.net/#/gist/c2092957576e4fe0343db18940c6338e/638dcac8116133edf8f0d977175a73a24afcaf32 "https://astexplorer.net/#/gist/c2092957576e4fe0343db18940c6338e/638dcac8116133edf8f0d977175a73a24afcaf32") 可以看到 AST。

然后把这些依次写到磁盘里就好了。

加上 typescript、vite、tailwind 等基础代码，就可以跑了。

那 bolt.new 在浏览器里执行 npm install、npm run dev，是怎么做到的呢？

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8c3584afa074f929fb7be5625e8952e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2218&h=1140&s=466453&e=png&b=161616)

这是用他家之前出的 [WebContainer](https://webcontainers.io/ "https://webcontainers.io/") 来做的。

前几年刚出的时候火过一段时间，不知道大家还有印象没：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc92b4711bdf446bae79a08d5a7c8e9e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2648&h=1432&s=1623840&e=png&b=f7f7f7)

就是基于 wasm 实现的浏览器里的 node 运行时，也可以跑 npm。

所以自然就可以执行 npm install、npm run dev，跑各种 node 代码。

个人用、不商用是免费的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ffecf7e8c514264917c399f52d55d97~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2478&h=978&s=411845&e=png&b=16171b)

那怎么用呢？

官方有个 demo：

[stackblitz.com/edit/stackb…](https://stackblitz.com/edit/stackblitz-webcontainer-api-starter-fn1dzd "https://stackblitz.com/edit/stackblitz-webcontainer-api-starter-fn1dzd")

最终效果就是在浏览器里跑起来了 express 服务：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b0f3062579d49b99f326b9154ab2ef7~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2836&h=1280&s=559236&e=png&b=1b1e25)

api 是这么用的：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c50aa9506bd344e7b80ec22eec4d7767~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=822&h=632&s=130971&e=png&b=15181e)

调用 @webcontainer/api 包的 WebContainer 的 api。

首先调用 boot，把它跑起来。

然后调用 mount 传入一些 files。

这里有两个 file：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9f92cda5aac4fb8986eff43b0289080~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1062&h=1170&s=166533&e=png&b=15181e)

一个 package.json，一个 index.js

后面还可以调用 fs 的 api 来写入文件：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4435523316204fcf8f398fc3a16f0ef5~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=940&h=116&s=30357&e=png&b=15181e)

总之，就是 webcontainer 里有一套文件系统。

然后有了 package.json 就可以跑 npm install 了：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc81b3e0af48467bafaa242e629034ba~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1054&h=826&s=160601&e=png&b=15181e)

调用 spawn 来执行命令，这里跑下 npm install。

然后结果可以在 WriteStream 里拿到。

这里只是打印了下，后面可以接入 xtrerm 在浏览器里显示终端。

也就是在这个东西：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03c40ad09455404c9dfa4db4299890e9~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=2282&h=1292&s=466524&e=png&b=15181e)

之后继续执行 npm run start 的命令就好了

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d73264ef9f074fe2a7b41382dd425c1e~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1040&h=490&s=92395&e=png&b=15181e)

这里是通过 iframe.src 来访问的这个 express 服务。

这样，我们就通过 @webcontainer/api 在浏览器里跑了一个 express。

其实 WebContaienr 用起来还是很简单的，

这俩都是他家的：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7f79aad3a8247729c275712de177771~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1724&h=230&s=151784&e=png&b=141519)

那自然会在实现 bolt.new 的时候用到 WebContainer 在浏览器里做构建。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70a4f761906246eaabda9ac1702e9e85~tplv-k3u1fbpfcp-jj-mark:1600:0:0:0:q75.jpg#?w=1938&h=1056&s=292109&e=png&b=181818)

那之后的部署呢？

在浏览器里跑完 npm run build 之后，拿到 dist 目录上传到服务器就好了。

这就是 bolt.new 的大概实现原理。

## 总结

今天我们了解了一下 bolt.new，它可以通过文本描述需求，让 AI 去生成前端项目的代码，然后在 webcontainer 里跑 npm install 和 npm run build 等，把产物部署到服务器上，之后直接通过 url 访问 AI 生成的代码的效果。

我们简单试了一下这个流程，首先我们用的国内的一个 AI 代理商的接口来试了下代码的生成，用的 cursor 一样的模型，生成的目录和代码都是可以直接跑的。

然后学习了下 @webcontainer/api，它是一个基于 wasm 实现的浏览器里的 node 环境，可以往其中写入一些文件，然后跑各种 node 脚本。

之后我们还可以在服务器上加一个接口来接收编译后的代码，实现部署。

把这个流程连起来，就是 bolt.new 的实现原理。

当然，具体细节肯定还有很多，但思路就是这样的。

那 CLI 里能不能实现这样的功能呢？下节我们来探讨下。