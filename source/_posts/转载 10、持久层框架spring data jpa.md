---
title: 持久层框架spring data jpa
date: 2020-09-14 14:55:04
tags: spring data 
excerpt: 转载 10、持久层框架spring data jpa
---

## [课程10_spring_data_jpa学习_预习_.xmind](https://uploader.shimo.im/f/rkWgjMrtZV0ng3Tx.xmind)[1_spring_data_jpa介绍.xmind](https://uploader.shimo.im/f/MlJ4jiAcw3UItnTk.xmind)[2_jpa_hibernate_与spring_data_jpa.xmind](https://uploader.shimo.im/f/i8TwxPPrcx8IFeYl.xmind)[3_常用注解与接口.xmind](https://uploader.shimo.im/f/3XteZtegs680QfHU.xmind)[4_jpa底层原理分析.xmind](https://uploader.shimo.im/f/4PUZIqR73dsnGgOR.xmind)[课程10、spring data jpa学习（预习）.xmind](https://uploader.shimo.im/f/7cO6lLnOFCIMxiOO.xmind)什么是jpa？

全称Java Persistence API，通过JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。

### JPA的出现有两个原因：

其一，简化现有Java EE和Java SE应用的对象持久化的开发工作；

其二，Sun希望整合对ORM技术，实现持久化领域的统一。

### JPA提供的技术：

1）ORM映射元数据：JPA支持XML和JDK 5.0注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中；

2）JPA 的API：用来操作实体对象，执行CRUD操作，框架在后台替我们完成所有的事情，开发者从繁琐的JDBC和SQL代码中解脱出来。

3）查询语言：通过面向对象而非面向数据库的查询语言查询数据，避免程序的SQL语句紧密耦合。

### JPA的规范提现在哪？

元数据映射、CRUD操作、JPQL查询语言

