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

1. 主构造器会执行类定义中的所有语句。

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

# 第6章 对象

本章要点概览：

用对象作为单例或存放工具方法。

* 类可以拥有一个同名的伴生对象。
* 对象可以扩展类或特质。
* 对象的apply方法通常用来构造伴生类的新实例。
* 如果不想显示定义main方法，可以用扩展App特质的对象。
* 可以通过扩展Enumeration对象来实现枚举。

## 6.1 单例对象

Scala没有静态方法或静态字段，可以用object这个语法结构来达到同样的目的。对象定义了某个类的单个实例，包含想要的特质。例如：

```
object Accounts{
    private var lastNumber = 0
    def newUniqueNumber() = { lastNumber += 1;lastNumber }
}
```

当在应用程序中需要一个新的唯一账号时，调用Accounts.newUniqueNumber\(\)即可。

对象的构造器在该对象第一次被使用时调用。如果一个对象从未被使用，那么其构造器也不会被执行。

对象本质上可以拥有类的所有特性——甚至可以扩展其他类或特质。只有一个例外：不能提供构造器参数。

对于任何在Java或C++中会使用单例的地方，在Scala中都可以用对象来实现：

* 作为存放工具函数或常量的地方。
* 高效地共享单个不可变实例。
* 需要用单个实例来协调某个服务时（参考单例模式）。

说明：Scala提供的是工具，可以做出好的设计，也可以做出糟糕的设计，需要做出自己的判断。

## 6.2 伴生对象

在Java或C++中，通常会用到既有实例方法又有静态方法的类。在Scala中，可以通过类和与类同名的“

伴生”对象

来达到同样的目的。

```
class Account{
    val id = Account.newUniqueNumber()
    private var balance = 0.0
    def deposit(amount: Double) { balance += amount }
    ......
}

object Account{      //伴生对象
    private var lastNumber = 0
    private def newUniqueNumber() = { lastNumber += 1; lastNumber }
}
```

类和它的伴生对象可以相互访问私有特性。它们必须存在于同一个源文件中。

说明：类的伴生对象可以被访问，但并不在作用域当中。如上面的例子，Account类必须通过Account.newUniqueNumber\(\)而不是newUniqueNumber\(\)来调用伴生对象的方法。

提示：在REPL中，要同时定义类和对象，必须使用黏贴模式。键入：

:paste

然后键入或黏贴类和对象的定义，最后一Ctrl + D退出黏贴模式。

## 6.3 扩展类或特质的对象

一个object可以扩展类以及一个或多个特质，其结果是一个扩展了指定类以及特质的类的对象，同时拥有在对象定义中给出的所有特性。一个有用的场景是给出可被共享的缺省对象。

如，考虑在程序中引入一个可撤销动作的类：

```
abstract class UndoableAction(val description:String){
    def undo(): Unit
    def redo():Unit
}
```

默认情况下可以是“什么都不做”。对于这个行为只需要一个实例即可。

```
object DoNothingAction extends UndoableAction("Do nothing"){
    override def undo() {}
    override def redo(){}
}
```

DoNothingAction对象可以被所有需要这个缺省行为的地方公用。

val actions = Map\("open" -&gt; DoNothingAction, "save" -&gt; DoNothingAction, ...\)

//打开和保存功能尚未实现

## 6.4 apply方法

我们通常会定义和使用对象的apply方法。当遇到如下形式的表达式时，apply方法就会被调用。

```
    Object\(参数1, 参数2, ... , 参数N\)
```

通常，这样的apply方法返回的是伴生类的对象。例如，Array对象定义了apply方法，可以用下面的表达式创建数组：

```
    Array\("Mary", "had", "a", "little", "lamb"\)
```

注意：不要把Array\(10\)和new Array\(10\)混淆。前一个表达式调用的是apply\(10\)，输出一个单元素（整数10）的Array\[Int\]；而第二个表达式调用的是构造器this\(10\)，结果是Array\[Nothing\]，包含10个null元素。

## 6.5 应用程序对象

每个Scala程序都必须从一个对象的main方法开始，这个方法的类型为Array\[String\] =&gt; Unit:

```
object Hello{
    def main(args: Array[String]){
        println("Hello, World!")
    }
}
```

除了每次都提供自己的main方法外，也可以扩展App特质，然后将程序代码放入构造器方法体内：

```
object Hello extends App{
    println("Hello, World!")
}
```

* 如果需要命令行参数，则可以通过args属性得到：

```
object Hello extends App{
    if(args.length 
>
 0)
        println("Hello, " + args(0)
    else 
        prinln("Hello, World!")
}
```

* 如果在调用该应用程序时设置了scala.time选项的话，程序退出时会显示逝去的时间。

