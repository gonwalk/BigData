# 第17章 类型参数

在Scala中，可以使用类型参数实现类和函数，这样的类和函数可以用于多种类型。如，Array\[T\]可以存放任意类型T的元素。有时，需要对类型做出限制。例如，要对元素排序，T必须提供一定的顺序定义。

本章要点概览：

* 类、特质、方法和函数都可以有类型参数。
* 将类型参数放置在名称之后，以方括号括起来。
* 类型界定的语法为T&lt;: UpperBound、 T&gt;: LowerBound、 T&lt;% ViewBound、 T：ContextBound。
* 可以使用类型约束来约束一个方法，比如\(implicit ev: T&lt;:&lt; UpperBound\)。
* 用+T（协变）来表示某个泛型类的子类型关系和参数T方法一致，或用-T（逆变）来表示方向相反。
* 协变适用于表示输出的类型参数，比如不可变集合中的元素。
* 逆变适用于表示输入的类型参数，比如函数参数。

## 17.1 泛型类

和Java或C++一样，类和特质可以带有类型参数。在Scala中，用方括号来定义类型参数。例如，定义一个带有两个类型参数T和S的类，在类的定义中可以使用类型参数来定义变量、方法参数、以及返回值的类型。

```
class Pair[T, S](val first: T, val second: S)
```

带有一个或多个类型参数的类是泛型的。如果把类型参数替换成实际的类型，将得到一个普通的类，比如Pair\[Int, String\] 。Scala会从构造参数推断出实际的类型 ：

```
val p1 = new Pair(43, "String)    //这是一个Pair[Int, String]
//也可以自己指定类型：
val p2 = new Pair[Any, Any](43, "String")
```

shell 运行过程：

```
scala> class Pair[T, S](val first: T, val second: S)
defined class Pair

scala> val p1 = new Pair(43, "String")
p1: Pair[Int,String] = Pair@6738ccc0

scala> val p2 = new Pair[Any, Any](43, "String")
p2: Pair[Any,Any] = Pair@4b2b9e36
```

## 17.2 泛型函数

函数和方法也可以带类型参数，和泛型类一样，需要把类型参数放在方法名之后。Scala会从调用该方法使用的实际参数来推断出类型。如有必要，也可以指定类型。

```
def getMiddle[T](a: Array[T]) = a(a.length / 2)      //返回a数组下标为 a数组长度的一半 位置处的字符串（下标从0开始）
getMiddle(Array("Mary", "had", "a", "little", "lamb"))  //将会调用getMiddle[String]
val f = getMiddle[String] _    //这是具体的函数，保存到f
```

shell运行过程

```
scala> def getMiddle[T](a: Array[T]) = a(a.length / 2)
getMiddle: [T](a: Array[T])T

scala> getMiddle(Array("Mary", "had", "a", "little", "lamb"))
res26: String = a

scala> val f = getMiddle[String] _
f: Array[String] => String = <function1>
```

## 17.3 类型变量界定

有时，需要对类型变量进行限制。如有一个Pair类型，要求它的两个组件类型相同，在类中添加一个方法，产出较小的那个值。

```
class Pair[T](val first: T, val second: T){
    def smaller = if(first.compareTo(second) < 0) first else second  //错误
}
```

上面的是错误的写法，因为并不知道first是否有compareTo方法。要解决这个问题，**可以添加一个上届T&lt;: Comparable\[T\]**。

```
class Pair[T <: Comparable[T]](val first: T, val second: T){
    def smaller = if(first.compareTo(second) < 0) first else second  
}
```

这意味着T必须是Comparable\[T\]的子类型。这样，可以实例化Pair\[java.lang.String\]，但不能实例化Pair\[java.io.File\]，因为String是Comparable\[String\]的子类型，而File并没有实现Comparable\[File\]。例如：

```
scala> class Pair[T <: Comparable[T]](val first: T, val second: T){
     | def smaller = if(first.compareTo(second) < 0) first else second  
     | }
defined class Pair

scala> val p = new Pair("Fred", "Brooks")
p: Pair[String] = Pair@7446bbe5

scala> println(p.smaller)
Brooks
```

也可以给类型指定一个下界。如：定义一个方法，用另一个值替换对偶的第一个组件。其中对偶是不可变的，因此需要返回一个新的对偶。假定有一个Pair\[Student\]，允许使用一个Person替换第一个组件，这样做的结果将会是一个Pair\[Person\]。通常，替换进来的类型必须是原类型的超类型。这样，返回值会被正确地推断为new Pair\[R\]。

```
class Pair[T](val first: T, val second: T){
    def replaceFirst[R >: T](newFirst: R) = new Pair[R](newFirst, second)
    //也可以写成def replaceFirst[R >: T](newFirst: R) = new Pair(newFirst, second)
}
```

