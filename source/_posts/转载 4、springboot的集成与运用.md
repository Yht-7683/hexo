---
title: springboot集成与运用
date: 2020-09-14 14:52:04
tags: springboot框架
excerpt: 转载4、springboot集成与运用
---

## [_课程4_springboot的集成与运用_预习_.xmind](https://uploader.shimo.im/f/HN6cedVfwcc0BmdD.xmind)[【课程4】springboot的集成与运用（预习）.xmind](https://uploader.shimo.im/f/P5pgcNh8j3ELenwI.xmind)[1_Spring事件.xmind](https://uploader.shimo.im/f/z6w5CzyPHDMe8EMs.xmind)[2_hibernate_validatior.xmind](https://uploader.shimo.im/f/bYNLMJBJJrQ2Yq6c.xmind)[3_异常处理.xmind](https://uploader.shimo.im/f/h7ANfjJNVuQuy84z.xmind)[4_多数据源.xmind](https://uploader.shimo.im/f/rU5U6Sx0dao014fG.xmind)自定义banner

我们只需要在Spring Boot工程的/src/main/resources目录下创建一个banner.txt文件，然后将ASCII字符画复制进去，就能替换默认的banner了。


* 属性设置

在banner.txt中，我们可以设置一些属性。比如：


* ${AnsiColor.BRIGHT_RED}：设置控制台中输出内容的颜色
* ${application.version}：用来获取MANIFEST.MF文件中的版本号
* ${application.formatted-version}：格式化后的${application.version}版本信息
* ${spring-boot.version}：Spring Boot的版本号
* ${spring-boot.formatted-version}：格式化后的${spring-boot.version}版本信息

* 生成工具

如果让我们手工的来编辑这些字符画，显然是一件非常困难的差事。所以，我们可以借助下面这些工具，轻松地根据文字或图片来生成用于Banner输出的字符画。


