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

对于递归函数，我们必须指定返回类型。例如：  
scala&gt; def fac\(n : Int\):Int = {  
     \| if\(n &lt;= 0\) 1 else n\*fac\(n - 1\)  
     \| }  
fac: \(n: Int\)Int  
scala&gt; fac\(3\)  
res0: Int = 6  
scala&gt; fac\(6\)  
res1: Int = 720

如果没有返回类型，Scala编译器无法校验n\*fac\(n - 1\)的类型是Int。

2.8 默认参数和带名参数

# 第3章 数组相关操作

本章要点包括：

* **若长度固定则使用Array，若长度可能有变化则使用ArrayBuffer。**
* _**提供初始值时不要使用new。**_
* _**Scala中使用\(\)而不是\[\]来访问元素。**_
* _**用for\(elem &lt;- arr\)来遍历元素。**_
* **用for\(elem &lt;- arr  if ...\) ...yield ...来将原数组转型为新数组。**
* **Scala数组和Java数组可以互操作；用ArrayBuffer，使用scala.collection.JavaConversions中的转换函数。**

## 3.1 定长数组

如果你需要一个长度不变的数组，可以使用Scala中的Array。例如：

```
scala> val name = new Array[Int](10)                               //10个整数的数组，所有元素初始化为0
scala> val a = new Array[String](10)                               //10个元素的字符串数组，所有元素初始化为null
a: Array[String] = Array(null, null, null, null, null, null, null, null, null, null)

scala> val s = Array("Hello", "World") //长度为2的Array[String]--类型是推断出来的，说明：已提供初始值就不需要使用new
s: Array[String] = Array(Hello, World)

scala> s(0)                            //Scala中使用()而不是[]来访问元素
res0: String = Hello

scala> s(0) = "Goodbye"                //修改数组s中下标为0的元素的值

scala> s                                
res2: Array[String] = Array(Goodbye, World)
```

**在JVM中，Scala的Array以Java数组方式实现。**示例中的数组在JVM中的类型为java.lang.String\[\]。Int、Double或其他与Java中基本类型对应的数组都是基本类型数组。如Array\(2, 3, 5, 7, 11\)在JVM中就是一个int\[\]。

## 3.2 变长数组：数组缓冲

**对于长度按需要变化的数组，Java有ArrayList，C++有vector。Scala中的等效数据结构为ArrayBuffer。在Scala中，可以使用+=在ArrayBuffer尾端添加同类型的元素 ；在尾端追加多个元素，以括号包起来； 用++=操作符**_**追加任何集合**_**。**

    scala> import scala.collection.mutable.ArrayBuffer
    import scala.collection.mutable.ArrayBuffer

    scala> val b = ArrayBuffer[Int]()                  //或者new ArrayBuffer[Int]，一个空的数组缓冲，准备存放整数，其长度可变
    b: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer()

    scala> b += 1                                      //用+=在尾端添加元素                  
    res3: b.type = ArrayBuffer(1)

    scala> b
    res4: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1)

    scala> b(0)
    res5: Int = 1

    scala> b(1)                                        //目前b数组中只有1个元素（下标为0），无法取出第二个元素（下标为1）
    java.lang.IndexOutOfBoundsException: 1
      at scala.collection.mutable.ResizableArray$class.apply(ResizableArray.scala:43)
      at scala.collection.mutable.ArrayBuffer.apply(ArrayBuffer.scala:48)
      ... 32 elided

    scala> b + = (1, 2, 3, 5)
    <console>:15: error: missing argument list for method + in class any2stringadd
    Unapplied methods are only converted to functions when a function type is expected.
    You can make this conversion explicit by writing `$plus _` or `$plus(_)` instead of `$plus`.
    val $ires0 = b.$plus
                   ^
    <console>:13: error: reassignment to val
           b + = (1, 2, 3, 5)
               ^

    scala> b += (1, 2, 3, 5)
    res7: b.type = ArrayBuffer(1, 1, 2, 3, 5)

    scala> b ++= Array(8, 13, 21)                      //用++=操作符追加任何集合
    res8: b.type = ArrayBuffer(1, 1, 2, 3, 5, 8, 13, 21)

    scala> b.trimEnd(5)                                //移除最后5个元素

    scala> b
    res10: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1, 1, 2)