```
javax.persistence.Entity
javax.persistence.EntityManager
javax.transaction.Transactional
javax.persistence.criteria.CriteriaQuery
#JPQL
select o from Order o 或  select o from Order as o
```
![图片](https://images-cdn.shimo.im/swtCLsfo0DAvDfP4/image.png!thumbnail)

### JPQL查询能力

JPA的查询语言是面向对象而非面向数据库的，它以面向对象的自然语法构造查询语句，可以看成是HibernateHQL的等价物。JPA定义了独特的JPQL（Java Persistence Query Language），JPQL是EJB QL的一种扩展，它是针对实体的一种查询语言，操作对象是实体，而不是关系数据库的表，而且能够支持批量更新和修改、JOIN、GROUP BY、HAVING 等通常只有 SQL 才能够提供的高级查询特性，甚至还能够支持子查询。

### 高级特性

JPA中能够支持面向对象的高级特性，如类之间的继承、多态和类之间的复杂关系，这样的支持能够让开发者最大限度的使用面向对象的模型设计企业应用，而不需要自行处理这些特性在关系数据库的持久化。

## 什么是spring data jpa？

Spring提供的一个用于简化JPA开发的框架。可以极大的简化JPA的写法，可以在几乎不用写实现的情况下，实现对数据的访问和操作。除了CRUD外，还包括如分页、排序等一些常用的功能。


* SpringData JPA只是SpringData中的一个子模块
* JPA是一套标准接口，而Hibernate是JPA的实现
* SpringData JPA 底层默认实现是使用Hibernate
* SpringDataJPA 的首个接口就是Repository，它是一个标记接口。只要我们的接口实现这个接口，那么我们就相当于在使用SpringDataJPA了。
* 就是说，只要我们实现了这个接口，我们就可以使用"按照方法命名规则"来进行查询。
### SpringDataJPA的核心接口


* **Repository**：最顶层的接口，是一个空的接口，目的是为了统一所有Repository的类型，且能让组件扫描的时候自动识别。
* **CrudRepository**：是Repository的子接口，提供CRUD的功能
* **PagingAndSortingRepository**：是CrudRepository的子接口，添加分页和排序的功能
* **JpaRepository**：是PagingAndSortingRepository的子接口，增加了一些实用的功能，比如：批量操作等。
* **JpaSpecificationExecutor**：用来做负责查询的接口,类似条件(QBC)查询
* **Specification**：是Spring Data JPA提供的一个查询规范，要做复杂的查询，只需围绕这个规范来设置查询条件即可
### CrudRepository 查询方法与命名规则

SpringDataJPA默认情况下, 提供了查询的相关的方法, 基本上能满足我们80%左右的需要. 但是还有一些是没有满足的,我们可以遵循它的命名规范来定义方法名. 如果没有满足命名规范的, 可以在方法上加@Query注解来写语句。

### JPA命令规范

![图片](https://images-cdn.shimo.im/CECehs5P9okiMZGA/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/YfJaEndOTUwbQcd7/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180924233204.png!thumbnail)

![图片](https://images-cdn.shimo.im/1vctXLroWvcPTEhw/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180924233232.png!thumbnail)

![图片](https://images-cdn.shimo.im/bE1GjcUAwPgQATKl/image.png!thumbnail)

### 条件查询的关键字


* And --- 等价于 SQL 中的 and 关键字，比如 findByUsernameAndPassword(String user, Striang pwd)；
* Or --- 等价于 SQL 中的 or 关键字，比如 findByUsernameOrAddress(String user, String addr)；
* Between --- 等价于 SQL 中的 between 关键字，比如 findBySalaryBetween(int max, int min)；
* LessThan --- 等价于 SQL 中的 "<"，比如 findBySalaryLessThan(int max)；
* GreaterThan --- 等价于 SQL 中的">"，比如 findBySalaryGreaterThan(int min)；
* IsNull --- 等价于 SQL 中的 "is null"，比如 findByUsernameIsNull()；
* IsNotNull --- 等价于 SQL 中的 "is not null"，比如 findByUsernameIsNotNull()；
* NotNull --- 与 IsNotNull 等价；
* Like --- 等价于 SQL 中的 "like"，比如 findByUsernameLike(String user)；
* NotLike --- 等价于 SQL 中的 "not like"，比如 findByUsernameNotLike(String user)；
* OrderBy --- 等价于 SQL 中的 "order by"，比如 findByUsernameOrderBySalaryAsc(String user)；
* Not --- 等价于 SQL 中的 "！ ="，比如 findByUsernameNot(String user)；
* In --- 等价于 SQL 中的 "in"，比如 findByUsernameIn(Collection<String> userList) ，方法的参数可以是 Collection 类型，也可以是数组或者不定长参数；
* NotIn --- 等价于 SQL 中的 "not in"，比如 findByUsernameNotIn(Collection<String> userList) ，方法的参数可以是 Collection 类型，也可以是数组或者不定长参数；
### @Query查询

## jpa、hibernate与spring data jpa三者关系

JPA作为一种规范——也就说JPA规范中提供的只是一些接口，显然接口不能直接拿来使用。虽然应用程序可以面向接口编程，但JPA底层一定需要某种JPA实现，否则JPA依然无法使用。

Sun之所以提出JPA规范，其目的是以官方的身份来统一各种ORM框架的规范，包括著名的Hibernate、TopLink等。不过JPA规范给开发者带来了福音：开发者面向JPA规范的接口，但底层的JPA实现可以任意切换：觉得Hibernate好的，可以选择Hibernate JPA实现；觉得TopLink好的，可以选择TopLink JPA实现……这样开发者可以避免为使用Hibernate学习一套ORM框架，为使用TopLink又要再学习一套ORM框架。

```
JPA是一套ORM规范，hibernate实现了JPA规范
hibernate中有自己的独立ORM操作数据库方式，也有JPA规范实现的操作数据库方式。
在数据库增删改查操作中，我们hibernate和JPA的操作都要会。
```
![图片](https://images-cdn.shimo.im/RDTerS27NLURSOsD/image.png!thumbnail)

虽然ORM框架都实现了JPA规范，但是在不同ORM框架之间切换是需要编写的代码有一些差异，而通过使用Spring Data Jpa能够方便大家在不同的ORM框架中间进行切换而不要更改代码。并且Spring Data Jpa对Repository层封装的很好，可以省去不少的麻烦。

Spring Data JPA 可以理解为 JPA 规范的再次封装抽象，底层还是使用了 Hibernate 的 JPA 技术实现。

![图片](https://images-cdn.shimo.im/F2td7SpAlDggIvdt/image.png!thumbnail)

## jpa实体相关注解

### @Entity

标注位置：实体类声明语句之前；

主要作用：标注该Java类为实体类，且将其映射到指定的数据库表。

### @Table

标注位置：实体类声明语句之前，与@Entity注释并列使用；

主要作用：标注当前实体类映射到数据库中的数据表名，当实体类与数据表名不同时使用。

### @Id

标注位置：实体类的属性声明语句之前，或属性的getter()方法之前；

主要作用：指定该实体类的当前属性映射到数据表的主键列。

### @GeneratedValue

标注位置：与@Id注释配合使用；

主要作用：通过其strategy属性指定数据表主键的生成策略。默认情况下，JPA自动选择最适合底层数据库的主键生成策略，即SqlServer对应identity，而MySQL对应auto increment。

![图片](https://images-cdn.shimo.im/zKAgenm2Ay0kN6OI/image.png!thumbnail)

### @Column

标注位置：实体类的属性声明语句之前，或属性的getter()方法之前；

主要作用：标注实体类的当前属性映射到数据库表的字段名，当属性名与数据表字段名不一致时使用。

![图片](https://images-cdn.shimo.im/AUGEctnTxV09qqxy/image.png!thumbnail)

### @Transient

标注位置：实体类的属性声明语句之前，或属性的getter()方法之前；

主要作用：标注实体类的当前属性不进行数据表字段的映射，ORM框架将忽略此映射，如实体类的getInfo()方法通常不需要映射到数据表的字段上。

### @Temporal

标注位置：实体类的属性声明语句之前，或属性的getter()方法之前；

主要作用：标注实体类中Date类型（Java核心API中未定义Date类型的精度）的属性映射到数据表字段的具体精度（数据库中Date类型的数据有DATE、TIME和TIMESTAMP三种精度）。

## 基本CRUD操作

### Repository接口

仅仅是一个标识，表明任何继承它的均为仓库接口类，方便Spring自动扫描识别。

### CRUDRespository接口

继承Repository，实现了一组CRUD相关的方法。

![图片](https://images-cdn.shimo.im/CDlbRxc3ToMZAN2P/image.png!thumbnail)

### 命名参数，索引参数


* **索引参数**如下所示，索引值从1开始，查询中 ”?X” 个数需要与方法定义的参数个数相一致，并且顺序也要一致
```
@Modifying 
@Query("update User u set u.firstname = ?1 where u.lastname = ?2") 
int setFixedFirstnameFor(String firstname, String lastname); 
```
* **命名参数**（推荐使用这种方式）
可以定义好参数名，赋值时采用@Param("参数名")，而不用管顺序。如下所示：
```
public interface UserRepository extends JpaRepository<User, Long> { 
  @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname") 
  User findByLastnameOrFirstname(@Param("lastname") String lastname, 
                                 @Param("firstname") String firstname); 
} 
```
### PagingAndSortRespository接口查询

该接口继承了CrudRepository接口，提供了两个方法，实现了分页和排序的功能了

![图片](https://images-cdn.shimo.im/fbx8iRj7mWgPzP5o/image.png!thumbnail)

### JpaRepository接口

该接口继承了PagingAndSortingRepository接口。同时也继承QueryByExampleExecutor接口，这是个用“实例”进行查询的接口

![图片](https://images-cdn.shimo.im/cPh4AsxFVqsegasa/image.png!thumbnail)

### 事务在spring data jpa中的使用，@Modifying @Transactional的综合使用

@Modifying表示是更新执行

@Transactional带事务

### JpaSpecificationExecutor接口

该接口提供了对JPA Criteria查询（动态查询）的支持。

spring data jpa 通过创建方法名来做查询，只能做简单的查询，那如果我们要做复杂一些的查询呢，多条件分页怎么办，这里，spring data jpa为我们提供了JpaSpecificationExecutor接口，只要简单实现toPredicate方法就可以实现复杂的查询

1.首先让我们的接口继承于JpaSpecificationExecutor

2.JpaSpecificationExecutor提供了以下接口

![图片](https://images-cdn.shimo.im/DvzrxR0TUw4Oc4Ac/image.png!thumbnail)

其中Specification就是需要我们传进去的参数，它是一个接口

![图片](https://images-cdn.shimo.im/ivGbyWxmSrQElFaY/image.png!thumbnail)

## spring集成spring data jpa

第一步：pom导入坐标

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```
第二步：配置文件中配置spring jpa和数据库相关信息

```
spring:
    http:
        encoding:
          charset: UTF-8
          force: true
          enabled: true
    datasource:
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost/jpademo?useSSL=false&characterEncoding=utf8
        username: root
        password: admin
    jpa:
        database: mysql
        show-sql: true
        hibernate:
            ddl-auto: update
```
第三步：根据需求，dao层文件继承Repository接口或者子集
![图片](https://images-cdn.shimo.im/wrzwZpYqa508x2AD/image.png!thumbnail)

## 可参考项目

TIMO后台管理系统


* [https://gitee.com/aun/Timo](https://gitee.com/aun/Timo)

mblog开源免费的博客系统


* [https://gitee.com/mtons/mblog](https://gitee.com/mtons/mblog)
## 原理分析

Spring Data JPA启动,主要分为几步:


1. 通过注解和包扫描获得配置信息 ,对应类为AnnotationRepositoryConfigurationSource
2. 通过配置信息,向Spring注册Repository的FactoryBean的BeanDefinition,即JpaRepositoryFactoryBean的BeanDefinition
3. Spring启动中,会根据BeanDefinition实例化FactoryBean,并调用它的getObject方法
4. 在getObject方法中,Spring Data JPA解析方法名并生成代理.

接下来,我们从@EnableJpaRepositories开始,可以看到在它上边有@Import(JpaRepositoriesRegistrar.class)注解,其中JpaRepositoriesRegistrar是ImportBeanDefinitionRegistrar的子类.

其中ImportBeanDefintionRegistrar是Spring提供的策略,spring会在加载BeanDefinition的时候调用它的方法:

```
public void registerBeanDefinitions(
      AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);
```
判断：

org.springframework.data.repository.config.RepositoryComponentProvider#isCandidateComponent

### SimpleJpaRepository

我们操作的repository层通常只有一个接口，没有实现类，一般继承2个接口(JpaRepository,JpaSpecificationExecutor)，这样就可以操作持久层的数据。

这其实是动态代理的运用，我们在调用repository层接口时候，会被代理成SimpleJpaRepository，SimpleJpaRepository其实也继承继承2个接口(JpaRepository,JpaSpecificationExecutor)，我们看看代码可以发现，其实SimpleJpaRepository的底层其实是运用了EntityManager来实现数据库的操作。

SimpleJpaRepository代码：

```
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    private final EntityManager em;
    ...
}
```
JpaRepositoryImplementation代码：
```
public interface JpaRepositoryImplementation<T, ID> extends JpaRepository<T, ID>, JpaSpecificationExecutor<T> {
    ...
}
```
**总结如下：**
我们在 Dao 接口里边 就可以调用 JPARepository 接口的方法, JPARepository有一个默认实现类:SimpleJpaRepository,当我们 调用获得Dao接口对象时,得到的是接口的子类代理对象proxy,当我们调用这个代理对象proxy的方法时, 实际调用的是 SimpleJpaRepository的方法;这样实现一些既定的CRUD操作;

## 好参考文章


* Spring Data JPA的启动过程

[https://blog.csdn.net/ysdssb/article/details/58619340](https://blog.csdn.net/ysdssb/article/details/58619340)


* spring-jpa 源码学习(1)—加载流程简介

[https://www.jianshu.com/p/63b186ededdf](https://www.jianshu.com/p/63b186ededdf)

[https://www.jianshu.com/p/96cecc296d6f](https://www.jianshu.com/p/96cecc296d6f)


