
---
title: mybatis源码分析
date: 2020-09-14 14:49:46
tags: mybatis框架
excerpt: 转载2、mybatis源码分析
---

## [4_mybatis_plus简单运用.xmind](https://uploader.shimo.im/f/ecvqhafTbFc9AjmX.xmind)[0_mybatis简介.xmind](https://uploader.shimo.im/f/DJkOQCDA4TgiRuMB.xmind)[1_mybatis源码分析.xmind](https://uploader.shimo.im/f/r7cGLLC5U6khPlwN.xmind)[2_mybatis处理流程图.xmind](https://uploader.shimo.im/f/WSwiSa1aXXggODvc.xmind)[3_Configuration配置路线.xmind](https://uploader.shimo.im/f/x1f02pzL2PEkBJFM.xmind)mybatis简介

mybatis官网系统学习：


* [http://www.mybatis.org/mybatis-3/zh/index.html](http://www.mybatis.org/mybatis-3/zh/index.html)

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

## jdbc与mybatis

需要先注册驱动和数据库信息、操作Connection、通过statement对象执行SQL，将结果返回给resultSet，然后从resultSet中读取数据并转换为pojo对象，最后需要关闭数据库相关资源。而且还需要自己对JDBC过程的异常进行捕捉和处理

MyBatis对JDBC的封装很好，几乎可以取代Jdbc。

MyBatis使用SqlSessionFactoryBuilder来连接完成JDBC需要代码完成的数据库获取和连接，减少了代码的重复。JDBC将SQL语句写到代码里，属于硬编码，非常不易维护，MyBatis可以将SQL代码写入xml中，易于修改和维护。JDBC的resultSet需要用户自己去读取并生成对应的POJO，MyBatis的mapper会自动将执行后的结果映射到对应的Java对象中。

## mybatis重要组件


* **Configuration**

MyBatis所有的配置信息都维持在Configuration对象之中。


* **SqlSession**

作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能


* **Executor**

MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护


* **StatementHandler**

封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数、将Statement结果集转换成List集合。


* **ParameterHandler**

负责对用户传递的参数转换成JDBC Statement 所需要的参数，


* **ResultSetHandler**

负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；


* **TypeHandler**

负责java数据类型和jdbc数据类型之间的映射和转换


* **MappedStatement**

MappedStatement维护了一条<select|update|delete|insert>节点的封装，


* **SqlSource**

负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回


* **BoundSql**

表示动态生成的SQL语句以及相应的参数信息

## 原理

![图片](https://images-cdn.shimo.im/ie6Sqa3g4L0iRScB/image.png!thumbnail)

![图片](https://uploader.shimo.im/f/VI4rxV5Ub9wToOW3.png!thumbnail)

## ﻿mybatis如何获取数据库中的字段

### 原理

information_schema数据库是MySQL自带的，它提供了访问数据库元数据的方式。什么是元数据呢？元数据是关于数据的数据，如数据库名或表名，列的数据类型，或访问权限等。有些时候用于表述该信息的其他术语包括“数据词典”和“系统目录”。

在MySQL中，把 information_schema 看作是一个数据库，确切说是信息数据库。其中保存着关于MySQL服务器所维护的所有其他数据库的信息。如数据库名，数据库的表，表栏的数据类型与访问权 限等。在INFORMATION_SCHEMA中，有数个只读表。它们实际上是视图，而不是基本表，因此，你将无法看到与之相关的任何文件。

### **information_schema表说明**

SCHEMATA表：提供了当前mysql实例中所有数据库的信息。是show databases的结果取之此表。

TABLES表：提供了关于数据库中的表的信息（包括视图）。详细表述了某个表属于哪个schema，表类型，表引擎，创建时间等信息。是show tables from schemaname的结果取之此表。

COLUMNS表：提供了表中的列信息。详细表述了某张表的所有列以及每个列的信息。是show columns from schemaname.tablename的结果取之此表。

STATISTICS表：提供了关于表索引的信息。是show index from schemaname.tablename的结果取之此表。

USER_PRIVILEGES（用户权限）表：给出了关于全程权限的信息。该信息源自mysql.user授权表。是非标准表。

SCHEMA_PRIVILEGES（方案权限）表：给出了关于方案（数据库）权限的信息。该信息来自mysql.db授权表。是非标准表。

TABLE_PRIVILEGES（表权限）表：给出了关于表权限的信息。该信息源自mysql.tables_priv授权表。是非标准表。

COLUMN_PRIVILEGES（列权限）表：给出了关于列权限的信息。该信息源自mysql.columns_priv授权表。是非标准表。

CHARACTER_SETS（字符集）表：提供了mysql实例可用字符集的信息。是SHOW CHARACTER SET结果集取之此表。

COLLATIONS表：提供了关于各字符集的对照信息。

COLLATION_CHARACTER_SET_APPLICABILITY表：指明了可用于校对的字符集。这些列等效于SHOW COLLATION的前两个显示字段。

TABLE_CONSTRAINTS表：描述了存在约束的表。以及表的约束类型。

KEY_COLUMN_USAGE表：描述了具有约束的键列。

ROUTINES表：提供了关于存储子程序（存储程序和函数）的信息。此时，ROUTINES表不包含自定义函数（UDF）。名为“mysql.proc name”的列指明了对应于INFORMATION_SCHEMA.ROUTINES表的mysql.proc表列。

VIEWS表：给出了关于数据库中的视图的信息。需要有show views权限，否则无法查看视图信息。

TRIGGERS表：提供了关于触发程序的信息。必须有super权限才能查看该表

![图片](https://images-cdn.shimo.im/9KHDIgsUyq4c8T8w/image.png!thumbnail)

查找当前数据库的表信息：

**第一种方法：**

```
select * from information_schema.TABLES where TABLE_SCHEMA=(select database())
```
**第二种方法：**
```
#获取表信息
show table status 
```
查找当前表的所有字段信息：

**第一种方法：**

```
select * from information_schema.COLUMNS where TABLE_SCHEMA = (select database()) and TABLE_NAME=#{tableName}
```
**第二种方法：**
```
show full fields from `student`;
```
### 集成spring的多数据源处理

1、通过扫描包区分

git demo:[https://gitee.com/lv-success/git-second/tree/master/course-3-mybatis/springBootMybatisMulidatasource](https://gitee.com/lv-success/git-second/tree/master/course-3-mybatis/springBootMybatisMulidatasource)

2、通过动态数据源区分

一般使用注解的形式，这里先留着，后面的renren-fast项目

## mybatis的读写分离

原理：

和多数据源处理差不多，mybatis做读写分离也有多种方法。通过拦截发起请求的方法或执行的sql来自动判断需要的数据源！

1、拦截发起操作的方法名

需要自己约定增删改查的前缀，然后根据前缀选择数据源！

git demo：[https://gitee.com/lv-success/git-second/tree/master/course-3-mybatis/bounterMybatis](https://gitee.com/lv-success/git-second/tree/master/course-3-mybatis/bounterMybatis)

2、拦截发起操作的sql

git demo:[https://github.com/shawntime/shawn-rwdb](https://github.com/shawntime/shawn-rwdb)

## mybatis源码分析

![图片](https://uploader.shimo.im/f/PMTBnkVXJ84Hmivq.png!thumbnail)

**课程mybatis源码注释git分支**


* [https://github.com/lv-success/mybatis-3](https://github.com/lv-success/mybatis-3)

源码分析部分因为内容比较多，我就不一一写下来的，大家去看看别人的博文


* 大闲人柴毛毛

[MyBatis源码解析(一)——MyBatis初始化过程解析](https://www.jianshu.com/p/7bc6d3b7fb45)

[MyBatis源码解析(二)——动态代理实现函数调用](https://www.jianshu.com/p/46c6e56d9774)


* zhjh256

[mybatis 3.x源码深度解析与最佳实践](https://www.cnblogs.com/zhjh256/p/8512392.html)


* 南轲梦

[随笔分类 - mybatis](https://www.cnblogs.com/dongying/category/620960.html)

## mybatis plus的简单运用

文章：[https://www.java-mindmap.com/view/21](https://www.java-mindmap.com/view/21)

官网：[https://mp.baomidou.com/guide](https://mp.baomidou.com/guide)

官网实例：[https://gitee.com/baomidou/mybatis-plus-samples](https://gitee.com/baomidou/mybatis-plus-samples)

