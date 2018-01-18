## 5.3 只带getter的属性

有时候需要一个只读属性，即有getter但没有setter。如果属性的值在对象构建完成后就不再改变，可以使用val字段：

```
class Message{
    val timeStamp = new java.util.Date
    ...
}
```

Scala会生成一个私有的final字段和一个getter方法，但没有setter。

不能通过val来实现这样一个属性——val用不改变。需要提供一个私有字段和一个属性的getter方法，像这样：

```
class Counter{
    private var value = 0
    def increment() { value += 1 }
    def current = value                //声明中没有()
}
```

注意，**在getter方法的定义中并没有\(\)。因此，必须以不带圆括号的方式来调用：**

val  myCounter = new Counter\(\)

val n = myCounter.current

```
scala> class Counter{
     | private var value = 0
     | def increment() { value += 1 }
     | def current = value //声明中没有()
     | }
defined class Counter

scala> val myCounter = new Counter()
myCounter: Counter = Counter@9225652

scala> val n = myCounter.current
n: Int = 0
```

总结，在实现属性时有如下四个选择（var foo表示var类型的字段）：

1. var foo:Scala自动合成一个getter和一个setter。
2.  var foo:Scala自动合成一个getter。
3.  自定义foo和foo\_=方法。
4.  自定义foo方法。

**说明：在Scala中，不能实现只写属性（即带有setter但不带getter的属性）。**

**提示：当在Scala类中看到字段的时候，记住它和Java或C++中的字段不同：它是一个私有字段，加上getter方法（对val字段而言）或者getter和setter方法（对var字段而言）。**

## 5.4 对象私有字段

在Scala中（Java和C++也一样），方法可以访问该类的所有对象的私有字段。例如：

```
class Counter{
    private var value = 0
    def increment() { value += 1  }
    def isLess(other: Counter) = value < other.value
    //可以访问另一个对象的私有字段
}
```