注意：上面的如果不写上界def replaceFirst\[R\]\(newFirst: R\) = new Pair\(newFirst, second\)，上述方法也可以编译通过，但是它将返回Pair\[Any\]。

## 17.4 视图界定

在上一节class Pair\[T

&lt;

: Comparable\[Int\]\]的示例中，如果试着new一个Pair\(3, 2\)，编译器会提示Comparable\[Int\]的子类型。

与java.lang.Integer包装类型不同，Scala的Int类型并没有实现Comparable。不过，RichInt实现了Comparable\[Int\]，同时还有一个从Int到RichInt的隐式转换。

对于上面问题的解决方法是使用“视图界定”，如下的形式：

class Pair\[T

&lt;

% Comparable\[T\]\]

&lt;

%关系意味着T可以被隐式转换为Comparable\[T\]。

说明：使用Ordered特质会更好，它在Comparable的基础上额外提供了关系操作符：

```
class Pair[T 
<
% Ordered[T]](val first: T, val second: T){
    def smaller = if(first 
<
second) first else second
}
```

java.lang.String实现了Comparable\[String\]，但没有实现Ordered\[String\]。有了视图之后，字符串可以被隐式转换为RichString，而RichString是Ordered\[String\]的子类型。

## 17.5 上下文界定

视图界定T

&lt;

% V要求必须存在一个从T到V的隐式转换。上下文界定的形式为T: M，其中M是另一个泛型类型。它要求必须存在一个类型为M\[T\]的“隐式值”。

```
class Pair[T: Ordering](val first: T, val second: T){
    def smaller(
implicit
 ord: Ordering[T] = if(ord.compare(first, second) 
<
 0) first else second
}
```

上面的定义中要求必须存在一个类型为Ordering\[T\]的隐式值，该隐式值可以被用在该类的方法中。

当声明一个使用隐式值的方法时需要添加一个“隐式参数”。

## 17.6 Manifest上下文界定

要实现一个泛型的Array\[T\]，需要一个Manifest\[T\]对象。要想让基本类型的数组能正常工作，这是必需的。如果T是Int，你会希望虚拟机中对应的是一个int\[\]数组。在Scala中，Array只不过是类库提供的一个类，编译器并不对它做特殊处理。如果要编写一个泛型函数来构造泛型数组的话，需要传入这个Manifest对象。由于它是构造器的隐式参数，可以使用上下文界定

，如下：

```
def makePair[T : Manifest](first: T, second: T) {
    val r = new Array[T](2); r(0) = first; r(1) = second; r
}
```

如果调用makePair\(4, 9\)，编译器将定位到隐式的Manifest\[Int\]并实际上调用makePair\(4, 9\)\(intManifest\)。这样，该Array方法调用的就是new Array\(2\)\(intManifest\)，返回基本类型的数组int\[2\]。在shell中的运行过程：

```
scala
>
 def makePair[T: Manifest](first: T, second: T){
     | val r = new Array[T](2);
     | r(0) = first;
     | r(1) = second;
     | r
     | }
makePair: [T](first: T, second: T)(implicit evidence$1: Manifest[T])Unit

scala
>
 makePair(3,6)
```

问什么搞得这么复杂？在虚拟机中，泛型相关的类型信息是被抹掉的。只会有一个makePair方法，却要处理所有类型T。

## 17.7 多重界定

类型变量可以同时又上届和下届。不能同时有多个上届或多个下届。不过，依然可以要求一个类型实现多个特质。类型变量可以有多个试图界定，也可以有多个上下文界定。

```
//类型变量同时有上届和下届：
T
>
: Lower 
<
: Upper

//一个类型实现多个特质：
T
<
: Comparable[T] with Serializable with Cloneable

//可以有多个视图界定：
T 
<
% Comparable[T] 
<
% String

//多个上下文界定：
T: Ordering : Manifest
```

## 17.8 类型约束

类型约束提供的是另一个限定类型的方式，总共有三种关系可供使用：

1. T =:= U
2. T 
   &lt;
   :
   &lt;
   U
3. T 
   &lt;
   %
   &lt;
   U

这些约束将回测试T是否等于U，是否为U的子类型，或者能否被视图（隐式）转换为U。要使用这样一个约束，需要添加“隐式类型证明参数”：

class Pair\[T\]\(val first: T, val second: T\) \(implicit ev: T

&lt;

:

&lt;

Comparable\[T\]\)

类型约束的用途一：在泛型类中定义只能在特定条件下使用的方法。

```
class Pair[T](val first: T, val second: T){
    def smaller(implicit ev: T 
<
: Ordered[T]) = if(first 
<
 second)first else second
}
```

另一个示例Option类的orNull方法：