* [http://patorjk.com/software/taag](http://patorjk.com/software/taag)
* [http://www.network-science.de/ascii/](http://www.network-science.de/ascii/)
* [http://www.degraeve.com/img2txt.php](http://www.degraeve.com/img2txt.php)
## freemarker集成

### 语法大全


* [https://www.cnblogs.com/JealousGirl/p/6914122.html](https://www.cnblogs.com/JealousGirl/p/6914122.html)
* [https://www.jianshu.com/p/7efc095dba57](https://www.jianshu.com/p/7efc095dba57)
### 集成

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```
```
spring:
  freemarker:
  template-loader-path:
  - classpath:/templates
  cache: false
  check-template-location: true
  charset: utf-8
  settings:
    classic_compatible: true #处理空值
    template_exception_handler: rethrow
    template_update_delay: 0
    datetime_format: yyyy-MM-dd HH:mm
    number_format: 0.##
```
## **Spring事件**

ApplicationEvent以及Listener是Spring为我们提供的一个事件监听、订阅的实现，内部实现原理是观察者设计模式，设计初衷也是为了系统业务逻辑之间的解耦，提高可扩展性以及可维护性。



通过 ApplicationEvent 类和 ApplicationListener 接口来提供在 ApplicationContext 中处理事件。如果一个 bean 实现 ApplicationListener，那么每次 ApplicationEvent 被发布到 ApplicationContext 上，那个 bean 会被通知。

注意：因为ApplicationEvent是基于Spring容器完成的监听，所以在与常用的MQ之间的区别你可以这样认为，但是也不能作为使用标准：


* ApplicationEvent用于Bean与Bean之间的业务解耦
* MQ（Rabbitmq等）用于应用之间的业务解耦
### 遵循流程


* 自定义事件，继承ApplicationEvent（org.springframework.context.ApplicationEvent）
* 定义监听事件，实现ApplicationListener（org.springframework.context.ApplicationListener）
* 使用容器触发事件。
* 发布事件，使用applicationContext发布事件。

ApplicationEvent中，在自定义事件的构造方法中除了第一个source参数，其他参数都可以去自定义，可以根据项目实际情况进行监听传参，这里就只定义个简单的String字符串的透传。

### **逻辑整理**

这里使用一种简单的单机实现Spring的事件模型ApplicationEvent。

第一步、分别自定义订阅和通知事件，继承ApplicationEvent

第二步、分别定义事件监听器，实现ApplicationListener

第三步、使用容器发布事件（订阅事件、通知事件）

### 项目运用


* 定义通知事件MessageEvent 。
```
@Data
public class MessageEvent extends ApplicationEvent {
    private long fromUserId;
    private long toUserId;
    private long postId;
    private int event;
    public MessageEvent(Object source, int event) {
        super(source);
        this.event = event;
    }
}
```

* MessageEventHandler，关注事件监听处理类，做相关业务处理。

注意ApplicationListener是带泛型的，这样MessageEventHandler 只会监听MessageEvent事件。另外可以用@Component来注册组件，这样就不需要在spring的配置文件中指定了。

另外要注意的一点是，发布事件的方法与事件处理是同步的，在service中有注册 @Transactional的情况下，listener中的do something和service中的是在同一个tansaction下。

解决方法也简单，可以吧事件处理以异步调用的方式来处理。在事件处理上加上@Async注解，表示异步调用，同时为了让注解起作用，需要开启@EnableAsync注解。可以加在application启动器上。

```
@Slf4j
@Component
public class MessageEventHandler implements ApplicationListener<MessageEvent> {
    /**
     * @Async 表示异步调用
     * @param event
     */
    @Async
    @Override
    public void onApplicationEvent(MessageEvent event) {
        //TODO 业务处理
        log.info("---------------> " + event.toString());
    }
}
```

* FollowController.sendNotify()-->发送关注通知。
```
@GetMapping("/test")
public User test() {
    MessageEvent event = new MessageEvent("MessagaEvent", Consts.MESSAGE_EVENT_FAVOR_POST);
    event.setPostId(1L);
    event.setFromUserId(2L);
    event.setToUserId(3L);
    applicationContext.publishEvent(event);
    User user = userService.getById(1L);
    log.info("---------> " + user.toString());
    return user;
}
```
### 知识拓展--@EventListener


* [https://blog.csdn.net/chszs/article/details/49097919](https://blog.csdn.net/chszs/article/details/49097919)

**有条件的事件处理**

为了使注释@EventListener的功能更强大，Spring 4.2支持用SpEL表达式表达事件类型的方式

**定义事件**

```
public class TestEvent extends ApplicationEvent {
    public boolean isImport;
    public TestEvent(Object source, boolean isImport) {
        super(source);
        this.isImport = isImport;
    }
    public boolean isImport() {
        return isImport;
    }
    public void setImport(boolean anImport) {
        isImport = anImport;
    }
    @Override
    public String toString() {
        return "TestEvent{" +
                "isImport=" + isImport +
                '}';
    }
}
```
**定义监听**
```
@Component
public class EventHandler {
    @EventListener(condition="#testEvent.isImport")
    public void TestEventTest(TestEvent testEvent) {
        System.out.println("==============TestEvent==============" + testEvent.toString());
    }
}
```
## 实体验证hibernate validatior

### 参数校验

在开发中经常需要写一些字段校验的代码，比如字段非空，字段长度限制，邮箱格式验证等等，写这些与业务逻辑关系不大的代码个人感觉有两个麻烦：


* 验证代码繁琐，重复劳动
* 方法内代码显得冗长
* 每次要看哪些参数验证是否完整，需要去翻阅验证逻辑代码

hibernate validator（[官方文档](http://hibernate.org/validator/documentation/)）提供了一套比较完善、便捷的验证实现方式。

spring-boot-starter-web包里面有hibernate-validator包，不需要引用hibernate validator依赖。

### 校验模式

1、普通模式（默认是这个模式）

普通模式(会校验完所有的属性，然后返回所有的验证失败信息)

2、快速失败返回模式

快速失败返回模式(只要有一个验证失败，则返回)

两种验证模式配置方式：（[参考官方文档](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#section-provider-specific-settings)）


* failFast：true  快速失败返回模式    false 普通模式
```
ValidatorFactory validatorFactory = Validation.byProvider( HibernateValidator.class )
        .configure()
        .failFast( true )
        .buildValidatorFactory();
Validator validator = validatorFactory.getValidator();
```
* 和 （hibernate.validator.fail_fast：true  快速失败返回模式    false 普通模式）
```
ValidatorFactory validatorFactory = Validation.byProvider( HibernateValidator.class )
        .configure()
        .addProperty( "hibernate.validator.fail_fast", "true" )
        .buildValidatorFactory();
Validator validator = validatorFactory.getValidator();
```
springboot配置hibernate Validator为快速失败返回模式：

```
@Configuration
public class ValidatorConfig {
    @Bean
    public Validator validator(){
        ValidatorFactory validatorFactory = Validation.byProvider( HibernateValidator.class )
                .configure()
                .failFast( true )
                .buildValidatorFactory();
        Validator validator = validatorFactory.getValidator();
        return validator;
    }
}
```
### 实例demo

```
@Data
@EqualsAndHashCode(callSuper = true)
@Accessors(chain = true)
public class User extends BaseEntity {
    private static final long serialVersionUID = 1L;
    
    @NotBlank(message="用户名不能为空")
    private String username;
    @NotBlank(message="密码不能为空")
    private String password;
    @Email
    private String email;
    
    ...
    
}
```
controller中校验，@Valid表示需要验证的实体，BindingResult是校验结果

```
@PostMapping("/add")
public Result addUser(@Valid User user, BindingResult result) {
    if(result.hasErrors()) {
        List<ObjectError> allErrors = result.getAllErrors();
        for (ObjectError error : allErrors) {
            log.error("here is error ----------> {}", error.getDefaultMessage());
        }
        return Result.failure(allErrors.get(0).getDefaultMessage());
    }
    userService.save(user);
    return Result.success();
}
```
## springboot的异常处理

在日常web开发中发生了异常，往往是需要通过一个统一的异常处理来保证客户端能够收到友好的提示。


* [https://www.jianshu.com/p/7c94d1ac2092](https://www.jianshu.com/p/7c94d1ac2092)
### jar包中自定义错误页面（内嵌tomcat）

创建Spring Boot项目，默认打包方式是jar，内部使用内嵌tomcat等servlet容器

最简单的方式是直接在resources/templates目录下创建error.html页面，此时如果访问不存在的画面就会直接进入此画面。

所有的错误都会返回这个页面：

我使用的是freemaker模板引擎，所以：

![图片](https://uploader.shimo.im/f/RIOmknd05gQcFop2.png!thumbnail)

如何区分404,500等？简单方法是在static先新建路径/error/404.html，注意这是静态页面！！！4xx表示4开头的其他错误码。

![图片](https://uploader.shimo.im/f/9R14OPtI0oslKnzF.png!thumbnail)

随便搞个链接得到：

![图片](https://uploader.shimo.im/f/KyULgscyq2YrgeHr.png!thumbnail)

### war包中自定义错误页面（内置或外置tomcat）

当服务器外置的时候，上面jar包形式的配置是不起作用的，这时候可以通过实现ErrorController来定义自定义错误。getErrorPath()表示当发现错误时候重定向的链接。

```
@Controller
public class IErrorController implements ErrorController {
    @Override
    public String getErrorPath() {
        return "/error";
    }
    @RequestMapping("/error")
    public String handleError(HttpServletRequest request) {
        Object status = request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        if (status != null) {
            Integer statusCode = Integer.valueOf(status.toString());
            if (statusCode == HttpStatus.NOT_FOUND.value()) {
                return "commons/404";
            } else if (statusCode == HttpStatus.INTERNAL_SERVER_ERROR.value()) {
                return "commons/500";
            }
        }
        return "commons/error";
    }
}
```
注入BasicErrorController（org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration#basicErrorController），默认errorViewResolver是DefaultErrorViewResolver

![图片](https://uploader.shimo.im/f/DhSBW3JrWMAPgEAK.png!thumbnail)

DefaultErrorViewResolver中处理页面（org.springframework.boot.autoconfigure.web.servlet.error.DefaultErrorViewResolver#resolve）

![图片](https://uploader.shimo.im/f/Tn1yE1fEkZQRjKd5.png!thumbnail)

### 单个控制器异常

@ExceptionHandler可用于控制器中，表示处理当前类的异常。

```
@ExceptionHandler(Exception.class)
public String exceptionHandler(Exception e) {
    log.error("---------------->捕获到局部异常", e);
    return "index";
}
```
### 全局异常

如果单使用@ExceptionHandler，只能在当前Controller中处理异常。但当配合@ControllerAdvice一起使用的时候，就可以摆脱那个限制了。

可通过@ExceptionHandler的参数进行异常分类处理。

**步骤：**

第一步、自定义异常

```
public class MyException extends Exception {
    public MyException(String message) {
        super(message);
    }
}
```
第二步、定义全局处理器，主要是@ControllerAdvice和@ExceptionHandler注解的运用

```
@Slf4j
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(value = Exception.class)
    public ModelAndView defaultErrorHandler(HttpServletRequest req, Exception e) throws Exception {
        log.error("------------------>捕捉到全局异常", e);
        ModelAndView mav = new ModelAndView();
        mav.addObject("exception", e);
        mav.addObject("url", req.getRequestURL());
        mav.setViewName("error");
        return mav;
    }
    @ExceptionHandler(value = MyException.class)
    @ResponseBody
    public R jsonErrorHandler(HttpServletRequest req, MyException e) throws Exception {
        //TODO 错误日志处理
        return R.fail(e.getMessage(), "some data");
    }
}
```
### 异常处理机制原理


* **关键类-HandlerExceptionResolver**

![图片](https://static.oschina.net/uploads/space/2016/0816/150118_JDoO_1258171.png)

Spring MVC通过HandlerExceptionResolver处理程序的异常，包括Handler的映射、数据绑定以及目标方法的执行。HandlerExceptionResolver时一个接口，该接口的实现类都有处理异常的功能。HandlerExceptionResolver是该接口应用广泛的一个实现类，并且DispatcherServlet默认装配了HandlerExceptionResolver 的Bean。

SpringMVC 提供的异常处理主要有两种方式：


* 一种是直接实现自己的HandlerExceptionResolver
* 一种是使用注解

通过注解的方式实现处理异常主要有以下两种方式：


* 1 @ControllerAdvice+@ExceptionHandler：配置对全局异常进行处理
* 2 @Controller + @ExceptionHandler：配置对当前所在Controller的异常进行处理

在SpringMVC中，处理异常类实际上是HandlerExceptionResolver子类。HandlerExceptionResolver处理所有controller类在执行过程中抛出的未被处理的异常。

本文演示如何使用以上多种处理异常的方式，最后演示以不同的方式将异常结果返回给调用者


* 返回 modelAndView
* 返回一个页面的地址
* 返回 JSON
* 返回 http 错误码
## 多数据源模块

### 预备知识-ThreadLocal

ThreadLocal是线程局部变量，所谓的线程局部变量，就是仅仅只能被本线程访问，不能在线程之间进行共享访问的变量。

### 关键抽象类-AbstractRoutingDataSource

官方注释如下：

```
* Abstract {@link javax.sql.DataSource} implementation that routes {@link #getConnection()}
* calls to one of various target DataSources based on a lookup key. The latter is usually
* (but not necessarily) determined through some thread-bound transaction context.
```
大概意思是：
就是getConnection()根据查找lookup key键对不同目标数据源的调用，通常是通过(但不一定)某些线程绑定的事物上下文来实现。

**通过这我们知道可以实现：**


* - 多数据源的动态切换，在程序运行时，把数据源数据源动态织入到程序中，灵活的进行数据源切换。
* - 基于多数据源的动态切换，我们可以实现读写分离，这么做缺点也很明显，无法动态的增加数据源。
### 逻辑思路


* DynamicDataSource继承AbstractRoutingDataSource类，并实现了determineCurrentLookupKey()方法。
* 我们配置的多个数据源会放在AbstractRoutingDataSource的 targetDataSources和defaultTargetDataSource中，然后通过afterPropertiesSet()方法将数据源分别进行复制到resolvedDataSources和resolvedDefaultDataSource中。
* AbstractRoutingDataSource的getConnection()的方法的时候，先调用determineTargetDataSource()方法返回DataSource在进行getConnection()。
### **实现多数据源**

步骤1，在spring boot中，增加多数据源的配置

步骤2，扩展Spring的AbstractRoutingDataSource抽象类，AbstractRoutingDataSource中的抽象方法determineCurrentLookupKey是实现多数据源的核心，并对该方法进行Override

步骤3，配置DataSource，指定数据源的信息

步骤4，通过注解，实现多数据源

步骤5、配置加上(exclude={DataSourceAutoConfiguration.**class**})

![图片](https://images-cdn.shimo.im/AmEM6jqp5w45PraA/image.png!thumbnail)

### 关于事务

只支持单库事务，也就是说切换数据源要在开启事务之前执行。  spring DataSourceTransactionManager进行事务管理，开启事务，会将数据源缓存到DataSourceTransactionObject对象中进行后续的commit rollback等事务操作。

![图片](https://images-cdn.shimo.im/0G1kxTZzC3McF69t/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/SgnevpHrDM4Wijf4/image.png!thumbnail)

### 使用经验

出现多数据源动态切换失败的原因是因为在事务开启后，数据源就不能再进行随意切换了，也就是说，一个事务对应一个数据源。那么传统的Spring管理事务是放在Service业务层操作的，所以更换数据源的操作要放在这个操作之前进行。也就是切换数据源操作放在Controller层，可是这样操作会造成Controller层代码混乱的结果。故而想到的解决方案是将事务管理在数据持久 (Dao层) 开启，切换数据源的操作放在业务层进行操作，就可在事务开启之前顺利进行数据源切换，不会再出现切换失败了。

## 一个动态数据源DEMO

### 第一步：做增删改查

新建：

![图片](https://images-cdn.shimo.im/RxRGlP1vJGo5iCBe/image.png!thumbnail)

暂时先导入两个包

![图片](https://images-cdn.shimo.im/IUJLSK06iFA7ZdRe/image.png!thumbnail)

因为项目还需要用到aop、mybatis plus、druid，所以先把maven包导入

```
<!--aop-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<!--mybatis plus-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.0.1</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<!--druid-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```
注意，这里使用的数据库就是renren-fast项目的数据库，所以sql文件在renren-fast项目中找哈。
运行MyGenerator的main方法，然后会自动生成sys_user表的一些基本service等~，然后复制到项目中，效果如下：

![图片](https://images-cdn.shimo.im/eiFQtj01pKYONaBC/image.png!thumbnail)

因为是springboot项目，所以还要手动添加扫描mapper的注解，加到springboot的启动类上。

```
@MapperScan("com.example.mapper")
@SpringBootApplication
public class DatasourceDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DatasourceDemoApplication.class, args);
    }
}
```
把配置文件修改成yml格式application.yml。然后添加数据库的信息。我们测试一下生成的代码是否有用！
```
# DataSource Config
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/renren_fast
    username: root
    password: admin
```
编写测试类：
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class DatasourceDemoApplicationTests {
    @Autowired
    SysUserService userService;
    @Test
    public void contextLoads() {
        SysUser user = userService.getById(1);
        System.out.println(user.toString());
    }
}
```
测试contextLoads（）方法，这时候可以看到测试结果啦~~
![图片](https://images-cdn.shimo.im/pKe8UW7fLmEBtO0w/image.png!thumbnail)

以上表明：生成的代码没毛病，可以查数据库了！

### 第二步：集成动态数据源模块

首先我们先梳理一下集成逻辑。

```
步骤1，在spring boot中，增加多数据源的配置
步骤2，扩展Spring的AbstractRoutingDataSource抽象类，
AbstractRoutingDataSource中的抽象方法determineCurrentLookupKey是实现多数据
源的核心，并对该方法进行Override
步骤3，配置DataSource，指定数据源的信息
步骤4，通过注解，实现多数据源
步骤5、配置加上(exclude={DataSourceAutoConfiguration.class})
```
下面我们按照上面的步骤一步步集成代码！

步骤一、配置多数据源的信息

首先在yml配置文件中添加数据源的信息，这里我新建了一个renren_fast2的数据库，然后把sys_user表的id为1的记录的名称改为了admin222。因为我们用到的druid的连接池，所以按照格式编写链接信息：如下

```
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driverClassName: com.mysql.jdbc.Driver
    druid:
      first:
        url: jdbc:mysql://localhost:3306/renren_fast?allowMultiQueries=true&useUnicode=true&characterEncoding=UTF-8
        username: root
        password: admin
      second:
        url: jdbc:mysql://localhost:3306/renren_fast2?allowMultiQueries=true&useUnicode=true&characterEncoding=UTF-8
        username: root
        password: admin
```
新建一个datasource包。用于放数据源模块的代码。

然后写一个配置类DynamicDataSourceConfig ，配置多数据源的信息，生成两个数据源：

```
/**
 * 配置多数据源
 * @author chenshun
 * @email sunlightcs@gmail.com
 * @date 2017/8/19 0:41
 */
@Configuration
public class DynamicDataSourceConfig {
    @Bean
    @ConfigurationProperties("spring.datasource.druid.first")
    public DataSource firstDataSource(){
        return DruidDataSourceBuilder.create().build();
    }
    @Bean
    @ConfigurationProperties("spring.datasource.druid.second")
    public DataSource secondDataSource(){
        return DruidDataSourceBuilder.create().build();
    }
}
```
另外，第五步骤中，其实现在就可以去做了，因为我们数据源是自己生成的，所以要去掉原先springboot启动时候自动装配的数据源配置。为了方便记忆，我们这里也导入自己写的配置。
```
@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
@Import({DynamicDataSourceConfig.class})
```
致此，第一步完成！效果：
![图片](https://images-cdn.shimo.im/6OvUkazkmQsXMDMR/image.png!thumbnail)

步骤二、扩展Spring的AbstractRoutingDataSource抽象类，重写lookupkey方法

```
/**
 * 动态数据源
 * determineCurrentLookupKey()决定使用哪个数据源
 */
public class DynamicDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return null;
    }
}
```
为了方便我们使用aop注解时候的得到的lookupkey参数能传递到这里。所以需要建一个线程安全的ThreadLocal变量，用于传参，避免复杂的参数传递过程。那么，我们获取这个lookupkey就从这个ThreadLocal里面去获取，所以determineCurrentLookupKey()直接返回getDataSource()。

```
/**
 * 动态数据源
 * determineCurrentLookupKey()决定使用哪个数据源
 */
public class DynamicDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return getDataSource();
    }
    
    /*
     *ThreadLocal 用于提供线程局部变量，在多线程环境可以保证各个线程里的变量独立于其它线程里的变量。
     * 也就是说 ThreadLocal 可以为每个线程创建一个【单独的变量副本】
     * 相当于线程的 private static 类型变量。
     */
    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();
    
    public static void setDataSource(String dataSource) {
        contextHolder.set(dataSource);
    }
    public static String getDataSource() {
        return contextHolder.get();
    }
    public static void clearDataSource() {
        contextHolder.remove();
    }
}
```
但是上面步骤还不够，因为自动装载数据源的这个过程我们在前面已经去掉了，所以需要我们自己手动去装配数据源的信息。调用determineCurrentLookupKey()方法的determineTargetDataSource()方法可以看到，里面有用到一些变量，比如resolvedDataSources，它是一个map，查看调用地方可以看到，它的初始化实在afterPropertiesSet()方法中，这初始化方法会用到一些变量，比如：targetDataSources、defaultTargetDataSource，然后你回发现，这个两个参数，都是一个set方法初始化的，setTargetDataSources(Map<Object, Object> targetDataSources)、setDefaultTargetDataSource(Object defaultTargetDataSource)，这两个就是所有数据源信息，和默认数据源信息，所以我们需要手动把我们配置的数据源信息set进去。
![图片](https://images-cdn.shimo.im/wVwD7D4865Yac743/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/gMAiX78TONkJJJWi/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/wwCAOSGA5vknTG51/image.png!thumbnail)

方法很多，因为涉及到一个顺序问题，调用determineCurrentLookupKey()前一定要把数据源的信息初始化化好，所以我们可以写一个DynamicDataSource的构造方法。两个参数，一个默认数据源，一个所有数据源。然后调用afterPropertiesSet()，初始化必要的参数。

```
/**
 * 决定使用哪个数据源之前需要把多个数据源的信息以及默认数据源信息配置好
 * @param defaultTargetDataSource
 * @param targetDataSources
 */
public DynamicDataSource(DataSource defaultTargetDataSource, Map<Object, Object> targetDataSources) {
    super.setDefaultTargetDataSource(defaultTargetDataSource);
    super.setTargetDataSources(targetDataSources);
    super.afterPropertiesSet();
}
```
因为这数据源的信息启动时候就需要初始化，因为后面事务等类的初始化都需要依赖数据源bean，所以在DynamicDataSourceConfig配置中，我们生成一个DynamicDataSource的bean。
```
@Bean
@Primary
public DynamicDataSource dataSource(DataSource firstDataSource, DataSource secondDataSource) {
    Map<Object, Object> targetDataSources = new HashMap<>();
    targetDataSources.put(DataSourceNames.FIRST, firstDataSource);
    targetDataSources.put(DataSourceNames.SECOND, secondDataSource);
    return new DynamicDataSource(firstDataSource, targetDataSources);
}
```
```
@Primary 优先考虑，优先考虑被注解的对象注入。
```
因为数据源的名称在我们后面注解的时候经常会用到，所以我们作为一个enum常亮来用。
```
/**
 * 增加多数据源，在此配置
 *
 * @author chenshun
 * @email sunlightcs@gmail.com
 * @date 2017/8/18 23:46
 */
public interface DataSourceNames {
    String FIRST = "first";
    String SECOND = "second";
}
```
通过上面的配置，我们可以说已经完成了手动装配我们自定义的多数据源的过程了。接下来的工作就是我们自己去指定ThreadLocal里面的值就行。
我们采用aop的方式，在需要修改数据源的地方使用注解方式去切换，然后切面修改ThreadLocal的内容。这里比较简单，我就直接贴代码：

**注解：**

```
/**
 * 多数据源注解
 * @author chenshun
 * @email sunlightcs@gmail.com
 * @date 2017/9/16 22:16
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DataSource {
    String name() default "";
}
```
**切面处理逻辑：**
```
/**
 * 多数据源，切面处理类
 * @author chenshun
 * @email sunlightcs@gmail.com
 * @date 2017/9/16 22:20
 */
@Aspect
@Component
public class DataSourceAspect implements Ordered {
    protected Logger logger = LoggerFactory.getLogger(getClass());
    @Pointcut("@annotation(com.example.datasource.DataSource)")
    public void dataSourcePointCut() {
    }
    @Around("dataSourcePointCut()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        MethodSignature signature = (MethodSignature) point.getSignature();
        Method method = signature.getMethod();
        DataSource ds = method.getAnnotation(DataSource.class);
        if(ds == null){
            DynamicDataSource.setDataSource(DataSourceNames.FIRST);
            logger.debug("set datasource is " + DataSourceNames.FIRST);
        }else {
            DynamicDataSource.setDataSource(ds.name());
            logger.debug("set datasource is " + ds.name());
        }
        try {
            return point.proceed();
        } finally {
            DynamicDataSource.clearDataSource();
            logger.debug("clean datasource");
        }
    }
    @Override
    public int getOrder() {
        return 1;
    }
}
```
![图片](https://images-cdn.shimo.im/7F9C0PRb4pYZ0I1S/image.png!thumbnail)

致此，完成了我们需要的所有准备工作，接下来我们只需要测试一下即可~

测试：

service中定义两个查询，分别查两个数据库：

```
public interface SysUserService extends IService<SysUser> {
    SysUser findUserByFirstDb(long id);
    SysUser findUserBySecondDb(long id);
    
}
```
实现类：因为默认是一，所以不用注解，数据库二需要添加注解@DataSource(name = DataSourceNames.SECOND)。
```
@Service
public class SysUserServiceImpl extends ServiceImpl<SysUserMapper, SysUser> implements SysUserService {
    
    @Override
    public SysUser findUserByFirstDb(long id) {
        return this.baseMapper.selectById(id);
    }
    @DataSource(name = DataSourceNames.SECOND)
    @Override
    public SysUser findUserBySecondDb(long id) {
        return this.baseMapper.selectById(id);
    }
}
```
测试方法：test()

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class DatasourceDemoApplicationTests {
    @Autowired
    SysUserService userService;
    @Test
    public void contextLoads() {
        SysUser user = userService.getById(1);
        System.out.println(user.toString());
    }
    @Test
    public void test() {
        SysUser user = userService.findUserByFirstDb(1);
        System.out.println("第one个数据库---------》" + user.toString());

        SysUser user2 = userService.findUserBySecondDb(1);
        System.out.println("第二个数据库---------》" + user2.toString());
    }
}
```
运行结果：
![图片](https://images-cdn.shimo.im/ouRhGJJdm9IOZ8Az/image.png!thumbnail)

ok，大功告成~~~~

