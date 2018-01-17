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

* * \* / % 等操作符完成的事它们通常的工作，位操作符& \| ^ &gt;&gt; &lt;&lt; 也一样。只是有一点比较特别：这些操作符实际上是方法。例如： a + b 是如下方法调用的简写：a.+ \(b\)

**这里的+是方法的名称。注意：Scala中对方法的命名规则与Java中有一点不同：在Java中，方法、属性、类的命名规则是要符合标识符命名规范的——可以使用字符、下划线、数字的组合，其中不能以数字开头，特殊符号，如\*&^%之类的字符；但是，在Scala中几乎可以使用任何符号来为方法命名。**比如，BigInt类中就定义了一个名为/%的方法，该方法返回一个对偶，而对偶的内容是除法操作得到的商和余数。

和Java或C++相比，Scala有一个显著的不同，Scala并没有提供++和--操作符，需要使用+= 1或者-= 1代替：

counter += 1                               //将counter递增——Scala没有++

**注意：在scala中，你无法简单地实现一个名为++的方法，因为Int类是不可变的，这样一个方法并不能改变某个整数类型的值。**

**说明：在Java中，不能对操作符进行重载**，Java的设计者们给出的解释是，这样做防止大家创造出类似!@$&\*这样的操作符，是程序变得没法读。而Scala允许你定义操作符，由你来决定是否要在必要时有分寸地使用这个特性。

## 1.5 调用函数和方法

除了方法之外，Scala还提供函数。相比Java，在Scala中使用数学函数

（比如min或pow）更为简单——不需要从某个类调用它的静态方法。

pow\(2, 4\)                            //将产生16.0

min\(3, Pi\)                            //将产生3.0

这些数学函数是在java.math包中定义的，使用时需要使用如下的语句进行引入：

import scala.math.\_            //在Scala中，\_字符是“通配符”，类似Java中的\*

说明：使用以scala.开头的包时，可以省去scala前缀。例如，import math.\_等同于import scala.math.\_，而math.sqrt\(2\)等同于scala.math.sqrt\(2\)。

**Scala没有静态方法，不过它有个类似的特性，叫做单例对象（singleton object）。通常，一个类对应有一个伴生对象（companion object），其方法就跟Java中的静态方法一样**。举例来说，BigInt类的BigInt伴生对象有一个生成指定位数的随机素数的方法probablePrime：

BigInt.probablePrime\(100, scala.util.Random\) ，这样的调用跟Java中的静态方法的调用很相似（都是通过类名.静态方法名的形式直接调用）。

_**注意：在Scala中，不带参数的Scala方法通常不适用圆括号。例如，StringOps类的API显示它有一个distinct方法，就没有带括号\(\)，起作用是获取字符串中不重复的字符。调用方法如下：**_

"Hello".distinct

**一般来讲，没有参数且不改变当前对象的方法不带圆括号。**

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

## 2.1 条件表达式

Scala的if/else语法结构和Java或C++一样。不过，在Scala中if/else表达式有值，这个值就是跟在if/else之后的表达式的值。例如：

if\(x &gt; 0\) 1 else -1

上述表达式的值是1或-1，具体是哪一个取决于x的值。同时，还可以将if/else表达式的值赋值给变量：

val  s  =  if\(x &gt; 0\) 1 else -1

这一句与if\(x &gt;0\) s = 1 else s = - 1 的效果是一样的。

**不过，第一种写法更好，因为它可以用来初始化一个val。而第二种写法当中，s必须是var。**