**在数组缓冲的尾端添加或移除元素是一个高效的操作。也可以在任意位置插入或移除元素，但这样的操作并不那么高效——所有在那个位置之后的元素都必须被平移。**举例如下：

```
scala> b.insert(2, 6)                    //在下标2的位置插入元素6，原来的元素后移

scala> b
res12: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1, 1, 6, 2)

scala> b.insert(2, 7, 8, 9)             //在下标2的位置插入多个元素7,8，9，原来在该位置的元素后移插入元素个数位

scala> b
res14: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1, 1, 7, 8, 9, 6, 2)

scala> b.remove(2)                     //移除下标2位置上的元素
res15: Int = 7

scala> b
res16: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1, 1, 8, 9, 6, 2)

scala> b.remove(2, 3)                  //从下标为2的位置开始移除元素，共移除3个元素

scala> b.toArray                       //将ArrayBuffer数组缓冲转换成Array
res18: Array[Int] = Array(1, 1, 2)
```

有时需要构建一个Array，但不知道最终需要装多少个元素。在这种情况下，先构建一个数组缓冲，然后调用：

b.toArray                              //Array\(1, 1, 2\)

反过来，调用c.toBuffer可以将一个数组c转换为一个数组缓冲。

```
scala> var c = b.toArray
c: Array[Int] = Array(1, 1, 2)

scala> c.toBuffer
res19: scala.collection.mutable.Buffer[Int] = ArrayBuffer(1, 1, 2)
```

## 3.3 遍历数组和数组缓冲。

在Java和C++中，数组和数组列表/向量有一些语法上的不同。Scala则更加统一。大多数时候可以用相同的代码处理这两种数据结构。以下是for循环遍历数组或数组缓冲的语法：

for\(i &lt;- 0 until  a.length\)                                    //变量i的取值从0到a.length - 1

```
println\(i + ":" + a\(i\)\)
```

```
scala> var a = 0 until 10
a: scala.collection.immutable.Range = Range(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

scala> for(i <- 0 until a.length)
     | println(i + ":" + a(i))
0:0
1:1
2:2
3:3
4:4
5:5
6:6
7:7
8:8
9:9
```

until是RichInt类的方法，返回所有小于（但不包含）上限的数字。例如：0 until 10      //Range\(0,1,2,3,4,5,6,7,8,9\)。注意：0 until 10实际上是一个方法调用——0.until\(10\)。

如果想要每两个元素一跳，可以让i这样来进行遍历：

0 until \(a.length, 2\)                                              //Range\(0, 2, 4, 6, 8\)

如果想要从数组的尾端开始，遍历的写法为：

\(0 until  a.length\).reverse                                  //Range\(9, 8, 7, 6, 5 ,4,3,2,1, 0\)

如果在循环体中不需要用到数组下标，也可以直接访问数组元素，如下的形式：

```
scala> for(elem <- a)
     | print(elem + " ")
0 1 2 3 4 5 6 7 8 9
```

## 3.4 数组转换

在Scala中，从一个数组或数组缓冲出发，以某种方式对它进行转换是很简单的。这些转换动作不会修改原始数组，而是产生一个全新的数组。

像这样是使用for推导式：

```
scala> val a = Array(2, 3, 5, 7, 11)
a: Array[Int] = Array(2, 3, 5, 7, 11)

scala> val result = for(elem <- a) yield 2 * elem
result: Array[Int] = Array(4, 6, 10, 14, 22)

scala> a
res23: Array[Int] = Array(2, 3, 5, 7, 11)

scala> result
res24: Array[Int] = Array(4, 6, 10, 14, 22)
```

_**for\(...\) yield 循环创建了一个类型与原始集合相同的新集合。**_**如果从数组出发，那么得到的是另一个数组。如果从数组缓冲你出发，那么在for\(...\) yield 之后得到的也是一个数组缓冲。结果包含yield之后的表达式的值，每次迭代对应一个。**

