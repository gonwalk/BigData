第

8章 继承

本章要点概览：

* extends、final关键字和Java中相同。
* 重写方法时必须用override。
* 只有主构造器可以调用超类的主构造器。
* 可以重写字段。
* 特质是将Java接口变得更为通用的Scala概念。

## 8.1 扩展类

Scala扩展类的方式和Java一样——使用extends关键字：

```
class Employee extends Person{
    var salary = 0.0
    ...
}
```

和Java一样，在定义中给出子类需要而超类没有的字段和方法，或者重写超类的方法。和Java一样，可以将类声明为final，这样它就不能被扩展。可以将单个方法或字段声明为final，以确保它们不能被重写。注意这和Java不同，在Java中，final字段是不可变的，类似Scala中的val。

## 8.2 重写方法

在Scala中，重写一个非抽象方法必须使用override修饰符。例如：

```
public class Person{
    ...
    override def toString = getClass.getName + "[name=" + name + "]"
}
```

override修饰符可以在多个常见情况下给出有用的错误提示，包括：

* 拼错要重写的方法名时。
* 在新方法中使用了错误的参数类型。
* 在超类中引入了新的方法，而这个新的方法与子类的方法相抵触。

在Scala中，调用超类的方式和Java完全一样，使用super关键字：

```
public class Employee extends Person{
    ...
    override def toString = super.toString + "[salary=" + salary + "]"
}
```

在这里，super.toString会调用超类的toString方法——即Person.toString。

## 8.3 类型检查和转换

要检查或测试某个对象是否属于某个给定的类，可以使用isInstanceOf方法。如果测试成功，就可以用asInstanceOf方法将引用转换为子类的引用：

```
if(p.isInstanceOf[Employee]){
    val s = p.asInstanceOf[Employee] //s的类型为Employee
    ...
}
```

如果p指向的是Employee类及其子类（比如Manager）的对象，则p.isInstanceOf\[Employee\]将会成功。如果p是null，则p.isInstanceOf\[Employee\]将返回false，且p.asInstanceOf\[Employee\]将返回null。如果不是一个Employee，则p.asInstanceOf\[Employee\]将抛出异常。如果想要测试p指向的是一个Employeee对象但又不是其子类的话，可以用：

if\(p.getClass == classOf\[Employee\]\)

其中classOf方法定义在scala.Predef对象中，因此会被自动引入。

不过，与类型检查和转换相比，模式匹配通常是更好的选择。例如：

```
p match{
    case s: Employee => ... //将s作为Employee处理
    case _ =>               //p不是Employee
}
```

## 8.4 受保护字段和方法

和Java或C++一样，可以将字段或方法声明为protected。这样的成员可以被任何子类访问，但不能从其他位置看到。

与Java不同，protected的成员对于类所属的包而言，是不可见的。（如果需要这样一种可见性，则可以用包修饰符。）

Scala还提供了一个protected\[this\]的变体，将访问权限定在当前的对象，类似于第5章中的private\[this\]。

## 8.5 超类的构造

第5章介绍过，类有一个主构造器和任意数量的辅助构造器，而每个辅助构造器都必须以对先前定义的辅助构造器或主构造器的调用开始。这样带来的后果是，辅助构造器永远都不可能直接调用超类的构造器。

子类的辅助构造器最终都会调用主构造器，只有主构造器可以调用超类的构造器。主构造器是和类定义交织在一起的，调用超类构造器的方式也同样交织在一起。如：

```
class Employee(name:String, age:Int, val salary: Double) extends Person(name, age)
```

这段代码定义了一个子类：class Employee\(name: String, age: Int, val salary: Double\) extends Person\(name, age\)

