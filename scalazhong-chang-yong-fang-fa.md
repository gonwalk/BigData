# 

在使用map映射/哈希表的时候出现如下错误：

    scala> var scores = new scala.collection.mutable.Map[String, Double]()
    <console>:17: error: trait Map is abstract; cannot be instantiated            
           var scores = new scala.collection.mutable.Map[String, Double]()
                        ^

    scala> var scores = scala.collection.mutable.Map[String, Double]()
    scores: scala.collection.mutable.Map[String,Double] = Map()

    scala> map += ("Alice" -> 89.0, "Tom" -> 73.2, "Jan" -> 93.1)
    <console>:18: error: missing argument list for method map in object functions
    Unapplied methods are only converted to functions when a function type is expected.
    You can make this conversion explicit by writing `map _` or `map(_)` instead of `map`.
           map += ("Alice" -> 89.0, "Tom" -> 73.2, "Jan" -> 93.1)
           ^

    scala> map += ("Alice" -> 87.2)
    <console>:18: error: missing argument list for method map in object functions
    Unapplied methods are only converted to functions when a function type is expected.
    You can make this conversion explicit by writing `map _` or `map(_)` instead of `map`.
           map += ("Alice" -> 87.2)
           ^

    scala> var a:Map[String, Double] = Map()
    a: Map[String,Double] = Map()

    scala> a += ("Alice" -> 86.6)

    scala> a += ("Alice" -> 89.0, "Tom" -> 73.2, "Jan" -> 93.1)

    scala> a
    res4: Map[String,Double] = Map(Alice -> 89.0, Tom -> 73.2, Jan -> 93.1)


原因分析：

①var scores = new scala.collection.mutable.Map\[String, Double\]\(\)   //mutable.Map\[\]是一个抽象接口（）特质，不能通过new直接实例化。

②**在Scala中方法不是值，而函数是。所以一个方法不能赋值给一个val变量，而函数可以。上面将Map**方法赋值给变量map失败。根据提示，可以通过将方法转化为函数的方式实现。

Scala中Method方法和Function函数的区别 - 简书

