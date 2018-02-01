# 1 Scala类和对象

## 1.1  类（class）与对象（object）的关系

**类是对象的抽象，而对象是类的具体实例。类是抽象的，不占用内存，而对象是具体的，占用存储空间。类是用于创建对象的蓝图，它是一个定义包括在特定类型的对象中的方法和变量的软件模板。**

定义完类之后，可以使用 new 关键字来创建类的对象，并通过对象名调用类中的方法。

```
package org.sym.test

class Point(xc: Int, yc: Int){
  var x = xc
  var y = yc
  def move(dx: Int, dy: Int): Unit ={
    x = x + dx
    y = y + dy
    println("x = " + x)
    println("y = " + y)
  }
}

object ClassTest {
  def main(args: Array[String]): Unit = {
    val p = new Point(3, 6)
    p.move(6, 16)
  }
}
```

按照上面的方式定义类之后，工程文件结构如下图1所示：

![](/assets/目录1.jpg)                                        ![](/assets/目录2.jpg)

```
      图1 class放object外对应的目录结构                               图2 class放object内对应的目录结构
```

类定义时，也可以将其放在object内部，这样产生的工程目录结构树如图2所示。从上面的对比中，可以发现将类定义放在object定义之外的会多出一个目录ClassTest，其下面才是定义的Object文件名，表示该源码文件中既有单独的类Class的定义，又有单独的Object的定义。而将类定义放在object里面时，只显式地在包下面包含一个Object文件。

## 1.2  使用IDEA创建Scala程序

在使用IDEA创建Scala程序时，有三种选择Class（存放单独的类文件的定义，类中可以定义各种方法）、Object（既可以存放类文件，还可以存放方法，最重要的是main函数的存放所在）、Trait（相当于 Java 的接口，实际上它比接口功能还强大。与接口不同的是，它还可以定义属性和方法的实现。）。一般在使用IDEA编写Scala程序时，如果代码中只定义了类则可以单独新建一个Class文件，如果单独定义一个接口或尚未实现的方法，则可以新建一个Trait文件，如果需要实例化类，通过main方法调用类中的方法运行检测代码的正确性，则可以创建一个新的Object文件。

![](/assets/Scala类、对象、特质.png)

```
           图3 使用IDEA创建Class、Object、Trait
```

```
package org.sym.test


object ClassTest {
  class Point(xc: Int, yc: Int){
    var x = xc
    var y = yc
    def move(dx: Int, dy: Int): Unit ={
      x = x + dx
      y = y + dy
      println("x = " + x)
      println("y = " + y)
    }
  }

  def main(args: Array[String]): Unit = {
    val p = new Point(3, 6)
    p.move(6, 16)
  }
}
```

Scala中的类不声明为public，一个Scala源文件中可以有多个类。

以上实例的类定义了两个变量**x**和**y**，一个方法：**move**，且该方法没有返回值。

**Scala 的类定义可以有参数，称为类参数，如上面的 xc, yc，类参数在整个类中都可以访问。**接着可以使用 new 来实例化类，并访问类中的方法和变量。

## 1.3 Scala 继承

Scala继承一个基类跟Java很相似, 但我们需要注意以下几点：

* **1、在子类中重写一个**_**非抽象方法需要**_**使用override修饰符。**
* **2、只有**_**主构造函数**_**才可以往基类的构造函数里写参数。**
* **3、在子类中重写**_**超类的抽象方法**_**时，你**_**不需要**_**使用override关键字。**

```
package org.sym.test

//定义父类（基类）
class Point(xc: Int, yc: Int){
  var x = xc
  var y = yc
  //基类中的方法
  def move(dx: Int, dy: Int): Unit ={
    x = x + dx
    y = y + dy
    println("x = " + x)
    println("y = " + y)
  }
}

//定义子类
class Location( val xc:Int, val yc: Int, val zc: Int)extends Point(xc, yc){
  var z: Int = zc
  //重写基类中的方法
  def move(dx: Int, dy: Int, dz: Int): Unit ={
    x = x + dx
    y = y + dy
    z = z + dz
    println("x的坐标为：" + x)
    println("y的坐标为：" + y)
    println("z的坐标为：" + z)
  }
}

object ClassTest {
  def main(args: Array[String]): Unit = {
    //初始化父类，并调用其中的方法
    val p = new Point(3, 6)
    p.move(6, 16)

    //初始化子类，并调用子类中的方法
    val pt = new Location(12, 23, 36)
    pt.move(6, 9, 10)
  }
}
```

运行结果为：

```
x = 9
y = 22
x的坐标为：18
y的坐标为：32
z的坐标为：46
```

