---
title: renren-fast脚手架学习（下）
date: 2020-09-14 14:56:04
tags: renren-fast
excerpt: 转载8、renren-fast脚手架学习（下）
---

# [2_定时任务Quartz.xmind](https://uploader.shimo.im/f/EFl1pRzJWhIjJZqA.xmind)[1_XSS与CSRF.xmind](https://uploader.shimo.im/f/UeuZQX3W4O0xLl0m.xmind)CSRF与XSS

## CSRF

CSRF（Cross-site request forgery）：跨站请求伪造。

用户是网站A的注册用户，且登录进去，于是网站A就给用户下发cookie。

从上图可以看出，要完成一次CSRF攻击，受害者必须满足两个必要的条件：

（1）登录受信任网站A，并在本地生成Cookie。（如果用户没有登录网站A，那么网站B在诱导的时候，请求网站A的api接口时，会提示你登录）

（2）在不登出A的情况下，访问危险网站B（其实是利用了网站A的漏洞）。

我们在讲CSRF时，一定要把上面的两点说清楚。

温馨提示一下，cookie保证了用户可以处于登录状态，但网站B其实拿不到 cookie。

### 预防CSRF攻击

跨站请求攻击，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并执行一定的操作。因为浏览器认证过，所以网站会认为是真正的用户在操作。

### **例子**

假如一家银行转账操作的URL地址如下：

