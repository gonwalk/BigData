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



