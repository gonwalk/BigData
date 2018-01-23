# My Awesome Book

This file file serves as your book's preface, a great place to describe your book's content and ideas.

1. Hive简介

Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供类SQL查询功能。本质是将SQL转换为MapReduce程序，然后利用Hadoop的Map和Reduce将大规模的数据进行分布式处理，其数据规模可以伸缩扩展到100PB+，数据形式可以是结构或非结构数据。

Hive与传统关系数据库比较有如下几个特点：

①侧重于分析，而非实时在线交易；

②无事务机制；

③不像关系数据库那样可以随机进行 insert或update；

④通过Hadoop的map/reduce进行分布式处理，传统数据库则没有；

⑤传统关系数据库只能拓展最多20个服务器，而Hive可以拓展到上百个服务器。

2. Hive架构

Hive架构主要包括shell、JDBC/ODBC、WebUI、Driver、Metastore等，其架构图如下：

![](file:///C:\Users\hasee\AppData\Local\Temp\ksohtml\wpsA546.tmp.jpg)  
Hive的体系结构可以分为以下几个部分：  
①用户接口：包括shell命令（也常称为命令行接口CLI）、Jdbc/Odbc和WebUI，其中最常用的是通过shell这个客户端方式对Hive进行交互式操作。  
②Hive解析器\(驱动器Driver\)：Hive解析器的核心功能就是根据用户编写的Sql语法匹配出相应的MapReduce模板，形成对应的MapReduce job进行执行。  
③Hive元数据库\(MetaStore\)：Hive将表中的元数据信息存储在数据库中，如derby\(自带的\)、Mysql\(实际工作中配置的\)，Hive中的元数据信息包括表的名字、表的列和分区、表的属性\(是否为外部表等\)、表的数据所在的目录等。Hive中的解析器在运行的时候会读取元数据库MetaStore中的相关信息。  
④Hadoop：Hive用HDFS进行存储，用MapReduce进行计算。Hive这个数据仓库的数据存储在HDFS中，业务实际分析计算是利用MapReduce执行的。  
从上面的体系结构中可以看出，在Hadoop的HDFS与MapReduce以及MySql的辅助下，Hive其实就是利用Hive解析器将用户的SQl语句解析成对应的MapReduce程序，即Hive仅仅是一个客户端工具

3. Hive的运行机制  
Hive的运行机制如下图所示：  
![](file:///C:\Users\hasee\AppData\Local\Temp\ksohtml\wpsA557.tmp.jpg)

创建完表之后，用户只需要根据业务需求编写Sql语句，之后Hive框架将Sql语句解析成对应的MapReduce程序，通过MapReduce计算框架运行job，便得到了最终的分析结果。  
在Hive的运行过程中，用户只需要创建表、导入数据、编写Sql分析语句即可，剩下的过程将由Hive框架自动完成，而创建表、导入数据、编写Sql分析语句其实就是数据库的知识，Hive的运行过程也说明了为什么Hive的存在大大降低了Hadoop的学习门槛以及为什么Hive在Hadoop家族中占有着那么重要的地位。



# 环境搭建：

hadoop学习笔记-hive安装及操作 - CSDN博客

[http://blog.csdn.net/lichangzai/article/details/8524504](http://blog.csdn.net/lichangzai/article/details/8524504)

## 实战练习：

1.**spark操作hive**

Spark2.1.0入门：连接Hive读写数据（DataFrame）\_厦大数据库实验室博客

[http://dblab.xmu.edu.cn/blog/1383-2/](http://dblab.xmu.edu.cn/blog/1383-2/)

2.大数据案例-步骤三：Hive、MySQL、HBase数据互导\_厦大数据库实验室博客

[http://dblab.xmu.edu.cn/blog/1059-2/](http://dblab.xmu.edu.cn/blog/1059-2/)

## 问题与解决：

**1.通过命令启动hive报错.org.apache.hadoop.hive.metastore.HiveMetaException: Failed to load driver** ......具体信息如下：

org.apache.hadoop.hive.metastore.HiveMetaException: Failed to load driver

Underlying cause: java.lang.ClassNotFoundException : com.mysql.jdbc.Driver

Use --verbose for detailed stacktrace.

\*\*\* schemaTool failed \*\*\*

可以参考文章：[http://blog.csdn.net/brotherdong90/article/details/49661731](http://blog.csdn.net/brotherdong90/article/details/49661731)

解决方式：

这种情况，猜测是缺少jdbc驱动，下载mysql jdbc 包, [下载地址](http://www.mysql.com/downloads/connector/j/)，之后解压复制jdbc驱动mysql-connector-java-5.1.36-bin.jar到hive安装目录的lib下面（这里为hive安装目录的lib下：/usr/local/hive/lib）。具体的可以参考

Ubuntu安装hive，并配置mysql作为元数据库\_厦大数据库实验室博客[http://dblab.xmu.edu.cn/blog/install-hive/](http://dblab.xmu.edu.cn/blog/install-hive/)

中的第二步：安装并设置mysql。

**2.创建表时出现异常HiveException: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient**

异常信息：

FAILED: SemanticException org.apache.hadoop.hive.ql.metadata.HiveException: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient

参考：

[http://blog.csdn.net/zheng911209/article/details/78718303](http://blog.csdn.net/zheng911209/article/details/78718303)

上面的不用看，只需要执行最后一句命令

schematool -dbType mysql -initSchema

即可，出错原因：重新安装Hive和MySQL，导致版本、配置不一致。在终端执行如下命令: schematool -dbType mysql -initSchema

Hive 分布现在包含一个用于 Hive Metastore 架构操控的脱机工具，名为 schematool.此工具可用于初始化当前 Hive 版本的 Metastore 架构。此外，其还可处理从较旧版本到新版本的架构升级。

3.[Hive之——启动问题及解决方案](http://blog.csdn.net/eason_oracle/article/details/52273954)

[http://blog.csdn.net/eason\_oracle/article/details/52273954](http://blog.csdn.net/eason_oracle/article/details/52273954)