[http://www.examplebank.com/withdraw?account=AccoutName&amount=1000&for=PayeeName](http://www.examplebank.com/withdraw?account=AccoutName&amount=1000&for=PayeeName)

那么，一个另外的网站可以放置如下代码

<img src="http://www.examplebank.com/withdraw?account=Alice&amount=1000&for=Badman">

如果用户之前刚访问过银行网站，登陆信息尚未过期，再次访问恶意网站点击了这个图片，那么就会损失1000元。事实上，这些地址还可以放在论坛、博客等地方，这种恶意访问的形式更加隐蔽，如果服务端没有相应措施，很容易受到威胁。

### 防御措施


* 检查referer字段

HTTP头中有一个Referer字段，这个字段是用来标明请求来源于哪一个网址。通常来说，Referer字段应和请求的地址是在同一个域名下的。服务器可以通过判断Referer字段来判断请求的来源。 这种方法简单易行，但也有其局限性。http协议无法保证来访的浏览器的具体实现，可以通过篡改Referer字段的方式来进行攻击


* Token 验证

（1）服务器发送给客户端一个token；

（2）客户端提交的表单中带着这个token。

（3）如果这个 token 不合法，那么服务器拒绝这个请求。

## XSS

XSS（Cross Site Scripting）：跨域脚本攻击。

XSS攻击的核心原理是：不需要你做任何的登录认证，它会通过合法的操作（比如在url中输入、在评论框中输入），向你的页面注入脚本（可能是js、hmtl代码块等）。

最后导致的结果可能是：


* 盗用Cookie
* 破坏页面的正常结构，插入广告等恶意内容
* D-doss攻击

1、百度百科的解释: XSS又叫CSS (Cross Site Script) ，跨站脚本攻击。它指的是恶意攻击者往Web页面里插入恶意html代码，当用户浏览该页之时，嵌入其中Web里面的html代码会被执行，从而达到恶意用户的特殊目的。

2、它与SQL注入攻击类似，SQL注入攻击中以SQL语句作为用户输入，从而达到查询/修改/删除数据的目的，而在xss攻击中，通过插入恶意脚本，实现对用户游览器的控制，获取用户的一些信息。

### 攻击方式

1、反射型

发出请求时，XSS代码出现在url中，作为输入提交到服务器端，服务器端解析后响应，XSS代码随响应内容一起传回给浏览器，最后浏览器解析执行XSS代码。这个过程像一次反射，所以叫反射型XSS。

2、存储型

存储型XSS和反射型XSS的差别在于，提交的代码会存储在服务器端（数据库、内存、文件系统等），下次请求时目标页面时不用再提交XSS代码。

### 项目运用

**Filter过滤器**

Filter过滤器是一种比较实用的东西，可以过滤不良信息，对提交来的信息进行处理。是Request和Response之间的传输纽带。

**HttpServletRequestWrapper**

Filter是这样一种Java对象，它能能在request到达servlet的服务方法之前拦截HttpServletRequest对象，而在服务方 法转移控制后又能拦截HttpServletResponse对象。

你可以使用filter来实现特定的任务，比如验证用户输入，以及压缩web内容。**但HttpServletRequest对象的参数是不可改变的**，这极大地缩减了filter的应用范围。至少在一半的时间里，你希望可以改变准备传送给 filter的对象。

幸运的是，尽管你不能改变不变对象本身，但你却可以通过使用**装饰模式**来改变其状态。

所以：在Filter中修改后台Controller中获取到的HttpServletRequest中的参数，只需要在Filter中自定义一个类继承于HttpServletRequestWrapper，并复写getParameterNames、getParameter、getParameterValues等方法即可

方法：

要创建HttpServletRequest的装饰类，你需要继承HttpServletRequestWrapper并且覆盖你希望改变的方法。

![图片](https://images-cdn.shimo.im/FGDuMsyylKISsSNp/image.png!thumbnail)


* *ServletRequest  抽象组件*
* *HttpServletRequest  抽象组件的一个子类，它的实例被称作“被装饰者”*
* *ServletRequestWrapper  一个基本的装饰类，这里是非抽象的*
* *HttpServletRequestWrapper  一个具体的装饰者，当然这里也继承了HttpServletRequest这个接口，是为了获取一些在ServletRequest中没有的方法*
* *ModifyParametersWrapper  同样是 一个具体的装饰者（PS：我自定义的一个类）*

**防XSS注入流程：**


* 自定义包装类XssHttpServletRequestWrapper，继承HttpServletRequestWrapper，重写getInputStream(),getParameter(String name),getParameterValues(String name)等方法。
* 自定义过滤器XssFilter，过滤所有链接，这样所有方法中request.getParameter();就能调用自定义包装类里面重写的方法，进行xss过滤。
# quartz

依赖关系：

spring-boot-starter-web --> spring-webmvc -> spring-context-support -> quartz

## @EnableScheduling

### @EnableScheduling定时任务

springboot的web项目最简单定时任务使用：

第一步、使用@EnableScheduling注解表示开启定时任务模块

```
@EnableScheduling
```
第二步、定义一个bean，在方法上加上@Scheduled注解
```
@Component
public class TestJob {
    @Scheduled(cron = "* * * * * ?")
    public void job1() {
        System.out.println("-------------> job1");
    }
}
```
启动项目之后，即可开启定时任务。
这样使用定时任务的缺陷是：


* 无法手动控制定时任务
* 难做到负载均衡
### @Scheduled注解的参数说明

1.cron是设置定时执行的表达式,如 0 0/5 * * * ?每隔五分钟执行一次

2.zone表示执行时间的时区

3.fixedDelay 和fixedDelayString 一个固定延迟时间执行,上个任务完成后,延迟多久执行

4.fixedRate 和fixedRateString一个固定频率执行,上个任务开始后多长时间后开始执行

5.initialDelay 和initialDelayString表示一个初始延迟时间,第一次被调用前延迟的时间


* 比如

//每30秒执行一次

@Scheduled(fixedRate = 1000* 30)

//在固定时间执行

@Scheduled(cron = "0 */1 *  * * * ")

### @Scheduled转成异步线程

采用异步的方式执行调度任务，配置 Spring 的 @EnableAsync，在执行定时任务的方法上标注 @Async

## quartz的三大组件


* [https://www.cnblogs.com/zhanghaoliang/p/7886110.html](https://www.cnblogs.com/zhanghaoliang/p/7886110.html)

Quartz对任务调度的领域问题进行了高度的抽象，提出了调度器、任务和触发器这3个核心的概念，并在org.quartz通过接口和类对重要的这些核心概念进行描述

### 分析


* **Job**：

是一个接口，只有一个方法void execute(JobExecutionContext context)，开发者实现该接口定义运行任务，JobExecutionContext类提供了调度上下文的各种信息。Job运行时的信息保存在JobDataMap实例中；


* **JobDetail**：

Quartz在每次执行Job时，都重新创建一个Job实例，所以它不直接接受一个Job的实例，相反它接收一个Job实现类，以便运行时通过newInstance()的反射机制实例化Job。因此需要通过一个类来描述Job的实现类及其它相关的静态信息，如Job名字、描述、关联监听器等信息，JobDetail承担了这一角色。

通过该类的构造函数可以更具体地了解它的功用：JobDetail(java.lang.String name, java.lang.String group, java.lang.Class jobClass)，该构造函数要求指定Job的实现类，以及任务在Scheduler中的组名和Job名称；


* **Trigger：**

是一个类，描述触发Job执行的时间触发规则。主要有SimpleTrigger和CronTrigger这两个子类。当仅需触发一次或者以固定时间间隔周期执行，SimpleTrigger是最适合的选择；而CronTrigger则可以通过Cron表达式定义出各种复杂时间规则的调度方案：如每早晨9:00执行，周一、周三、周五下午5:00执行等；


* **Calendar：**

org.quartz.Calendar和java.util.Calendar不同，它是一些日历特定时间点的集合（可以简单地将org.quartz.Calendar看作java.util.Calendar的集合——java.util.Calendar代表一个日历时间点，无特殊说明后面的Calendar即指org.quartz.Calendar）。一个Trigger可以和多个Calendar关联，以便排除或包含某些时间点。

假设，我们安排每周星期一早上10:00执行任务，但是如果碰到法定的节日，任务则不执行，这时就需要在Trigger触发机制的基础上使用Calendar进行定点排除。针对不同时间段类型，Quartz在org.quartz.impl.calendar包下提供了若干个Calendar的实现类，如AnnualCalendar、MonthlyCalendar、WeeklyCalendar分别针对每年、每月和每周进行定义；


* **Scheduler：**

代表一个Quartz的独立运行容器，Trigger和JobDetail可以注册到Scheduler中，两者在Scheduler中拥有各自的组及名称，组及名称是Scheduler查找定位容器中某一对象的依据，Trigger的组及名称必须唯一，JobDetail的组和名称也必须唯一（但可以和Trigger的组和名称相同，因为它们是不同类型的）。Scheduler定义了多个接口方法，允许外部通过组及名称访问和控制容器中Trigger和JobDetail。

Scheduler可以将Trigger绑定到某一JobDetail中，这样当Trigger触发时，对应的Job就被执行。一个Job可以对应多个Trigger，但一个Trigger只能对应一个Job。可以通过SchedulerFactory创建一个Scheduler实例。Scheduler拥有一个SchedulerContext，它类似于ServletContext，保存着Scheduler上下文信息，Job和Trigger都可以访问SchedulerContext内的信息。SchedulerContext内部通过一个Map，以键值对的方式维护这些上下文数据，SchedulerContext为保存和获取数据提供了多个put()和getXxx()的方法。可以通过Scheduler# getContext()获取对应的SchedulerContext实例；


* **ThreadPool：**

Scheduler使用一个线程池作为任务运行的基础设施，任务通过共享线程池中的线程提高运行效率。

Job有一个StatefulJob子接口，代表有状态的任务，该接口是一个没有方法的标签接口，其目的是让Quartz知道任务的类型，以便采用不同的执行方案。无状态任务在执行时拥有自己的JobDataMap拷贝，对JobDataMap的更改不会影响下次的执行。而有状态任务共享共享同一个JobDataMap实例，每次任务执行对JobDataMap所做的更改会保存下来，后面的执行可以看到这个更改，也即每次执行任务后都会对后面的执行发生影响。

正因为这个原因，无状态的Job可以并发执行，而有状态的StatefulJob不能并发执行，这意味着如果前次的StatefulJob还没有执行完毕，下一次的任务将阻塞等待，直到前次任务执行完毕。有状态任务比无状态任务需要考虑更多的因素，程序往往拥有更高的复杂度，因此除非必要，应该尽量使用无状态的Job。

如果Quartz使用了数据库持久化任务调度信息，无状态的JobDataMap仅会在Scheduler注册任务时保持一次，而有状态任务对应的JobDataMap在每次执行任务后都会进行保存。

Trigger自身也可以拥有一个JobDataMap，其关联的Job可以通过JobExecutionContext#getTrigger().getJobDataMap()获取Trigger中的JobDataMap。不管是有状态还是无状态的任务，在任务执行期间对Trigger的JobDataMap所做的更改都不会进行持久，也即不会对下次的执行产生影响。

Quartz拥有完善的事件和监听体系，大部分组件都拥有事件，如任务执行前事件、任务执行后事件、触发器触发前事件、触发后事件、调度器开始事件、关闭事件等等，可以注册相应的监听器处理感兴趣的事件。满意答案

### 总结

Job:是任务的接口，实现该接口定义运行任务。

JobDetail:用来描述保存Job的实现类及其它相关信息，如Job名字、描述、关联监听器等信息

Trigger:是触发器,描述触发Job执行时间的规则。(2个子类)

SimpleTrigger:适用于仅需触发一次或以固定时间间隔周期执行

CronTrigger:适用于各种复杂时间规则。通过Cron表达式定义

Scheduler:是调度器,代表一个Quartz独立运行容器，管理Quartz的运行环境。协调被注册到Scheduler中的Trigger和JobDetails的工作， 一个Scheduler中可以注册多个JobDetail和多个Trigger



## cron表达式

cron表达式在线生成

[http://cron.qqe2.com/](http://cron.qqe2.com/)

![图片](https://uploader.shimo.im/f/OpsbRXwDnAQl55ps.png!thumbnail)

如上面的表达式所示:

“*”字符被用来指定所有的值。如：”*“在分钟的字段域里表示“每分钟”。

“-”字符被用来指定一个范围。如：“10-12”在小时域意味着“10点、11点、12点”。



“,”字符被用来指定另外的值。如：“MON,WED,FRI”在星期域里表示”星期一、星期三、星期五”.

“?”字符只在日期域和星期域中使用。它被用来指定“非明确的值”。当你需要通过在这两个域中的一个来指定一些东西的时候，它是有用的。看下面的例子你就会明白。

“L”字符指定在月或者星期中的某天（最后一天）。即“Last ”的缩写。但是在星期和月中“Ｌ”表示不同的意思，如：在月子段中“L”指月份的最后一天-1月31日，2月28日，如果在星期字段中则简单的表示为“7”或者“SAT”。如果在星期字段中在某个value值得后面，则表示“某月的最后一个星期value”,如“6L”表示某月的最后一个星期五。

“W”字符只能用在月份字段中，该字段指定了离指定日期最近的那个星期日。

“#”字符只能用在星期字段，该字段指定了第几个星期value在某月中

![图片](https://uploader.shimo.im/f/bk0Cq8poBvYS4TuW.png!thumbnail)

