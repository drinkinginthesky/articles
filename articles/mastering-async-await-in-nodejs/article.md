https://blog.risingstack.com/mastering-async-await-in-nodejs/
#在Nodejs中掌握async/await

##在这篇文章中，你将学到如何使用async/await来简化nodejs中的callback、promise代码

异步语言结构在其他语言中已经出现一段时间了，像C#中的async/await，Kotlin中的协程和GO语言中的goroutines。随着Nodejs V8的发布，期待已久的async函数也在nodejs中落地了。

学完这个教程之后，你应该可以回答下面的这些问题：
async/await是否是自从切片面包发明以来的最好的事情？

##什么是async函数

async函数声明返回一个async函数对象。这些和Generator-s具有相似的意义，它们的执行都可以被停止。唯一的不同就是async一直返回一个Promise对象，而不是{ value: any, done: Boolean }对象。事实上，它可以和co包有着相似的作用。

在一个async函数中，你可以使用await来等待任意一个promise或者抓住它们的reject情况。

所以如果你有一些用promise实现的逻辑：
```js
function handler (req, res) {
  return request('https://user-handler-service')
    .catch((err) => {
      logger.error('Http error', err)
      error.logged = true
      throw err
    })
    .then((response) => Mongo.findOne({ user: response.body.user }))
    .catch((err) => {
      !error.logged && logger.error('Mongo error', err)
      error.logged = true
      throw err
    })
    .then((document) => executeLogic(req, res, document))
    .catch((err) => {
      !error.logged && console.error(err)
      res.status(500).send()
    })
}
```
你可以使用async/await方法来使它看起来更像同步代码
```js
async function handler (req, res) {
  let response
  try {
    response = await request('https://user-handler-service')
  } catch (err) {
    logger.error('Http error', err)
    return res.status(500).send()
  }

  let document
  try {
    document = await Mongo.findOne({ user: response.body.user })
  } catch (err) {
    logger.error('Mongo error', err)
    return res.status(500).send()
  }

  executeLogic(document, req, res)
}
```
在旧版本的V8引擎中，没有处理的错误将会被抛出来。现在至少你能从Nodejs中获取一个警告，所以你不用去创建一个错误监听器。然而，在你不处理错误的时候，建议在这种情况下崩溃你的程序，你的应用会处于一个未知的状态：
```js
process.on('unhandledRejection', (err) => {
  console.error(err)
  process.exit(1)
})
```
##async函数模式

下面有很多例子，展示了异步函数就像同步函数一样，用起来非常方便，为解决他们的Promise或回调函数需要复杂的模式或外部库的使用。

下面这些例子展示了当你需要循环一个异步函数并获取数据或者使用if-else条件语句。
#Retry with exponential backoff
#根据返回指数进行重试

使用promise实现重试逻辑是非常笨拙的：
```js
function requestWithRetry (url, retryCount) {
  if (retryCount) {
    return new Promise((resolve, reject) => {
      const timeout = Math.pow(2, retryCount)

      setTimeout(() => {
        console.log('Waiting', timeout, 'ms')
        _requestWithRetry(url, retryCount)
          .then(resolve)
          .catch(reject)
      }, timeout)
    })
  } else {
    return _requestWithRetry(url, 0)
  }
}

function _requestWithRetry (url, retryCount) {
  return request(url, retryCount)
    .catch((err) => {
      if (err.statusCode && err.statusCode >= 500) {
        console.log('Retrying', err.message, retryCount)
        return requestWithRetry(url, ++retryCount)
      }
      throw err
    })
}

requestWithRetry('http://localhost:3000')
  .then((res) => {
    console.log(res)
  })
  .catch(err => {
    console.error(err)
  })
```
我一看到这些代码就感到头痛，我们可以使用async/await来重写它并让它看上去更简单。
```js
function wait (timeout) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve()
    }, timeout)
  })
}

async function requestWithRetry (url) {
  const MAX_RETRIES = 10
  for (let i = 0; i <= MAX_RETRIES; i++) {
    try {
      return await request(url)
    } catch (err) {
      const timeout = Math.pow(2, i)
      console.log('Waiting', timeout, 'ms')
      await wait(timeout)
      console.log('Retrying', err.message, i)
    }
  }
}
```
看起来更赏心悦目了，不是嘛？
##中间值

不像上一个例子那么可怕，但是如果有一种情况，三个异步函数相互依赖的情况，然后你需要从几个丑陋的方法中去做出选择。

函数A 返回一个Promise对象，然后函数B需要函数A的返回值，函数C需要A和B的返回值。