和一个调用超类构造器的主构造器：class Employee \(name: String, age: Int, val salary: Double\) extends Person\(name, age）

将类和构造器交织在一起可以带来更精简的代码。把主构造器的参数当做是类的参数可能更容易理解。本例中的Employee类有三个参数：name、age和salary，其中的两个（name和age）被“传递”到了超类（即父类）。与上述定义等效的Java代码如下：

```
public class Employee extends Person{ //Java代码
    private double salary;
    public Employee(String name, int age, double salary){
        super(name, age);
        this.salary = salary;
    }
}
```

注意：在Scala的构造器中，不能调用super\(params\)。不像Java，可以用这种方式来调用超类构造器。

Scala类可以扩展Java类。这种情况下，它的主构造器必须调用Java超类的某一个构造方法。例如：

```
class Square(x: Int, y: Int, width: Int) extends java.awt.Rectangle(x, y, width, width)
```

## 8.6 重写字段

Scala的字段由一个私有字段和取值器/改值器方法构成。可以用另一个同名的val字段重写一个val（或不带参数的def）。子类由一个私有字段和一个共有的getter方法，而这个getter方法重写了超类的getter方法。例如：

```
class Person(val name:String){
    override def toString = getClass.getName + "[name=" + name + "]"
}


class SecretAgent(codename: String) extends Person(codename){
    override val name = "secret" //不像暴露真名... ，用另一个同名的val name字段重写一个val
    override val toString = //...或类名， 重写toString方法
}
```

上面的实例展示了工作机制，但是比较做作。更常见的案例是用val重写抽象的def，如：

```
abstract class Person{ //8.8节会介绍到抽象类
    def id: Int //每个人都有一个以某种方式计算出来的ID
    ...
}

class Student(override val id: Int) extends Person
//学生ID通过构造器输入
```

注意如下限制（同时参照表8-2）：

* def只能重写另一个def
* val只能重写另一个val或不带参数的def
* var只能重写另一个抽象的var（参见8.8节“抽象类”）

## 8.7 匿名子类

和 Java一样，可以通过包含带有定义或重写的代码块的方式创建一个匿名的子类，比如：

```
val alien = new Person("Fred"){
    def greeting = "Greetings, Earthling! My name is Fred."
}
```

从技术上讲，这将会创建出一个结果类型的对象。该类型标记为Person{def greeting:String}。可以用这个类型作为参数类型的定义：

```
def meet(p: Person{ def greeting: String }) {
    println(p.name + "says: " + p.greeting)
}
```

## 8.8 抽象类

和Java一样，可以用abstract关键字标记不能被实例化的类，通常这是因为它的某个或某几个方法没有被完整定义。例如：

```
abstract class Person{
    def id:Int //没有方法体，这里是一个抽象方法
}
```

在Scala中，不像Java，不需要对抽象方法使用abstract关键字，只是省去其方法体。但和Java一样，如果某个类至少存在一个抽象方法，则该类必须声明为abstract。

在子类中重写超类的抽象方法时，不需要使用override关键字。

```
class Employee(name: String) extends Person(name){
    def id = name.hashCode //不需要override关键字
}
```

## 8.9 抽象字段

除了抽象方法外，类还可以拥有抽象字段。抽象字段就是一个没有初始值的字段。例如：

```
abstract class Person{
    val id:Int //没有初始化——这是一个带有抽象的getter方法的抽象字段（val类型的）
    var name:String //另一个抽象字段(var类型)，带有抽象的getter和setter方法
}
```

该类为id和name字段定义了抽象的getter方法，为name字段定义了抽象的setter方法。生成的Java类并不带字段。具体的子类必须提供具体的字段，例如：

```
class Employee(val id: Int) extends Person{ //子类有具体的id属性
    var name = " " //和具体的name属性
}
```

和方法一样，在子类中重写超类中的抽象字段时，不需要override关键字。可以随时用匿名类型定制抽象字段：

```
val fred = new Person{
    val id = 1729
    var name = "Fred"
}
```

## 8.10 构造顺序和提前定义

![](/assets/构造顺序.png)

![](/assets/提前定义.png)  


说明：构造顺序问题的根本原因来自Java语言的一个设计决定——即允许在超类的构造方法中调用子类的方法。在C++中，对象的虚函数表的指针在超类构造方法执行的时候被设置成指向超类的虚函数表。之后，才指向子类的虚函数表。因此，在C++中，没有办法通过重写修改构造方法的行为。Java设计者们觉得这个细微差别是多余的，Java虚拟机因此在构造过程中并不调整虚拟函数表。

## 8.11 Scala继承层级

  


  

