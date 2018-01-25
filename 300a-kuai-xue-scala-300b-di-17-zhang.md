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
def getMiddle[T](a: Array[T]) = a(a.length / 2)
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