```
$scalac Hello.scala
$scala -Dscala.time Hello Fred
Hello, Fred
[total 4ms]
```

App特质扩展自另一个特质DelayedInit，编译器对该特质有特殊处理。所有带有该特质的类，其初始化方法都会被挪到delayedinit方法中。App特制的main方法捕获到命令行参数，调用delayedInit方法，并且还可以根据要求打印出逝去的时间。

## 6.6 枚举

和Java或C++不同，Scala并没有枚举类型。不过，标准类库提供了一个Enumeration助手类，可以用于产出枚举。

例如，定义一个扩展Enumeration类的对象并以Value方法调用初始化枚举中的所有可选值。

```
object TrafficLightColor extends Enumeration{
    val Red, Yellow, Green = Value
}
```

这里定义了三个字段：Red、Yellow、Green，然后用Value调用将它们初始化。得到val Red = Value；val Yellow  = Value; val Green = Value。每次调用Value方法都返回内部类的新实例，该内部类也叫做Value。

记住枚举的类型是TrafficLightColor.Value而不是TrafficLightColor——后者是握有这些值的对象。

```
object TrafficLightColor extends Enumeration{
    type TrafficLightColor = value      //增加一个类型别名
    val Red, Yellow, Green = Value
}

//现在枚举的类型变成了TrafficLightColor.TrafficLightColor，但仅当使用import语句时这样做才有意义。
import TrafficLightColor._
def doWhat(color: TrafficLightColor) = {
    if(color == Red) "stop"
    else if(color == Yellow) "hurry up"
    else "go"
}
```

枚举值得ID可通过id方法返回，名称通过toString方法返回。对TrafficLightColor.values的调用输出所有枚举值的集：

for\(c -&gt; TrafficLightColor.values\) println\(c.id + ":" + c\)

最后，可以通过枚举的ID或名称来进行查找定位，以下两段代码都输出TrafficLightColor.Red对象：

```
TrafficLightColor(0)        //调用Enumeration.apply
TrafficLightColor.withName("Red")
```

# 第7章 包和引入

本章要点概览：

* 包也可以像内部类那样嵌套。
* 包路径不是绝对路径。
* 包声明链x.y.z并不自动将中间包x和x.y变成可见。
* 位于文件顶部不带花括号的包声明在整个文件范围内有效。
* 包对象可以持有函数和变量。
* 引入语句可以引入包、类和对象。
* 引入语句可以出现在任何位置。
* 引入语句可以重命名和隐藏特定成员。
* java.lang、scala和Predef总是被引入。

## 7.1 包

Scala的包和Java中的包或者C++中的命名空间的目的是相同的：管理大型程序中的名称。

例如：Map这个名称可以同时出现在scala.collection.immutable和scala.collection.mutable包中而不会发生冲突。要访问它们中的任何一个，可以使用完全限定的名称scala.collection.imutable.Map或scala.collection.mutable.Map，也可以使用引入语句提供一个更短小的别名。

要增加条目到包中，可以将其包含在包语句中，比如：

```
package com{
    package horstman{
        package impatient{
            class Employee
            ...
        }
    }
}
```

这样类名Employee就可以在任意位置以com.horstmann.impatient.Employee访问到了。

与对象或类的定义不同，同一个包可以定义在多个文件当中。

说明：源文件的目录和包之间并没有强制的关联关系。

## 7.2 作用域规则

在Scala中，包的作用域比起Java来更加前后一致。Scala的包和其他作用域一样地支持嵌套。

可以访问上层作用域中的名称。

Scala中有一个特性，那就是scala包总是被引入。因此包中没有其他collection包，引入collection包中的mutable.ArrayBuffer时可以使用相对路径collection.mutable.ArrayBuffer而不出错，但是当其他包中有collection这个包，如com.horstmann.collection，再使用相对路径collection.mutable.ArrayBuffer就可能因编译器找不到对应的包而引发错误。

在Java中，这个问题不会发生，因为包名总是绝对的，从包层级的最顶端开始。但是在Scala中，包名是相对的，就像内部类的名称一样。内部类通常不会遇到这个问题，因为所有代码都在同一个文件当中，由负责该文件的人直接控制。但是包不一样了，任何人都可以在任何时候向任何包添加内容。

解决方法之一是使用绝对包名，以\_root\_开始

，例如：

val subordinates = new \_root\_.scala.collection.mutable.ArrayBuffer\[Employee\]

另一种做法是使用“串联式”包语句。

说明：大多数程序员都是用完整的包名，知识不加\_root\_前缀。只要避免用scala、java、com、org等名称来命名嵌套的包，这样做就是安全的。

## 7.3 串联式包语句

包语句可以包含一个“串”，或者说路径区段，例如：scala.collection.mutable包中的ArrayBuffer，这样的包语句限定了可见的成员。

