# 第8章 继承

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

这段代码定义了一个子类：class Employee\(name: String, age: Int, val salary: Double\) extends Person\(name, age\)

和一个调用超类构造器的主构造器：class Employee \(name: String, age: Int, val salary: Double\) extends Person\(name, age）

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

![](/assets/Scala继承层级.png)

说明：和Java一样，建议远离wait、notify和synchronized——除非你有充分的理由使用这些关键字而不是更高层次的并发结构。

所有的Scala类都实现ScalaObject这个接口，这个接口没有定义任何方法。

在继承层级的另一端是Nothing和Null类型。

Null类型的唯一实例是null值。可以将null赋值给任何引用，但不能赋值给值类型的变量。如，不能将Int设为null。这比Java更好，在Java中可以将Integer包装类引用设为null。Nothing类型没有实例。它对于泛型结构时常有用。举例，空列表Nil的类型是List\[Nothing\]，它是List\[T\]的子类型，T可以是任何类。

**注意：Nothing类型和Java或C++中的void完全是两个概念。在Scala中，void由Unit类型表示，该类型只有一个值，那就是**

**\( \)。注意Unit并不是任何其他类型的超类型。但是，**_**编译器依然允许任何值被替换成\( \)。**_

## 8.12 对象相等性

在Scala中，AnyRef的eq方法检查两个引用是否指向同一个对象。AnyRef的equals方法调用eq。当实现类的时候，应该考虑重写equals方法，以提供一个自然、与实际情况相称的相等性判断。

示例如，当定义class Item\(val description: String, val price: Double\)，可能会认为当两个物件有着相同描述和价格的时候它们是相等的。以下是相应的equals方法定义：

```
final override def equals(other:Any) = {
    val that = other.asInstanceOf[Item]
    if(that == null) false
    else description == that.description && price == that.price
}
```

**说明：将方法定义为final，是因为通常而言在子类中正确地扩展相等性判断非常困难。问题出在对称性上。想让a.equals\(b\)和b.equals\(a\)的结果相同，尽管b属于a的子类。**

_**注意：请确保定义的equals方法参数类型为Any。**_以下代码是错误的：

final def equals\(other: Item\) = { ... }

这是一个不相关的方法，并不会重写AnyRef的equals方法。

当定义equals是，记得同时也定义hashCode。在计算哈希码时，只应使用那些用来做相等性判断的字段。拿Item这个示例来说，可以将两个字段的哈希码结合起来：

```
final override def hashCode = 13 * description.hashCode + 17 * price.hashCode
```

提示：并不需要觉得重写equals哈hashCode是义务。对很多类而言，将不同的对象看做不相等是很正常的。举例来说，如果有两个不同的输入流或者单选按钮，则完全不需要考虑它们是否相等的问题。

_**在应用程序中，通常对于数值型的比较使用==进行；对于引用类型而言，他会在做完必要的null检查之后调用equals方法。**_

## 第9章 文件和正则表达式

本章要点提要：

* Source.fromFile\(...\).getLines.toArray输出文件的所有行。
* Source.fromFile\(...\)。mkString以字符串形式输出文件内容。
* 将字符串转换为数字，可以用toInt或toDouble方法。
* 使用Java的PrintWriter来写入文本文件。
* “正则”.r是一个Regex对象。
* 如果正则表达式包含反斜杠\或者引号' 或“的话，用"""..."""。
* 如果正则模块包含分组，可以用如下语法来提取它们的内容for\( regex\(变量1, ... , 变量n\) &lt;- 字符串\)。

## 9.1 读取行

要读取文件中的所有行，可以调用scala.io.Source对象的getLines方法：

```
import scala.io.Source

val source = Source.fromFile("myfile.txt", "UTF-8)
    //第一个参数可以是字符串或者是java.io.File，如果知道文件使用的是当前平台缺省的字符编码，则可以略去第二个字符编码参数
val lineIterator = source.getLines
```

结果是一个迭代器，可以用它来逐条处理这些行：

```
for(l <- lineIterator) 处理l
```

或者也可以将对迭代器应用toArray或toBuffer方法，将这些行放在数组或数组缓冲当中：

```
val lines = source.getLines.toArray
//有时候，只想把整个文件读取到一个字符串中，可以使用如下语句：
val contents = source.mkString
```

**注意：在用完Source对象后，记得调用close去关闭。**

## 9.2 读取字符

要从文件中读取单个字符，可以直接把Source对象当做迭代器，因为Source类扩展自Iterator\[Char\]:

```
import scala.io.Source

val source = Source.fromFile("myfile.txt", "UTF-8)
for(c <- source) 处理c
```

**要想查看某个字符而不处理掉它的话（类似C++中的istream::peek或Java中的PushbackInputStreamReader），调用Source对象的buffered方法。这样就可以用head方法查看下一个字符，但同时并不把它当做是已处理的字符**：

```
import scala.io.Source

val source = Source.fromFile("myfile.txt", "UTF-8)
val iter = source.buffered
while(iter.hasNext){
    if(iter.head 是符合预期的)
        处理 iter.next
    else
        ...
}
source.close()
```

如果文件不是很大，也可以把它读取成一个字符串进行处理：

val contents = source.mkString

## 9.2 读取词法单元和数字

一个快而脏的读取源文件中所有以空格隔开的词法单元：

```
val tokens = source.mkString.split("\\S+")
```

将字符转换为数字，可以使用toInt或toDouble方法。如，将一个包含浮点数的文件读取到数组中：

```
val numbers = for(w <- tokens ) yield w.toDouble
//或者val numbers = tokens.map(_.toDouble)
```

**提示：总是可以使用java.util.Scanner类处理同时包含文本和数字的文件。**

从控制台读取数字：

```
print("How old are you?")
//缺省情况下，系统会自动引入Console，因此并不需要对print和readInt使用限定词
val age = readInt()        //或使用readDouble或readLong
```

**注意：这些方法假定下一行输入包含单个数字，且前后都没有空格。否则会报错NumberFormatException。**

## 9.4 从URL或其他源读取

Source对象有读取非文件源的方法：

```
val source1 = Source.fromURL("http://horstmann.com", "UTF-8")
val source2 = Source.fromSting("Hello, World!")                    //从给定的字符串读取——对调试很有用
val source3 = Source.stdin                                         //从标准输入读取
```

注意：当从URL读取时，需要事先知道字符集，可能是通过HTTP头获取。

## 9.5 读取二进制文件

Scala并没有提供读取二进制文件的方法，需要使用Java类库。如，将文件读取成字节数组：

```
val file = new File(filename)
val in = new FileInputStream(file)
val bytes = new Array[Byte](file.length.toInt)
in.read(bytes)
in.close
```

## 9.6 写入文本文件

Scala没有內建的对写入文件的支持。要写入文本文件，可使用java.io.PrintWriter，例如：

```
val out = new PrintWriter("numbers.txt")
for(i <- 1 to 100) out.println(i)
out.close()
```

注意，当传递数字给printf时，编译器要求将它转换成AnyRef，如下形式：

```
out.printf("%6d %10.2f", quantity.asInstanceOf[AnyRef], price.asInstanceOf[AnyRef])
```

**为了避免这个麻烦，可以用String类的format方法：**

```
out.print("%6d %10.2f".format(quantity, price))
```

说明：Console类的printf没有这个问题，可以使用printf\("%6d %10.2f", quantity, price\)来输出消息到控制台。

## 9.7 访问目录

目前，Scala并没有“正式的”用来访问某个目录总的所有文件的方法，或者递归地遍历所有目录的类。但可以使用其他的替代方案，如编写遍历某目录下所有子目录的函数：

```
import java.io.File

def subdirs(dir: File): Iterator[File] = {
    val children = dir.listFiles.filter(_.isDirectory)
    children.toIterator ++ children.toIterator.flatMap(subdirs _)
}

//利用这个函数，可以访问所有的子目录
for(d <- subdirs(dir)) 处理d
```

在Java7中，也可以使用java.nio.file.Files类中的walkFileTree方法，该类用到了FileVisitor接口。在Scala中，通常喜欢用函数对象指定工作内容，而不是接口。以下隐式转换让函数可以与接口相匹配：

```
import java.nio.file._
implicit def makeFileVisitor(f: (Path) => Unit) = new SimpleFileVisitor[Path]{
    override def visitFile(p: Path, attrs: attribute.BasicFileAttributes) = {
        f(p)
        FileVisitResult.CONTINUE
    }
}

//通过如下调用来打印所有的子目录
Files.walkFileTree(dir.toPath, (f: Path) => println(f))
```

## 9.8 序列化

_**Java中，用序列化将对象传输到其他虚拟机，或临时存储。**_（对于长期存储而言，序列化可能会比较笨拙——随着类的演进更新，处理不同版本之间的对象是很麻烦的一件事。）在Java和Scala中声明一个可被序列化的类：

```
//Java代码
public class Person implements java.io.Serializable{
    private static final long serialVersionUID = 42L;
    ...
}

//Scala代码
@SerialVersionUID(42L) class Person extends Serializable
```

Serializable特质定义在scala包，因此不需要显示引入。

说明：如果能接受缺省的ID，也可略去@SerialVersionUID注释。

可以按照常规的方式对对象进行序列化和反序列化：

```
val fred = new Person( ... )
import java.io_
val out = new ObjectOutputStream(new FileOutputStream("/tmp/test.obj"))
out.writeObject(fred)
out.close()
val in = new ObjectInputStream(new FileInputStream("/tmp/test.obj"))
val savedFred = in.readObject().asInstanceOf[Person]
```

Scala集合类都是可序列化的，因此可以把它们用做可序列化类的成员：

```
class Person extends Serializable {
    private val friends = new ArrayBuffer[Person]   //ArrayBuffer是可序列化的
    ...
}
```

## 9.9 进程控制

第5章 类

  


本章的要点包括：

* 类中的字段自动带有getter方法和setter方法。
* 可以用定制的getter/setter方法替换掉字段的定义，而不必修改使用类的客户端——这就是所谓的“统一访问原则”。
* 用@BeanProperty注解来生成JavaBeans的getXxx/setXxx方法。
* 每个类都有一个主要的构造器，这个构造器和类定义“交织”在一起。它的参数直接成为类的字段。主构造器执行类体中所有的语句。
* 辅助构造器是可选的，他们叫做this。



