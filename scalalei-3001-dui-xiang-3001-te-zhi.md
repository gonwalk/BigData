# 1 Scala类和对象

类是对象的抽象，而对象是类的具体实例。类是抽象的，不占用内存，而对象是具体的，占用存储空间。类是用于创建对象的蓝图，它是一个定义包括在特定类型的对象中的方法和变量的软件模板。

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

                    ![](/assets/工程文件结构1.png)                         ![](/assets/工程文件结构2.png)

                              图1   类定义放在object外面的工程结构树                                                图2   类定义放在object里面的工程结构树

类定义时，也可以将其放在object内部，这样产生的工程目录结构树如图2所示。从上面的对比中，可以发现将类定义放在object定义之外的会多出一个目录ClassTest，其下面才是定义的Object文件名，表示该源码文件中既有单独的类Class的定义，又有单独的Object的定义。而将类定义放在object里面时，只显式地在包下面包含一个Object文件。

而在使用IDEA创建Scala程序时，有三种选择Class（存放单独的类文件的定义，类中可以定义各种方法）、Object（既可以存放类文件，还可以存放方法，最重要的是main函数的存放所在）、Trait（相当于 Java 的接口，实际上它比接口功能还强大。与接口不同的是，它还可以定义属性和方法的实现。）。一般在使用IDEA编写Scala程序时，如果代码中只定义了类则可以单独新建一个Class文件，如果单独定义一个

![](/assets/Scala类、对象、特质.png)

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



