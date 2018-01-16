# 《快学Scala》学习笔记



Scala是一门以Java虚拟机（JVM）为目标运行环境并将面向对象和函数式编程语言的最佳特性结合在一起的编程语言。由于Scala运行在JVM智商，因此它可以访问任何Java类库并且与Java框架进行互操作。Scala可以直接编译成Java字节码，因此可以重根利用JVM这个高性能的运行平台为我们提供便利和保障。



# 1 Scala基础



## 1.1 Scala解释器

Scala解释器读到一个表达式，对它进行求值，将它打印出来，接着继续再读下一个表达式。这个过程被称作读取-求值-打印-循环，即REPL。

## 1.2 声明值和变量

以val定义的值实际上是一个常量——无法改变它的内容。如果要声明值可变的变量，可以使用var关键字声明。在Scala中，鼓励使用val——除非真的需要改变它的值。

注意：在Scala中不需要给出值（由val关键字声明）或者变量（由var关键字声明）的类型，这个信息可以从初始化的表达式中推断出来，但在必要的时候也可以指定类型，如下面的例子。同时，声明值和变量的时候如果不做初始化，则会报错。

val greeting:String = null

val greeting:Any = "Hello"

说明：在Scala中，变量或函数的类型总是写在变量或函数名称的后面，中间使用英文冒号:隔开，这样容易阅读复杂类型的声明。在Scala中，仅当同一行代码中存在多条语句时才需要使用逗号隔开，其他情况不需要使用分号作为一条语句的结束。如下例：

val xmax, ymax = 100            //将xmax和ymax设为100

var greeting, message:String = null            //greeting和message都是字符串，被初始化为null

## 1.3 常用类型

和Java一样，Scala也有7种数值类型：Byte、Char、Short、Int、Long、Float和Double，以及1个Boolean类型。跟Java不同的是，这些类型是类，Scala并不刻意区分基本类型和引用类型。你可以对数字执行方法，例如：

1.toString\(\)                //产生字符串“1”

1.to\(10\)                    //也可写成1 to 10，都是产生Range\(1,2,3,4,5,6,7,8,9,10\)

在Scala中，不需要包装类型。在基本类型和包装类型之间的转换是Scala编译器的工作。举例来说，如果创建一个Int类型的数组，最终在虚拟机中得到的是一个int\[\] 数组。

Scala使用底层的java.lang.String类来表示字符串，不过，它通过StringOps类给字符串追加了上百种操作。如，intersect\(\)方法输出两个字符串中共通（共有的）一组字符。

"Hello".intersect\("World"\)            //输出为"lo"

在这个表达式中，java.lang.String对象“Hello”被隐式地转换成了一个StringOps对象，接着StringOps类的intersect方法被应用。在使用Scala文档时，记得要看一下StringOps类。

同样地，Scala还提供了RichInt、RichDouble、RichChar等。它们分别提供了它们可伶的堂兄弟们——Int、Double、Char等——所不具备的便捷方法。之前用到的to\(\)方法事实上就是RichInt类中的方法，在表达式1.to\(10\)中，Int值1首先被转换成RichInt，然后再应用to方法。

最后，还有BigInt和BigDecimal类，用于任一大小（但有穷）的数字。这些类背后分别对应的是java.math.BigInteger和java.math.BigDecimal。它们用起来更加方便，可以用常规的数学操作符来操作它们。



说明：在Scala中，使用方法而不是强制类型转换，来做数值类型之间的转换。举例，99.44.toInt\(\)得到99，99.toChar\(\)得到'c'。和Java一样，toString\(\)将任意的对象转换成字符串。

要将包含了数字的字符创转换成数值，使用toInt\(\)或toDouble\(\)方法。 如，"99".toDouble\(\)得到Double类型的99.44。

## 1.4 算术和操作符重载

Scala的算术操作符和在Java和C++中预期的效果一样。

+ - \* / % 等操作符完成的事它们通常的工作，位操作符& \| ^ &gt;&gt; &lt;&lt; 也一样。只是有一点比较特别：这些操作符实际上是方法。例如： a + b 是如下方法调用的简写：a.+ \(b\)

