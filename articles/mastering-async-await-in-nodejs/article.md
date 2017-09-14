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

不像上一个例子那么可怕，但是下面这种三个异步函数相互依赖的情况，你需要从几个丑陋的方法中去做出选择。

函数A 返回一个Promise对象，然后函数B需要函数A的返回值，函数C需要A和B的返回值。

###方法1：.then树
```js
function executeAsyncTask () {
  return functionA()
    .then((valueA) => {
      return functionB(valueA)
        .then((valueB) => {
          return functionC(valueA, valueB)
        })
    })
}
```
With this solution, we get valueA from the surrounding closure of the 3rd then and valueB as the value the previous Promise resolves to. We cannot flatten out the Christmas tree as we would lose the closure and valueA would be unavailable for functionC.

###方法2: 提升作用域
```js
function executeAsyncTask () {
  let valueA
  return functionA()
    .then((v) => {
      valueA = v
      return functionB(valueA)
    })
    .then((valueB) => {
      return functionC(valueA, valueB)
    })
}
```
在这个圣诞树中，我们使用了一个更高的作用域使valueA同样有效。这中方法同样有效，但是我们在.then作用域外创建了valueA，所以我们可以把第一个promise resolved的结果赋值给他。

这个一定管用，简化.then调用链并且语义正确。然而，如果变量名valueA在函数的其他地方也有用的话，将会引入新的bug。而且对于一个变量我们需要使用两个变量名：valueA 和 v。

###方法3：不必要的数组
```js
function executeAsyncTask () {
  return functionA()
    .then(valueA => {
      return Promise.all([valueA, functionB(valueA)])
    })
    .then(([valueA, valueB]) => {
      return functionC(valueA, valueB)
    })
}
```
除了扁平化函数之外没有更多的理由去让valueA和Promise函数functionB放到同一个数组中。它们可能是完全不同的类型，所以它们有很大的可能不属于一个数组。

###方法4：写一个helper函数
```js
const converge = (...promises) => (...args) => {
  let [head, ...tail] = promises
  if (tail.length) {
    return head(...args)
      .then((value) => converge(...tail)(...args.concat([value])))
  } else {
    return head(...args)
  }
}
functionA(2)
  .then((valueA) => converge(functionB, functionC)(valueA))
```
你当然可以写一个helper函数去隐藏语义，但是这会使代码很难阅读，并且对于那些不太懂函数魔法的人来说很难理解。

###使用async/await 方法，我们的问题神奇地解决了
```js
async function executeAsyncTask () {
  const valueA = await functionA()
  const valueB = await functionB(valueA)
  return function3(valueA, valueB)
}
```
###使用async/await并发执行多个请求

这个例子和上个例子很相似，在例子中你想去同时执行数个异步任务，然后在不同的地方使用它们的返回值，你可以使用async/await轻松地实现：
```js
async function executeParallelAsyncTasks () {
  const [ valueA, valueB, valueC ] = await Promise.all([ functionA(), functionB(), functionC() ])
  doSomethingWith(valueA)
  doSomethingElseWith(valueB)
  doAnotherThingWith(valueC)
}
```
正如我们在这个例子中看到的，我们不需要将这些返回值放入一个更高的作用域中或者创建一个不相关的数组去传递这些值。

###数组遍历方法
You can use map, filter and reduce with async functions, although they behave pretty unintuitively. Try guessing what the following scripts will print to the console:

你可以在async函数中使用map、filter、reduce方法，虽然他们表现的不是很直观。试着猜测下面的代码会打印什么出来：

1.map
```js
function asyncThing (value) {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(value), 100)
  })
}

async function main () {
  return [1,2,3,4].map(async (value) => {
    const v = await asyncThing(value)
    return v * 2
  })
}

main()
  .then(v => console.log(v))
  .catch(err => console.error(err))
```
filter
```js
function asyncThing (value) {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(value), 100)
  })
}

async function main () {
  return [1,2,3,4].filter(async (value) => {
    const v = await asyncThing(value)
    return v % 2 === 0
  })
}

main()
  .then(v => console.log(v))
  .catch(err => console.error(err))

```

reduce
```js
function asyncThing (value) {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(value), 100)
  })
}

async function main () {
  return [1,2,3,4].reduce(async (acc, value) => {
    return await acc + await asyncThing(value)
  }, Promise.resolve(0))
}

main()
  .then(v => console.log(v))
  .catch(err => console.error(err))
```
###答案：
1.[ Promise { <pending> }, Promise { <pending> }, Promise { <pending> }, Promise { <pending> } ]
2.[ 1, 2, 3, 4 ]
3.10

If you log the returned values of the iteratee with map you will see the array we expect: [ 2, 4, 6, 8 ]. The only problem is that each value is wrapped in a Promise by the AsyncFunction.
如果你记录map函数的返回值，我们期望的是[ 2, 4, 6, 8 ]，然而真实的返回值是包裹在Promise中的异步函数。

如果你想得到你的期望值，你需要将函数返回的数组用Promise.all方法去解除Promise的包裹。
```js
main()
  .then(v => Promise.all(v))
  .then(v => console.log(v))
  .catch(err => console.error(err))
```
Originally, you would first wait for all your promises to resolve and then map over the values:
最初，你先等待你的所有的promises对象resolve之后再去遍历他们的值：
```js
function main () {
  return Promise.all([1,2,3,4].map((value) => asyncThing(value)))
}

main()
  .then(values => values.map((value) => value * 2))
  .then(v => console.log(v))
  .catch(err => console.error(err))
```

这看起来更简单一点，不是吗？
如果你在循环里有长时间运行的同步逻辑和其他长时间运行的异步逻辑，这时使用async/await同样是很有用的。
This way you can start calculating as soon as you have the first value -
you don't have to wait for all the Promises to be resolved to run your computations. Even though the results will still be wrapped in Promises, those are resolved a lot faster then if you did it the sequential way.
这样你就可以在你得到第一个值的时候就开始计算 - 你不必等待所有的Promise在resolve之后再去运行你的计算，尽管这些结果都被包裹在promise中