[https://www.jianshu.com/p/d5ce4c683703](https://www.jianshu.com/p/d5ce4c683703)





[scala通过mkString方法把一个集合转化为一个字符串](http://blog.csdn.net/qq_36330643/article/details/76489573)

[http://blog.csdn.net/qq\_36330643/article/details/76489573](http://blog.csdn.net/qq_36330643/article/details/76489573)

#### Problem {#h4_0}

```
如果你想要把集合元素转化为字符串，可能还会添加分隔符，前缀，后缀。
```

#### Solution {#h4_1}

```
**使用mkString方法，以一定的分隔符重组集合中的内容**，下面给一个简单的例子：
```

```
scala> val a = Array("apple", "banana", "cherry")
a: Array[String] = Array(apple, banana, cherry)

scala> a.mkString
res1: String = applebananacherry

// 使用mkString方法你会看到结果并不漂亮，我们来加一个分隔符：

scala> a.mkString(",")
res2: String = apple,banana,cherry

scala> a.mkString(" ")
res3: String = apple banana cherry

//同样可以添加一个前缀和一个后缀：
scala> a.mkString("[", ", ", "]")
res4: String = [apple, banana, cherry]
```

**如果想把一个潜逃集合转化为一个字符串，比如嵌套数组，首先可以使用flatten方法，将多个嵌套数组（Array）展开为一个Array，然后调用mkString方法，以一定的分隔符组成字符串：**

```
scala> val a = Array(Array("a", "b"), Array("c", "d"))
a: Array[Array[String]] = Array(Array(a, b), Array(c, d))

scala> a.flatten
res5: Array[String] = Array(a, b, c, d)

scala> a.flatten.mkString(",")
res6: String = a,b,c,d
```

#### Discussion {#h4_2}

**    你可以调用集合的toString方法，但是它返回带有集合元素信息的集合名称**：

```
scala> val v = Vector("apple", "banana", "cherry")
v: scala.collection.immutable.Vector[String] = Vector(apple, banana, cherry)

scala> v.toString
res7: String = Vector(apple, banana, cherry)

scala> v.mkString
res8: String = applebananacherry

scala> v.mkString(", ")
res9: String = apple, banana, cherry

scala> v.mkString("(", ", ", ")")
res10: String = (apple, banana, cherry)
```

# scala 解析json字符串

**scala中自带了一个scala.util.parsing.json.JSON ，然后可以通过JSON.parseFull\(jsonString:String\)来解析一个json字符串**，如果解析成功的话则返回一个Some\(map: Map\[String, Any\]\)，如果解析失败的话返回None。

所以我们可以通过模式匹配来处理解析结果：

```
    val str2 = "{\"et\":\"kanqiu_client_join\",\"vtm\":1435898329434,\"body\":{\"client\":\"866963024862254\",\"client_type\":\"android\",\"room\":\"NBA_HOME\",\"gid\":\"\",\"type\":\"\",\"roomid\":\"\"},\"time\":1435898329}"

    val b = JSON.parseFull(str2)
    b match {
      // Matches if jsonStr is valid JSON and represents a Map of Strings to Any
      case Some(map: Map[String, Any]) => println(map)
      case None => println("Parsing failed")
      case other => println("Unknown data structure: " + other)
    }
```

spark-shell运行过程：

```
scala> import scala.util.parsing.json.JSON
import scala.util.parsing.json.JSON


scala> val str2 = "{\"et\":\"kanqiu_client_join\",\"vtm\":1435898329434,\"body\":{\"client\":\"866963024862254\",\"client_type\":\"android\",\"room\":\"NBA_HOME\",\"gid\":\"\",\"type\":\"\",\"roomid\":\"\"},\"time\":1435898329}"  
str2: String = {"et":"kanqiu_client_join","vtm":1435898329434,"body":{"client":"866963024862254","client_type":"android","room":"NBA_HOME","gid":"","type":"","roomid":""},"time":1435898329}

scala> val b = JSON.parseFull(str2)
b: Option[Any] = Some(Map(et -> kanqiu_client_join, vtm -> 1.435898329434E12, body -> Map(gid -> , roomid -> , client -> 866963024862254, client_type -> android, room -> NBA_HOME, type -> ), time -> 1.435898329E9))

scala> b match {
     | case Some(map: Map[String, Any]) => println(map)
     | case None => println("Parsing failed")
     | case other => println("Unknown data structure: " + other)
     | }
<console>:28: warning: non-variable type argument String in type pattern scala.collection.immutable.Map[String,Any] (the underlying of Map[String,Any]) is unchecked since it is eliminated by erasure
       case Some(map: Map[String, Any]) => println(map)
                      ^
Map(et -> kanqiu_client_join, vtm -> 1.435898329434E12, body -> Map(gid -> , roomid -> , client -> 866963024862254, client_type -> android, room -> NBA_HOME, type -> ), time -> 1.435898329E9)
```

# Scala HashMap排序

[https://segmentfault.com/q/1010000004862906](https://segmentfault.com/q/1010000004862906)

WorkerInfo是一个类，包括很多属性，我现在需要根据里面的一个cpuUsage这个字段排序，cpuUsage是double类型，对idToWorker 进行排序  
`val idToWorker = new HashMap[String, WorkerInfo]`

本人找到一个类似的，但是不是很清楚sortby以及里面的case用法，所以不知道怎么与我的WorkerInfo中的double那个字段联系，因为对scala语法不是很熟悉~

```
case class WorkerInfo(id: String, cpuUsage: Double)

    object SortObj extends App {

  var sortHash = new HashMap[String, WorkerInfo]

  sortHash+= ("1" -> WorkerInfo("a", 0.4), 
      "2" -> WorkerInfo("b", 0.2), 
      "3" -> WorkerInfo("c", 0.3))

  sortHash.toList.sortBy(_._2.cpuUsage) foreach {
    case (key, value) => println(key + "==" + value)
  }
}
```

spark shell运行过程

```
scala> import scala.collection.mutable.HashMap
import scala.collection.mutable.HashMap

scala> case class WorkerInfo(id:String, cpuUsage: Double)
defined class WorkerInfo

scala> var sortHash = new HashMap[String, WorkerInfo]
sortHash: scala.collection.mutable.HashMap[String,WorkerInfo] = Map()

scala> sortHash += ("1" -> WorkerInfo("a", 0.4),
     | "2" -> WorkerInfo("b", 0.2),
     | "3" -> WorkerInfo("c", 0.3))
res17: scala.collection.mutable.HashMap[String,WorkerInfo] = Map(2 -> WorkerInfo(b,0.2), 1 -> WorkerInfo(a,0.4), 3 -> WorkerInfo(c,0.3))

scala> sortHash.toList.sortBy(_._2.cpuUsage) foreach {
     | case (key, value) => println(key +  "==" + value)
     | }
2==WorkerInfo(b,0.2)
3==WorkerInfo(c,0.3)
1==WorkerInfo(a,0.4)
```



