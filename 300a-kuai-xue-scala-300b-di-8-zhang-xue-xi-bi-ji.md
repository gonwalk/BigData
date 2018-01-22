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









# 第10章 特质

一个类扩展自一个或多个特质，以便使用这些特质提供的服务。特质可能会要求使用它的类支持某个特定的特性。不过，和Java接口不同，Scala特质可以给出这些特性的缺省实现。因此，与接口相比，特质要有用得多。  
本章要点概述

* 类可以实现任意数量的特质。
* 特质可以要求实现它们的类具备特定的字段、方法或超类。
* **和Java接口不同，Scala特质可以提供方法和字段的实现。**
* 将多个特质叠加在一起时，顺序很重要——其方法先被执行的特质排在更后面。

## 10.1 为什么没有多重继承 {#101-为什么没有多重继承}

Scala和Java一样不允许从多个超类继承。某些编程语言，特别是C++允许多重继承，但代价也是很高的。如果要将毫不相干的多个类组装在一起，但是这些类具备某些共通的方法或子弹，那么麻烦就来了。  
![](http://img.blog.csdn.net/20180120162709568?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG92ZTY2NjY2NnNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")  
![](http://img.blog.csdn.net/20180120162924730?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG92ZTY2NjY2NnNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

## 10.2 当做接口使用的特质 {#102-当做接口使用的特质}

Scala特质完全可以像Java接口那样公共。例如：

```
trait Logger
 {
    def log(msg: String)  //这是一个抽象方法
}

```

**注意：在Scala特质中，不需要（显式地）将方法声明为abstract——特质中未被实现的方法默认就是抽象的。在子类中，可以给出具体的实现。在重写特质的抽象方法时不需要给出override关键字。**

```
class ConsoleLogger extends Logger {	//用extends而不是implements
    def log(msg: String) { //不需要写override
        println(msg)
    }
}
```

说明：Scala并没有一个特殊的关键字用来标记对特质的实现。**比起Java接口，特质跟类更为相像。这样就对上面子类实现特质时，使用extends而不是implements更容易理解了。**  
如果需要多个特质，可以使用with关键字来添加额外的特质：class ConsoleLogger extends Logger**with Cloneable with Serializable**  
正如上面使用到的Java类库中的Cloneable和Serializable接口一样，**所有Java接口都可以作为Scala特质使用**。**和Java一样，Scala类只能有一个超类，但可以用任意数量的特质。**

## 10.3 带有具体实现的特质 {#103-带有具体实现的特质}

**在Scala中，特质中的方法可以是抽象的，也可以带有具体的实现，这与Java接口中的方法只能是抽象的（且不能含有字段的实现:接口里面不能有私有的方法或变量，定义的变量一般都是静态常量，起数据共享的作用）是不同的。**

```
trait ConsoleLogger {
    def log(msg: String){
        println(msg)
    }
}

class SavingsAccount extends Account with ConsoleLogger{
	def withdraw(amount: Double) {
		if(amount > balance) log("Insufficient funds")
		else balance -= amount
	}
	...
}
```

注意：上面SavingAccount从ConsoleLogger特质得到了一个具体的log方法实现，这对于Java接口的话是不可能做到的，这是Scala和Java的一个区别。**让特质拥有具体行为存在一个弊端：当特质改变时，所有混入了该特质的类（即实现该特质的子类）都必须重新编译。**

## 10.4 带有特质的对象 {#104-带有特质的对象}

在构造单个对象时，可以为它添加特质。下面的示例中，用标准Scala库中的Logged特质，其和上面的Logger很像，但它自带的是一个什么都不做的实现：

```
trait Logged {
    def log(msg:String) { }
}
//在类定义中使用上面的特质
class SavingsAccount extends Account with Logged {
	def withdraw(amount: Double) {
		if(amount > balance) log("Insufficient funds")
		else ...
	}
}
```

现在，什么都不会被记录到日志，看上去似乎毫无意义。但是可以在构造具体对象的时候“混入”一个更好的日志记录器的实现。标准的ConsoleLogger扩展自Logged特质：

```
trait ConsoleLogger extends Logged{
	override def log(msg: String) { println(msg) }
}
```

可以在构造对象的时候加入这个特质：  
val acct = new SavingsAccount with ConsoleLogger  
当在acct对象上调用log方法时，ConsoleLogger特质的log方法就会被执行。另一个对象可以加入不同的特质：  
val acct2 = new SavingsAccount with FileLogger

## 10.5 叠加在一起的特质 {#105-叠加在一起的特质}

可以为类或者对象添加多个互相调用的特质，从最后一个开始。这对于需要分阶段加工处理某个值的场景很有用。如，给所有日志消息添加时间戳：

```
trait TimestampLogger extends Logged{
	override def log(msg: String) {
		super.log(new java.util.Date() + " " + msg)
	}
}
```

截断过于冗长的日志消息：

```
trait ShortLogger extends Logged{
	val maxLength = 15  //关于特质中的字段，参见10.8节
	override def log(msg: String) {
		super.log(
			if(msg.length <= maxLength) msg 
			else msg.substring(0, maxLength - 3) + "...") //大于最大长度时，去前面的字段，后面超过长度的内容使用三个点号组成省略号代替
	}
}
```

上面的log方法每一个都将修改过的消息传递给super.log。  
对特质而言，super.log并不像类那样拥有相同的含义。（如果真有相同的含义，那么这些特质就毫无用处——他们扩展自Logged，其log方法什么也不做。）实际上，super.log调用的是特质层级中的下一个特质。具体是哪一个，要根据特质添加的顺序来决定。一般来说，特质从最后一个开始被处理。  
![](http://img.blog.csdn.net/20180120205803985?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG92ZTY2NjY2NnNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

## 10.6 在特质中重写抽象方法 {#106-在特质中重写抽象方法}

在自定义的Logger特质中，并没有提供log方法的实现。现在用时间戳特质来扩展它，这样TimestampLogger类就不能编译了。

```
trait Logger{
	def log(msg: String)   //这个是抽象方法
}
trait TimestampLogger extends Logger{
	override def log(msg:String){   //重写抽象方法
		super.log(new java.util.Date() + " " + msg)    //super.log定义了吗？
	}
}

```

在上面程序中，编译器将super.log调用标记为错误。根据正常的继承规则，这个调用永远都是错的——Logger.log方法没有实现。但实际上，我们没有办法指导哪个log方法最终被调用——这取决于特质被混入的顺序。  
Scala认为TimestampLogger依旧是抽象的——它需要混入一个具体的log方法。因此必须给方法打上abstract关键字以及override关键字，就像这样：

```
abstract override def log(msg: String) {
	super.log(new java.util.Date() + " " + msg)
}
```

## 10.7 当做富接口使用的特质 {#107-当做富接口使用的特质}

特质可以包含大量工具方法，而这些工具方法可以依赖一些抽象方法来实现。例如Scala中的Iterator特质就是利用抽象的next和hasNext定义了几十个方法。如下面给日志API中的每个日志指定一个级别以区分信息类的消息和警告、错误等。

```
trait Logger{
	def log(msg: String)
	def info(msg: String) { log("INFO: " + msg) }
	def warn(msg: String) { log("WARN: " + msg) }
	def severe(msg: String) { log("SEVERE: " + msg) }

//这样，使用Logger特质的类就可以任意调用这些日志消息方法了
class SavingsAccount extends Account with Logger{
	def withdraw(amount: Double) {
		if(amount > balance) severe("Insufficient funds)
		else ...
	}
	...
	override def log(msg: String) { println(msg)
	}
}
```

在Scala中像这样在特质中使用具体和抽象方法十分普遍。在Java中，就需要声明一个接口和一个额外的扩展该接口的类（比如Collection/AbstractCollection或MouseListener/MouseAdapter）。

## 10.8 特质中的具体字段 {#108-特质中的具体字段}

特质中的字段可以是具体的，也可以是抽象的。如果给出了初始值，那么该字段就是具体的。而Java中的接口，字段（变量）都是静态常量，用于传递共享字段。

```
trait ShortLogger extends Logged{
	val maxLength = 15 //具体的字段
	...
}

```

混入该特质的类自动获得一个maxLength字段。通常，对于特质中的每一个具体字段，使用该特质的（子）类都会获得一个字段与之对应。这些字段不是被继承的，它们只是简单地被加到了子类当中。这看上去像是一个很细微的差别，但这个区别很重要。

```
class SavingsAccount extends Account with ConsoleLogger with ShortLogger{
	var interest = 0.0
	def withdraw(amount: Double) {
		if(amount > balance) log("Insufficient funds")
		else ...
	}
}
```

注意，子类有一个字段interest，这个是子类中一个普通的字段。假定Account也有一个字段：

```
class Account {
	var balance = 0.0
}

```

在JVM中，一个类只能扩展一个超类，因此来自特质的字段不能以相同的方式继承。由于这个限制，maxLength被直接加到了SavingsAccont类中，跟interest字段排在一起。如下图所示：  
\| balance \| //超类对象  
\| interest \| //子类字段  
\| maxLength \| // 子类字段

可以把具体的特质字段当做是针对使用该特质的（子）类的“（自动）装配指令”，任何通过这种方式被混入的字段都自动成为该类自己的字段。

## 10.9 特质中的抽象字段 {#109-特质中的抽象字段}

**特质中未被初始化的字段在具体的子类中必须被重写。**如下maxLength字段时抽象的：

```
trait ShortLogger extends Logged {
	val maxLength: Int   //抽象字段
	override def log(msg: String){
		super.log(
		  if(msg.length <= maxLength) msg 
		  else msg.substring(0, maxLength - 3) + "...")
		  //在这个实现中用到了maxLength字段
    }
    ...
}

//当在一个具体的类中使用上面的特质时，必须提供maxLength字段
class SavingsAccount extends with ConsoleLogger with ShortLogger {
	val maxLength = 20  //不需要override,这样所有的日志消息都将在第20个字符处被截断
	...
}
```

这种提供特质参数值的方式在临时要构造某种对象时尤为重要。

## 10.10 特质构造顺序 {#1010-特质构造顺序}

和类一样，特质也可以有构造器，由字段的初始化和其他特质体中的语句构成。

```
trait FileLogger extends Logger {
	val out = new PrintWriter("app.log") //这个是特质构造器的一部分
	out.println("# " + new Date().toString)//同样是特质构造器的一部分
	def log(msg: String) { out.println(msg); out.flush() }
}
```

![](http://img.blog.csdn.net/20180120223021163?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG92ZTY2NjY2NnNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")  
![](http://img.blog.csdn.net/20180120223304367?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG92ZTY2NjY2NnNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

## 10.11 初始化特质中的字段 {#1011-初始化特质中的字段}

**特质不能有构造器参数，每个特质都有一个无参数的构造器。**  
**说明：**_**缺少构造器参数是特质与类之间唯一的技术差别**_**。除此之外，特质可以具备类的所有特性，比如具体的和抽象的字段，以及超类。**

```
val acct = new SavingsAccount with FileLogger("myapp.log") //错误，特质不能使用构造器参数

//一个可行的方案：FileLogger可以有一个抽象的字段用来存放文件名
trait FileLogger extends Logger{
	val filename： String 
	val out = new PrintStream(filename)
	def log(msg: String) { out.println(msg); out.flush() }
}
```

使用该特质的类可以重写filename字段。不过，像这样直截了当的方案并不可行：

```
val acct = new SavingsAccount with FileLogger {
	val filename = "myapp.log   //这样行不通
}
```

问题出在构造顺序上，FileLogger构造器先于子类构造器执行。new语句构造的其实是一个扩展自SavingsAccount\(超类\)并混入了FileLogger特质的匿名类的实例。filename的初始化只发生在这个匿名子类中，实际上，它根本不会发生——在轮到子类之前，FileLogger的构造器就会抛出一个空指针异常。  
**解决办法之一就是在第8章介绍过的一个很隐晦的特性：提前定义。**以下是正确的版本：

```
val acct = new {	//new之后的是提前定义块
	val filename = "myapp.log"
} with SavingsAccount with FileLogger
```

**提前定义发生在常规的构造顺序之前。在FileLogger被构造时，filename已经是初始化过的了。**  
**另一个解决办法是在FileLogger构造器中使用懒值：**

```
trait FileLogger extends Logger {
	val filename:String 
	lazy val out = new PrintStream(filename)
	def log(msg: String) { out.println(msg) } //不需要写override
}

```

这样，out字段将在初次被使用时才会初始化。而在那个时候，filename字段应该已经被设置好值了。**由于懒值在每次使用前都会检查是否已经初始化，它们使用起来并不那么高效。**

## 10.12 扩展类的特质 {#1012-扩展类的特质}

**特质可以扩展另一个特质，而由特质组成的继承层级也很常见。不那么常见的一种用法是，特质也可以扩展类，这个类将会自动成为所有混入该特质的超类。**

```
//LoggedException特质扩展自Exception类：
trait LoggedException extends Exception with Logged{
	def log() { log(getMessage()) }
}
```

LoggedException有一个log方法用来记录异常的消息，log方法调用了Exception超类继承下来的getMessage\(\)方法。创建一个混入该特质的类：

```
class UnhappyException extends LoggedException {		//该类扩展自一个特质
	override def getMessage() = "arggh!"
}
```

**特质的超类自动成为任何混入该特质的类的超类。**  
![](http://img.blog.csdn.net/20180121152220060?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG92ZTY2NjY2NnNoZW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

## 10.13 自身类型 {#1013-自身类型}

**当特质扩展类时，编译器能够确保的一件事是所有混入该特质的类都认这个类作为超类。Scala还有另一套机制可以保证这一点：自身类型（self type）。**  
当特质以如下代码开始定义时： this: 类型 =&gt; 它便只能被混入指定类型的子类。

```
trait LoggedException extends Logged{
	this: Exception =>
	  def log() { log(getMessage()) }
}
```

注意该特质并不扩展Exception类，而是有一个自身类型Exception。这意味着，它只能被混入Exception的子类。  
在特质的方法中，可以调用该自身类型的任何方法。如，log方法中的getMessage\(\)调用就是合法的，因为this必定是一个Exception。  
带有自身类型的特质和带有超类型的特质很相似。两种情况都能够确保混入该特质的类能够使用某个特定类型的特质。在某些情况下，自身类型这种写法比超类型的特质更灵活。自身类型可以解决特质间的循环依赖。如果有两个彼此需要的特质时循环依赖就会产生。  
自身类型也同样可以处理结构类型（structural type）——这种类型只给出类必须拥有的方法，而不是类的名称。以下是使用结构类型的LoggedException定义：

```
trait LoggedException extends Logged{
	this: { def getMessage(): String } => 
	  def log() { log(getMessage()) }
}
```

这个特质可以被混入任何拥有getMessage方法的类。

## 10.14 背后发生了什么 {#1014-背后发生了什么}

**Scala需要将特质翻译成JVM的类和接口。**你不需要知道背后是如何做到的，但了解背后的原理有助于深入地理解特质。  
**只有抽象方法的特质被简单地变成一个Java接口。**

```
trait Logger
{
    def log(msg: String)
}
```

直接被翻译成Java代码为：

```
public interface
 Logger{    //生成的Java接口
    void log(String msg);
}
```

**如果特质有具体的方法，Scala会创建一个伴生类，该伴生类会用静态方法存放特质的方法**：

```
trait ConsoleLogger extends Logger {
    def log(msg:String) { println(msg) }
}

//在Java虚拟机中被翻译成
public interface ConsoleLogger extend Logger{ 
//生成的Java接口
   void log(String msg);
}


public class ConsoleLogger$class
{  
//生成的Java伴生类
    public static void log(ConsoleLogger self, String msg) {
        println(msg);
    }
    ...
}

```

这些伴生类不会有任何字段。特质中的字段对应到接口中的抽象的getter和setter方法。当某个类实现该特质时，字段被自动加入。例如：

```
trait ShortLogger extends Logger {
	val maxLength = 15 //具体的字段
	...
}
//被翻译成
public interface ShortLogger extends Logger{
	public abstract int maxLength();
	public abstract void weird_prefix$maxLength_$eq(int);
	...
}
```

上面以weird开头的setter方法是需要的，用来初始化该字段。初始化发生在伴生类的一个初始化方法内：

```
public class ShortLogger$class
{
    public void $init$(ShortLogger self){
        self.weird_prefix$maxLength_$eq(15)
    }
    ...
}
```

当特质被混入类的时候，类将会得到一个带有getter和setter的maxLength字段。那个类的构造器会调用初始化方法。如果特质扩展自某个超类，则伴生类并不继承这个超类。该超类会被任何实现该特质的类继承。







