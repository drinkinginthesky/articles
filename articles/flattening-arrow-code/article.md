#扁平化箭头函数

我经常遇到这些代码：
```js
if (rowCount > rowIdx)
    {
      if (drc[rowIdx].Table.Columns.Contains("avalId"))
      {
        do
        {
          if (Attributes[attrVal.AttributeClassId] == null)
          {
            // do stuff
          }
          else
          {
            if (!(Attributes[attrVal.AttributeClassId] is ArrayList))
            {
              // do stuff
            }
            else
            {
              if (!isChecking)
              {
                // do stuff
              }
              else
              {
                // do stuff
              }
            }
          }
          rowIdx++;
        }
        while (rowIdx < rowCount && GetIdAsInt32(drc[rowIdx]) == Id);
      }
      else
        rowIdx++;
    }
    return rowIdx;
  }
```
条件语句的过度嵌套将代码形成了一个箭头

并且当你在一个典型的1280x1024显示器中，你阅读的代码一定会超出右边距。这是在实践中的反箭头模式。

我的重构代码的重要任务之一是像这样扁平化箭头代码。这些箭头代码是危险的！箭头代码具有很高的复杂度---通过测量代码有多少个不同的路径：

研究表明一个代码的环路复杂性与代码的错误频率有着密切的联系。一个低环路复杂度的代码有助于提高程序的可理解性，并且表明它比更复杂的程序具有较低的风险。一个程序的环路复杂性是一个强大的可测试性的指标。

适当的情况下，我会用以下方法去扁平化箭头代码：
###1.利用保卫条款替换条件
```js
if (SomeNecessaryCondition)
{
  // function body code
}
```
 利用守卫条款更好一些
```js
if (!SomeNecessaryCondition)
{
  throw new RequiredConditionMissingException;
}
// function body code
```
###2.将条件块分解成为单独的函数，在上面的例子中，我们将其分解到一个do...while循环中
```js
do
{
  ValidateRowAttribute(drc[rowIdx]);
  rowIdx++;
}
while(rowIdx < rowCount && GetIdAsInt32(drc[rowIdx]) == Id);
```

###3.将消极检查转换为积极检查，作为一个宽泛的规则，我喜欢首先进行积极的判断，然后消极的判断自然进入else分支，我认为这样更容易阅读，更重要的是，避免‘我不是从来没有这样做过’这样的拗口：
```js
if (Attributes[attrVal.AttributeClassId] is ArrayList)
{
  // do stuff
}
else
{
  // do stuff
}
```

###4.永远尽快从函数中返回，一旦你的功能完成了，尽快返回这个函数！虽然这并不是一直可能的，你可能有一些资源需要去释放。但是无论你怎么做，你需要去遗弃只有一个return在函数底部的想法。

我们的目标是滚动的代码很多，但是不需要这么多的水平编码。