通常，当遍历一个集合时，只想处理那些满足特定条件的元素。这个需求可以通过守卫：for中的if来实现。如：

for\(elem &lt;-  a  if  elem  %  2 == 0\) yield 2 \* elem

```
scala> for(elem <- a if elem % 2 == 0) yield 2 * elem
res25: Array[Int] = Array(4)

scala> for(elem <- result if elem % 2 == 0) yield 2 * elem
res26: Array[Int] = Array(8, 12, 20, 28, 44)
scala> a
res28: Array[Int] = Array(2, 3, 5, 7, 11)

scala> result
res29: Array[Int] = Array(4, 6, 10, 14, 22)
```

**注意：使用for\(...\) yield 语句的结果是一个新的集合，原始集合并没有收到yield之后语句的影响而改变。yield有产生、产出、生成；屈服；收益的意思，这里应该理解为“产出，生成，输出”的意思。**

说明：另一种做法是a.filter\(_ % 2 ==\).map\(2 \* _\)，甚至a.filter{ _  %  2 == 0} map {2 \* _\_}。

```
scala> a.filter(_ % 2 == 0).map(2 * _)
res30: Array[Int] = Array(4)

scala> a.filter{ _ % 2 == 0 } map { _ *2 } 
res31: Array[Int] = Array(4)
```

示例：给定一个整数的数组缓冲，想要移除除第一个负数之外的所有负数。传统的依次执行的解决方案会在遇到第一个负数时置一个标记，然后移除后续出现的负数元素。

```
var first = true
var n = a.length
var i = 0
while(i < n){
    if(a(i)>= 0) i += 1
        else {
            if(first) { first = false; i += 1 }
            else { a.remove(i); n -= 1 }
        }
    }
```

上述方案并不太好：从数组缓冲中移除元素并不高效，把非负值拷贝到前端要好得多。

```
//首先收集需要保留的下标
var first = true
val indexes = for(i <- 0 until a.length if first || a(i) >= 0) yield {
    if(a(i) < 0) first = false; i
}
//然后将元素移动到该去的位置，并截断尾端：
for(j <- 0 until indexes.length) a(j) = a(indexes(j))
a.trimEnd(a.length - indexes.length)
```

这里的关键是，拿到所有的下标好过逐个处理。

## 3.5 常用算法

Scala有內建的函数来处理业务运算，如求和与排序等。

**要使用sum方法求和，元素的类型必须是数值类型：整型、浮点数、BigInteger、BigDecimal等。同理，min和max输出数组或数组缓冲中最小和最大的元素，但其除了可以比较数值类型数据的大小外，还可以比较字符串的大小。**

_**sorted方法将数组或数组缓冲排序并返回经过排序的数组或数组缓冲，这个过程并不会修改原始版本。**_

```
scala> Array(1, 7, 2, 9).sum            //对ArrayBuffer同样适用
res32: Int = 19

scala> ArrayBuffer("Mary", "had", "a", "little", "lamb").max
res33: String = little

scala> val b = ArrayBuffer(1, 7, 2, 9)
b: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1, 7, 2, 9)

scala> val bSorted = b.sorted(_ < _)                    //b没有被改变，bSorted是ArrayBuffer(1,2, 7, 9)
```

还可以提供一个比较函数，不过需要使用到sortWith方法：

```
scala> val bDescending = b.sorted(_ > _)                 //ArrayBuffer(9,7,2,1)
```

**注意：可以直接对一个数组进行排序，但是不能对数组缓冲进行排序。使用scala.util.Sorting中的quickSort\(\)方法进行排序后，原来的待排序的集合发生改变，变成排序之后的内容。**

```
scala> val a = Array(1, 7, 2, 9)
a: Array[Int] = Array(1, 7, 2, 9)

scala> scala.util.Sorting.quickSort(a)

scala> a
res35: Array[Int] = Array(1, 2, 7, 9)
```

**对于min、max和quickSort方法，元素类型必须支持比较操作，这包括了数字、字符串及其他带有Ordered特质的类型。**

如果要显示数组或数组缓冲的内容，可以用mkString方法，它允许指定元素之间的分隔符。该方法的另一个重载版本可以指定前缀和后缀。

