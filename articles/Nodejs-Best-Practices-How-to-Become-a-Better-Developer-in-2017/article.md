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
当然，现在对于fs模块来说还没有readFileAsync接口去返回一个Promise方法。你可以引入[promisifyAll]()模块来添加readFileAsync接口。