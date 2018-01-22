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
val fun = ceil _	//将fun设为ceil函数
fun(num) 	//调用fun中的函数，返回值为4.0
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



