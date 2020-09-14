---
title: springboot自动装配与运行原理
date: 2020-09-14 14:50:04
tags: springboot框架
excerpt: 转载3、springboot自动装配与运行原理
---

[_课程3_spring装配方式.xmind](https://uploader.shimo.im/f/Kk4BfgEnmX8MnDqL.xmind)[【课程3】0、springboot（预习）.xmind](https://uploader.shimo.im/f/IO54Sa2unmADIR7e.xmind)[_课程3_springboot的启动过程.xmind](https://uploader.shimo.im/f/j6VyKprfz3ERamIW.xmind)[_课程3_springboot基本概念.xmind](https://uploader.shimo.im/f/RpMZAt66nwYdZm7J.xmind)[_课程3_springboot基本概念.xmind](https://uploader.shimo.im/f/RpMZAt66nwYdZm7J.xmind)

程代码（请收藏，以后的代码都上传到这里）：


* [https://gitee.com/lv-success/git-third](https://gitee.com/lv-success/git-third)
## 什么是spring boot

其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。

以前在写spring项目的时候，要配置各种xml文件。特别是做这种框架集成，比如说ssm框架，就需要配置一堆的xml，还经常会出问题，这样就降低了开发效率。

[_课程3_springboot的启动过程.xmind](https://uploader.shimo.im/f/j6VyKprfz3ERamIW.xmind)

着spring3，spring4的相继推出，约定大于配置逐渐成为了开发者的共识，大家也渐渐的从写xml转为各种注解的形式，在spring4的项目里，你甚至可以一行xml都不写。

但虽然spring4已经可以做到无xml，写一个大项目需要导入很多的第三方包，maven配置要写几百行，这是一件很可怕的事。为了简化我们的配置，快速集成常用的第三方框架，就有了springboot的诞生。

那么，spring boot可以做什么呢？

### Spring Boot的功能

Spring Boot实现了自动配置，降低了项目搭建的复杂度。

他本身并不提供Spring框架的核心特性以及扩展功能，只是用于快速、敏捷地开发新的基于Spring框架的应用程序。也就是说，它并不是用来替代Spring的解决方案，而是和Spring框架紧密结合用于提升Spring开发者体验的工具。同时呢，因为它集成了大量常用的第三方库配置(例如JDBC, Mongodb, Redis, Mail，rabbitmq等等)，所以在Spring Boot应用中这些第三方库几乎可以零配置的开箱即用(out-of-the-box)，大部分的Spring Boot应用都只需要非常少量的配置代码，开发者能够更加专注于业务逻辑。

所以说，Spring Boot只是承载者，辅助你简化项目搭建过程的。如果承载的是WEB项目，使用Spring MVC作为MVC框架，那么工作流程和spring mvc是完全一样的，因为这部分工作是Spring MVC做的而不是Spring Boot。

## springboot与springmvc的关系

### 二者区别

所以说我们谈到springboot与spring mvc的关系区别的时候，有这么一句总结：

Spring 是一个“引擎”；

springmvc是框架,web项目中实际运行的代码；

spring boot只是一个配置工具,整合工具,辅助工具。是一套快速开发整合包。

Spring Boot ：J2EE一站式解决方案

Spring Cloud ： 分布式整体解决方案

Spring 最初利用“工厂模式”（DI）和“代理模式”（AOP）解耦应用组件。大家觉得挺好用，于是按照这种模式搞了一个 MVC框架（一些用Spring 解耦的组件），用开发 web 应用（ SpringMVC ）。然后发现每次开发都写很多样板代码，为了简化工作流程，于是开发出了一些“懒人整合包”（starter），这套就是 Spring Boot。

## springboot与微服务

为什么说springboot与我们的微服务有关系呢？可以来看下springboot的一些特性~

### Spring Boot 特性


* 使用 Spring 项目引导页面可以在几秒构建一个项目
* 方便对外输出各种形式的服务，如 REST API、WebSocket、Web、Streaming、Tasks
* 非常简洁的安全策略集成
* 支持关系数据库和非关系数据库
* 支持运行期内嵌容器，如 Tomcat、Jetty
* 强大的开发包，支持热启动
* 自动管理依赖
* 自带应用监控
* 支持各种 IED，如 IntelliJ IDEA 、NetBeans

正是因为Spring Boot 的这些特性非常方便、快速构建独立的微服务。所以我们使用 Spring Boot 开发项目，会给我们传统开发带来非常大的便利度，可以说如果你使用过 Spring Boot 开发过项目，就不会再愿意以以前的方式去开发项目了。

总结一下，使用 Spring Boot 至少可以给我们带来以下几方面的改进：


* Spring Boot 使编码变简单，Spring Boot 提供了丰富的解决方案，快速集成各种解决方案提升开发效率。
* Spring Boot 使配置变简单，Spring Boot 提供了丰富的 Starters，集成主流开源产品往往只需要简单的配置即可。
* Spring Boot 使部署变简单，Spring Boot 本身内嵌启动容器，仅仅需要一个命令即可启动项目，结合 Jenkins 、Docker 自动化运维非常容易实现。
* Spring Boot 使监控变简单，Spring Boot 自带监控组件，使用 Actuator 轻松监控服务各项状态。

总结，Spring Boot 是 Java 领域最优秀的微服务架构落地技术，没有之一。

## springboot的三大特性

组件自动装配：web mvc、jdbc、MongoDB

嵌入式web容器：tomcat、jetty

生产准备特性：指标、健康检查、外部化配置


* 不影响功能开发，但是有指导意义
* 应用是否监控的，微服务是否健康
* 外部化配置：如server.port，可以改变服务端口等，以前在服务器上改端口
## 初识springboot

### 整合mybatis plus

**步骤1：**

首先我们来看下mybatis plus的官网并且找到整合包：[https://mp.baomidou.com/guide/install.html](https://mp.baomidou.com/guide/install.html)

官网中提示springboot的集成包：

```
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.1.0</version>
</dependency>
```
我们在用idea创建了一个springboot项目之后，把mybatis plus导入pom中。
然后在application.yml中写入我们的数据源链接信息：

```
# DataSource Config
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/homework?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=UTC
    username: root
    password: admin
```
好了，我们的mybatis plus已经集成成功。
**步骤2：**

我们借用官网给我们提供的代码生成工具：[https://mp.baomidou.com/guide/generator.html](https://mp.baomidou.com/guide/generator.html)

注意需要导入代码生成工具类的jar包，还有模板引擎相关的包：

```
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```
接着就是修改我们的代码生成器的相关配置，具体修改如下：
```
// 演示例子，执行 main 方法控制台输入模块表名回车自动生成对应项目目录中
public class CodeGenerator {
    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }
    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();
        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
//        gc.setOutputDir("D:\\test");
        gc.setAuthor("lv-success");
        gc.setOpen(false);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        gc.setServiceName("%sService");
        mpg.setGlobalConfig(gc);
        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/homework?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=UTC");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("admin");
        mpg.setDataSource(dsc);
        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(null);
        pc.setParent("com.example");
        mpg.setPackageInfo(pc);
        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };
        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";
        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/src/main/resources/mapper/" + pc.getModuleName()
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);
        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();
        // 配置自定义输出模板
        //指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();
        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);
        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setSuperEntityClass("com.example.entity.BaseEntity");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        strategy.setSuperControllerClass("com.example.controller.BaseController");
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setSuperEntityColumns("id", "created", "modified", "status");
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }
}
```
注意安装配置来生成BaseEntity、BaseController、还有包路径相关等东西。
## spring的手动装配

### spring的模式注解装配

@Component注解的拓展，“派生注解”-->@Controller、@Service、@Repository、分别和控制层（Web 层）、业务层和持久层相对应...

模式注解的层次性

![图片](https://images-cdn.shimo.im/78Tz8Bs4T00xE8fV/image.png!thumbnail)

### spring的@Enable模块装配

激活：@EnableAutoConfiguration

配置：/META-INF/spring.factoryes

实现：XXXAutoConfiguration

**框架实现**

@Enable 注解模块 激活模块

**Spring Framework**

@EnableWebMvc     Web MVC 模块

@EnableTransaction    Management 事务管理模块

@EnableCaching     Caching 模块

@EnableMBeanExport   JMX 模块

@EnableAsync   异步处理模块

@EnableWebFlux   Web Flux 模块

@EnableAspectJAutoProxy   AspectJ 代理模块



**Spring Boot**

@EnableAutoConfiguration   自动装配模块

@EnableManagementContext   Actuator 管理模块

@EnableConfigurationProperties   配置属性绑定模块

@EnableOAuth2Sso   OAuth2 单点登录模块



**Spring Cloud**

@EnableEurekaServer   Eureka服务器模块

@EnableConfigServer   配置服务器模块

@EnableFeignClients  Feign客户端模块

@EnableZuulProxy 服务网关   Zuul 模块

@EnableCircuitBreaker   服务熔断模块

### 自定义@Enable模块

**1、基于注解驱动实现，可参考@EnableWebMvc注解**

***第一步***：自定义@EnableXXX注解，其中@Import表示把用到的资源导入到当前容器中，相当于导入一个@Configuration配置类

```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({SayHelloWorldConfiguration.class})
public @interface EnableSayHelloWorld {
}
```
***第二步***、编写配置类，配置类中把需要的bean注入到spring容器中，因为已经有了第一步的@Import，所以这里不再需要@Configuration注解。
```
//@Configuration
public class SayHelloWorldConfiguration {
    @Bean
    SayHelloWorld sayHelloWorld() {
        System.out.println("here to loading bean sayhelloworld!");
        return new SayHelloWorld();
    }
}
```
```
//需要初始化的bean
public class SayHelloWorld {
    public String say() {
        return "hello world";
    }
}
```
***第三步***、在启动项目时候加上@EnableSayHelloWorld注解，表示加载自定义的配置。@EnableSayHelloWorld是@Import的拓展注解，实际起作用的是@Import注解。
```
@EnableSayHelloWorld
@SpringBootApplication
public class SpringBootDeepinApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootDeepinApplication.class, args);
    }
}
```
**2、基于接口驱动实现，参考@EnableCaching注解**

相比注解形式，接口驱动实现更加灵活，可以添加分支或根据条件判断来初始化程序。

大概步骤：

HelloWorldImportSelector -> HelloWorldConfiguration -> HelloWorld

***第一步、***自定义注解@EnableSeletorHelloWorld

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({HelloWorldImportSeletor.class})
public @interface EnableSeletorHelloWorld {
    String model() default "first";
}
```
***第二步、***编写Seletor，实现ImportSeletor这个借口，并重写SelectImports方法，返回的数组表示需要初始化注入到spring容器的中的bean名称。
```
public class HelloWorldImportSeletor implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        //获取注解上的属性的值
        Map<String, Object> annotationAttributes = annotationMetadata.getAnnotationAttributes(EnableSeletorHelloWorld.class.getName());
        String model = (String) annotationAttributes.get("model");
        System.out.println(model);
        //可以返回多个加载的配置或bean
        return new String[]{SayHelloWorldConfiguration.class.getName()};
    }
}
```
其余与注解形式相同，这里略~
### spring条件装配

定义：Bean装配的前置条件

举例：@Profile、@Conditional

**1、注解方式**

@Profile 配置条件装配

略~

**2、编程方式**

@Conditional 编程条件装配，

可参考org.springframework.boot.autoconfigure.condition.ConditionalOnProperty

***第一步***、编写条件注解。用于判断系统是否是linux，当linux的时候初始化对应的配置。

```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@Conditional({OnSystemPropertyCondition.class})
public @interface ConditionalOnSystemProperty {
    String value();
}
```
第二步、编写条件判断类OnSystemPropertyCondition，实现org.springframework.context.annotation.Condition接口，重写match方法用于判断条件返回true或false；
```
public class OnSystemPropertyCondition implements Condition {
    /**
     * 判断是否满足条件
     * @param conditionContext
     * @param annotatedTypeMetadata 注解的元信息
     * @return
     */
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Map<String, Object> attrs = annotatedTypeMetadata.getAnnotationAttributes(ConditionalOnSystemProperty.class.getName());
        String system = String.valueOf(attrs.get("value"));
        String currentOs = System.getProperty("os.name");
        return currentOs.endsWith(system);
    }
}
```
第三步、在需要做判断才能初始化的bean上添加此注解即可完成条件装配
```
@ConditionalOnSystemProperty(value = "linux")
```
## springboot的自动装配模式

### Java SPI机制

**Java SPI是什么？**

SPI的全名为Service Provider Interface.大多数开发人员可能不熟悉，因为这个是针对厂商或者插件的。

我们系统里抽象的各个模块，往往有很多不同的实现方案，比如日志模块的方案，xml解析模块、jdbc模块的方案等。

面向的对象的设计里，我们一般推荐模块之间基于接口编程，模块之间不对实现类进行硬编码。一旦代码里涉及具体的实现类，就违反了可拔插的原则，如果需要替换一种实现，就需要修改代码。为了实现在模块装配的时候能不在程序里动态指明，这就需要一种服务发现机制。

Java SPI就是提供这样的一个机制：为某个接口寻×××实现的机制。

**Java SPI的约定**

当服务的提供者，提供了服务接口的一种实现之后，在jar包的META-INF/services/目录里同时创建一个以服务接口命名的文件。

该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。

基于这样一个约定就能很好的找到服务接口的实现类，而不需要再代码里制定。

**SpringBoot中的SPI机制**

在Spring中也有一种类似与Java SPI的加载机制。它在META-INF/spring.factories文件中配置接口的实现类名称，然后在程序中读取这些配置文件并实例化。

这种自定义的SPI机制是Spring Boot Starter实现的基础。

### 实现原则

**理念**：约定大于配置原则

**装配**：模式装配、@Enable注解装配、条件装配

**步骤**：激活自动装配、实现自动装配、配置自动装配

starter：启动器

### 底层装配技术


* Spring 模式注解装配
* Spring @Enable 模块装配
* Spring 条件装配装配
* Spring 工厂加载机制
    * 实现类： SpringFactoriesLoader （类似与Java SPI机制的一个加载类）
    * 配置资源： META-INF/spring.factories

**加载类SpringFactoriesLoader说明：**

spring的工厂加载类SpringFactoriesLoader，用于加载自动配置，主要方法loadFactories()，参数factoryClass表示要加载的工厂配置类，classLoader表示类加载器。

**配置文件META-INF/spring.factories说明：**

![图片](https://images-cdn.shimo.im/BFdpDt5yFFY7xpTC/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/mP2NuG2zK8gOB77J/image.png!thumbnail)

### 自定义自动条件装配

步骤：


* HelloWorldAutoConfiguration
* 条件判断： user.name == "Mercy"
* 模式注解： @Configuration @Enable 模块： @EnableHelloWorld -> HelloWorldImportSelector -> HelloWorldConfiguration - > helloWorld

查看这个类WebMvcAutoConfiguration，可以看到这个配置类就用了条件装配。

```
@Configuration
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
@ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class})
@ConditionalOnMissingBean({WebMvcConfigurationSupport.class})
@AutoConfigureOrder(-2147483638)
@AutoConfigureAfter({DispatcherServletAutoConfiguration.class, ValidationAutoConfiguration.class})
public class WebMvcAutoConfiguration {
 ...
}
```
@ConditionalOnMissingBean表示当容器中没有这个bean的时候
@ConditionalOnClass表示对应的类在classpath目录下存在时

@ConditionalOnWebApplication表示当前项目是个servlet项目的时候

***第一步***、编写需要自动装载的配置类，因为@Configuration注解的存在，因此我把需要自动装载的配置类放在了springboot扫描不到的包上。

说明：

@Configuration表示是个配置类

@ConditionalOnSystemProperty表示需要满足当前系统是win10系统

@EnableSeletorHelloWorld表示启动这个模块的加载

```
@Configuration
@ConditionalOnSystemProperty(value = "Windows 10")
@EnableSeletorHelloWorld
public class SayHelloWorldAutoConfiguration {
    @Bean
    SayHelloWorld autoSayHelloWorld() {
        System.out.println("here to ！！auto！！ loading bean autoSayHelloWorld!");
        return new SayHelloWorld();
    }
}
```
***第二步***、在resources目录下新建META-INF文件夹，编写spring.factories。
```
# Auto Configure 自动装配自定义的配置
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.other.configuration.SayHelloWorldAutoConfiguration
```
启动springboot之后就会自动加载这个配置类~

## springboot源码分析

源码分析系列的文章有很多，同学们找些来看看，有些都比较详细了，这里我就不写这么多了。


* 张书康 - SpringBoot源码深入解析（专栏）

[https://blog.csdn.net/woshilijiuyi/column/info/26175](https://blog.csdn.net/woshilijiuyi/column/info/26175)


* Java技术栈 -Spring Boot 2.x 启动全过程源码分析（全）

·[https://cloud.tencent.com/developer/article/1189168](https://cloud.tencent.com/developer/article/1189168)

同时我在我们的演示项目中已经给一些比较重点的代码写了一些注释，大家在学源码分析的过程中可以学习下这种方式，把这个类单独拷贝一份出来，然后写上注释。

![图片](https://uploader.shimo.im/f/oUX9Nk9MJZQjCpWH.png!thumbnail)

### main方法解释

springboot项目的启动一般使用Java的main方法来启动项目，代码里面有个run方法，使用SpringBootDeepinApplication来作为参数，为什么要使用SpringBootDeepinApplication来作为参数，因为@SpringBootApplication也是@Configuration拓展注解，所以初始化的时候其实就是最先加载这个配置类，然后根据注解拓展开来加载各个模块。


* 所以，参数不一定是主类。只要是有@SpringBootApplication注解的类都可以成为参数
```
@SpringBootApplication
public class SpringBootDeepinApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootDeepinApplication.class, args);
    }
}
```
### boot的准备阶段

类加载实现思路：


1. 实现类
2. 配置资源META-INF/spring.factories
3. 排序：@AutoConfigureOrder，@Ordered，实现Ordered接口等方式

* 加载spring.favorites的自动配置
* 加载应用监听器

首先SpringApplication使用主类作为配置参数初始化并加载资源。

```
SpringApplication application = new SpringApplication(SpringBootDeepinApplication.class);
```
![图片](https://images-cdn.shimo.im/qq9nQT4hz4o2LKAC/image.png!thumbnail)

每个模块中，只要classpath中有spring.favorites就可以装载有这个两个配置类的资源。

```
#加载Spring.favorites中ApplicationContextInitializer的配置类
this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
#加载Spring.favorites中ApplicationListener的配置类
this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
```
![图片](https://images-cdn.shimo.im/QVIH5XwRuhcNfVMo/image.png!thumbnail)

关键代码：

```
#配置加载器，包括注解配置或xml配置
BeanDefinitionLoader
```
![图片](https://images-cdn.shimo.im/K8s2ikHJ6R8szIOz/image.png!thumbnail)

### boot的运行阶段

SpringApplication的run方法就是boot的运行阶段的，需要初始化环境，context，发布事件，启动监听器等等。

![图片](https://images-cdn.shimo.im/9Clt3v5LT1o9cFA9/image.png!thumbnail)

refresh()方法做了很多核心工作比如BeanFactory的设置，BeanFactoryPostProcessor接口的执行、BeanPostProcessor接口的执行、自动化配置类的解析、spring.factories的加载、bean的实例化、条件注解的解析、国际化的初始化等等。

