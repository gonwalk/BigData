## 5.1 简单类和无参方法

Scala类最简单的形式看上去和Java或C++中的很相似：

```
scala> class Counter{
     | private var value = 0                    //必须初始化否则会出错
     | def increment(){ value += 1 }
     | def current = value
     | }
defined class Counter

scala> val myCounter = new Counter             //或new Counter()
myCounter: Counter = Counter@60addb54

scala> myCounter.increment()

scala> println(myCounter.current)
1
```

**无参方法的调用:**

* **对于定义时带圆括号\(\)的无参方法：调用时，可以写上圆括号，也可以不写，如上面对increment的调用，可以写成myCounter.increment或者myCounter.increment\(\);**
* **对于定义时不带圆括号\(\)的无参方法：调用时，不能带圆括号，如：myCounter.current带上会出错，提示该方法不带参数does not take parameters  **



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
2. var foo:Scala自动合成一个getter。
3. 自定义foo和foo\_=方法。
4. 自定义foo方法。

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

Scala允许定义更加严格的访问限制，通过private\[this\]这个修饰符来实现。

private\[this\] var value = 0        //类似某个对象.value这样的访问将不被允许

这样一来，Counter类的方法只能访问到当前对象的value字段，而不能访问同样是Counter类型的其他对象的该字段。这样的访问有时被称为对象私有的，这在某些OO（面向对象）语言，比如SmalTalk中十分常见。

**对于类私有的字段，Scala生成私有的getter和setter方法，但是对于对象私有的字段，Scala根本不会生成getter或setter方法。**

**说明：Scala允许将访问权赋予指定的类。private\[类名\]修饰符可以定义仅有指定类的方法可以访问给定的字段。**_**这里的类名必须是当前定义的类，或者是包含该类的外部类。**_

**在这种情况下，编译器会生成辅助的getter和setter方法，运行外部类访问该字段。这些类将会使公有的，因为JVM并没有更细粒度的访问控制系统，并且它们的名称也会随着JVM实现不同而不同。**