Scala 使用 extends 关键字来继承一个类。在上面的示例中， Location 类继承了 Point 类。Point 称为父类\(基类\)，Location 称为子类。

**override val xc**为重写了父类的字段，其中Location类中的xc、yc、x、y是继承（重写）自父类的字段

**extends会继承父类的所有属性和方法，Scala 只允许继承一个父类。**

Scala重写一个非抽象方法，必须用override修饰符。

```
package org.sym.test
class Person{
  var name = ""
  override def toString: String = getClass.getName + "[name = " + name + "]"
}

class Employee extends Person {
  var salary = 0.0
  override def toString: String = super.toString + "[salary = " + salary + "]"
}

object ExtendsTest {
  def main(args: Array[String]): Unit = {
    //初始化基类
    val person = new Person()
    person.name = "zhangsan"
    println(person)

    //初始化子类
    val employee = new Employee
    employee.name = "zhangsan"
    employee.salary = 16688
    println(employee)
  }
}
```

程序运行结果为：

```
org.sym.test.Person[name = zhangsan]
org.sym.test.Employee[name = zhangsan][salary = 16688.0]
```

## 1.4 Scala 单例对象（object）

**在 Scala 中，是没有 static 这个关键字的，但是Scala借鉴了Java中的静态的思想，也为我们提供了单例模式的实现方法，那就是使用关键字 object。**

**在Scala 中使用单例模式时，除了定义的类之外，还要定义一个同名的 object 对象，它和类的区别是，object对象不能带参数。**

**当单例对象与某个类共享同一个名称时，他被称作是这个类的伴生对象：companion object，这就意味着必须在同一个源文件里定义类和它的伴生对象。类被称为是这个单例对象的伴生类：companion class。类和它的伴生对象可以互相访问其私有成员。**

### 1.4.1 单例对象示例

```
package org.sym.singleton

class  Point(xc: Int, yc: Int){
  var x: Int = xc
  var y: Int = yc
  def move(dx: Int, dy: Int): Unit ={
    x = x + dx
    y = y + dy
  }
}
object SingletonObject {
  def main(args: Array[String]): Unit = {
    val p = new Point(10, 28)
    printPoint

    def printPoint: Unit ={
      println("x的坐标为： " + p.x)      //注意，这里的printPoint方法需要放在main函数里，否则会出错
      println("y的坐标为： " + p.y)
    }
  }

}
```

程序运行结果：

```
x的坐标为： 10
y的坐标为： 28
```

### 1.4.2  伴生对象示例

```
package org.sym.singleton

//私有构造方法
class Marker private(val color: String){
  println("创建" + this)
  override def toString: String = "颜色标记" + color
}

// 伴生对象，与类共享名字，可以访问类的私有属性和方法
object Marker {
  private val markers: Map[String, Marker] = Map(
    "red" -> new Marker("red"),
    "blue" -> new Marker("blue"),
    "green" -> new Marker("green")
  )

  def apply(color: String) = (
    if(markers.contains(color))
      markers(color)
    else
      null
  )

  def getMarker(color: String) = {
    if(markers.contains(color))
      markers(color)
    else
      null
  }

  def main(args: Array[String]): Unit = {
    println(Marker("red"))

    println(Marker.getMarker("yellow"))
    //单例对象调用，下面的调用方式省略了.()
    println(Marker getMarker("blue"))
  }
}
```

程序运行结果：

```
创建颜色标记red
创建颜色标记blue
创建颜色标记green
颜色标记red
null
颜色标记blue
```

## 2 特质——Trait

Scala中的 Trait\(特质\) 相当于 Java 的接口，实际上它比接口功能还强大。与接口不同的是，它还可以定义属性和方法的实现。

一般情况下Scala的类只能够继承单一父类，但是对于Trait\(特质\) 可以继承多个，从结果来看就是实现了多重继承（跟Java中的（子）类只能继承自一个父类，接口可以实现多个）。注意：Scala中，不管是类还是特质的继承都是用关键字extends，而在Java中类的继承使用关键字extends，接口的实现使用implements。

Trait\(特质\) 定义的方式与类类似，但它使用的关键字是**trait**，如下所示：

```
trait Equal{
    def isEqual(x: Any):Boolean                         //该方法未被实现
    def isNotEqual(x: Any): Boolean = !isEqual(x)       //该方法定义有方法体，即定义了方法的实现
```

以上Trait\(特质\)由两个方法组成：**isEqual**和**isNotEqual**。isEqual 方法没有定义方法的实现，isNotEqual定义了方法的实现。**子类继承特质可以实现特质中未被实现的（抽象）方法。所以其实 Scala Trait\(特质\)更像 Java 的抽象类。**