```
scala> a
res35: Array[Int] = Array(1, 2, 7, 9)

scala> a.mkString(" and ")                    //结果为"1 and 2 and 7 and 9"
res36: String = 1 and 2 and 7 and 9

scala> a.mkString("<", ",", ">")              //结果为"<1,2,7,9>"
res37: String = <1,2,7,9>

scala> a.toString                             //这里被调用的是来自Java的毫无意义的toString方法
res38: String = [I@3ed2983b

scala> b.toString                             //结果为"ArrayBuffer(1, 7, 2, 9)" ，toString方法报告了类型，便于调试
res39: String = ArrayBuffer(1, 7, 2, 9)
```

## 3.6 解读Scaladoc

**由于Scala的类型系统比Java更丰富，在浏览Scala的文档时，可能会遇到一些看上去很奇怪的语法。所幸，你并不需要理解类型系统的所有细节就可以完成很多有用的工作。可以把表3-1用做“解码指环”。**

![](/assets/Scaladoc解码一.png)![](/assets/Scaladoc解码二.png)

```
scala> b
res42: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1, 7, 2, 9)

scala> b.append(3, 6, 8)                //在数组缓冲b中添加多个元素：3,6，9

scala> b
res44: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer(1, 7, 2, 9, 3, 6, 8)

scala> a
res45: Array[Int] = Array(1, 2, 7, 9)

scala> a.count(_ > 6)                    //统计数组a中大于6的元素的个数
res46: Int = 2

scala> b += 4 -= 5
res47: b.type = ArrayBuffer(1, 7, 2, 9, 3, 6, 8, 4)

scala> b += 5 -= 4                      //在数组缓冲中添加元素5，删去元素4
res48: b.type = ArrayBuffer(1, 7, 2, 9, 3, 6, 8, 5)
```

## 3.7 多维数组

**和Java一样，多维数组是通过数组的数组来实现的。如：Double的二维数组类型为Array\[Array\[Double\]\]。要构造这样一个数组，可以用ofDim方法：**

```
scala> val matrix = Array.ofDim[Double](3,4)                         //定义三行四列的数组，初始值都为0.0
matrix: Array[Array[Double]] = Array(Array(0.0, 0.0, 0.0, 0.0), Array(0.0, 0.0, 0.0, 0.0), Array(0.0, 0.0, 0.0, 0.0))

scala> matrix(2)(3) = 23                                             //将二行三列的元素设置为23

scala> matrix
res50: Array[Array[Double]] = Array(Array(0.0, 0.0, 0.0, 0.0), Array(0.0, 0.0, 0.0, 0.0), Array(0.0, 0.0, 0.0, 23.0))

//创建不规则的数组，每一行的长度各不相同
scala> val triangle = new Array[Array[Int]](10)
triangle: Array[Array[Int]] = Array(null, null, null, null, null, null, null, null, null, null)

scala> for(i <- 0 until triangle.length)
     | triangle(i) = new Array[Int](i + 1)

scala> triangle
res52: Array[Array[Int]] = Array(Array(0), Array(0, 0), Array(0, 0, 0), Array(0, 0, 0, 0), Array(0, 0, 0, 0, 0), Array(0, 0, 0, 0, 0, 0), Array(0, 0, 0, 0, 0, 0, 0), Array(0, 0, 0, 0, 0, 0, 0, 0), Array(0, 0, 0, 0, 0, 0, 0, 0, 0), Array(0, 0, 0, 0, 0, 0, 0, 0, 0, 0))
```

要访问数组中的元素，使用两对圆括号，如：matrix\(行号\)\(列号\) = 数值

## 3.8 与Java的互操作

由于Scala数组是用Java数组实现的，可以在Java和Scala之间来回传递。

**可以引入scala.collection.JavaConversions里的隐式转换方法，这样就可以在代码中使用Scala缓冲，在调用Java方法时，这些对象会被自动包装成Java列表。**

举例来说，java.lang.ProcessBuilder类有一个以List&lt;String&gt;为参数的构造器。以下是在Scala中调用它的写法：

