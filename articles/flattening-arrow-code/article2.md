http://wiki.c2.com/?ArrowAntiPattern
#箭头反模式
像下面这样，嵌套if语句组成一个箭头形状：
```javascript
 if
   if
     if
       if
         do something
       endif
     endif
   endif
 endif
```
通常当程序员盲目应用"one return per function"原则或者混合了条件和循环。
最好的解决方法是 [RefactorLowHangingFruit](http://wiki.c2.com/?RefactorLowHangingFruit):简单地将整个方法分开将导致两个巨大且丑陋的函数，争论很多，没有概念。相反，重构代码的组件：经常这些条件可以颠倒，并作为保护条款，从结构中移除他们； ExtractMethod可以将循环的内容转换为独立的函数。并且ExecuteAroundPattern是一个可以无限使用的魔杖。

有趣的是，IBM的LanceMiller博士将这种结构与英语语法进行对比，你在哪里经常开始一个动词并且修饰它。例如："heat water until boiling"，我想这是Perl语法的争吵？-- PaulMorrison

在[code complete](http://wiki.c2.com/?CodeComplete)中，SteveMcConnell指出在三层'if'嵌套中代码的可读性将大大降低，根据 NoamChomsky和GeraldWeinberg在1986的研究。McConnell 引用了[AntiPattern](http://wiki.c2.com/?AntiPattern)中“危险的深层嵌套”。

If the arrow is an AntiPattern, consider a whole mountain range: http://thedailywtf.com/Articles/Coding-Like-the-Tour-de-France.aspx

----------
下面这些代码虽然很烂，但还不至于是最坏的：
```javascript
 if get_resource
   if get_resource
     if get_resource
       if get_resource
         do something
         free_resource
       else
         error_path
       endif
       free_resource
     else
       error_path
     endif
     free_resource
   else
     error_path
   endif
   free_resource
 else
   error_path
 endif
```
This style puts much error recovery and clean-up code very far from the thing that spawned the error. Hours Of Fun!
一个好的方法：只需保证这段代码完全异常安全，这样将保证人们不必去关心谁或者怎样去处理异常。
是的。“只需保证代码完全异常安全”，怎样去实现呢？这在嵌入式系统中是一个异常棘手的问题，资源获取的通常是一个硬件不同的设置/确定条件。
 (with-resource-foo (foo ...) ... ) ?这样如何，我认为这是在作弊。
 这是一个很好的解决方法，使这些对象异常安全。重构因素将会清理大部分的函数，使留下来的更加明白。

如果你有一种像CeePlusPlus一样支持资源获取初始化的语言，那就可以干净，优雅地实施资源安全的异常处理。（这种方法决不是排他性的处理这样的事情，但恰好是我个人的最爱）。然而，建议你只是“使”代码异常驱动，虽然正确，但是并不实用（[Just Isa Dangerous Word](http://wiki.c2.com/?JustIsaDangerousWord)）。通常这样的重构有级联效应，当你改变错误处理（通常更高）的调用栈。根据我的经验， Arrow Anti Pattern code is symptomatic of code written without attention being paid to proper Modular Programming techniques。因此可能是由一个不好的程序员编写的。这个意见将加强我对这个问题的规模比例：[Three Strikes And You Refactor](http://wiki.c2.com/?ThreeStrikesAndYouRefactor)。通常，一种可能的解决方案是使用多态性将语义与控制流解耦。你可以将所有的if else块分为小的函数，被称为适当（多态）。而额外照顾那些错误路径。

----------
这个方法多年前被采用，它使我们的代码干净，清晰，简短，优雅，并且大大减少了出现bug的几率。要求每个”get_resource“调用失败返回“false”，不需要进一步清理，成功时返回“true”并且提供了一个清理资源释放的回调。回调被添加在一个list结构中，将在程序的最后被调用，代码将会像这样：
```c
    callback_list = []
    if ( get_resource_A(&callback_list,...) &&
         get_resource_B(&callback_list,...) &&
         ...
         )
    {
        success_flag = do_work();
    }
    for closure in callback:
        eval(closure);
```
这个语法并不是太坏，但是我已经简化了代码以避免模糊要点。我们更复杂的系统中提供了一个包括成功和失败回调的list结构，确保错误报告和错误恢复被正确处理，并且保证在距离错误最近的地方。

----------
另一个解决方法，请看[Function Wrapper](http://wiki.c2.com/?FunctionWrapper)
也可以看: [BouncerPattern](http://wiki.c2.com/?BouncerPattern), [TrivialDoWhileLoop](http://wiki.c2.com/?TrivialDoWhileLoop), [ElseConsideredSmelly](http://wiki.c2.com/?ElseConsideredSmelly)
如果你使用java的话，[PeeEmDee](http://wiki.c2.com/?PeeEmDee)代码检查工具可以帮助你找到深层嵌套的if语句

----------
我经常像下面这样写代码：
```c
  bool open_some_object(Object *object)
  {
    assert(object);
    if(open_resource_a(&object->a))
    {
      if(open_resource_b(&object->b))
      {
        if(open_resource_c(&object->c))
        {
          return true;
        }
        close_resource_b(&object->b);
      }
      close_resource_a(&object->a);
    }
    return false;
  }

```
这看起来和反模式很像，除了中间的“return true”。如果函数执行成功的话，函数就返回成功。如果有任何的失败，函数将会关闭任何被打开的事物，然后返回失败。我一直认为这是一个初始化对象的优雅方式，但也许我误导了我自己？这是反模式吗，或者只是看起来像？有没有一个更好的方式去打开一个由子对象组成的对象？

----------
是的，这仍然是反模式，因为反模式很难去匹配开/关。在这个例子中，只有三步，所以还不算差。但是考虑一下如果你有10个或者更多资源去初始化。这里真正的问题是open_resource_X()函数在功能出错后无法自行清除。如果可能的话，将cleanup方法放到处理错误的函数中。

如果你不能重写这些函数，考虑将它们包裹在一个函数中，对错误进行适当的处理。你可以编写n个不同的函数包裹全部的open_resource_X()，但是这样做会更容易一些：
```c
  typedef void (*Close)(void*);
  typedef bool (*Open)(void*);

  bool try_open(Open open, Close on_error, void* arg)
  {
      if(open(arg))
        return true;
      on_error(arg);
      return false;
  }

  bool open_some_object(Object *object)
  {
    assert(object);
    return try_open(open_a, close_a, &object->a) &&
           try_open(open_b, close_b, &object->b) &&
           try_open(open_c, close_c, &object->c);
  }

```

----------
一个我喜欢的反箭头模式重构，当守卫条款模式不工作时，将这些代码块分割成更小的代码块，所以每一个都取前面块的部分结果，并且生成结果给下一个代码块。
```c
use_items = []
 list.each do |item|
   if !item.nil? then
      if item.category == 'foo' then
        use_items << item
      end
   end
 end
```
上面的嵌套循环可以被重写成：
```ruby
 non_nil_items = list.filter {|i| !i.nil?}
 use_items = non_nil_items.filter {|i| i.category == 'foo'}
```
这个例子看起来有点牵强，因为代码是用ruby实现的，因为ruby有短路与，所以在循环中没有这两个if语句。尽管很难想象Ruby中的病理起点。

----------
在处理低级的服务时，我经常看到这样的代码，必须逐步获得，但没有看到简单的修复。至少避免深层嵌套的一种方法是类似于：
```c
  good = true;
  if (good) {
    if (! get_resource_1()) {
      error_handling...
      close_resource_1;  // if applicable (perhaps call it "clean up")
      good = false;
    }
  }
  if (good) {
    if (! get_resource_2()) {
      error_handling...
      close_resource_2;
      good = false;
    }
  }
  if (good) {
    if (! get_resource_3()) {
      error_handling...
      close_resource_3;
      good = false;
    }
  }
  ...
  if (good) {
    regular_process...
    close_resource_1;
    close_resource_2;
    close_resource_3;
  }
  ...
```