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

reduceLeft方法接受一个二元函数——即一个带有两个参数的函数——并将它应用到序列中的所有元素，从左到右。例如：

\(1 to 9 \).reduceLeft\(_ \_ \* \_ \)        //等同于1 \* 2 \* 3 \* 4 \* 5\* 6 \* 7 \*8 \* 9_

注意：乘法函数的紧凑写法_ \_ \* \_，每个下划线代表一个参数。_

