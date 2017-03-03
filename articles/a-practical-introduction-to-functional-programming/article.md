https://maryrosecook.com/blog/post/a-practical-introduction-to-functional-programming

#函数式编程的实用建议

许多函数式编程的文章教一些抽象的函数技术。即一些composition, pipelining, higher order function。但我这篇文章是与众不同的。文章中主要举例一些人们日常编写的非函数式编程代码并将这些代码转换成函数式编程风格。

The first section of the article takes short, data transforming loops and translates them into functional maps and reduces. The second section takes longer loops, breaks them up into units and makes each unit functional. The third section takes a loop that is a long series of successive data transformations and decomposes it into a functional pipeline.

文章中的例子都用Python来实现，因为很多人都觉的Python更加容易阅读和理解。很大部分的例子避开了python的一些特性，为了给大家示范函数式编程技巧在很多语言中都是通用的：map, reduce, pipeline。所有的例子都是用Python2来实现的。

##引导
当人们谈论函数式编程时，他们总会提及一大堆令人眩晕的函数式编程特性：不可改变数据（immutable data），一等函数（first class functions）和 tail call optimisation，这些都是可以帮助函数式编程的语言特性。当提及mapping, reducing, pipelining, recursing, currying以及高阶函数，这些都是是用来编写函数代码的编程技术。至于
parallelization5, lazy evaluation6 和 determinism。这些都是函数式编程的有利特性。

请忽略这些，函数式编程的核心特征就是：消除副作用。它并不依赖当前函数之外的数据，并且不会改变存在于当前函数之外的数据。所有其他的函数式编程特性都起源于这个属性。在你学习的时候要牢记这个属性。

这是一个非函数式编程的例子：
```py
a = 0
def increment():
    global a
    a += 1
```

这是一个函数式编程的例子
```py
def increment(a):
    return a + 1
```
##不要直接遍历数组，使用map和reduce

##Map

map需要一个函数和一个存放元素的数组。它将生成一个新的空集合，在原始集合的每一个元素上运行函数，将每一个函数运行的返回值放入新的集合中。最后返回这个新集合。

下面这个例子中map传入一个名字列表并返回名字长度的数组。
```py
name_lengths = map(len, ["mary", "isla", "sam"])
print name_lengths
```

下面这个例子将数组中的每个数字进行平方
```py
squares = map(lambda x: x * x, [0, 1, 2, 3, 4])
print squares
```
这个map传入了一个没有名字的函数。它传入了一个匿名的lambda定义的内联函数。冒号左边定义了匿名函数的参数，冒号右边定义了函数体。函数运行的结果被（隐性）返回。

下面这段非函数式代码传入了一个名字的数组并将它们替换为代码名字。

```py
import random

names = ['mary', 'isla', 'sam']
code_names = ['mr.pink', 'mr.orange', 'mr.blonde']
for i in range(len(names)):
    name[i] = random.choice(code_names)
print names
```
正如你所看到的，这个算法可以潜在地分配相同的密码给多个代理。希望这个不是某个加密的算法。

这个算法可以用map来实现：

```py
import random

names = ['mary', 'isla', 'sam']
secret_names = map(lambda x : random.choice(['mr.pink', 'mr.orange', 'mr.blonde']), names)
```

练习 1：使用map重写以下方法，它传入了一个真实的名字列表，并用一个更健壮的策略产生的代码名替换它们。

```py
names = ['mary', 'isla', 'sam']

for i in range(len(names)):
    names[i] = hash(names[i])
print names
```
希望秘密特工在秘密任务中有美好的回忆，不会忘记他们的密码。
我的答案：
```py
names = ['mary', 'isla', 'sam']
secret_names = map(hash, names)
```

###Reduce
Reduce需要传入一个函数和一个集合。它返回通过组合项创建的值。
下面是一个简单的例子，它返回集合中所有值的和。
```py
sum = reduce(lambda a, x : a + x, [0, 1, 2, 3, 4])
print sum
```
x是当前循环中的元素。a是累加器，是lambda函数上一次运行的结果.redude()方法遍历集合中的每一个元素。对于每一次循环，程序都会在当前的a和x上运行lambda函数，并将函数的返回值作为下次循环a的值。

a在第一次迭代的时候是多少呢，在之前并没有循环的结果传给它。reduce()函数使用集合里的第一个元素作为a，并在第二个元素上开始迭代。所以，第一次迭代的时候x是集合中的第二个元素。
下面这段代码统计'Sam'在数组中出现的次数：
```py
sentences = ['Mary read a story to Sam and Isla.',
             'Isla cuddled Sam.',
             'Sam chortled.']
sam_count = 0
for sentence in sentences:
    sam_count += sentence.count('Sam')
print sam_count
```
使用reduce重写：
```py
sentences = ['Mary read a story to Sam and Isla.',
             'Isla cuddled Sam.',
             'Sam chortled.']
sam_count = reduce(lambda a, x : a + x.count('Sam'), sentences, 0)
```
这段代码是怎么产生a的初始值的呢？最开始'Sam'的数量并不能成为'Mary read a story to Sam and Isla.'，所以初始值被赋值为reduce()函数的第三个参数。这种做法允许我们给a赋值一个与集合中元素不同种类的值。

为什么使用map reduce更好呢？

第一，它们经常被提及
第二，


























