---
title: spring security与oauth2
date: 2020-09-14 14:58:04
tags: spring security
excerpt: 转载 12、spring security与oauth2
---


## [3_权限鉴定.xmind](https://uploader.shimo.im/f/ZDVYpP2VmN4MbAwt.xmind)[4_oauth2.xmind](https://uploader.shimo.im/f/WN68E4gDAWIhXIOY.xmind)[5_spring_oauth.xmind](https://uploader.shimo.im/f/97HO6FoRi9ADwr9M.xmind)[1_spring_security简介.xmind](https://uploader.shimo.im/f/BoS2yNBL0O8iwerV.xmind)[2_认证过程.xmind](https://uploader.shimo.im/f/EZyYunOY0dQPaMqk.xmind)spring security

### 简介

一个能够为基于Spring的企业应用系统提供声明式的安全訪问控制解决方式的安全框架（简单说是对访问权限进行控制嘛），应用的安全性包括用户认证（Authentication）和用户授权（Authorization）两个部分。

用户认证指的是验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。

用户授权指的是验证某个用户是否有权限执行某个操作。

在一个系统中，不同用户所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。   spring security的主要核心功能为 认证和授权，所有的架构也是基于这两个核心功能去实现的。

### 核心组件


* **SecurityContextHolder**：提供对SecurityContext的访问
* **SecurityContext**,：持有Authentication对象和其他可能需要的信息
* **AuthenticationManager**其中可以包含多个AuthenticationProvider
* **ProviderManager**对象为AuthenticationManager接口的实现AuthenticationProvider 主要用来进行认证操作的类 调用其中的authenticate()方法去进行认证操作
* **Authentication**：Spring Security方式的认证主体
* **GrantedAuthority**：对认证主题的应用层面的授权，含当前用户的权限信息，通常使用角色表示
* **UserDetails**：构建Authentication对象必须的信息，可以自定义，可能需要访问DB得到
* **UserDetailsService**：通过username构建UserDetails对象，通过loadUserByUsername根据userName获取UserDetail对象 （可以在这里基于自身业务进行自定义的实现  如通过数据库，xml,缓存获取等）。
## Filter顺序

Spring Security的官方文档向我们提供了filter的顺序，无论实际应用中你用到了哪些，整体的顺序是保持不变的：


* **ChannelProcessingFilter**，重定向到其他协议的过滤器。也就是说如果你访问的channel错了，那首先就会在channel之间进行跳转，如http变为https。
* **SecurityContextPersistenceFilter**，请求来临时在SecurityContextHolder中建立一个SecurityContext，然后在请求结束的时候，清空SecurityContextHolder。并且任何对SecurityContext的改变都可以被copy到HttpSession。
* **ConcurrentSessionFilter**，因为它需要使用SecurityContextHolder的功能，而且更新对应session的最后更新时间，以及通过SessionRegistry获取当前的SessionInformation以检查当前的session是否已经过期，过期则会调用LogoutHandler。
* **认证处理机制**，如UsernamePasswordAuthenticationFilter，CasAuthenticationFilter，BasicAuthenticationFilter等，以至于SecurityContextHolder可以被更新为包含一个有效的Authentication请求。
* **SecurityContextHolderAwareRequestFilter**，它将会把HttpServletRequest封装成一个继承自HttpServletRequestWrapper的SecurityContextHolderAwareRequestWrapper，同时使用SecurityContext实现了HttpServletRequest中与安全相关的方法。
* **JaasApiIntegrationFilter**，如果SecurityContextHolder中拥有的Authentication是一个JaasAuthenticationToken，那么该Filter将使用包含在JaasAuthenticationToken中的Subject继续执行FilterChain。
* **RememberMeAuthenticationFilter**，如果之前的认证处理机制没有更新SecurityContextHolder，并且用户请求包含了一个Remember-Me对应的cookie，那么一个对应的Authentication将会设给SecurityContextHolder。
* **AnonymousAuthenticationFilter**，如果之前的认证机制都没有更新SecurityContextHolder拥有的Authentication，那么一个AnonymousAuthenticationToken将会设给SecurityContextHolder。
* **ExceptionTransactionFilter**，用于处理在FilterChain范围内抛出的AccessDeniedException和AuthenticationException，并把它们转换为对应的Http错误码返回或者对应的页面。
* **FilterSecurityInterceptor**，保护Web URI，进行权限认证，并且在访问被拒绝时抛出异常
## 基础知识-验证


* Spring Security 验证身份的方式是利用 Filter，再加上 HttpServletRequest 的一些信息进行过滤。
* 类 Authentication 保存的是身份认证信息。
* 类 AuthenticationProvider 提供身份认证途径。
* 类 AuthenticationManager 保存的 AuthenticationProvider 集合，并调用 AuthenticationProvider 进行身份认证。
### AbstractAuthenticationProcessingFilter

