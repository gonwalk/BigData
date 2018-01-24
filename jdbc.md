# 浅谈 JDBC 中 CreateStatement 和 PrepareStatement 的区别与优劣。

2015年09月12日 16:02:16             [http://blog.csdn.net/u011161786/article/details/48394751](http://blog.csdn.net/u011161786/article/details/48394751)

本人的几点浅见，各位大大不喜勿喷。

先说下这俩到底是干啥的吧。其实这俩干的活儿都一样，就是创建了一个对象然后去通过对象调用executeQuery方法来执行sql语句。

说是CreateStatement和PrepareStatement的区别，但其实说的就是Statement和PrepareStatement的区别，相信大家在网上已经看到过  
不少这方面的资料和博客，我在此处提几点，大家看到过的，就当重记忆，没看到就当补充~下面开始谈谈他们的区别。

### 首先**：PrepareStatement跟Statement的主要区别就是把sql语句中的变量抽出来了**

**最明显的区别，就是执行的sql语句格式不同。**我们往上放两段代码来看看他们的区别吧：

代码背景：我们有一个数据库，里面有一个user表，有username,userpwd两列。我们要查出这两列的数据。

这是使用CreateStatement方法创建了stmt对象，再通过他查询的一部分语句片段。

```
String sql = "select * from users where  username= '"+username+"' and userpwd='"+userpwd+"'";
stmt = conn.createStatement();
rs = stmt.executeQuery(sql);
```

而下面则是使用了PrepareStatement方法创建了pstmt对象，再通过这个对象查询的一部分语句片段。

```
String sql = "select * from users where  username=? and userpwd=?";
pstmt = conn.prepareStatement(sql);
pstmt.setString(1, username);
pstmt.setString(2, userpwd);
rs = pstmt.executeQuery();
```

相信写到这，大家很多人就能看出来了，原来**PrepareStatement跟Statement的主要区别就是把上面sql语句中的变量抽出来了。这就是我要说的第一大优点，PrepareStatement可以提高代码的可读性。**什么？你没觉得这有什么可以提高可读性的？那好，咱来看看下面这两段代码，看完你再说话。

代码背景：我们有一个数据库，里面有一个book表，有bookid,bookname,bookauthor,booksort,bookprice五列。我们要向这个表中添加一部分数据。

Statement版

```
String sql = "insert into book (bookid,bookname,bookauthor,booksort,bookprice) values ('"+var1+"',
                                '"+var2+"',"+var3+",'"+var4+","+var5+"')";
stmt = conn.createStatement();
rs = stmt.executeUpdate(sql);
```

PrepareStatement版

```
String sql = "insert into book (bookid,bookname,bookauthor,booksort,bookprice) values (?,?,?,?,?)";
pstmt = conn.prepareStatement(sql);
pstmt.setString(1,var1);
pstmt.setString(2,var2);
pstmt.setString(3,var3);
pstmt.setString(4,var4);
pstmt.setString(5,var5);
pstmt.executeUpdate();
```

怎么样。反正我打这行代码的时候，整个引号逗号就给我刺激懵了。

#### 下面说说第二点优点。ParperStatement提高了代码的灵活性和执行效率。

**PrepareStatement接口是Statement接口的子接口，他继承了Statement接口的所有功能。它主要是拿来解决我们使用Statement对象多次执行同一个SQL语句的效率问题的。ParperStatement接口的机制是在数据库支持预编译的情况下预先将SQL语句编译，当多次执行这条SQL语句时，可以直接执行编译好的SQL语句，这样就大大提高了程序的灵活性和执行效率。**

#### 最后但也是最重要的一个大大的比Statement好的优点，那就是安全！

你说啥？这还关安全啥事儿，那我给你一行代码，你来给我说说这是干嘛的。

```
String sql = "select * from user where username= '"+varname+"' and userpwd='"+varpasswd+"'";
stmt = conn.createStatement();
rs = stmt.executeUpdate(sql);
```

这是验证用户名密码的，对吧。但要是我们把'or '1' = 1'当作密码传进去，你猜猜会发生啥。

```
select * from user where username = 'user' and userpwd = '' or '1' = '1';
```

发现了吧！这是个永真式，因为1永远等于1。所以不管怎样都能获取到权限。哇。这就坏咯！这还不是最坏的，你再看！

```
String sql = "select * from user where username= '"+varname+"' and userpwd='"+varpasswd+"'";  
stmt = conn.createStatement();  
rs = stmt.executeUpdate(sql);
```

依旧是这行代码。这次我们把'or '1' = 1';drop table book;当成密码传进去。哇！又坏了！这次直接把表给删了。但是，你**如果用PrepareStatement的话就不会出现这种问题。你传入的这些数据根本不会跟原来的数据有任何的交集，也不会发生这些问题。**



# statement 、prepareStatement的用法和解释 - CSDN博客

[http://blog.csdn.net/qh\_java/article/details/48245945](http://blog.csdn.net/qh_java/article/details/48245945)



# java攻城狮之路--复习JDBC\(PrepareStatement\) - 骄阳08 - 博客园

[https://www.cnblogs.com/shellway/p/3933403.html](https://www.cnblogs.com/shellway/p/3933403.html)