在Scala中，每个表达式都有一个类型。举例来说，表达式if \( x &gt; 0）1 else -1 的类型是Int，因为两个分支的类型都是Int。混合类型表达式，比如：

if \( x &gt; 0\) "positive" else -1

**上述表达式的类型是两个分支类型的公共超类型。在本例中，其中一个分支是java.lang.String，而另一个分支是Int。它们的公共超类型叫做Any。**

如果else部分缺失，比如：if\( x &gt; 0\) 1 ， 在Scala中，每个表达式都应该有某种值。这个问题的解决方案是引入一个Unit类，写做\(\)。**不带else的这个if语句等同于if\( x &gt; 0\) 1 else \(\) 。**

**可以把上面的\(\)当做是表示“无 有用值”的占位符，将Unit当做Java或C++中的void。**

**从技术上讲，void没有值但是Unit有一个表示“无值”的值。**

说明：Scala没有switch语句，不过它有一个强大得多的模式匹配机制，将在第14章中看到。

## 2.2 语句终止

**在Java或C++中，每个语句都以分号结束。而在Scala中——与JavaScript和其他脚本语言类似——行尾的位置不需要分号。同样，在}以及类似的位置不必写分号，只能够从上下文明确地判断出这里是语句的终止即可。**

**不过，如果想在单行中写下多个语句，就需要将它们以**_**分号**_**隔开。**例如：

if \( n &gt; 0 \) { r = r \* n **; ** n -= 1 }

如果在写较长的语句，需要分两行来写的话，就要确保第一行以一个不能用做语句结尾的符号结尾。通常来说一个比较好的选择是操作符，比如+ - { } 等：

s = s0 + \(v - v0\) \* t +                        //+ 告诉解析器这里不是语句的末尾

```
0.5 \* \(a -a0 \)  \* t \* t
```

## 2.3 块表达式和赋值

**在Java或C++中，块语句是一个包含于{ }中的语句序列。每当需要在逻辑分支或循环中放置多个动作时，都可以使用块语句。**

**在Scala中，{ } 块包含一系列表达式，其结果也是一个表达式。**_**块中最后一个表达式的值就是块的值**_**。**

**在Scala中，赋值动作本身是没有值的——或者说，它们的值是Unit类型的**。Scala中，Unit类型等同于Java和C++中的void，而这个类型只有一个值，写做\(\) 。

由于赋值语句的值是Unit类型的，别把它们串联（联系赋值）在一起。如：

x = y = 1              //不要这样写， y = 1 的值是\(\)，几乎不太可能想把一个Unit类型的值赋值给x。

## 2.4 输入和输出

如果要打印一个值，使用print或println函数。后者在打印完内容后会追加一个换行符。

另外，还有一个带有C风格格式化字符串的printf函数：

printf\("Hello , %s ! You are %d years old . \n ", "Fred", 36\)

可以使用readLine函数从控制台读取一行输入。如果要读取数字、Boolean或者是字符，可以使用readInt、readDouble、readByte、readShort、readLong、readFloat、readBoolean或者readChar。与其他方法不同，readLine带一个参数作为提示字符串：

val name = readLine\("Your name: "\)

print\("Your age: "\)

val age = readInt\(\)

printf\(" Hello, %s! Next year, your will be %d. \n", name , age + 1\)

**注意：早期的Scala中**`Console`**类提供了一系列的终端输入方法，在现在的版本中这些方法已经被废弃。**

* **当前版本的Scala获取终端输入需要使用包**`scala.io.StdIn`**中的相关方法。**
* `scala.io.StdIn`中的相关方法签名与先前的`Console`类中完全相同。
* **使用**`readLine()`**获取单行文本输入，返回**`String`**类型。**
* **使用**`readInt()/readFloat()/readChar()/readLong()...`**等方法获取特定类型的输出，当输入的内容不匹配时，会抛出异常。**
* **使用**`readf()/readf1()/readf2()/readf3()`**等方法能以**`java.text.MessageFormat`**语法格式化接收的终端输入。**