AbstractAuthenticationProcessingFilter 是一个抽象类，主要的功能是身份认证。OAuth2ClientAuthenticationProcessingFilter（Spriing OAuth2）、UsernamePasswordAuthenticationFilter都继承了 AbstractAuthenticationProcessingFilter ，并重写了方法 attemptAuthentication 进行身份认证。


![图片](https://uploader.shimo.im/f/imk68veuNAAENP0x.png!thumbnail)

以上意思：AuthenticationManger --> UserDetailService --> AccessDecisionManger --> has Authority

认证 --> 验证权限

UserDetail 与 Authorition的关系


* UserDetail 是从数据库中查出来的用户基本数据
* Authorition是security已认证之后的用户数据封装
### 权限的注解使用

@PreAuthorize("hasRole('ROLE_ADMIN') orhasRole('ROLE_USER)  ")

@PostAuthorize("hasRole('ROLE_ADMIN') orhasRole('ROLE_USER)  ")

@PreFilter()

@PostFilter()

@EnableGlobalMethodSecurity(prePostEnable = true)

![图片](https://uploader.shimo.im/f/AuAUAzYEJZYVEGT6.png!thumbnail)

![图片](https://uploader.shimo.im/f/OdEP9mKmcK4J2mQO.png!thumbnail)

## 入门demo

官网demo：


* [https://docs.spring.io/spring-security/site/docs/current/guides/html5/helloworld-boot.html](https://docs.spring.io/spring-security/site/docs/current/guides/html5/helloworld-boot.html)
* [https://github.com/spring-projects/spring-security-javaconfig](https://github.com/spring-projects/spring-security-javaconfig)

常见异常类型


* UsernameNotFoundException（用户不存在）
* DisabledException（用户已被禁用）
* BadCredentialsException（坏的凭据）
* LockedException（账户锁定）
* AccountExpiredException （账户过期）
* CredentialsExpiredException（证书过期）
## springboot集成

第一步，导入jar包

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<!--如果页面模板是thymeleaf-->
<!-- thymeleaf页面模板-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<!--Thymeleaf Spring Security依赖-->
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
```
第二步、配置基本信息
```
@Configuration
@EnableWebSecurity //开启security自动配置
@EnableGlobalMethodSecurity(prePostEnabled = true) //开启注解
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    CustomUserDetailsService customUserDetailsService;
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/css/**", "/", "/index").permitAll()
                .antMatchers("/user/**").hasRole("USER")
                .and()
                .formLogin().loginPage("/login").failureUrl("/login-error")
        ;
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
//        auth.inMemoryAuthentication().passwordEncoder(new IPasswordEncoder())
//                .withUser("user").password("1").roles("USER");
        auth.userDetailsService(customUserDetailsService).passwordEncoder(new IPasswordEncoder());
    }
}
```

1. 通过 @EnableWebMvcSecurity 注解开启Spring Security的功能

@EnableGlobalMethodSecurity(prePostEnabled = true)开启security的注解功能，如@PreAuthorize("hasRole('ROLE_VIP')")


1. 继承 WebSecurityConfigurerAdapter ，并重写它的方法来设置一些web安全的细节
2. configure(HttpSecurity http) 方法配置与http连接相关
3. 通过 authorizeRequests() 定义哪些URL需要被保护、哪些不需要被保护。例如以上代码指定了 /css/** 和 /index不需要任何认证就可以访问，/user/**需要USER角色权限。
4. 通过 formLogin() 定义当需要用户登录时候，转到的登录页面。
5. configure(AuthenticationManagerBuilder auth) 方法，注释的部分表示在内存中创建了一个用户，该用户的名称为user，密码为1，用户角色为USER。一般我们都是从数据库中查询用户数据，所以这里实现了UserDetailsService接口，定义了一个CustomUserDetailsService 类。loadUserByUsername（）方法中返回User，带有用户名称、密码、和用户的角色权限。同时在security5.0后，需要自定义一个密码匹配器，所以要重写一个PasswordEncoder，定义匹配规则。

第三步、定义UserDetailsService 和PasswordEncorder

```
@Component
public class CustomUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        String username = "lv";
        String password = "1";
        Collection<GrantedAuthority> authorities = new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
        UserDetails userDetails = User.withUsername(username)
                .password(password)
                .authorities(authorities)
                .build();
        return userDetails;
    }
}
```
还有PasswordEncorder
```
/**
 * 创建PasswordEncorder的实现类
 */
public class IPasswordEncoder implements PasswordEncoder {
    @Override
    public String encode(CharSequence charSequence) {
        return charSequence.toString();
    }
    @Override
    public boolean matches(CharSequence charSequence, String s) {
        return s.equals(charSequence.toString());
    }
}
```
ok，上面的配置完毕之后，可以说我们已经集成了spring security了，那么我们正在用到我们项目资源中，从上面的配置可以知道，我们要使得资源拥有权限才能访问。现在我们建一下资源
```
@Slf4j
@Controller
public class IndexController {
    @GetMapping("/login")
    public String login() {
        log.info("-------------> login");
        return "login";
    }
    @GetMapping({"/", "", "/index"})
    public String index() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        log.info("用户----------->{}", authentication.getPrincipal());
        return "index";
    }
}
```
访问受限资源或者点击登录时候，我们都会跳转到/login页面，


* login.html
```
<form name='f' th:action='@{/login}' method='POST'>
    <table>
        <tr><td>User:</td><td>
            <input type='text' name='username' value='' /></td></tr>
        <tr><td>Password:</td>
            <td><input type='password' name='password'/></td></tr>
        <tr><td colspan='2'>
            <input id="remember_me" name="remember-me" type="checkbox"/>
            <label for="remember_me" class="inline">Remember me</label></td></tr>
        <tr><td colspan='2'>
            <input name="submit" type="submit" value="Login"/></td></tr>
    </table>
</form>
```
用户的登陆认证是由Spring Security进行处理的，请求路径默认为/login，方法是Post方式。用户名字段默认为username，密码字段默认为password。可以研究一下UsernamePasswordAuthenticationFilter就会发现。
![图片](https://uploader.shimo.im/f/8dgm2nQMlrQZez2B.png!thumbnail)

说明了，点击post的login表单之后会在这个过滤器中被处理。

登录成功之后跳转到/index


* index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>index</title>
</head>
<body>
here is index;
</body>
</html>
```
另外受限资源还可以使用注解来拦截，@PreAuthorize表示前缀权限拦截
```
    @ResponseBody
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    @GetMapping("/admin")
    public Object admin() {
        return "说明有admin角色权限";
    }
    @ResponseBody
    @PreAuthorize("hasRole('ROLE_VIP')")
    @GetMapping("/vip")
    public Object vip() {
        return "说明有vip角色权限";
    }
```
## OAuth2

OAuth 就是一种授权机制。数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。

### 令牌与密码

令牌（token）与密码（password）的作用是一样的，都可以进入系统，但是有三点差异。

（1）令牌是短期的，到期会自动失效，用户自己无法修改。密码一般长期有效，用户不修改，就不会发生变化。

（2）令牌可以被数据所有者撤销，会立即失效。密码一般不允许被他人撤销。

（3）令牌有权限范围（scope）,密码一般是完整权限。

### 四种授权方式


* 授权码（authorization-code）
    * 授权码（authorization code）方式，指的是第三方应用先申请一个授权码，然后再用该码获取令牌。
* 隐藏式（implicit）
    * 允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）
* 密码式（password）：
    * 允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）
* 客户端凭证（client credentials）
    * 适用于没有前端的命令行应用，即在命令行下请求令牌。

Oauth2授权主要由两部分组成：


* Authorization server：认证服务器
    * 授权模式
    * 生成token的存储
* Resource server：资源服务器
    * springsecurity过滤器链上加了OAuth2AuthenticationProcessingFilter（决定是否能访问保护资源）
    * 资源（rest服务）

在实际项目中以上两个服务可以在一个服务器上，也可以分开部署。

核心配置主要分为授权应用和客户端应用两部分，如下：


* 授权应用：即Oauth2授权服务，主要包括Spring Security、认证服务和资源服务两部分配置
* 客户端应用：即通过授权应用进行认证的应用，多个客户端应用间支持单点登录

![图片](https://uploader.shimo.im/f/ThP9QSbx8eIOpq1P.png!thumbnail)

授权模式第一步：

[http://localhost:8081/oauth/authorize?client_id=client&response_type=code&redirect_uri=http://www.baidu.com](http://localhost:8081/oauth/authorize?client_id=client&response_type=code&redirect_uri=http://www.baidu.com)

授权模式第二步：


* Spring Security 与 OAuth2（授权服务器）

[https://www.jianshu.com/p/227f7e7503cb](https://www.jianshu.com/p/227f7e7503cb)

### 授权码模式

关于授权码模式，参考qq、微博、github登录的模式

看mblog项目

### 密码模式

通过输入密码得到token

### 授权服务配置


* 配置一个授权服务，需要考虑 授权类型（GrantType）、不同授权类型为客户端（Client）提供了不同的获取令牌（Token）方式，每一个客户端（Client）都能够通过明确的配置以及权限来实现不同的授权访问机制，也就是说如果你提供了一个 “client_credentials” 授权方式，并不意味着其它客户端就要采用这种方式来授权
* 使用 @EnableAuthorizationServer 来配置授权服务机制，并继承 AuthorizationServerConfigurerAdapter 该类重写 configure 方法定义授权服务器策略
### 资源服务器


* 要访问资源服务器受保护的资源需要携带令牌（从授权服务器获得）
* 客户端往往同时也是一个资源服务器，各个服务之间的通信（访问需要权限的资源）时需携带访问令牌
* 资源服务器通过 @EnableResourceServer 注解来开启一个 OAuth2AuthenticationProcessingFilter 类型的过滤器
* 通过继承 ResourceServerConfigurerAdapter 类来配置资源服务器