```
scala
>
 val friends = Map("Fred" -
>
 "Barney", "Tom" -
>
 "Wilma")
friends: scala.collection.immutable.Map[java.lang.String,java.lang.String] = Map(Fred -
>
 Barney, Tom -
>
 Wilma)

scala
>
 val friendOpt = friends.get("Wilma")
friendOpt: Option[java.lang.String] = None

scala
>
 val friendOpt = friends.get("Fred")
friendOpt: Option[java.lang.String] = Some(Barney)

scala
>
 val friendOrNull = friendOpt.orNull
friendOrNull: java.lang.String = Barney
```

在和Java代码打交道时，orNull方法很有用，因为Java中通常习惯使用null表示缺少某值。不过这种做法并不适用于值类型，比如Int，它们并不把null看做是合法的值。因为orNull的实现带有约束Null

&lt;

:

&lt;

A，仍然可以实例化Option\[Int\]，只要不对这些实例使用orNull。

类型约束的用途二：改进类型推断：

```

```

```
scala
>
 def firstLast[A, C 
<
: Iterable[A]](it: C) = (it.head, it.last)
firstLast: [A, C 
<
: Iterable[A]](it: C)(A, A)

scala
>
 firstLast(List(1, 2, 3))

<
console
>
:9: error: inferred type arguments [Nothing,List[Int]] do not conform to method firstLast's type parameter bounds [A,C 
<
: Iterable[A]]
              firstLast(List(1, 2, 3))
              ^
//推断出类型参数[Nothing, List[Int]]不符合[A, C
<
: Iterable[A]],由于类型推断期单凭List(1, 2, 3)无法判断A是什么，因为它是在同一个步骤中匹配到A和C的。
```

```
//解决方式， 首先匹配C，然后匹配A:
```

```
scala
>
 def firstLast[A, C](it: C)(implicit ev: C 
<
:
<
 Iterable[A]) = (it.head, it.last)
firstLast: [A, C](it: C)(implicit ev: 
<
:
<
[C,Iterable[A]])(A, A)

scala
>
 firstLast(List(1, 2, 3))
res5: (Int, Int) = (1,3)
```

## 17.9 型变

如果Student是Person的子类，也不能使用Pair\[Student\]作为参数调用函数 def makeFriends\(p: Pair\[Person\]\)。因为尽管Student是Person的子类型，但是Pair\[Student\]和Pair\[Person\]之间没有任何关系。

如果要想要有这样的关系，则必须在定义Pair类时表明这一点：

class Pair\[+T\]\(val first: T, val second: T\)

+号意味着该类型是与T协变的——即它与T按照同样方向型变。由于Student是Person的子类型，同方向型变也就意味着Pair\[Student\]同样是Pair\[Person\]的子类型。

同样，-号表示逆变，即如有Student是Person的子类型，按照下面的例子则有Friend\[Student\]是Friend\[Person\]的超类型。

```
class Person extends Friend[Person]
class Student extends Person

trait Friend[-T] {
    def befriend(someont: T)
}
```

在一个泛型的类型声明中，可以同时使用协变和逆变这两种类型。

## 17.10 协变和逆变点

函数在参数上式逆变的，在返回值上则是协变的。通常而言，对于某个对象消费的值使用逆变，而对于它产出的值则适用协变。如果一个对象同时消费和产出某值，则该类型应该保持不变。这通常适用于可变数据结构。如，在Scala中数组是不支持形变的。不能将一个Array\[Student\]转换为Array\[Person\]，或者反过来。

## 17.11 对象不能泛型

没法给对象添加类型参数，比如可变列表。元素类型为T的列表要么为空，要么是一个头部类型为T，尾部类型类List\[T\]的节点：

```
abstract class List[+T] {
    def isEmpty: Boolean
    def head: T
    def tail: List[T]
}

class Node[T](val head: T, val tail: List[T]) extends List[T]{
    def isEmpty = false
}

class Empty[T] extends List[T] {
    def isEmpty = true
    def head = throw new UnsupportedOperationException
    def tail = throw new UnsupportedOperationException
}
//Nothing类型是所有类型的子类型。
object Empty extends List[Nothing]
//调用下面的语句，将把Lsit[Nothing]转换为List[Int]
val lst = new Node(43, Empty)
```

说明：上卖弄使用Node和Empty是为了识记，在Scala列表中，只要把它们替换成::和Nil即可。

## 17.12 类型通配符

在Java中，所有泛型都是不变的。不过，可以在使用时使用通配符改变它们的类型。如，方法void makeFriend\(Pair&lt;? extends Person&gt; people\)  可以用List&lt;Student&gt;

作为参数调用。也可以在Scala中使用通配符：

```
def process(people:java.util.List[_ <: Person]  //这是Scala
```

在Scala中，对于协变的Pair类，不需要使用通配符。也可以对逆变使用通配符。类型通配符是用来指代存在类型的“语法糖”。

