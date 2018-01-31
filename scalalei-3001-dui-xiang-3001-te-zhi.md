# 1 Scala类和对象

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

          图1 



类定义时，也可以将其放在object内部，这样产生的工程目录结构树如图2所示。从上面的对比中，可以发现将类定义放在object定义之外的会多出一个目录ClassTest，其下面才是定义的Object文件名，表示该源码文件中既有单独的类Class的定义，又有单独的Object的定义。而将类定义放在object里面时，只显式地在包下面包含一个Object文件。

而在使用IDEA创建Scala程序时，有三种选择Class（存放单独的类文件的定义，类中可以定义各种方法）、Object（既可以存放类文件，还可以存放方法，最重要的是main函数的存放所在）、Trait（相当于 Java 的接口，实际上它比接口功能还强大。与接口不同的是，它还可以定义属性和方法的实现。）。一般在使用IDEA编写Scala程序时，如果代码中只定义了类则可以单独新建一个Class文件，如果单独定义一个接口或尚未实现的方法，则可以新建一个Trait文件，如果需要实例化类，通过main方法调用类中的方法运行检测代码的正确性，则可以创建一个新的Object文件。

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

**Scala 的类定义可以有参数，称为类参数，如上面的 xc, yc，类参数在整个类中都可以访问。**

接着我们可以使用 new 来实例化类，并访问类中的方法和变量：

