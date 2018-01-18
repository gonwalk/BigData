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

* **对于定义时不带圆括号\(\)的无参方法：调用时，不能带圆括号，如：myCounter.current带上会出错，提示该方法不带参数does not take parameters**

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

**在这种情况下，编译器会生成辅助的getter和setter方法，运行外部类访问该字段。这些类将会是公有的，因为JVM并没有更细粒度的访问控制系统，并且它们的名称也会随着JVM实现不同而不同。**

## 5.5 Bean属性

Scala对于定义的字段提供了getter和setter方法。不过，这些方法的名称并不是Java工具所预期的。JavaBean规范（[www.oracle.com/technetwork/java/javase/tech/index-jsp-138759.html](/www.oracle.com/technetwork/java/javase/tech/index-jsp-138759.html)）把Java属性定义为一堆getFoo/setFoo方法（或者对于只读属性而言为单个getFoo方法）。许多Java工具都依赖这样的明明习惯。

_**当将Scala字段标注为@BeanProperty时，这样的方法会自动生成**_。例如：

```
import scala.bean.BeanProperty
class Person {
    @BeanProperty var name:String = _
}
```

shell运行过程：

```
scala> import scala.beans.BeanProperty
import scala.beans.BeanProperty

scala> class Person{
     | @BeanProperty var name:String = _
     | }
defined class Person

scala> var na = new Person
na: Person = Person@dc4a691


scala> na.getName()
res11: String = null

scala> na.setName("zhangsan")

scala> na
res13: Person = Person@dc4a691

scala> na.name
res14: String = zhangsan
```

将会生成如下四个方法，这些方法就可以直接调用：

1. name:String
2. name\_ = \(newValue:String\):Unit
3. getName\(\):String
4. setName\(newValue:String\):Unit

说明：如果以主构造器参数的方式定义了某字段，并且需要JavaBeans版的getter和setter方法，像如下这样给构造器参数加上注释即可：class Person\(@BeanProperty var name:String\)

表5-1显示了在各种情况下哪些方法会被生成。

![](/assets/字段生成的方法.png)

## 5.6 辅助构造器

和Java或C++一样，Scala也可以有任意多的构造器。不过，Scala类有一个构造器比其他所有构造器都更为重要，它就是主构造器（primary constructor）。除了主构造器之外，类还可以有任意多的辅助构造器（auxiliary constructor）。

**辅助构造器同Java或C++的构造器十分相似，只有两处不同：**

1. ** 辅助构造器的名称为this。在Java或C++中，构造器的名称和类名相同——当修改类名时不那么方便了。**
2. _** 每一个辅助构造器都必须以一个对先前已定义的其他辅助构造器或主构造器的调用开始。**_

示例：这里有一个带有两个辅助构造器的类

```
class Person {
    private var name = ""
    private var age = 0

    def this(name:String) {        //一个辅助构造器
        this()                     //调用主构造器
        this.name = name
    }

    def this(name:String, age: Int) {    //另一个辅助构造器
        this(name)                       //调用前一个辅助构造器
        this.age = age
    }
}
```

如果一个类没有显示定义主构造器，则自动拥有一个无参的主构造器。

对于上面的类，可以以三种方式构建对象：

```
val p1 = new Person                //主构造器
val p2 = new Person("Fred")        //第一个辅助构造器
val p3 = new Person("Fred", 42)    //第二个辅助构造器
```

## 5.7 主构造器

在Scala中，每个类都有主构造器。主构造器并不以this方法定义，而是与类定义交织在一起。

1. 主构造器的参数直接放置在类名之后。

```
class Person(val name:String, val age:Int) {
    //（...）中的内容就是主构造器的参数
    ...
}
```

主构造器的参数被编译成字段，其值被初始化成构造时传入的参数。在本例中，name和age成为Person类的字段。如new Person\("Fred", 42\)这样的构造器调用将设置name和age字段。

   2. 主构造器会执行类定义中的所有语句。

```
class Person(val name:String, val age:Int){
    println("Just constructed another person")
    def description = name + " is " + age + " years old"
}
```

println语句是主构造器的一部分，每当有对象被构造出来时，上述代码就会被执行。当需要再构造过程当中配置某个字段时，这个特性特别有用：

```
class MyProg {
    private val props = new Properties
    props.load（new FileReader("myprog.properties"))
    //上述语句是主构造器的一部分
    ...
}
```

**说明：如果类名之后没有参数，则该类具备一个无参主构造器。这样一个构造器仅仅是简单地执行类体中的所有语句而已。**

提示：通常可以通过在主构造器中使用默认参数来避免过多地使用辅助构造器。例如：

class Person\(val name:String = " ", val age: Int = 0\)

主构造器的参数可以采用表5-1中列出的任意形态。例如：class Person\(**val** name: String, **private var** age: Int\)

构造参数也可以是普通的方法参数，不带val或var。这样的参数如何处理取决于它们在类中如何被使用。

* **如果不带val或者var的参数至少被一个方法所使用，它将被升格为字段。下面这段代码声明并初始化了不可变字段name和age，而这两个字段都是对象私有的。类似这样的字段等同于private\[this\]val 字段的效果。**例如：

```
class Person(name: String, age: Int) {
    def description = name + " is " + age + " years old"
}
```

* 否则，该参数将不被保存为字段。它仅仅是一个可以被主构造器中的代码访问的普通参数。（严格地说，这是一个具体实现相关的优化。）

表5-2总结了不同类型的主构造器参数对应会生成的字段和方法。

![](/assets/主构造器参数生成的字段和方法.png)

**如果对主构造器的使用不熟悉，可以按照常规的做法提供一个或多个辅助构造器即可，不过要记得调用this\(\)。**_**为了便于理解，建议这样看待主构造器：在Scala中，类也接受参数，就像方法一样。**_

_**说明：当把主构造器的参数看做是类参数时，不带val或var的参数就变得易于理解了。这样的参数的作用域涵盖了整个类。因此，可以在方法中使用它们。而一旦这样做了，编译器就自动帮你将它保存为字段。**_

**如果想让主构造器变成私有的，可以像这样放置private关键字：**

**class Person private\(val id: Int\) { ... } **

**这样一来类用户就必须通过辅助构造器来构造Person对象了。**

## 5.8 嵌套类

在Scala中，几乎可在任何语法结构中内嵌任何语法结构。可以在函数中定义函数，在类中定义类。以下代码是在类中定义类的一个示例：

```
import scala.collection.mutable.ArrayBuffer

class Network {
    class Member（val name: String) {
        val contacts = new ArrayBuffer[Member]
    }
    
    private val members = new ArrayBuffer[Member]
    
    def join(name: String) = {
        val m = new Member(name)
        members += m
        m                    //返回m的值，相当于return m
    }
}
```