```
scala> import scala.collection.JavaConversions.bufferAsJavaList
import scala.collection.JavaConversions.bufferAsJavaList

scala> import scala.collection.mutable.ArrayBu
ArrayBuffer   ArrayBuilder

scala> import scala.collection.mutable.ArrayBuffer
import scala.collection.mutable.ArrayBuffer

scala> val command = ArrayBuffer("ls", "-al", "/home/test")
command: scala.collection.mutable.ArrayBuffer[String] = ArrayBuffer(ls, -al, /home/test)

scala> val pb = new ProcessBuilder(command)            //Scala到Java的转换
pb: ProcessBuilder = java.lang.ProcessBuilder@45967df8
```

Scala缓冲被包装成了一个实现了java.util.List接口的Java类的对象。

反过来，当Java方法返回java.util.List时，可以让它自动转换成一个Buffer：

```
scala> import scala.collection.JavaConversions.asScalaBuffer
import scala.collection.JavaConversions.asScalaBuffer

scala> import scala.collection.mutable.Buffer
import scala.collection.mutable.Buffer

scala> val cmd:Buffer[String] = pb.command()            //不能使用ArrayBuffer——包装起来的对象仅能包装是个Buffer
cmd: scala.collection.mutable.Buffer[String] = ArrayBuffer(ls, -al, /home/test)
```

如果Java方法返回一个包装过的Scala缓冲，那么隐式转换会将原始的对象解包出来。拿本例来说，cmd == command。

课后习题练习及答案

