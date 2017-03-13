https://blog.risingstack.com/node-js-best-practices-2017/
#Node.js最佳实践-怎样在2017成为一个更加优秀的Nodejs开发者

一年前我们发表了一篇很成功的文章[怎样在2016成为一个更优秀的Nodejs开发者](https://blog.risingstack.com/how-to-become-a-better-node-js-developer-in-2016/) - 所以我认为现在是一个合适的时间去重温这些话题并为2017做出准备。

在这篇文章中，我们将提及2017年最重要的Nodejs最佳实践，以及你需要关注的话题和要学习的知识。

##2017年Nodejs最佳实践

###使用ES2015
去年我们建议你使用ES2015 - 然而到目前为止，已经有了很大的改变。

那时，Nodejs v4版本还是LTS版本，它已经支持了57%的ES2015方法。一年过去了Node v6版本对ES2015的支持上升到了99%。

如果你使用的是最新的Nodejs LTS版本你可以随心所欲地去使用ES2015的各种特性而不再需要babel。但是就像他们所说的在客户端仍然需要！

你可以登录[node.green](http://node.green/)来获取更多关于Nodejs版本对ES2015新特性的支持信息。

###使用Promises

Promises是concurrency primitive（并发的？），首次出现在80年代。现在基本上成为了所有流行编程语言的一部分，可以帮助你更编程更容易。

想象一段代码，读取一个文件并解析，再打印文件的名字。如果使用callback的话，代码就像下面这样：

```js
fs.readFile('./package.json', 'utf-8', function (err, data) {
    if (err) {
        return console.log(err);
    }

    try {
        JSON.parse(data);
    } catch (ex) {
        return console.log(ex);
    }
    console.log(data.name);
});
```
使用Promises重写这段代码的话可以更具可读性：

```js
fs.readFileAsync('./package.json').then(JSON.parse).then((data) => {
    console.log(data.name);
})
.catch((e) => {
    console.error('error reading/parsing file', e);
});
```
当然，现在对于fs模块来说还没有readFileAsync接口去返回一个Promise方法。你可以引入[promisifyAll](http://bluebirdjs.com/docs/api/promise.promisifyall.html)模块来添加readFileAsync接口。

###使用JavaScript标准样式

当提及到代码风格时，拥有一个公司的代码规范是十分重要的，所以当你需要换项目时，你可以非常高效地从零起步，不需要去根据不同的预设去构造项目的结构。

在RisiStack，我们所有的项目都纳入了[JavaScript Standard Style](https://github.com/feross/standard)编程标准。

使用了编程标准后，我们不需要去再做更多的决定，没有.eslintrc, .jshintrc, .jscsrc文件去管理。你可以在[这里](http://standardjs.com/rules.html#javascript-standard-style)找到我们的标准规范。

###使用Docker - 容器已经做好了投入生产环境的准备了

你可以考虑把Docker镜像作为部署工具了 - Docker容器在一个完整的文件系统中封装了一个包含所有需要运行的文件系统的软件：代码，运行环境，系统工具，系统库 - 任何你可以安装在服务器上的都可以包含在容器中。

####但是为什么需要使用容器呢？

它可以使你的应用独立运行，
良心来说，它可以使你的应用更加安全，
Docker镜像更加轻量级，
支持不可变部署，
拥有了以上特性，你就可以保存生产镜像。

在使用Docker之前，可以先阅读官方入门教程。同样的，我们也建议你阅读我们的[Kubernetes best practices](https://blog.risingstack.com/moving-node-js-from-paas-to-kubernetes-tutorial/)这篇文章。

###监控你的应用

如果你的Nodejs应用发生异常，你应该第一时间知道，而不是你的客户们。

一个可以帮助你实现监听程序的比较新的开源解决方案是[Prometheus](https://prometheus.io/)。Prometheus是一个构建于SoundCloud的开源系统，可以实现监听并发送提醒。唯一的缺点就是你需要自己去搭建并设置。

如果你需要一个开箱即用的解决方案，[Trace by RisingStack](https://trace.risingstack.com/)是我们研发的一款极佳的监听系统。

Trace会帮助你：
报警
生产系统中的内存和CPU分析
分布式跟踪和错误搜索
性能检测
保证你的npm模块安全

###后台进程使用消息传递

如果你使用HTTP来发送消息，那么当接收方宕机的时候，你所有的消息都会丢失。如果你选择一个持久传输层，像使用消息队列去发送消息，你就不会出现这样的问题。

如果接收服务器宕机的话，这些消息将会被保存下来，可以稍后处理。如果接收服务器并没有宕机，只是出现了问题的话，这些任务将会被重新执行，将不会有数据丢失。

例如：将要发送成千上万的email。在这中情况下，你只需要把一些基本信息，如email地址和收件人的名字，后台将很轻松的把email的内容整合在一起并发送。

这种方法真正的优点是，只要你愿意，你就可以缩放它，不会丢失任何任务。如果你要发送数百万的email，你可以添加额外的进程，这些进程可以消耗相同的队列。

对于消息队列，你可以有很多的选项：
[RabbitMQ](https://www.rabbitmq.com/)
[Kafka](https://kafka.apache.org/)
[NSQ](http://nsq.io/)
[AWS SQS](https://aws.amazon.com/cn/sqs/)

###使用最新的LTS Nodejs版本

为了获得最佳的稳定性和新特性，我们推荐你使用最新的LTS（长期支持）的Nodejs版本。当写这篇文章的时候，最新版本是6.9.2.

为了很方便地切换Nodejs版本，你可以使用[nvm](https://github.com/creationix/nvm)。当你成功安装了nvm的时候，切换LTS版本只需要两个命令：

```js
nvm install 6.9.2
nvm use 6.9.2
```

###使用语义化版本

几个月前我们进行了[Nodejs 开发者调查](https://blog.risingstack.com/node-js-developer-survey-results-2016/),这让我们了解对开发者们如何使用语义版本的一些见解。
不幸的是，我们发现只有71%的受访人群在发布模块的时候会使用语义化版本。我认为这个数字应该更高，每个人都应该使用语义化版本！为什么呢？因为在我们更新模块的时候而没有semver将会很容易击垮我们的Nodejs应用。
![image](https://blog-assets.risingstack.com/2016/Sep/node-js-survey/node-js-survey-semantic-versioning.png)<br/>

版本化你的应用/模块是非常重要的 - 当你发布一个新版本时，你的用户必须知道他们应该做些什么去兼容这些新版本。
这就是语义化版本的来源，给定一个版本号MAJOR.MINOR.PATCH等描述：

主版本号(MAJOR) 当你对API进行一些不兼容的修改
次版本号(MINOR) 当你添加一些新的功能（并不影响API）
修订号(PATCH) 当你做一些向后兼容的bug修复

npm也使用SemVer当安装你的依赖项时，所以当你发布模块时，一定要重视起来。否则，你将会影响到其他的应用。

###确保你的应用程序的安全

确保你的用户的数据安全应该是2017年的头等大事。在2016年中，因为不重视程序安全造成数百万用户信息泄露。
要开始确保Nodejs安全，可以阅读我们的文章[Node.js Security Checklist](https://blog.risingstack.com/node-js-security-checklist/),其中涵盖了：

加密HTTP请求头
Brute Force Protection
session管理
不安全的依赖
数据校验

在你掌握了这些基本信息之后，你可以来看看我的演讲[Surviving Web Security with Node.js!](https://www.youtube.com/watch?v=80LbyikAUqI)

###学习Serverless

Serverless源于AWS Lambda，从诞生起就快速生长，并拥有一个爆炸生长的开源社区。
在接下来的几年里，serverless将成为构建新应用的关键因素，如果你想站在技术的前沿，快去学习serverless吧。
一个最为流行的框架是[Serverless Framework](https://serverless.com/),这将有助于部署AWS Lanbda函数。

###参加聚会或者会议并发言

参加会议或者聚会是一个非常棒的方式去学习新的潮流。use-cases或者最佳实践。同样也是个非常棒的方式去结交新的朋友。
更进一步，我强烈建议你在这些会议中发言[one of these events](https://github.com/watson/conferences)!
在公共场合发表演讲是艰难的，因为“想象所有的人都是裸体的”是最坏的建议，所以建议你上[speaking.io](http://speaking.io/)去寻找一些演讲的小技巧！

###在2017年成为一个更加优秀的Nodejs开发者

2017年将成为Nodejs的一年，我们将帮助你从中得到更多知识！
我们刚刚推出了一个新的项目["Owning Node.js"](https://risingstack.com/owning-nodejs),它将帮助你在这些问题中更加自信：

使用Nodejs进行异步编程
使用Express创建应用
使用Nodejs来操作数据库
构建项目和可扩展的应用程序