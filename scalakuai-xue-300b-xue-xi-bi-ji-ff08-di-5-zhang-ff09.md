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

注意，在getter方法的定义中并没有\(\)。因此，必须以不带圆括号的方式来调用：

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