详情参见：Scala学习笔记\(4\) - CSDN博客  
[http://blog.csdn.net/u011152627/article/details/50934769](http://blog.csdn.net/u011152627/article/details/50934769)

如下代码所示：

```
scala> val str = scala.io.StdIn.readLine()      //自行脑补终端输入"Test input"
str: String = Test input
scala> val int = scala.io.StdIn.readInt()       //自行脑补终端输入"200"
int: Int = 200

//输入内容不匹配读取类型时会抛出异常
scala> val double = scala.io.StdIn.readDouble()
java.lang.NumberFormatException: For input string: "test"
  at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:2043)
  at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
  at java.lang.Double.parseDouble(Double.java:538)
  at scala.collection.immutable.StringLike$class.toDouble(StringLike.scala:284)
  at scala.collection.immutable.StringOps.toDouble(StringOps.scala:30)
  at scala.io.StdIn$class.readDouble(StdIn.scala:155)
  at scala.io.StdIn$.readDouble(StdIn.scala:229)
  ... 33 elided

//readf()可以接收任意数量的值，返回值为List[Any]类型
scala> val list = scala.io.StdIn.readf("{0} + {1}")             //输入该语句后，在光标提示处输入字符串——Test + Input
list: List[Any] = List(Test, Input)                             //按照格式化字符串提取出了输入内容，list为List类型的数据
scala> list foreach { println }                                 //循环输出list中的数据："Test"和"Input"，每次输出后换行
Test
Input

//readf1()仅能接收一个值，返回接收的值
scala> val num = scala.io.StdIn.readf1("This is {0}")           //输入该语句后，在光标提示处输入666
num: Any = 666
//readf2()/readf3()接收两个/三个值，返回值为Tuple类型
scala> val tuple = scala.io.StdIn.readf3("{0} + {1} + {2}")     //自行脑补终端输入"One + Two + Three"
tuple: (Any, Any, Any) = (On,Two,Three)
```

```
scala> print("Answer:")
Answer:
scala> println("Answer: " + 26)
Answer: 26

scala> printf("Hello , %s ! You are %d years old . \n ", "Fred", 36)
Hello , Fred ! You are 36 years old . 

scala> val name = readLine("Your name:")
warning: there was one deprecation warning; re-run with -deprecation for details
Your name:name: String = sym

scala> print("Your age:")
Your age:
scala> val ageInt = scala.io.StdIn.readInt()
ageInt: Int = 20

scala> val age = readInt()
warning: there was one deprecation warning; re-run with -deprecation for details
age: Int = 36
scala> val str = scala.io.StdIn.readLine()
str: String = hello

scala> val int = scala.io.StdIn.readInt()
int: Int = 200

scala> val double = scala.io.StdIn.readDouble()
double: Double = 36.66
```

## **2.5 循环**

Scala拥有与Java和C++相同的while和do循环。Scala没有与for（初始化变量；检查变量是否满足某条件；更新变量）循环直接对应的结构。如果要使用这样的循环，有两个选择：一是使用while循环，二是使用如下for语句：

```
scala> var r = 0
r: Int = 0

scala> for(x <- 1 to 10)
     | r = r + x

scala> r
res10: Int = 55
```

在第1章中的RichInt类中曾提到过这个to方法。1 to n这个调用返回数字1到数字n（包含）的Range（区间）。

下面的这个语法结构

for\( i &lt;-  表达式\)

**让变量i遍历&lt;-右边的表达式的所有值（注意：这里的**_**符号“&lt;-”应该理解为一个整体，中间没有空格**_**，要写成&lt; -中间多一个空格的形式则会报错）。对于Scala集合，比如Range而言，这个循环会让i依次取得区间中的每一个值。**

**说明：在for循环的变量之前并没有val或var的指定。该变量的类型是集合的元素类型。循环变量的作用域一直持续到循环结束。**

**遍历字符串或数组时，通常需要**_**使用从0到n - 1的区间。这个时候可以用until方法而不是to方法**_**。until方法返回一个并不包含上限的区间。**

```
scala> val s = "Hello"
s: String = Hello

scala> var sum = 0
sum: Int = 0

scala> for (i <- 0 until s.length)               //i的最后一个取值是s.length - 1
     | sum += s(i)

scala> sum
res18: Int = 500
```

在本例中，事实上我们并不需要使用下标。可以直接遍历对应的字符序列：

```
scala> var sum = 0
sum: Int = 0

scala> for( ch <- "Hello")
     | sum += ch

scala> sum
res20: Int = 500
```

**在Scala中，对循环的使用通常可以通过对序列中的所有值应用某个函数的方式来处理，而完成这项工作只需要一次方法调用即可。**

**说明：Scala并没有提供break或continue语句来退出循环。如果需要break时，有如下几个选项：**

**1.使用Boolean型的控制变量。**

**2.使用嵌套函数——可以从函数当中return。**

**3.使用Breaks对象中的break方法：**

```
scala> var res = 0
res: Int = 0

scala> breakable{
     | for(i <- 1 to 100){
     | if(i % 2 == 0)break else res += i
     | }
     | }

scala> res
res23: Int = 1
```

在这里，控制权的转移是通过抛出和捕获异常完成的，因此，如果时间很重要的话，应该尽量避免使用这套机制。

## 2.6 高级for循环和for推导式

在2.5节，是for循环的基本形态。Scala中的for循环比起Java和C++功能要丰富得多。

可以以变量 &lt;- 表达式 的形式提供多个生成器，用分号将它们隔开。例如，

```
scala> for( i <- 1 to 3; j <- 1 to 3)print(( 10 * i + j) + " ") //将打印11 12 13 21 22 23 31 32 33 
11 12 13 21 22 23 31 32 33
```

每个生成器都可以带一个守卫，以if开头的Boolean表达式（注意在if之前没有分号）：

```
scala> for( i <- 1 to 3;j <- 1 to 3 if i != j)print((10 * i + j) + " ")
12 13 21 23 31 32
```

可以使用任意多的定义，引入可以在循环中使用的变量：

```
scala> for( i <- 1 to 3; from = 4 - i; j <- from to 3) print((10 * i + j) + " ")
13 22 23 31 32 33
```

如果for循环的循环体以yield开始，则该循环会构造出一个集合，每次迭代生成集合中的一个值：

for\( i &lt;- 1 to 10 \) yield i % 3             //生成Vector\(1, 2, 0, 1, 2, 0, 1, 2, 0, 1\)

这类循环叫做for推导式。for推导式生成的集合与它的第一个生成器是类型兼容的。

```
scala> for( c <- "Hello"; i <- 0 to 1) yield (c + i).toChar
res0: String = HIeflmlmop                    //将字符串c中的每个字符对应的ASCII码加一，然后转换为对应的字符。

scala> for( i <- 0 to 1; c <- "Hello") yield (c + i).toChar
res1: scala.collection.immutable.IndexedSeq[Char] = Vector(H, e, l, l, o, I, f, m, m, p)
```

说明：如果愿意，也可以将生成器、守卫和定义包含在花括号当中，并可以以换行的方式而不是分号来隔开它们：

```
for{ i <- 1 to 3 
     | from = 4 - i
     | j <- from to 3 }
```

## 2.7 函数

Scala除了方法外还支持函数。方法对对象进行操作，函数不是。C++也有函数，不过在Java中我们只能使用静态方法来模拟。

**要定义函数，需要给出函数的名称、参数和函数体**，就像这样：

def abs\(x: Double\)  = if \(x &gt;= 0 ） x else  -x

**在Scala函数中必须给出所有参数的类型。不过，只要函数不是递归的，就不需要指定返回类型。Scala编译器可以通过=符号右侧的表达式的类型推断出返回类型。**

如果函数体需要多个表达式完成，可以用代码块。块中最后一个表达式的值就是函数的返回值。下面的函数返回值就是for循环之后的r的值：

```
scala> def fac (n: Int) = {
     | var r = 1 
     | for( i <- 1 to n) r = r * i
     | r
     | }
fac: (n: Int)Int

scala> fac(6)
res5: Int = 720
```

提示：虽然在带名函数中使用return并没有什么不对（除了多打几个字母之外），我们最好适应没有return的日子。之后在会使用到大量的匿名函数，这些函数中return并不返回值给调用者。它跳出到包含它的带名函数中。我们可以把return当做是函数版的break语句，仅在需要时使用。