以下演示了特征的完整实例：

```
package org.sym.singleton

trait Equal{
  //注意：如果这里写成class Equal,怎会出错，因为类中存在抽象方法（未实现的方法）则需要声明为abstract的类，并在子类中实现该方法。
  //而特质中可以存在尚未实现的方法，所以存在抽象方法的要声明为trait。
  def isEqual(x: Any): Boolean                      //未实现的（抽象）方法
  def isNotEqual(x: Any): Boolean = !isEqual(x)     //定义有方法体，实现了该方法
}

class Point(xc: Int, yc: Int)extends Equal {
  var x: Int  = xc
  var y: Int  = yc

  def isEqual(obj: Any):Boolean = {
    obj.isInstanceOf[Point] && obj.asInstanceOf[Point].x == x
    //isInstanceOf[T]方法判断对象是否为T类型的实例;asInstanceOf[T]方法将对象类型强制转换为T类型。
  }


}
object TraitTest {
  def main(args: Array[String]): Unit = {
    val p1 = new Point(2, 3)
    val p2 = new Point(2, 4)
    val p3 = new Point(3, 3)

    println(p1.isNotEqual(p2))
    println(p1.isNotEqual(p3))
    println(p1.isNotEqual(2))
  }
}
```

执行以上代码，输出结果为：

```
false
true
true
```

---

## 特征构造顺序

特征也可以有构造器，由字段的初始化和其他特征体中的语句构成。这些语句在任何混入该特征的对象在构造时都会被执行。

构造器的执行顺序：

* 调用超类的构造器；
* 特征构造器在超类构造器之后、类构造器之前执行；
* 特征由左到右被构造；
* 每个特征当中，父特征先被构造；
* 如果多个特征共有一个父特征，父特征不会被重复构造
* 所有特征被构造完毕，子类被构造。

构造器的顺序是类的线性化的反向。线性化是描述某个类型的所有超类型的一种技术规格。

# 3 Scala 提取器\(Extractor\)

**提取器的作用是从传递给它的对象中提取出构造该对象的参数。**

## 3.1 apply和unapply方法的使用

**Scala 标准库包含了一些预定义的提取器，Scala 提取器是一个带有unapply方法的对象。unapply方法算是apply方法的反向操作：unapply接受一个对象，然后从对象中提取值，提取的值通常是用来构造该对象的值。**

以下实例演示了邮件地址的提取器对象：

```
package org.sym.extractor

object EmailExtractor {
  def main(args: Array[String]): Unit = {
    println("Apply方法:" + apply("netease", "163.com"))
    println("Unapply方法:" + unapply("netease@163.com"))
    println("Unapply方法：" + unapply("tengxunmail"))
  }

  def apply(user: String, domain: String):String = {
    user + "@" + domain
  }
  def unapply(str: String): Option[(String, String)] = {
    val sp = str.split("@")             //以@为分隔符切分字符串
    if(sp.length == 2)
      Some(sp(0), sp(1))
    else
      None
  }
}
```

程序运行结果：

```
Apply方法 : netease@gmail.com
Unapply方法 : Some((netease, 163.com))
Unapply方法 : None
```

以上对象定义了两个方法：**apply**和**unapply**方法。**通过 apply 方法我们无需使用 new 操作就可以创建对象。**所以你可以通过语句 EmailExtractor\("netease", "163.com"\) 来构造一个字符串 "netease@163.com"。

**unapply方法算是apply方法的反向操作：unapply接受一个对象，然后从对象中提取值，提取的值通常是用来构造该对象的值。**实例中我们使用 unapply 方法从对象中提取用户名和邮件地址的后缀。

当 unapply 方法传入的字符串不是邮箱地址时返回 None。

## 3.2 提取器使用模式匹配

**在实例化一个类的时，可以带上0个或者多个参数，编译器在实例化时会调用 apply 方法。可以在类和对象中都定义 apply 方法。**

**unapply 用于提取指定要查找的值，它与 apply 的操作相反。 当在提取器对象中使用 match 语句时，unapply 将自动执行，**如下所示：

```
object Test {
   def main(args: Array[String]) {
      
      val x = Test(5)
      println(x)

      x match
      {
         case Test(num) => println(x + " 是 " + num + " 的两倍！")
         //unapply 被调用
         case _ => println("无法计算")
      }

   }
   def apply(x: Int) = x*2
   def unapply(z: Int): Option[Int] = if (z%2==0) Some(z/2) else None
}
```

程序运行结果：

```
10
10 是 5 的两倍！
```



