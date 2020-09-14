---
title: renren-fast脚手架学习（上）
date: 2020-09-14 14:55:04
tags: renren-fast
excerpt: 转载8、renren-fast脚手架学习（上）
---

## [5_jwt机制.xmind](https://uploader.shimo.im/f/aP4BEprUKa8dVuTc.xmind)[6_动态数据源.xmind](https://uploader.shimo.im/f/h1L8tkHSVEALcsnt.xmind)[课程8、renren-fast项目解读 （预习）.xmind](https://uploader.shimo.im/f/3o4j1434AUwXnL61.xmind)[课程8_renren-fast项目解读_预习_.xmind](https://uploader.shimo.im/f/3Do0DbqCPcMkgAE4.xmind)[1_renren-fast项目.xmind](https://uploader.shimo.im/f/lUDXFVKHsE8BVtWM.xmind)[2_自定义异常与redis开关.xmind](https://uploader.shimo.im/f/SFTWgnhyYdASGwdZ.xmind)[3_日志处理与数据校验模块.xmind](https://uploader.shimo.im/f/kq9mQSb0TPUNO36X.xmind)[4_token机制.xmind](https://uploader.shimo.im/f/zlwn8t4g5Ic0yuiS.xmind)项目介绍


* 官方网址：[http://www.renren.io/](http://www.renren.io/)
* 项目演示地址：[http://fast.demo.renren.io/#/login](http://fast.demo.renren.io/#/login)账号密码admin/admin
* renren-fast是一个轻量级的 Spring Boot 快速开发平台，能快速开发项目并交付【接私活利器】
* 完善的 XSS 防范及脚本过滤，彻底杜绝 XSS 攻击
* 实现前后端分离，通过 token 进行数据

后端：[https://gitee.com/renrenio/renren-fast](https://gitee.com/renrenio/renren-fast)

前端：[https://gitee.com/renrenio/renren-fast-vue](https://gitee.com/renrenio/renren-fast-vue)

## 技术选型

后端


* spring boot
* mybatis plus
* shiro
* swagger2
* redis
* mysql

前段


* vue.js
* element-ui
## 前端部署

由于前端使用vue开发，因此需要安装node.js环境。

node.js安装教程：[http://nodejs.cn/download/](http://nodejs.cn/download/)下载msi版本安装。

安装之后，命令行窗口，表示安装成功。

![图片](https://uploader.shimo.im/f/xTyWqz8xpac8Qz8t.png!thumbnail)

启动步骤：

```
# 克隆项目
git clone https://github.com/daxiongYang/renren-fast-vue.git
# 切换到项目目录根目录renren-fast-vue里面
# 1、安装淘宝镜像依赖
npm install -g cnpm --registry=https://registry.npm.taobao.org
# 2、安装项目依赖
cnpm install
# 启动服务
npm run dev
```
## 系统日志

系统管理员的有很多，操作权限也不同，但每个人的操作都需要有个操作记录才能跟踪每个管理员的操作，便于排查事故。

### 关键类


* io.renren.common.annotation.SysLog（日志注解）
* io.renren.common.aspect.SysLogAspect（系统日志，切面处理类）
### **项目逻辑**


* 其实Spring Aop切面编程的使用。
* 自定义方法级别的注解SysLog，在需要说明操作日志的方法上添加此注解，并说明操作的意义。
```
@SysLog("保存菜单")
```
* 同时编写切面处理类SysLogAspect，使用SysLog作为切入点，监测注解的方法。并把操作内容保存到数据库中。**操作内容包括：调用的方法，参数，时间点，操作用户，操作ip地址**等~
### 总结-自定义注解步骤

第一步、首先新建一个注解

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface SysLog {
   String value() default "";
}
```
第二步、新建一个切面处理类
```
@Aspect
@Component
public class SysLogAspect {
}
```
第三步、定切点在注解上，切面需要处理的拦截点
```
@Pointcut("@annotation(io.renren.common.annotation.SysLog)")
public void logPointCut() { 
   
}
```
第四步、定义切点的环绕处理方法、如前置、后置、环绕处理
```
@Around("logPointCut()")
public Object around(ProceedingJoinPoint point) throws Throwable {
。   
   ...
   return result;
}
```
## 自定义异常

### **自定义异常类RRException**


* io.renren.common.exception.RRException（自定义异常）
* io.renren.common.exception.RRExceptionHandler（异常处理类）

系统自带的，系统自己处理，但是很多时候项目会出现特有问题，而这些问题并未被java所描述并封装成对象，所以对于这些特有的问题可以按照java的对问题封装的思想，将特有的问题进行自定义异常封装。在Java中要想创建自定义异常，需要继承Throwable或者他的子Exception。

```
class  自定义异常类 extends 异常类型(RuntimeException){
 // 因为父类已经把异常信息的操作都完成了，所在子类只要在构造时，将异常信息传递给父类通过super 语句即可。
  // 重写 有参 和 无参  构造方法
}
```
### **spring boot统一异常处理**

***主要注解：@ControllerAdvice、@ExceptionHandler***

全局异常处理@ControllerAdvice。

添加@ControllerAdvice注解的类是集中处理异常的地方，可以同时存在多个这样的类，用来做更细粒度的划分。

在这个类中，我们可以对每一种异常编写一种处理逻辑，在方法上使用@ExceptionHandler注解修饰，传入指定的异常类型即可。

如果是RESTful风格，不返回视图，也可使用@RestControllerAdvice

@RestControllerAdvice是一个组合注解，组合了@ControllerAdvice、@ResponseBody

![图片](https://uploader.shimo.im/f/hKkMcC7T91Qj71fb.png!thumbnail)

## 数据校验

后端实体效验使用的是Hibernate Validator校验框架。

且自定义ValidatorUtils工具类，用来效验数据。


* io.renren.common.validator.ValidatorUtils（校验工具）
* io.renren.common.validator.Assert（自定义的断言，用于抛出自定义的RRException）

![图片](https://images-cdn.shimo.im/twDJqCc4MTUwDVzL/image.png!thumbnail)

通过上面的实体类代码，我们来理解Hibernate Validator校验框架的使用。

其中，username属性，表示保存或修改用户时，都会效验username属性； 而password属

性，表示只有保存用户时，才会效验password属性，也就是说，修改用户时，password可以

不填写，允许为空。

如果不指定属性的groups，则默认属于javax.validation.groups.Default.class分组，可以通

过ValidatorUtils.validateEntity(user)进行效验。

校验分组


* io.renren.common.validator.group.AddGroup
* io.renren.common.validator.group.AliyunGroup
* io.renren.common.validator.group.Group
* io.renren.common.validator.group.QcloudGroup
* io.renren.common.validator.group.QiniuGroup
* io.renren.common.validator.group.UpdateGroup
### **项目逻辑**


* 1、首先定义分组，根据实际情况，可以分为添加组AddGroup和修改组UpdateGroup等。
* 2、在实体上添加hibernate.validator规则注解@NotBlank、@Email等，并分组。
* 3、编写规则校验工具类ValidatorUtils。校验出有不符合规则的内容抛出自定义异常RRException
* 4、再保存、更新等操作中使用ValidatorUtils.validateEntity(user, AddGroup.class);校验实体规则情况。
### **常用注解**

```
Bean Validation 中内置的 constraint         
@Null   被注释的元素必须为 null    
@NotNull    被注释的元素必须不为 null    
@AssertTrue     被注释的元素必须为 true    
@AssertFalse    被注释的元素必须为 false    
@Min(value)     被注释的元素必须是一个数字，其值必须大于等于指定的最小值    
@Max(value)     被注释的元素必须是一个数字，其值必须小于等于指定的最大值    
@DecimalMin(value)  被注释的元素必须是一个数字，其值必须大于等于指定的最小值    
@DecimalMax(value)  被注释的元素必须是一个数字，其值必须小于等于指定的最大值    
@Size(max=, min=)   被注释的元素的大小必须在指定的范围内    
@Digits (integer, fraction)     被注释的元素必须是一个数字，其值必须在可接受的范围内    
@Past   被注释的元素必须是一个过去的日期    
@Future     被注释的元素必须是一个将来的日期    
@Pattern(regex=,flag=)  被注释的元素必须符合指定的正则表达式    
    
Hibernate Validator 附加的 constraint    
@NotBlank(message =)   验证字符串非null，且长度必须大于0    
@Email  被注释的元素必须是电子邮箱地址    
@Length(min=,max=)  被注释的字符串的大小必须在指定的范围内    
@NotEmpty   被注释的字符串的必须非空    
@Range(min=,max=,message=)  被注释的元素必须在合适的范围内 
```
## redis的使用

### **renren官方给出的redis使用原则**

1. 查询数据的时候，尽量减少DB查询，DB主要负责写数

2. 尽量不使用 LEFt JOIN 等关联查询，缓存命中率不高，还浪费内存

3. 多使用单表查询，缓存命中率最高

4. 数据库 insert 、 update 、 delete 时，同步更新缓存数据

5. 合理运用Redis数据结构，也许有质的飞跃

6. 对于访问量不大的项目，使用缓存只会增加项目的复杂性。

Redis切面处理类RedisAspect，使用切面定义是否启用redis缓存。

### **redis开关原理**

ProceedingJoinPoint：用于环绕通知，使用proceed()方法来执行目标方法，当不打开redis缓存时候，跳过RedisUtils的所有方法执行。


* io.renren.common.utils.RedisUtils，redis操作的工具类
* io.renren.modules.sys.redis.SysConfigRedis，系统配置缓存操作的工具类
* io.renren.common.aspect.RedisAspect
## Token机制

## JWT

### 概要

JWT是一种用于双方之间传递安全信息的简洁的、URL安全的表述性声明规范。JWT定义了一种简洁的，自包含的方法用于通信双方之间以Json对象的形式安全的传递信息。因为数字签名的存在，这些信息是可信的，JWT可以使用HMAC算法或者是RSA的公私秘钥对进行签名。


* **简洁(Compact)**: 可以通过URL，POST参数或者在HTTP header发送，因为数据量小，传输速度也很快
* **自包含(Self-contained)**：负载中包含了所有用户所需要的信息，避免了多次查询数据库
### 主要应用场景

身份认证在这种场景下，一旦用户完成了登陆，在接下来的每个请求中包含JWT，可以用来验证用户身份以及对路由，服务和资源的访问权限进行验证。

由于它的开销非常小，可以轻松的在不同域名的系统中传递，所有目前在**单点登录（SSO）**中比较广泛的使用了该技术。

信息交换在通信的双方之间使用JWT对数据进行编码是一种非常安全的方式，由于它的信息是经过签名的，可以确保发送者发送的信息是没有经过伪造的。

### jwt消息结构

一个token分3部分，按顺序为


* **头部（header)**
* **载荷（payload)**
* **签证（signature)**

由三部分生成token3部分之间用“**.**”号做分隔。

例如：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```
### 头部（header）

Jwt的头部承载两部分信息：


* 声明类型，这里是jwt
* 声明加密的算法 通常直接使用 HMAC SHA256
```
{ "alg": "HS256", "typ": "JWT"}
```
然后将头部进行base64加密（该加密是可以对称解密的),构成了第一部分：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```
### **载荷（payload)**

载荷就是存放有效信息的地方。这个名字像是特指飞机上承载的货品，这些有效信息包含三个部分


* **标准中注册的声明**
* **公共的声明**
* **私有的声明**

***payload-标准中注册的声明 (建议但不强制使用) ：***


* **iss**: jwt签发者
* **sub**: jwt所面向的用户
* **aud**: 接收jwt的一方
* **exp**: jwt的过期时间，这个过期时间必须要大于签发时间
* **nbf**: 定义在什么时间之前，该jwt都是不可用的.
* **iat**: jwt的签发时间
* **jti**: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。

***payload-公共的声明 ：***

公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密。

***payload-私有的声明 ：***

私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

定义一个payload：

```
{"name":"Free码农","age":"28","org":"今日头条"}
```
然后将其进行base64加密，得到Jwt的第二部分：
```
eyJvcmciOiLku4rml6XlpLTmnaEiLCJuYW1lIjoiRnJlZeeggeWGnCIsImV4cCI6MTUxNDM1NjEwMywiaWF0IjoxNTE0MzU2MDQzLCJhZ2UiOiIyOCJ9
```
### **签证（signature）**

jwt的第三部分是一个签证信息，这个签证信息由三部分组成：


* header (base64后的)
* payload (base64后的)
* secret

这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行加盐secret组合加密，然后就构成了jwt的第三部分：

```
49UF72vSkj-sA4aHHiYN5eoZ9Nb4w5Vb45PsLF7x_NY
```
密钥secret是保存在服务端的，服务端会根据这个密钥进行生成token和验证，所以需要保护好。
### 说明


* 在Web应用中，别再把JWT当做session使用，绝大多数情况下，传统的cookie-session机制工作得更好
* JWT适合一次性的命令认证，颁发一个有效期极短的JWT，即使暴露了危险也很小，由于每次操作都会生成新的JWT，因此也没必要保存JWT，真正实现无状态。
## JWT的代码实现

**第一步**、导入maven坐标

```
<dependency>
   <groupId>io.jsonwebtoken</groupId>
   <artifactId>jjwt</artifactId>
   <version>0.9.0</version>#renren-fast用的是0.7.0版本
</dependency>
```
**第二步**、封装一个util工具类统一头部和载荷部分的信息，应包含生成jwt和校验jwt。
io.renren.modules.app.utils.JwtUtils

```
@ConfigurationProperties(prefix = "renren.jwt")
@Component
public class JwtUtils {
    private Logger logger = LoggerFactory.getLogger(getClass());
    private String secret;
    private long expire;
    private String header;
    /**
     * 生成jwt token
     */
    public String generateToken(long userId) {
        Date nowDate = new Date();
        //过期时间
        Date expireDate = new Date(nowDate.getTime() + expire * 1000);
        return Jwts.builder()
                .setHeaderParam("typ", "JWT")
                .setSubject(userId+"")
                .setIssuedAt(nowDate)
                .setExpiration(expireDate)
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }
    public Claims getClaimByToken(String token) {
        try {
            return Jwts.parser()
                    .setSigningKey(secret)
                    .parseClaimsJws(token)
                    .getBody();
        }catch (Exception e){
            logger.debug("validate is token error ", e);
            return null;
        }
    }
    
    /**
     * token是否过期
     * @return  true：过期
     */
    public boolean isTokenExpired(Date expiration) {
        return expiration.before(new Date());
    }
    
    ....getter、setter
}
```
**第三步、**为了区分需要拦截和不需要拦截的资源，项目添加了一个@login注解
```
/**
 * app登录效验
 * @author chenshun
 * @email sunlightcs@gmail.com
 * @date 2017/9/23 14:30
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Login {
}
```
**第四步**、登录成功后，生成一个jwt的 token，用于返回给前段。

```
/**
 * 登录
 */
@PostMapping("login")
@ApiOperation("登录")
public R login(@RequestBody LoginForm form){
    //表单校验
    ValidatorUtils.validateEntity(form);
    //用户登录
    long userId = userService.login(form);
    //生成token
    String token = jwtUtils.generateToken(userId);
    Map<String, Object> map = new HashMap<>();
    map.put("token", token);
    map.put("expire", jwtUtils.getExpire());
    return R.ok(map);
}
```
**第五步、**编写一个拦截器，拦截所有需要校验的资源模块的url（有加了@login注解的），访问前校验jwt是否合法。
```
/**
 * 权限(Token)验证
 * @author chenshun
 * @email sunlightcs@gmail.com
 * @date 2017-03-23 15:38
 */
@Component
public class AuthorizationInterceptor extends HandlerInterceptorAdapter {
    @Autowired
    private JwtUtils jwtUtils;
    public static final String USER_KEY = "userId";
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Login annotation;
        if(handler instanceof HandlerMethod) {
            annotation = ((HandlerMethod) handler).getMethodAnnotation(Login.class);
        }else{
            return true;
        }
        if(annotation == null){
            return true;
        }
        //获取用户凭证
        String token = request.getHeader(jwtUtils.getHeader());
        if(StringUtils.isBlank(token)){
            token = request.getParameter(jwtUtils.getHeader());
        }
        //凭证为空
        if(StringUtils.isBlank(token)){
            throw new RRException(jwtUtils.getHeader() + "不能为空", HttpStatus.UNAUTHORIZED.value());
        }
        Claims claims = jwtUtils.getClaimByToken(token);
        if(claims == null || jwtUtils.isTokenExpired(claims.getExpiration())){
            throw new RRException(jwtUtils.getHeader() + "失效，请重新登录", HttpStatus.UNAUTHORIZED.value());
        }
        //设置userId到request里，后续根据userId，获取用户信息
        request.setAttribute(USER_KEY, Long.parseLong(claims.getSubject()));
        return true;
    }
}
```
### 获取用户信息

项目用了一个一种特殊的方法来获取用户信息，一般我们再baseController中获取用户信息，但renren-fast使用了注解的形式，@loginUser

```
/**
 * 登录用户信息
 *
 * @author chenshun
 * @email sunlightcs@gmail.com
 * @date 2017-03-23 20:39
 */
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}
```
运用是这样的，参数中添加@LoginUser UserEntity user作为参数：
```
@RestController
@RequestMapping("/app")
@Api("APP测试接口")
public class AppTestController {
    @Login
    @GetMapping("userInfo")
    @ApiOperation("获取用户信息")
    public R userInfo(@LoginUser UserEntity user){
        return R.ok().put("user", user);
    }
}
```
然后写一个全局解析注解的类：其中HandlerMethodArgumentResolver是用来为处理器解析参数的。
HandlerMethodArgumentResolver的接口定义如下：

（1）supportsParameter 用于判断是否支持对某种参数的解析

（2）resolveArgument  将请求中的参数值解析为某种对象

```
/**
 * 有@LoginUser注解的方法参数，注入当前登录用户
 * @author chenshun
 * @email sunlightcs@gmail.com
 * @date 2017-03-23 22:02
 */
@Component
public class LoginUserHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {
    @Autowired
    private UserService userService;
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.getParameterType().isAssignableFrom(UserEntity.class) && parameter.hasParameterAnnotation(LoginUser.class);
    }
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer container,
                                  NativeWebRequest request, WebDataBinderFactory factory) throws Exception {
        //获取用户ID
        Object object = request.getAttribute(AuthorizationInterceptor.USER_KEY, RequestAttributes.SCOPE_REQUEST);
        if(object == null){
            return null;
        }
        //获取用户信息
        UserEntity user = userService.selectById((Long)object);
        return user;
    }
}
```
最后在mvc配置中添加这个解析器
```
/**
 * MVC配置
 *
 * @author chenshun
 * @email sunlightcs@gmail.com
 * @date 2017-04-20 22:30
 */
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Autowired
    private LoginUserHandlerMethodArgumentResolver loginUserHandlerMethodArgumentResolver;
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(loginUserHandlerMethodArgumentResolver);
    }
}
```
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

步骤2，扩展Spring的AbstractRoutingDataSource抽象类，

AbstractRoutingDataSource中的抽象方法determineCurrentLookupKey是实现多数据

源的核心，并对该方法进行Override

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
然后先使用mybatis plus做一个增删改查例子。这里我们使用自动生成工具~
[MyGenerator.java](https://attachments-cdn.shimo.im/jT6aih25uvc9UUrB/MyGenerator.java)注意，这里使用的数据库就是renren-fast项目的数据库，所以sql文件在renren-fast项目中找哈。

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