## 7.4 文件顶部标记法

除了嵌套标记法外，也可以在文件顶部使用package语句，不带或括号。例如：

```
package com.horstmann.impatient
package people

class Person
...

//上面的代码等价于
package com.horstmann.impatient{
    package people{
        class Person
        ...
        //直到文件末尾
    }
}
```

注意在上面的示例中，文件的所有内容都属于com.horstmann.impatient.people，但com.horstmann.impatient包中的内容是可见的，可以被直接引用。

## 7.5 包对象

包可以包含类、对象和特质，但不能包含函数或变量的定义，这是Java虚拟机的局限。把工具函数或常量添加到包而不是某个Utils对象，这是更加合理的做法。包对象的出现正是为了解决这个局限。

每个包都可以有一个包对象。需要再父包中定义它，且名称与子包一样。例如：

```
package com.horstmann.impatient


package object
 people{
    val defaultName = "John Q, Public"
}

package people {
    class Person{
        var name = defaultName  //此包对象拿到的常量
    }
}
```

注意defaultName不需要加限定词（父包的路径），因为它位于同一个包内。在其他地方，这个常量可以使用com.horstmann.impatient.people.defaultName访问到。

在幕后，包对象被编译成带有静态方法和字段的JVM类，名为package.class，位于相应的包下。对应到本例中，就是com.horstmann.impatient.people.package，其中有一个静态字段defaultName。（在JVM中，可以使用package作为类名。）

## 7.6 包可见性

在Java中，没有被声明为public、private或protected的类成员在包含该类的包中可见。在Scala中，可以通过修饰符达到同样的效果。以下方法在它自己的包中可见：

```
package com.horstmann.impatient.people
class Person{
    private[people] def description = "A person with name " + name 
    ...
}
```

可以将可见度延展到上层包：

private\[impatient\] def description = "A person with name " + name

## 7.7 引用

引用语句可以使用更短的名称而不是原来较长的名称。如：import java.awt.Color 这样就可以在代码中写Color而不是java.awt.Color了，同时还可以通过Color.类名 的方式调用Color类中的方法实现想要的功能。

在Scala中，可以通过\_引用某个包的全部成员：import java.awt.\_  这和Java中的通配符\*一样。（在Scala中，\*是合法的表示符。在上面的例子中完全可以定义com.horstmann.\*.people这样的包，但建议还是不要这样做。）

在Scala中，也可以通过\_引入类或对象的所有成员。一旦引入了某个包，就可以用较短的名称访问其子包。例如：

```
import java.awt._

def handler(evt: event.ActionEvent){//java.awt.event.ActionEvent
    ...
}
```

其中，event包时java.awt包的成员，因此引用语句把它带进了作用域。

## 7.8 任何地方都可以声明引入

在Scala中，import语句可以出现在任何地方，并不仅限于文件顶部。import语句的效果一直延伸到包含该语句的块末尾。例如：

```
class Manager {
    import scala.collection.mutable._
    val subordinates = new ArrayBuffer[Employee]
    ...
}
```

通过将引入放置在需要这些引入的地方，可以大幅减少可能的名称冲突。

## 7.9 重命名和隐藏方法

如果想要引入包中的几个成员，可以像下面这样使用选取器（selector）：

imprt java.awt.{Color, Font}

选取器语法（将用到的包中的某几个类通过花括号{}括起来）还允许重命名选到的成员：

```
import java.util.{HashMap => JavaHashMap}
import scala.collection.mutable._
```

这样，JavaHashMap就是java.util.HashMap，而HashMap则对应scala.collection.mutable.HashMap。

选取器HashMap =&gt; \_将隐藏某个成员而不是重命名它。这仅在需要引入其他成员时有用：

```
import java.util.{HashMap => _, _}
import scala.collection.mutable._
```

现在，HashMap无二义地指向scala.collection.mutable.HashMap，因为java.util.HashMap被隐藏起来了。

## 7.10 隐式引入

每个Scala程序都隐式地（默认）以如下代码开始（即默认自动导入以下包，这些包下面的子包不需要再带上这些前缀）：

```
import java.lang._
import scala._
import Predef._
```

和Java程序一样，java.lang总是被引入。接下来，scala包也被引入，不过方式有些特殊。不像所有其他引入，这个引入被允许可以覆盖之前的而引入。例如，scala.StringBuilder会覆盖java.lang.StringBuffer而不是与之冲突。最后，Predef对象被引入。它包含了相当多有用的函数。（这些同样可以被放置在scala包对象中，不过Predef在Scala还没有加入包对象之前就已经存在了。）

由于scala包默认被引入，对于那些以scala开头的包，完全不需要写全这个前缀。例如：collection.mutable.HashMap和以下这个写法scala.collection.mutable.HashMap效果一样。

