Scala混合了面向对象和函数式的特性。在函数式编程语言中，函数时“头等公民”，可以像任何其他类型一样被传递和操作；只需要将明细动作包含在函数当中作为参数传入即可。  
本章要点概述：

* 在Scala中函数时“头等公民”，就和数字一样。
* 可以创建匿名函数，通常还会把它们交给其他函数。
* 函数参数可以给出需要稍后执行的行为。
* 许多集合方法都接受函数参数，将函数应用到集合中的值里面。
* 有许多语法上的简写以简短且易读的方式表达函数参数。
* 可以创建操作代码块的函数，它们看上去就像是內建的控制语句。

## 12.1 作为值的函数 {#121-作为值的函数}

在Scala中，可以在变量中存放函数：

```
import scala.math._

val num = 3.14
val fun = ceil _    //将fun设为ceil函数
fun(num)     //调用fun中的函数，返回值为4.0
Array(3.14, 1.42, 2.0).map(fun)
```

上面的代码中，将fun设为ceil函数。ceil函数后的_意味着确实指的是这个函数，而不是碰巧忘记了给它传送参数。如果不带_会出错。

    scala> import scala.math._
    import scala.math._

    scala> val num = 3.14
    num: Double = 3.14

    scala> val fun = ceil _
    fun: Double => Double = <function1>

    scala> val fun = ceil 
    <console>:10: error: missing arguments for method ceil in class MathCommon;
    follow this method with `_' if you want to treat it as a partially applied function
           val fun = ceil 

    scala> fun(num)
    res0: Double = 4.0
    scala> Array(3.14, 1.42, 2.0).map(fun)
    res1: Array[Double] = Array(4.0, 2.0, 2.0)

说明：从技术上讲， \_将ceil方法转成了函数。在Scala中，无法直接操纵方法，而只能直接操纵函数。  
函数的用处：

* 调用它；
* 传递它，存放在变量中，或者作为参数传递给另一个函数。
  如何将fun传递给另一个函数：
  Array\(3.14, 1.42, 2.0\).map\(fun\)
  map方法接受一个函数参数，将它应用到数组中的所有值，然后返回结果的数组。

## 12.2 匿名函数 {#122-匿名函数}

在Scala中，不需要给每一个函数命名，正如不需要给每个数字命名一样。以下是一个命名函数：

```
(x: Double) => 3 * x  //该函数将传给它的参数乘以3
val triple = (x: Double) => 3 * x //将函数存放在变量中
def triple(x: Double） = 3 * x  //跟def一样
Array(3.14, 1.42, 2.0).map((x:Double) => 3 * x)
//不需要给函数命名，可以直接将其传递给另一个函数，结果为Array(9.42, 4.26, 6.0)
//上一条语句也可以写成中置表达式的方格（不需要加句点）：Array(3.14, 1.42, 2.0) map ((x:Double) => 3 * x)
```

## 12.3 带函数参数的函数 {#123-带函数参数的函数}

接受另一个函数作为参数的函数：

```
def valueAtOneQuarter(f: (Double) => Double) = f(0.25)
valueAtOneQuarter(ceil _)         //结果为1.0，实际上该函数返回的是ceil(0.25) = 1.0的值
valueAtOneQuarter(sqrt _)         //结果为0.5，因为0.5 * 0.5 = 0.25，其实该函数返回的是sqrt(0.25) = 0.5
```

注意，这里的参数可以是任何接受Double并返回Double的函数。valueAtOneQuarter函数将计算那个函数在0.25位置的值。

valueAtOneQuarter的类型是一个带有单个参数的函数，因为它的类型写作：\(参数类型\) =&gt; 结果类型

```
valueAtOneQuarter(f: (Double) => Double) = f(0.25)
//其中f代表作为函数valueAtOneQuarter的参数的函数名，第一个Double为作为函数f的参数的类型，第二个Double为作为函数
//valueAtOneQuarter输入参数的f的结果类型，之后的f(0.25)表示整个函数valueAtOneQuarter的最后的返回值，其结果类型也为Double
```

由于valueAtOneQuarter是一个接收函数作为参数的函数类型，因此被称为高阶函数（higher-order function\)。

高阶函数也可以产出另一个函数：

```
def mulBy(factor: Double) = (x: Double) => factor * x
//相当于mulBy(factor: Double) = ((x: Double) => factor * x)
val quintuple = mulBy(5)   //相当于quintuple = 5*x
quintuple(20)              //等于 5 * 20
```

mulBy函数有一个类型为Double的参数，返回一个类型为\(Double\) =&gt; Double的函数。因此，它的类型为： \(Double\) =&gt; \(\(Double\) =&gt; Double\)

## 12.4 参数（类型）推断

当将一个匿名函数传递给另一个函数或方法时，Scala会尽可能推断出类型信息。例如，不需要将上述代码写成：

valueAtOneQuarter\(\(x: Double\) =&gt; 3 \* x\)   //相当于f\(0.25\) =3 \* 0.25 ，也可以将其写为valueAtOneQuarter\(x =&gt; 3 \* x\)

如果参数在=&gt;右侧只出现一次，可以用\_替换掉它：

valueAtOneQuarter\(3 \* \_\)        //可以理解为：一个将某值乘以3的函数

注意，这些简写形式仅在参数类型已知的情况下有效。

val fun = 3 \* \_      //错误：无法推断出类型

val fun  = 3 \* \(\_ :Double\)  //正确，fun\(3\) 得到结果为9.0

val fun : \(Double\) = &gt; Double = 3 \* \_    //Ok，因为给出了fun的类型

## 12.5 一些有用的高阶函数

Scala集合库中的一些常用的接受函数参数的方法：

**map方法，将一个函数应用到某个集合的所有元素并返回结果**。如，下面是快速产生

0.1, 0.2，...， 0.9的集合的方式: \(1 to 9\).map\(0.1 \* \_\)

说明：有一个通用的规则：如果想要得到的是一个序列的值，那么想办法从一个简单的序列转化得到即可。如打印一个三角形：

\*

\*\*

\*\*\*

........

可以使用\(1 to 9\).map\("\*" \* \_ _\).foreach\( println  \__ \)

**foreach，它和map很像，只不过它的函数并不返回任何值，foreach只是简单地将函数应用到每个元素。**

**filter方法输出所有匹配某个特定条件的元素。**  如得到一个序列中的所有偶数：

\(1 to 9\).filter\(\_ % 2 == 0\)         //2, 4, 6, 8

**reduceLeft方法接受一个二元函数——即一个带有两个参数的函数——并将它应用到序列中的所有元素，从左到右。**例如：

\(1 to 9 \).reduceLeft\(_ \_ _\* \_ \)        //等同于1 \* 2 \* 3 \* 4 \* 5\* 6 \* 7 \*8 \* 9\_

**注意：乘法函数的紧凑写法**_\*\* \_ **\_** \* \_，每个下划线代表一个参数。\*\*

**sortWith方法用来对二元函数排序**。例如：

"Mary has a little labm".split\(" "\).sortWith\(_.length &lt; \_.length\)\_

输出一个按长度递增排序的数组：Array\("a", "had", "Mary", "lamb", "little"\)。

示例：取map中前几项的值，思路：先使用toList方法将Map转换为List类型，然后使用列表中的take\(n\)方法取出前n项的值，再将取出的前n项值通过toMap方法转换为Map类型：

```
var aMap = Map("a" -> 1, "b" -> 2, "c" -> 3, "d" -> 4, "e" -> 5)  
//aMap的类型为scala.collection.immutable.Map[String, Int]
var bList = aMap.toList
//bList: List[(String, Int)] = List((a, 1), (b, 2), (c, 3), (d, 4), (e, 5))
var cList = bList.take(3)      //取出列表中的前3项
//cList: List[(String, Int)] = List((a, 1), (b, 2), (c, 3))
var nMap = cList.toMap     //将列表转为Map
//nMap: scala.collection.immutable.Map[String, Int] = Map(a -> 1, b -> 2, c -> 3)
```

## 12.6 闭包

在Scala中，可以在任何作用域内定义函数：包、类甚至是另一个函数或方法。在函数体内， 可以访问到相应作用域内的任何变量。

注意，函数可以在变量不再处于作用域内时被调用。

闭包由代码和代码用到的任何非局部变量定义构成。

示例：12.3节的mulBy函数：

![](/assets/闭包.png)

## 12.7 SAM转换

在Scala中，一个函数可以作为另一个函数的参数进行传递。但Java并不支持函数，Java程序员需要付出更多才能达到相同的效果。其通常的做法是将动作放在一个实现某接口的类中，然后将该类的一个实例传递给另一个方法。

在很多时候，这些接口都只有单个抽象方法（single abstract method），在Java中它们被称为SAM类型。例如，在按钮被点击时递增一个计数器：

```
var counter = 0
val button = new JButton("Increment")
button.addActionListener(new ActionListener{
    override def actionPerformed(event: ActionEvent){
        counter += 1
    }
})
```

好多样板代码，如果可以只传递一个函数给addActionListener 就好了，像这样：

button.addActionListener\(\(event: ActionEvent\) =&gt; counter += 1\)

为了启用这个语法，需要提供一个隐式转换把一个函数转换成一个ActionListener的实例：

```
implicit def makeAction(action: (ActionEvent) => Unit) =
  new ActionListener {
      override def actionPerformed(event: ActionEvent) { action(event) }
  }
```

只要简单地将这'个函数和界面代码放在一起，就可以在所有预期ActionListener对象的地方传入任何\(ActionEvent\) =&gt; Unit函数了。

## 12.8 柯里化

柯里化（currying）指的是将原来接受两个参数的函数变成新的接受一个参数的函数的过程。新的函数返回一个以原有第二个参数作为参数的函数。

```
//该函数接受两个参数：
def mul(x: Int, y: Int) = x * y
//以下函数接受一个参数，生成另一个接受单个参数的函数：
def mulOneAtATime(x: Int) = (y: Int) => x * y
```

使用上面的接受一个参数的函数，要计算两个数的乘积，需要调用： mulOneAtATime\(6\)\(7\)  //等价于val m = mulOneAtATime\(6\)  val res = m\(7\)   结果都为42

Scala支持如下简写来定义上面的那个柯里化函数：def mulOneAtATime\(x: Int\)\(y: Int\) = x \* y

**corresponds方法可以比较两个序列是否在某个比对条件下相同**。例如：

```
val a = Array("Hello", "World")
val b = Array("hello", "world")
a.corresponds(b)(_.equalsIgnoreCase(_))  //相当于a.equalsIgnoreCase(b)
```

上面的函数.equalsIgnoreCase\( \_ \)是以一个经过柯里化的参数的形式传递的，有自己独立的\(...\)。查看Scaladoc，会看到corresponds的类型声明如下：def conrresponds\[B\]\(that: Seq\[B\]\)\(p: \(A, B\) =&gt; Boolean\) : Boolean

在这里，that序列和前提函数p是分开的两个柯里化的参数。类型推断器可以分析出B出自that的类型，因此就可以利用这个信息来分析作为参数p传入的函数。

## 12.9 控制抽象

在Scala中，可以将一系列语句归组成不带参数也没有返回值的函数。如下函数在线程中执行某段代码：

```
def runInThread(block: () => Unit) {
    new Thread {
        override def run() { block() }
    }.start()
}
```

这段代码以类型为\(\) =&gt; Unit的函数的形式给出，当调用该函数时，需要写成:

runInThread{ \(\) =&gt; println\("Hi"\); Thread.sleep\(10000\); println\("Bye"\) }

Scala程序员可以构建控制抽象：看上去像是编程语言的关键字的函数。

换名调用参数，和一个常规（或者说换值调用）的参数不同，函数在被调用时，参数表达式不会被其值。

## 12.10 return表达式

在Scala中，不需要使用return语句来返回函数值，函数的返回值就是函数体的值。不过，可以使用return来从一个匿名函数中返回值给包含这个匿名函数的带名函数。这对于控制抽象是很有用的。如下函数：

```
def indexOf(str:String, ch: Char):Int = {
    var i = 0
    until (i == str.length) {
        if(str(i) == ch) return i
        i += 1
    }
    return -1
}
```

在这里，匿名函数{ if\(str\(i\) == ch\) return i;  i += 1 }被传递给until。当return表达式被执行时，包含它的带名函数indexOf终止并返回给定的值。如果要在带名函数中使用return的话，则需要给出其返回类型。如在上述的indexOf函数中，编译器没法推断出他会返回Int。

控制流程的实现依赖一个在匿名函数的return表达式中抛出的特殊异常，该异常从until函数传出，并被indexOf函数捕获。

**注意：如果异常在被送往带名函数值前，在一个try代码块中被捕获掉了，那么相应的值就不会被返回。**

# 第13 章 集合

本章要点概述：

* 所有集合都扩展自Iterable特质。
* 集合有三大类，分别为序列、集合、映射。
* 对于几乎所有集合类，Scala都同时提供了可变的和不可变的版本。
* Scala列表要么是空的，要么拥有一头一尾，其中尾部本身又是一个列表。
* 集是无先后次序的集合。
* 用LinkedHashMapSet来保留插入顺序，或者用SortendSet来按顺序进行迭代。
* +将元素添加到无先后次序的集合中；+:和:+向前或向后追加到序列； ++将两个集合串接在一起； -和--移除元素。
* Iterable和Seq特质有数十个用于常见操作的方法。在编写冗长烦琐的循环之前，先看看这些方法是否能满足需求。
* 映射、折叠和拉链操作是很有用的技巧，用来将函数或操作应用到集合中的元素。

## 13.1 主要的集合特质

![](/assets/Scala结合继承层级中的关键特质.png)

Iterable指的是那些能生成用来访问集合中所有元素的Iterator的集合：

```
val coll = ...    //某种Iterable
val iter = coll.iterator
while(iter.hasNext)
    对iter.next()执行某种操作
```



