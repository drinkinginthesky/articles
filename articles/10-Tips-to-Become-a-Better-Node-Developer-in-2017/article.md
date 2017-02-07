https://www.sitepoint.com/10-tips-to-become-a-better-node-developer/?utm_source=nodeweekly&utm_medium=email#avoidblockingrequire
# 10个技巧让你在2017年成为一个更优秀的Node开发者

###这篇文章由客座作者Azat Mardan编写。SitePoint旨在给你带来web社区优秀的作家、演讲者的文章。

NOTE：这篇文章原始的标题是“来自平台大师的Node最佳实践”。这篇文章包含真实，尝试和测试模式，尽管并不是2017最新的和最好的，但是这些大师们的优秀实践同样适用于2017、2018甚至2019，本文并不包含像async/await,promise等新特性。因为这些新特性并不包含在Node的核心代码或像npm、Express等流行项目之中，文章的第二部分将反射出内容的适当的特性。

在2012年，我加入了Storify公司，开始了Node全栈生涯。从那时开始，我便从来没有后悔过放弃过去十年中使用过的语言，比如Python，Ruby，Java，PHP。

Storify对我来说是一个有趣的工作，因为不像其他公司一样，Storify所有的项目都运行在JavaScript上（也许现在仍是这样）。正如你所知的，所有的公司，尤其像PayPal，Walmart或Capital One等大公司，只使用Node作为技术栈的一部分。一般使用Node来提供API或者用在业务层。这很棒，但是作为一名软件工程师，没有什么比完全使用Node更棒的了。

在这一部分，我将列出十条建议能帮助你在2017成为一名更棒的Node开发者。这些建议一部分来自我工作中趟过的一些坑，另一些是从那些写了最流行的Node和npm模块的人们身上学到的：

1.避免复杂性 -- 尽可能将你的代码块拆分到最小，要小到极致。
2.使用异步代码 -- 要像躲避瘟疫一样避免同步代码。
3.避免块级引用 -- 要将所有的require放在文件头部，因为require行为是同步的，将阻塞你代码的执行。
4.要知道require是可以被缓存的 -- 这将是一把双刃剑，使用得当将有助于代码，否则将造成bug。
5.要始终检查错误 -- 错误并不像足球一样，永远不要抛出错误或者忽略错误检查。
6.只在同步代码中使用try...catch语句 -- try...catch语句对于异步代码是无效的，V8引擎无法优化try...catch语句。
7.return callback或者使用if...else语句 -- 执行callback代码的时候一定要执行return语句，防止代码继续执行。
8.监听异常事件 -- 几乎所有的Node class/object都具有观察者模式，并会广播错误事件。确保你监听了错误事件。
9.了解你的npm -- 通过-s或者-d来代替--save和--save-dev来安装模块。
10.在packa.json中使用明确的版本号 -- 当你使用-s时npm添加默认的模块，所以摆脱这些束缚，人工指定版本除了开源模块，永远不要在你的应用之中相信这些。
11.附加项 --

##避免复杂性

来看一看由npm作者Isaac Z. Schlueter写的一些模块。例如，使用'use strict'强迫使模块使用严格模式，仅仅需要三行代码：

```js
var module = require('module');
module.wrapper[0] += '"use strict";'
Object.freeze(module.warp)
```

为什么要避免程序的复杂性呢？美国海军有一个著名的短语’KISS‘， KEEP IT SIMPLE STUPID。事实证明，人类的大脑在工作时只能记住5-7个项目。

使你的代码模块化至更小的部分，使你和其他开发者能够更好的了理解这些模块，你也可以更方便的测试这些模块。看看下面这个例子：

```js
app.use(function (req, res, next) {
    if (req.session.admin === true) return next()
    else return next(new Error('Not authorized'))
}, function (req, res, next) {
    req.db = db
    next()
})
```

或者像下面这些代码：
```js
const auth = require('./middleware/auth.js')
const db = require('./middleware/db.js')

app.use(auth, db)
```

我敢肯定你们大部分人都会选择第二个例子，尤其是这些名字明确的描述了该模块的作用。当然，当你写下这些代码的时候你会知道这些代码是如何工作的。你甚至将几个方法放在一行中来展示你是多么机智。但事实上你写了一些愚蠢的代码。想象六个月以后你再读这些代码，就像是喝醉了之后写的。如果你是在你智商的巅峰写下这些代码，那么你以后将会很难去读懂这些代码，更不用说不懂你算法复杂性的同事了。保持代码简洁，尤其是在Node中使用异步的代码。

当然也会有left-pad 事件，但是其实它只是影响了依赖于left-pad模块的项目而且11分钟后就发布了替代品。代码的最小化带来的好处超过了它的缺点。npm已经改变了发布策略,任何重要的项目都应该使用缓存或私有的源（作为临时解决方案）。

##使用异步代码

