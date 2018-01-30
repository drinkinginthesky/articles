http://www.jongleberry.com/understanding-possible-eventemitter-leaks.html

# 了解nodejs可能的事件内存泄露监听错误信息

在node.js和io.js中，你总会看到这样的错误信息：

> (node) warning: possible EventEmitter memory leak detected. 11 a listeners added. Use emitter.setMaxListeners() to increase limit.

## 什么时候会发生泄露？

当你持续添加事件监听却不移除它们，就会发生泄露。这种特殊情况发生在多次使用一个事件监听器的时候。让我们写一个返回流中下一个值的函数：

```javascript
function next(stream) {
  // if the stream has data buffered, return that
  {
    let data = stream.read()
    if (data) return Promise.resolve(data)
  }

  // if the stream has already ended, return nothing
  if (!data.readable) return Promise.resolve(null)

  // wait for data
  return new Promise(function (resolve, reject) {
    stream.once('readable', () => resolve(stream.read()))
    stream.on('error', reject)
    stream.on('end', resolve)
  })
}
```

每次在stream上调用next()函数时，就会添加一个`readable`, `error`, 和 `end`事件。当你第十一次调用时，就会得到这个错误：

> (node) warning: possible EventEmitter memory leak detected. 11 a listeners added. Use emitter.setMaxListeners() to increase limit.

你连续不断地添加了`error`和`end`事件，但是并没有移除它们，即使数据被成功读取，这些处理函数也不再有用。

## 清除事件处理程序

正确的方式是保证在promise执行完成之后清理你的事件监听函数，确保没有事件监听函数被添加到net上。

```javascript
return new Promise(function (resolve, reject) {
  stream.on('readable', onreadable)
  stream.on('error', onerror)
  stream.on('end', cleanup)

  // define all functions in scope
  // so they can be referenced by cleanup and vice-versa
  function onreadable() {
    cleanup()
    resolve(stream.read())
  }

  function onerror(err) {
    cleanup()
    reject(err)
  }

  function cleanup() {
    // remove all event listeners created in this promise
    stream.removeListener('readable', onreadable)
    stream.removeListener('error', onerror)
    stream.removeListener('end', cleanup)
  }
})
```

在promise执行完成之后，将不会产生事件监听导致的内存泄露，net上没有任何事件监听的变化。

## 并行处理

如果你想在一个事件上添加多个监听函数呢？例如，你可能有很多函数监听在这个事件上：

```javascript
doThis1(stream)
doThis2(stream)
doThis3(stream)
doThis4(stream)
doThis5(stream)
doThis6(stream)
doThis7(stream)
doThis8(stream)
doThis9(stream)
doThis10(stream)
doThis11(stream)
doThis12(stream)
doThis13(stream)
```

如果上面所有的函数被添加到`data`事件上，你将会得到一样的内存泄露的报错，但是你知道并没有实际的泄露。在这一点上，你应该修改最大的事件监听数量：

```javascript
return new Promise(function (resolve, reject) {
  // increase the maximum number of listeners by 1
  // while this promise is in progress
  stream.setMaxListeners(stream.getMaxListeners() + 1)
  stream.on('readable', onreadable)
  stream.on('error', onerror)
  stream.on('end', cleanup)

  function onreadable() {
    cleanup()
    resolve(stream.read())
  }

  function onerror(err) {
    cleanup()
    reject(err)
  }

  function cleanup() {
    stream.removeListener('readable', onreadable)
    stream.removeListener('error', onerror)
    stream.removeListener('end', cleanup)
    // this promise is done, so we lower the maximum number of listeners
    stream.setMaxListeners(stream.getMaxListeners() - 1)
  }
})
```

这允许你掌控事件监听的数量并且保持你对事件监听的控制，如果允许nodejs打印错误信息当真正的内存泄露发生的时候。

## 帮助你写出更棒的代码

如果你简单的setMaxListener(0)的话，那么内存泄露会在不知不觉中发生。如果你在任何代码（尤其是开源代码）中看到使用setMaxListener(0)的话，pull request去修复它！不要走任何捷径！









































