参考  快学Scala-第三章 数组相关操作 - 开心咿呀 - 博客园[ https://www.cnblogs.com/yiruparadise/p/5519649.html](https://www.cnblogs.com/yiruparadise/p/5519649.html)

1.编写一段代码，将a设置为一个n个随机整数的数组，要求随机数介于0（包含）和n\(不包含\)之间

```
class test{
  def main(args:Array[Int]){
    getArr(10).foreach(println)
  }
  
  def getArr(n:Int): Array[Int] = {
    val a = new Array[Int](n)
    val rand = new scala.util.Random()
    for(i <- a) yield rand.nextInt()
  }
  
}
```

2.编写一个循环，将整数数组中相邻的元素置换

```
class test{
  def main(args:Array[Int]){
    val arr = Array(1,2,3,4,5)
    revert(arr)
    arr.foreach(println)
  }
  
  def revert(a:Array[Int]) = {
    for(i <- 0 until (a.length - 1,2)){
      val t = a(i)
      a(i) = a(i+1)
      a(i+1) = t
    }
  }
}
```

3.重复前一个练习，不过这次生成一个新的值交换过的数组。用for/yield。

```
class test{
  def main(args:Array[Int]){
    val a = Array(1,2,3,4,5)
    val b = revertYield(a)
    b.foreach(println)
  }
  
  def revertYield(a:Array[Int]) = {
    for(i <- 0 until a.length) yield { 
      if( i < (a.length - 1) && i % 2 == 0){
        //偶数下标时，则交换下一个相邻的元素值
        val t = a(i)
        a(i) = a(i+1)
        a(i+1) = t
      }
      a(i) //因为生成新的数组，每一个元素都要返回
    }
  }
}
```

4.给定一个整数数组，产出一个新的数组，包含元数组中的所有正值，以原有顺序排列，之后的元素是所有零或负值，以原有顺序排列。

```
class test{
  def main(args:Array[Int]){
    val a = Array(1,2,3,4,5)
    val b = revertYield(a)
    b.foreach(println)
  }
  
  def revertYield(a:Array[Int]) = {
    for(i <- 0 until a.length) yield { 
      if( i < (a.length - 1) && i % 2 == 0){
        //偶数下标时，则交换下一个相邻的元素值
        val t = a(i)
        a(i) = a(i+1)
        a(i+1) = t
      }
      a(i) //因为生成新的数组，每一个元素都要返回
    }
  }
}
```

5.如何计算Array\[Double\]的平均值？

```
class test{
  def main(args:Array[Int]){
    val a = Array(1.0,5.6,0.0,-3.0)
    val b = average(a)
    println(b)
  }
  
  def average(a:Array[Double]) = {
    var t = 0.0
    for(i <- a){
      t += i
    }
    t/a.length
  }
  
  def ave(a:Array[Double]) = {
     a.sum / a.length
  }
}
```

6.如何重新组织Array\[Int\]的元素将它们反序排列？对于ArrayBuffer\[Int\]你又会怎么做呢？

```
import scala.collection.mutable.ArrayBuffer

object Hello {       
  def main(args: Array[String]) {          
  val a = Array(1,2,3,4,5)
    reverseArray(a)
    println("array reverse:")
    a.foreach(println)
    val b = a.reverse //将a的值逆序回去了
     b.foreach(println)
    
    println("bufferArray reverse:")
    val c = ArrayBuffer(6,7,8,9,0);
    val d = c.reverse        
    d.foreach(println)
  }     
  def reverseArray(a:Array[Int]) = {
    for( i <- 0 until (a.length / 2)){
      val t = a(i)
      a(i) = a(a.length-1-i)
      a(a.length-1-i)=t
    }
  }
}
```

7.编写一段代码，产出数组中的所有值，去掉重复项

```
import scala.collection.mutable.ArrayBuffer

object Hello {       
  def main(args: Array[String]) {          
    val ab = new ArrayBuffer[Int]()
    val c = new ArrayBuffer[Int]()
    println("Input the array line,separated by a space,ended with an enter.")
    val input = readLine().toString().split(' ');
    for(i <- input){
      ab += i.toInt
    }
//    ab.foreach(println)
    c ++= ab.distinct
    c.foreach(println) 
  }     
}
```

8.重新编写3.4节结尾的示例。收集负值元素的下标，反序，去掉最后一个下标，然后对每一个下标调用a.remove\(i\)。比较这样做的效率和3.4节中另外两种方法的效率

```
import scala.collection.mutable.ArrayBuffer

object Hello {       
  def main(args: Array[String]) {          
    val a = ArrayBuffer(1,2,4,-2,0,-1,-4,8)
    deleteNeg(a)
    a.foreach(println) 
  }  
  def deleteNeg(a:ArrayBuffer[Int]) = {
    val indexes = for(i <- 0 until a.length if a(i) < 0 ) yield  i 
    //不能val b = indexex.reverse.trimEnd(1) reverse后，它是一个Seq序列。
    //value trimEnd is not a member of scala.collection.immutable.IndexedSeq[Int]
    val b = new ArrayBuffer[Int]()
    b ++= indexes.reverse
    b.trimEnd(1)
    //remove是ArrayBuffer的函数,如果传入的是Array，则需要调用toBuffer
    for(i <- b){
      a.remove(i)
    }
  }
}
```

9.创建一个由java.util.TimeZone.getAvailableIDs返回的时区集合，判断条件是它们在美洲，去掉”America/“前缀并排序。

```
import scala.collection.mutable.ArrayBuffer
import scala.collection.JavaConversions.asScalaBuffer

object Hello
{
  def main(args: Array[String])  = {
    var c = timeZoneName()
    c.foreach(println)
  }        
  
  def timeZoneName() = {
    val arr = java.util.TimeZone.getAvailableIDs();
    val tmp = (for (i <- arr if i.startsWith("America/")) yield {
      i.drop("America/".length)
    })
    scala.util.Sorting.quickSort(tmp)
    tmp
  }
}
```

10.引入java.awt.datatransfer.\_并构建一个类型为SystemFlavorMap类型的对象，然后以DataFlavor.imageFlavor为参数调用getNativesForFlavor方法，以Scala缓冲保存返回值。

```
import scala.collection.JavaConversions.asScalaBuffer
import scala.collection.mutable.Buffer
import java.awt.datatransfer._

object Hello
{
  def main(args: Array[String])  = {
     val flavors = SystemFlavorMap.getDefaultFlavorMap().asInstanceOf[SystemFlavorMap]
     val buf : Buffer[String] = flavors.getNativesForFlavor(DataFlavor.imageFlavor);
     buf.foreach(println);
  }
}
```