这里的+是方法的名称。注意：Scala中对方法的命名规则与Java中有一点不同：在Java中，方法、属性、类的命名规则是要符合标识符命名规范的——可以使用字符、下划线、数字的组合，其中不能以数字开头，特殊符号，如\*&^%之类的字符；但是，在Scala中几乎可以使用任何符号来为方法命名。比如，BigInt类中就定义了一个名为/%的方法，该方法返回一个对偶，而对偶的内容是除法操作得到的商和余数。

和Java或C++相比，Scala有一个显著的不同，Scala并没有提供++和--操作符，需要使用+= 1或者-= 1代替：

counter += 1                               //将counter递增——Scala没有++

注意：在scala中，你无法简单地实现一个名为++的方法，因为Int类是不可变的，这样一个方法并不能改变某个整数类型的值。

说明：在Java中，不能对操作符进行重载，Java的设计者们给出的解释是，这样做防止大家创造出类似!@$&\*这样的操作符，是程序变得没法读。而Scala允许你定义操作符，由你来决定是否要在必要时有分寸地使用这个特性。

## 1.5 调用函数和方法

除了方法之外，Scala还提供函数。相比Java，在Scala中使用数学函数

（比如min或pow）更为简单——不需要从某个类调用它的静态方法。

pow\(2, 4\)                            //将产生16.0

min\(3, Pi\)                            //将产生3.0

这些数学函数是在java.math包中定义的，使用时需要使用如下的语句进行引入：

import scala.math.\_            //在Scala中，\_字符是“通配符”，类似Java中的\*



说明：使用以scala.开头的包时，可以省去scala前缀。例如，import math.\_等同于import scala.math.\_，而math.sqrt\(2\)等同于scala.math.sqrt\(2\)。



Scala没有静态方法，不过它有个类似的特性，叫做单例对象（singleton object）。通常，一个类对应有一个伴生对象（companion object），其方法就跟Java中的静态方法一样。举例来说，BigInt类的BigInt伴生对象有一个生成指定位数的随机素数的方法probablePrime：

BigInt.probablePrime\(100, scala.util.Random\) ，这样的调用跟Java中的静态方法的调用很相似（都是通过类名.静态方法名的形式直接调用）。





注意：在Scala中，不带参数的Scala方法通常不适用圆括号。例如，StringOps类的API显示它有一个distinct方法，就没有带括号\(\)，起作用是获取字符串中不重复的字符。调用方法如下：

"Hello".distinct

一般来讲，没有参数且不改变当前对象的方法不带圆括号。

## 1.6 apply方法

在Scala中，我们通常都会使用类似函数调用的语法。举例来说，如果s是一个字符串，那么s\(i\)就是该字符串的第i个字符。（在C++中，写成s\[i\]；在Java中，写成s.charAt\(i\)。）在REPL中运行一下代码：

"Hello"\(4\)                    //将产生'o'

你可以把这种用法当做是\(\)操作符的重载形式，它背后的实现原理是一个名为apply的方法。举例来说，在StringOps类的文档中，你会发现这样一个方法：

def  apply\(n: Int\): Char

也就是说，"Hello"\(4\)是如下语句的简写：

"Hello".apply\(4\)

在BigInt伴生对象的文档中，同样有将字符串或数字转换为BigInt对象的apply方法。如：BigInt\("123456789"\)就是BigInt.apply\("123456789"\)的简写。这个语句产生出一个新的BigInt对象，不需要使用new。

## 1.7 Scaladoc

Java程序员们使用Javadoc来浏览Java API。Scala也有它自己的版本，兼做Scaladoc。

可以在www.scala-lang.org/api在线浏览Scaladoc，不过，从www.scala-lang.org/downloads\#api下载一个副本安装到本地也是个不错的主意。

和Javadoc按照字母顺序列出清单不同，Scaladoc的类清单按照包排序。如果知道类名但不知道包名，可以用位于左上角的过滤器。注意每个类名旁边的O和C，它们分别链接到对应的类（C）或伴生对象（O）。

![](/assets/scaladoc窍门一.png)

![](/assets/scaladoc窍门二.png)



# 第2章 控制结构和函数



在Java和C++中，把表达式（如3 + 4）和语句（比如if语句）看做两样不同的东西。表达式有值，而语句执行动作。在Scala中，几乎所有构造出来的结构都有值。这个特性使得程序更加精简，也更易读。

![](/assets/第二章要点.png)