同步代码在Node中只占有很小一部分。大部分主要用在与web应用无关的CLI命令或者一些其他脚本之中。开发者们主要用Node来构建web应用，因此他们用异步代码来避免堵塞线程。

就像下面的脚本，如果我们只用来构建一个连接数据库脚本或者用在一个并发不是很高的地方：

```js
let data = fs.readFileSync('./accounts.json')
db.collection('accounts').insert(data, (results))=>{
    fs.writeFileSync('./accountIDs.json', results, ()=>{process.exit(1)})
})
```
但是下面的脚本在构建web应用的时候可以表现更好：
```js
app.use('/seed/:name', (req, res) => {
  let data = fs.readFile(`./${req.params.name}.json`, ()=>{
    db.collection(req.params.name).insert(data, (results))=>{
      fs.writeFile(`./${req.params.name}IDs.json`, results, ()={res.status(201).send()})
    })
  })
})
```
区别在于你在写一个并发（通常长期运行）或者非并发（短期运行）的系统。根据经验来说，总是在Node中使用异步代码。

##避免块级引用
Node采用了CommonJS模块格式的简单模块加载系统。它内嵌的require方法可以很方便地在不同文件中引入模块。不像AMD/requirejs，Node/CommonJS加载模块的方法是同步的。require是这样工作的：引入一个模块或者文件。
```js
const react = require('react')
```
但是大部分的开发者并不知道require是可以缓存的。所以，只要解析的文件名没有剧烈的变化（在npm模块没有的情况下），模块的代码将被执行并存入变量中（在当前进程），这是一个很好的优化。然而，即使有了缓存，你最好还是把require语句放在文件开头。看下面这段代码，只有在路由中真正使用到axios模块的时候才会去加载它。所以执行这个/connect路由时会因为加载模块而变得很慢。
```js
app.post('/connect', (req, res) => {
    const axios = require('axios');
    axios.post('/api/authorize', req.body.auth)
        .then((response) => res.sent(response));
});
```
一个更好，更高性能的方法是在这个服务没有定义之前就去引入这个模块，而不是在路由中去引入：
```js
const axios = require('axios');
const express = require('express');
app = express();
app.post('/connect', (req, res) => {
    axios.post('/api/authorize', req.body.auth)
        .then((response) => res.end(response));
});
```

##知道require是被缓存的
在上一节我提到require是被缓存的，有趣的是我们可以在module.exports外也会有代码。例如
```js
console.log('I will not be cached and only run once, the first time')

module.exports = () => {
    console.log('I will be cached and will run every time this module is invoked')
}
```
要明白一些代买只执行一次，你可以利用这个特性来优化你的代码。

##始终要检查错误

并不像java一样，在java中，你可以抛出错误，因为大多数情况下你不想再让应用继续运行。在java中，你可以在外层使用一个try...catch语句就可以捕捉多个错误。

然而在Node中并不是这样。自从Node使用了时间循环机制和异步执行方式，任何错误都和上下文的错误处理器分离开了（例如try...catch）当错误发生的时候，这在Node中是没有用的：
```js
try {
    request.get('/accounts', (error, response) => {
        data = JOSN.parse(response);
    });
} catch (error) {
    //will not be called
    console.log(error);
}
```
但是try...catch语句在同步代码中仍然有效，所以前面的代码可以被更好的重构为：
```js
request.get('/accounts', (error, response) => {
    try {
        data = JSON.parse(response);
    } catch (error) {
        //will be called
        console.log(error);
    }
});
```
如果我们不能把request返回内容包裹在try...catch中，那么我们将没有办法去处理请求错误。Node开发者通过将error作为一个回调参数来解决错误处理问题。这样，你需要手动在每个回调中去处理错误。你需要检查错误（判断不为null），然后将错误信息展示给用户或者在客户端通过日志记录下来，或者把错误传递给callback函数，把错误传递给上一级调用栈（如果调用栈上有callback函数的话）。
```js
request.get('/accounts', (error, response) => {
    if (error) return console.error(error);
    try {
        data = JSON.parse(response);
    } catch(error) {
        console.error(error);
    }
});
```
一个小技巧是你可以使用okay库，你可以像下面的例子一样去使用它避免太深的回调（回调地狱）。

```js
var ok = require('okay');
request.get('/accounts', ok(console.error, (response) => {
    try {
        data = JSON.parse(response);
    } catch(error) {
        console.error(error);
    }
}));
```

##返回回调或者使用if...else语句
Node是并行的。如果你不够细心这个特性将会导致bug。安全起见，应该使用return来终止代码的继续运行：

```js
let error = true;
if (error) return callback(error);
console.log('i will nerver be run - good');
```
这样可以避免一些因为代码逻辑处理不当导致一些不该执行的内容（或者错误）被执行。

```js
let error = true;
if (error) callback(error);
console.log('i will run - not good');
```
确保使用return去避免程序的继续执行。

##监听error事件
