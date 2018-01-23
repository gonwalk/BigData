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

![](/assets/Scala结合继承层级.png)

关于集合的分类：Seq、Set、Map及其比较，可以参考博客：

[http://blog.csdn.net/bluishglc/article/details/51085917](http://blog.csdn.net/bluishglc/article/details/51085917)

Iterable指的是那些能生成用来访问集合中所有元素的Iterator的集合：

```
val coll = ...    //某种Iterable
val iter = coll.iterator
while(iter.hasNext)
    对iter.next()执行某种操作
```

这是遍历一个集合最基本的方式。  
**Seq是一个有先后次序的值的序列，比如数组或列表。IndexedSeq运行通过整型的下标快速地访问任意元素。比如，ArrayBuffer是带有下标的，而链表不是。**  
**Set是一组没有先后次序的值。在SortedSet中，元素以某种排过序的顺序被访问。Map是一组键值对偶。SortedMap按照键的排序访问其中的实体。**  
这个继承层级和Java很相似，同时有一些对其有一些不错的改进：  
1. 映射隶属于同一个继承层级而不是一个单独的层级关系。  
2. IndexedSeq是数组的超类型，但不是列表的超类型，以便于区分。  
说明：在Java中，ArrayList和LinkedList实现同一个List接口，使得编写那种需要优先考虑随机访问效率的代码十分困难。例如，在一个已排序的序列中进行查找的时候。这是最初的Java集合框架中一个有问题的设计决定。后来的版本加入了RandomAccess这个标记接口来应对这个缺陷。  
每个Scala集合特质或类都有一个带有apply方法的伴生对象，这个apply方法可以用来构建该集合中的实例。

## 13.2 可变和不可变集合 {#132-可变和不可变集合}

Scala同时支持可变的和不可变的集合。不可变的集合从不改变，因此可以安全地共享其引用，甚至是在一个多线程的应用程序中也没问题。举例，既有scala.collection.mutable.Map，也有scala.collection.immutable.Map，它们有一个共有的超类型scala.collection.Map（这个超类型没有定义任何该值操作）。  
**Scala优先采用不可变集合**。scala.collection包中的半生对象产出不可变的集合。如，scala.collection.Map\(“Hello” -&gt; 42\)是一个不可变的映射。  
**总被引入的scala包中和Predef对象里有指向不可变特质的类型别名List、Set和Map。如，Predef.Map和scala.collection.immutable.Map是一回事。**  
提示：使用如下语句：  
import scala.collection.mutable  
就可以通过Map方法得到不可变的映射，用mutable.Map得到可变的映射，因为引入了上述可变的包，而默认情况下的Map指的是scala.collection.immutable下的不可变的映射。

```
scala> import scala.collection.mutable
import scala.collection.mutable

scala> val b = mutable.Map("a" -> 1, "b" -> 2, "c" -> 3)
b: scala.collection.mutable.Map[java.lang.String,Int] = Map(c -> 3, a -> 1, b -> 2)

scala> val a = Map("a" -> 1, "b" -> 2, "c" -> 3)
a: scala.collection.immutable.Map[java.lang.String,Int] = Map(a -> 1, b -> 2, c -> 3)
```

不可变集合的用处：可以基于老的集合创建新的集合。如果numbers是一个不可变的集（且不包含元素9），那么numbers + 9就是一个包含了numbers和9的新集。如果9已经在集中，则得到的是指向老集的引用。这在递归计算中很有用。

```
scala> val numbers = Set(1,2,3,4,5,6)
numbers: scala.collection.immutable.Set[Int] = Set(5, 1, 6, 2, 3, 4)

scala> numbers + 9
res8: scala.collection.immutable.Set[Int] = Set(5, 1, 6, 9, 2, 3, 4)

scala> numbers
res9: scala.collection.immutable.Set[Int] = Set(5, 1, 6, 2, 3, 4)

scala> numbers + 6
res10: scala.collection.immutable.Set[Int] = Set(5, 1, 6, 2, 3, 4)

scala> numbers
res11: scala.collection.immutable.Set[Int] = Set(5, 1, 6, 2, 3, 4)
```

例如，在计算某个整数中所有出现过的阿拉伯数字的集：

```
def digits(n: Int):Set[Int] =
    if(n < 0) digits(-n)  //小于0的取反，为非负
    else if(n < 10) Set(n)//0~10放入集合
    else digits( n/10 ) + ( n % 10)
```

这个方法从包含单个数字的集开始，每一步，添加进另外一个数字。添加某个数字并不会改变原有的集，而是构造出一个新的集。

## 13.3 序列 {#133-序列}

![](/assets/不可变序列.png)  
Vector是ArrayBuffer的不可变版本：一个带有下边的序列，支持快速的随机访问。向量是以树形结构的形式实现的，每个节点可以有不超过32个子节点。对于一个有100万个元素的向量而言，只需要四层节点（10^3 ~ 2^10， 10^6 ~32^4）。访问这样的一个列表中的某个元素只需要4跳，而在链表中，同样的操作评价需要500000跳。

Range表示一个整数序列，Range对昂并不存储所有值而只是起始值、结束值和增量值。可以用to和until方法构造Range对象。  
![](/assets/可变序列.png "这里写图片描述")  
栈、队列、优先级队列都是标准的数据结构，用来实现特定算法。不过，链表有些特殊，它们和在Java、C++或数据结构中接触的可能不太一样。

## 13.4 列表 {#134-列表}

**在Scala中，列表要么是Nil（即空表），要么是一个head元素加上一个tail，而tail又是一个列表。::操作符从给定的头和尾创建一个新的列表。**例如：  
9 :: List\(4, 2\) //List\(9, 4, 2\)  
亦可以将这个列表写成：  
9 :: 4 :: 2 :: Nil  
**注意::是右结合的，通过::操作符，列表将从末端开始构建**，上式构建方向即是9::\(4::\(2::Nil\)\)  
在Java或C++中，使用迭代器遍历链表。在Scala中，也可以这样做，但是使用递归会更加自然。例如，如下函数计算整型链表中所有元素之和：

```
def sum(lst: List[Int]): Int = 
    if(lst == Nil) 0 else lst.head += sum(lst.tail)
```

或者，也可以使用模式匹配：

```
def sum(lst: List[Int]): Int = lst match {
    case Nil => 0
    case h :: t => h + sum(t) //h是lst.head而t 是lst.tail
}
```

```
scala> val list  = List(1, 2, 3, 4, 5, 6)
list: List[Int] = List(1, 2, 3, 4, 5, 6)
scala> def sum(lst:List[Int]): Int = 
     | if(lst == Nil) 0 else lst.head + sum(lst.tail)
sum: (lst: List[Int])Int
scala> sum(list)
res15: Int = 21
scala> List(1, 2, 3, 4, 5, 6).sum  //直接调用Scala类库中的sum方法
res17: Int = 21
```

注意：第二个模式中的::操作符，它将列表“析构”成头部和尾部。

## 13.5 可变列表 {#135-可变列表}

**可变的LinkedList和不可变的List相似，只不过可以通过对elem引用赋值来修改其头部，对next引用赋值来修改尾部。**注意，这里并不是给head和tail赋值。  
示例，如下循环把所有负值都改成0：

```
val lst = scala.collection.mutable.LinkedList(1, -2, 7, -9)
var cur = lst 
while( cur != Nil) {
    if(cur.elem < 0) cur.elem = 0
    cur = cur.next
}
```

如下循环将取出每两个元素中的一个：

```
var cur = lst 
while(cur != Nil && cur.next != Nil){
    cur.next = cur.next.next
    cur = cur.next
}
```

这里，变量cur用起来就像是迭代器，但是实际上它的类型是LinkedList。除LinkedList之外，Scala还提供了一个DoubleLinkedList，区别是它多带一个prev因用。  
注意：如果要把列表中的某个节点变成列表中的最后一个节点，不能将next因用设为Nil，而应该将它设为LinkedList.empty。也不要将它设成null，不然在遍历该链表时会遇到空指针错误。

## 13.6 集 {#136-集}

集是不重复元素的集合。尝试将已有的元素加入集值没有效果。如Set\(2, 0, 1 \) + 1 和Set\(2, 0, 1 \) + 1是一样的。  
和列表不同，集并不保留元素插入的顺序。缺省情况下，集是以哈希集实现的，其元素根据hashCode方法的值进行组织。（Scala和Java一样，每个对象都有hashCode方法。）如，遍历Set\(1,2,3,4,5,6\)元素被访问到的次序为：5 1 6 2 3 4。  
在哈希集中查找元素要比在数组或列表中快得多。链式哈希集可以记住元素被插入的顺序，它会维护一个链表来达到这个目的。  
val weekdays = scala.collection.mutable.LinkedHashSet\(“Mo”, “Tu”, “We”, “Th”, “Fr”\)  
如果想要按照已排序的顺序访问集中的元素，用已排序的集：  
scala.collection.immutable.SortedSet\(1,2,3,4,5,6\)  
**已排序的集是用红黑树实现的。**  
位集（bit set）是集的一种实现，以一个字符序列的方式存放非负整数。如果集中有i，则第i个字位是1.这是个高效的实现，只要最大元素不是特别大。Scala提供了可变和不可变的两个BitSet类。  
contains方法检查某个集是否包含给定的值。subsetOf方法检查某个集中的所有元素是否都被另一个集包含。

```
scala> val digits = Set(1,7, 2, 9)
digits: scala.collection.immutable.Set[Int] = Set(1, 7, 2, 9)

scala> digits contains 0
res29: Boolean = false

scala> digits contains 2
res30: Boolean = true

scala> Set(1,2) subsetOf digits
res31: Boolean = true

scala> digits.contains(2)
res32: Boolean = true

scala> Set(1,2).subsetOf(digits)
res33: Boolean = true
```

union、intersect和diff方法执行通常的集操作，也可以将它们写做\| 、&和&~。还可以将联合（union）写做++，将差异（diff）写做–。

```
val digits = Set(1,7, 2, 9)
val primes = Set(2, 3, 5, 7)
digits union primes  //Set(1, 2, 3, 5, 7, 9)
digits & primes //等于Set(2, 7)
digits -- primes       //等于Set(1, 9)
```

## 13.7 用于添加或去除元素的操作符 {#137-用于添加或去除元素的操作符}

![](/assets/添加和移除元素的操作符.png "这里写图片描述")

```
scala> Vector(1,2,3) :+ 5
res34: scala.collection.immutable.Vector[Int] = Vector(1, 2, 3, 5)

scala> 1 +: Vector(1,2,3)
res35: scala.collection.immutable.Vector[Int] = Vector(1, 1, 2, 3)
```

注意：和其他以冒号结尾的操作符一样，+:是右结合的，是右侧操作元的方法。  
这些操作符都返回新的集合（和原集合类型保持一致），不会修改原有的集合。而可变集合有+=操作符用于修改左侧操作元。

```
val numbers = ArrayBuffer(1, 2, 3)

numbers += 5 //将5添加进numbers
```

对于不可变集合，可以在var上使用+=或 :+=，比如：

```
var numbers = Set(1, 2, 3)
 numbers += 5 //将numbers设为不可变的集numbers + 5
 var numberVector = Vector(1, 2, 3)
 numberVector :+= 5 //这里没法用+=，因为向量没有+操作符
```

要移除元素，使用-操作符。可以用++操作符，一次添加多个元素。下面的++操作符将产出一个与col1类型相同，且包含col1和col2中所有元素的集合。类似地，--操作符将一次移除多个元素。

```
Set(1, 2, 3) -2 //结果为Set(1, 3)

col1 ++ col2
```

提示：Scala提供了许多用于添加和移除元素的操作符。以下是一个汇总：

1. 向后（:+）或向前（+:）追加元素到序列当中。
2. 添加（+）元素到无先后次序的集合中。
3. 用-移除元素。
4. 用++和--来批量添加和移除元素。
5. 对于列表，优先使用::和:::。
6. 改值操作-有+=、++=、-=和- - =。
7. 对于集合，个人更偏向于++、&和- -。
8. 尽量不用++:、+=：和++=：。

说明：对于列表，可以用+:而不是::来保持与其他集合操作的一致性，但有一个例外：模式匹配（case h::t）不认+:操作符。

## 13.8 常用方法







