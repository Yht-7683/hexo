---
title: 负载均衡工具nginx
date: 2020-09-14 15:02:04
tags: nginx
excerpt: 转载 14、负载均衡工具nginx
---

### [5_蓝绿部署.xmind](https://uploader.shimo.im/f/EydmWFmo3yYVqF1o.xmind)[课程14预习_负载均衡nginx.xmind](https://uploader.shimo.im/f/Jo10Wyc41vEzk7Yq.xmind)[1_负载均衡nginx.xmind](https://uploader.shimo.im/f/hf3uuddQhfwb5ca9.xmind)[2_nginx安装.xmind](https://uploader.shimo.im/f/FCMKTT0QXFQsTNrs.xmind)[3_Nginx能做什么.xmind](https://uploader.shimo.im/f/VtIxkj1XOdofYkfn.xmind)[4_正向代理与反向代理.xmind](https://uploader.shimo.im/f/sGnF1SBIdjwp2J8m.xmind)概念

广义：

负载平衡（Load balancing）是一种计算机技术，用来在多个计算机（计算机集群）、网络连接、CPU、磁盘驱动器或其他资源中分配负载，以达到最优化资源使用、最大化吞吐率、最小化响应时间、同时避免过载的目的。 使用带有负载平衡的多个服务器组件，取代单一的组件，可以通过冗余提高可靠性。负载平衡服务通常是由专用软件和硬件来完成。

狭义（web服务器）：

负载均衡(Load Balance)是集群技术(Cluster)的一种应用。负载均衡可以将工作任务分摊到多个处理单元,从而提高并发处理能力。目前最常见的负载均衡应用是Web负载均衡。根据实现的原理不同,常见的web负载均衡技术包括:DNS轮询、IP负载均衡和CDN。其中IP负载均衡可以使用硬件设备或软件方式来实现。

### 常用软件、硬件

F5、思科

LVS、Nginx、Apache

### 与故障转移

负载均衡经常被用于实现故障转移 -- 当一个或多个组件出现故障时能持续提供服务。这些组件都在持续监控（例如：Web服务器通过请求一个已知页面来监控是否正常工作）中，当一个组件没有响应，负载均衡器就会发现并不再向其发送数据。同样当一个组件重新上线，负载均衡器会重新开始向其发送数据。为了能够如前所述正常工作，负载均衡体系中至少要有一个冗余服务。采用一用一备方案（一个组件提供服务，一个备用，当主组件故障时备用组件将接管继续提供服务）比故障转移方案更加经济，灵活。有些类型的RAID系统使用的热备份功能跟这是类似的作用。

![图片](https://uploader.shimo.im/f/ymwUdOKL7kgq4pRr.png!thumbnail)

## 常用负载算法

### **DNS轮询**

大多数域名注册商都支持对统一主机添加多条A记录，这就是DNS轮询，DNS服务器将解析请求按照A记录的顺序，随机分配到不同的IP上，这样就完成了简单的负载均衡。


* 零成本：只是在DNS服务器上绑定几个A记录，域名注册商一般都免费提供解析服务；
* 部署简单：就是在网络拓扑进行设备扩增，然后在DNS服务器上添加记录。
* 可靠性低：
    * 假设一个域名DNS轮询多台服务器，如果其中的一台服务器发生故障，那么所有的访问该服务器的请求将不会有所回应，这是任何人都不愿意看到的。即使从DNS中去掉该服务器的IP，但在Internet上，各地区电信、网通等宽带接入商将众多的DNS存放在缓存中，以节省访问时间，DNS记录全部生效需要几个小时，甚至更久。所以，尽管DNS轮询在一定程度上解决了负载均衡问题，但是却存在可靠性不高的缺点。
* 负载分配不均匀：
    * DNS负载均衡采用的是简单的轮询算法，不能区分服务器的差异，不能反映服务器的当前运行状态，不能做到为性能较好的服务器多分配请求，甚至会出现客户请求集中在某一台服务器上的情况。
## 腾讯云的负载均衡

首先打开腾讯云的云解析模块，可以看到菜单栏有个负载均衡的栏目，要实现这个负载均衡很简单，只需要添加同样的主机记录即可，即：www同时指向两个记录值，然后负载均衡中就会出现负载的基本算法可以选择。


* [https://cloud.tencent.com/document/product/302/11358](https://cloud.tencent.com/document/product/302/11358)

![图片](https://images-cdn.shimo.im/kvzNSUY2xSwjIyPB/image.png!thumbnail)

可以通过一下nslookup命令来查看域名的解析：

```
nslookup www.java-mindmap.com
```
## nginx的安装

### 简介

Nginx（发音同engine x）是一个异步框架的 Web服务器，也可以用作反向代理，负载平衡器 和 HTTP缓存。相较于Apache\lighttpd具有占有内存少，稳定性高等优势，并且依靠并发能力强，丰富的模块库以及友好灵活的配置而闻名。在Linux操作系统下，nginx使用epoll事件模型,得益于此，nginx在Linux操作系统下效率相当高。

### 安装步骤

**gcc安装******

```
yum -y install gcc-c++
```
**pcre安装**
```
yum -y install  pcre pcre-devel
```
**zlib安装**
```
yum -y install zlib zlib-devel
```
**OpenSSL安装**
```
yum -y install openssl openssl-devel
```
**nginx安装**
```
官网：https://nginx.org/en/download.html
首先官网下载nginx-1.12.2.tar.gz压缩包
tar  -zxvf nginx-1.12.2.tar.gz
./configure
make && make install
cd /usr/local/nginx/sbin/
nginx -v
```
![图片](https://images-cdn.shimo.im/HC5s1RStpTk3q2lT/image.png!thumbnail)

**访问nginx-关闭防火墙**

```
#查看防火墙状态
firewall-cmd --state
#关闭防火墙
systemctl stop firewalld.service
firewall-cmd --states
```
## 配置文件nginx.conf

### 目录结构

### ![图片](https://images-cdn.shimo.im/Z7e0ZkpTqkAJVdhE/image.png!thumbnail)

### nginx.conf

```
#指定nginx worker进程运行用户以及用户组
#user  nobody; 
#nginx进程数，建议设置为等于CPU总核心数。
worker_processes  1; 
#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#进程文件
#pid        logs/nginx.pid;
#工作模式与连接数上限
events {
    #单个进程最大连接数（最大连接数=连接数*进程数）
    worker_connections  1024;
}
#设定http服务器
http {
    #文件扩展名与文件类型映射表
    include       mime.types;
    #默认文件类型
    default_type  application/octet-stream;
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    #access_log  logs/access.log  main;
    #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改 成off。
    sendfile        on;
    #防止网络阻塞
    #tcp_nopush     on;

    #长连接超时时间，单位是秒
    #keepalive_timeout  0;
    keepalive_timeout  65;
    #开启gzip压缩输出
    #gzip  on;
    #虚拟主机的配置
    server {
        #监听端口
        listen       80;
        #域名可以有多个，用空格隔开
        server_name  localhost;
        #默认编码
        #charset utf-8;
        #定义本虚拟主机的访问日志
        #access_log  logs/host.access.log  main;
        location / {
            root   html;
            index  index.html index.htm;
        }
        #error_page  404              /404.html;
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;
    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
}
```
### location正则写法

语法规则

```
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
```
原文：[http://seanlook.com/2015/05/17/nginx-location-rewrite/](http://seanlook.com/2015/05/17/nginx-location-rewrite/)


* 以=开头表示精确匹配，如 A 中只匹配根目录结尾的请求，后面不能带任何字符串。
* ^~开头表示uri以某个常规字符串开头，不是正则匹配
* ~ 开头表示区分大小写的正则匹配;
* ~* 开头表示不区分大小写的正则匹配
* / 通用匹配, 如果没有其它匹配,任何请求都会匹配到
```
location  = / {
  # 精确匹配 / ，主机名后面不能带任何字符串
  [ configuration A ]
}
location  / {
  # 因为所有的地址都以 / 开头，所以这条规则将匹配到所有请求
  # 但是正则和最长字符串会优先匹配
  [ configuration B ]
}
location /documents/ {
  # 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索
  # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
  [ configuration C ]
}
location ~ /documents/Abc {
  # 匹配任何以 /documents/Abc 开头的地址，匹配符合以后，还要继续往下搜索
  # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
  [ configuration CC ]
}
location ^~ /images{
  # 匹配任何以 /images/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条。
  [ configuration D ]
}
location ~* \.(gif|jpg|jpeg)$ {
  # 匹配所有以 gif,jpg或jpeg 结尾的请求
  # 然而，所有请求 /images/ 下的图片会被 config D 处理，因为 ^~ 到达不了这一条正则
  [ configuration E ]
}
location /images/ {
  # 字符匹配到 /images/，继续往下，会发现 ^~ 存在
  [ configuration F ]
}
location /images/abc {
  # 最长字符匹配到 /images/abc，继续往下，会发现 ^~ 存在
  # F与G的放置顺序是没有关系的
  [ configuration G ]
}
location ~ /images/abc/ {
  # 只有去掉 config D 才有效：先最长匹配 config G 开头的地址，继续往下搜索，匹配到这一条正则，采用
    [ configuration H ]
}
location ~* /js/.*/\.js
```
* **顺序 no优先级：**

***(location =) > (location 完整路径) > (location ^~ 路径) > (location ~,~* 正则顺序) > (location 部分起始路径) > (/)***

按照上面的location写法，以下的匹配示例成立：

/ -> config A


* 精确完全匹配，即使/index.html也匹配不了

/downloads/download.html -> config B


* 匹配B以后，往下没有任何匹配，采用B

/images/1.gif -> configuration D


* 匹配到F，往下匹配到D，停止往下

/images/abc/def -> config D

最长匹配到G，往下匹配D，停止往下


* 你可以看到 任何以/images/开头的都会匹配到D并停止，FG写在这里是没有任何意义的，H是永远轮不到的，这里只是为了说明匹配顺序

/documents/document.html -> config C


* 匹配到C，往下没有任何匹配，采用C

/documents/1.jpg -> configuration E


* 匹配到C，往下正则匹配到E

/documents/Abc.jpg -> config CC


* 最长匹配到C，往下正则顺序匹配到CC，不会往下到E

**实际使用建议**

```
所以实际使用中，个人觉得至少有三个匹配规则定义，如下：
#直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。
#这里是直接转发给后端应用服务器了，也可以是一个静态首页
# 第一个必选规则
location = / {
    proxy_pass http://tomcat:8080/index
}
# 第二个必选规则是处理静态文件请求，这是nginx作为http服务器的强项
# 有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用
location ^~ /static/ {
    root /webroot/static/;
}
#第三个规则就是通用规则，用来转发动态请求到后端应用服务器
#非静态文件请求就默认是动态请求，自己根据实际把握
#毕竟目前的一些框架的流行，带.php,.jsp后缀的情况很少了
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
    root /webroot/res/;
}
```
## 基本命令

```
#启动
nginx   -c /etc/nginx/nginx.conf
#停止
nginx -s stop 或者
nginx -s quit 或者
pkill -9 nginx
#重载配置
nginx -s reload
#配置文件测试
nginx -t -c /etc/nginx/nginx.conf
```
## 正向代理与反向代理

### ***正向代理***

一般情况下，如果没有特别说明，代理技术默认说的是正向代理技术。关于正向代理的概念如下：正向代理(forward)是一个位于客户端【用户A】和原始服务器(origin server)【服务器B】之间的服务器【代理服务器Z】，为了从原始服务器取得内容，用户A向代理服务器Z发送一个请求并指定目标(服务器B)，然后代理服务器Z向服务器B转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。

如下图1.1

![图片](https://images-cdn.shimo.im/RItfN7MbGrgC0PHg/image.png!thumbnail)

（图1.1）

从上面的概念中，我们看出，文中所谓的正向代理就是代理服务器替代访问方【用户A】去访问目标服务器【服务器B】这就是正向代理的意义所在。

**正向代理运用**


* 访问本无法访问的服务器B
* 加速访问服务器B
* Cache作用（Cache命中）
* 客户端访问授权
* 隐藏访问者的行踪----“肉鸡”
### ***反向代理***

反向代理正好与正向代理相反，对于客户端而言代理服务器就像是原始服务器，并且客户端不需要进行任何特别的设置。客户端向反向代理的命名空间(name-space)中的内容发送普通请求，接着反向代理将判断向何处(原始服务器)转交请求，并将获得的内容返回给客户端。

**反向代理作用**


* 保护和隐藏原始资源服务器
* 负载均衡
## Nginx能做什么


1. 资源服务器
2. 静态服务器
3. 反向代理
4. 负载均衡
5. HTTP服务器（包含动静分离）
### ***资源服务器***

```
location ^~ /static/ {
    root /opt/resource/;
    autoindex on;#开启目录访问，默认off，不允许访问
    autoindex_exact_size off;#关闭具体大小，显示kb或mb或gb;默认on，显示具体大小，单位byte
    autoindex_localtime on;#显示服务器的文件修改时间，默认off，不显示
}
```
注意：static路径为/opt/resource/static，即static在/opt/resource/下
访问路径[http://localhost/static/](http://localhost/static/)，点击对应资源，可以查看图片或者下载文件

![图片](https://images-cdn.shimo.im/7by7PKRPWXQspx6g/image.png!thumbnail)

### *权限控制*

```
location ^~ /static/ {
    root /opt/resource/;
    autoindex on;#开启目录访问，默认off，不允许访问
    autoindex_exact_size off;#关闭具体大小，显示kb或mb或gb;默认on，显示具体大小，单位byte
    autoindex_localtime on;#显示服务器的文件修改时间，默认off，不显示
    auth_basic           "my site";
    auth_basic_user_file /etc/nginx/11.conf;
}
```
如上面的代码，之前我们可以直接访问[http://localhost/static/](http://localhost/static/)，现在我们添加一个权限控制，需要账号密码才能访问，这时候可以使用nginx的auth basic模块。
添加

```
 auth_basic           "my site";
 auth_basic_user_file /etc/nginx/11.conf;
```
**auth_basic**
语法： auth_basic [ text|off ]

默认值： auth_basic off

作用域： http, server, location, limit_except

该指令包含用于 HTTP 基本认证 的测试名和密码。分配的参数用于认证领域。值 "off" 可以使其覆盖来自上层指令的继承性。

**auth_basic_user_file**

语法： auth_basic_user_file the_file

默认值： no

作用域： http, server, location, limit_except

该指令为某认证领域指定 htpasswd 文件名。

文件格式类似于下面的内容：

>用户名:密码
>用户名2:密码2:注释
>用户名3:密码3
>密码必须使用函数 crypt(3) 加密。 你可以使用来自 Apache 的 htpasswd 工具来创建密码文件。

这里介绍一个生成密码的工具：

[htpasswd.sh](https://uploader.shimo.im/f/QHkULnHk9w4cw5JA.sh)

也可以通过**wget -c**[http://www.huzs.net/soft/htpasswd.sh](http://www.huzs.net/soft/htpasswd.sh)****来下载

然后执行命令

```
 bash htpasswd.sh
```
就会按照提示生成账号密码，文件生成在/etc/nginx目录下。
![图片](https://uploader.shimo.im/f/HuiKCIEgteYUpwQL.png!thumbnail)

### ***静态文件***

**gzip。**开启nginx gzip压缩后，网页、css、js等静态资源的大小会大大的减少，从而可以节约大量的带宽，提高传输效率，给用户快的体验。虽然会消耗cpu资源，但是为了给用户更好的体验是值得的。


* [https://blog.csdn.net/huangbaokang/article/details/79931429](https://blog.csdn.net/huangbaokang/article/details/79931429)

![图片](https://images-cdn.shimo.im/wviR1Pblt30XfX4o/image.png!thumbnail)

```
location ~ .*\.(txt|xml|log)$ {
    gzip on;
    gzip_min_length 1k; #低于1kb的资源不压缩
    gzip_comp_level 3; #压缩级别【1-9】，越大压缩率越高，同时消耗cpu资源也越多，建议设置>在4左右。
    gzip_types text/plain application/javascript application/x-javascript text/javascript text/xml text/css;  #需要压缩哪些响应类型的资源，多个空格隔开。不建议压缩图片，下面会讲为什么。
    gzip_disable "MSIE [1-6]\.";  #配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
    gzip_vary on;  #是否添加“Vary: Accept-Encoding”响应头
    add_header Content-Type text/plain;#文件显示为text/plain格式
    root /opt/resource/;
}
```
*上述配置，可以全局配置~*
### *防盗链配置*

```
# 加入至指定location 即可实现
valid_referers none blocked *.aaa.com;
 if ($invalid_referer) {
       return 403;
}
```
### *黑名单*

主要做法就是创建一个文件，格式是deny + ip，然后http配置块中直接include就行了。

```
# 创建黑名单文件
echo 'deny 192.168.0.132;' >> ip.black
#http 配置块中引入 黑名单文件
include       ip.black;
```
### ***反向代理***

反向代理应该是Nginx做的最多的一件事了，什么是反向代理呢，以下是百度百科的说法：反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。简单来说就是真实的服务器不能直接被外部网络访问，所以需要一台代理服务器，而代理服务器能被外部网络访问的同时又跟真实服务器在同一个网络环境，当然也可能是同一台服务器，端口不同而已。 下面贴上一段简单的实现反向代理的代码

```
server {  
    listen       80;                                                         
    server_name  localhost;                                               
    client_max_body_size 1024M;
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host:$server_port;
    }
}
```
保存配置文件后启动Nginx，这样当我们访问localhost的时候，就相当于访问localhost:8080了
### ***负载均衡***

负载均衡也是Nginx常用的一个功能，负载均衡其意思就是分摊到多个操作单元上进行执行，例如Web服务器、FTP服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。简单而言就是当有2台或以上服务器时，根据规则随机的将请求分发到指定的服务器上处理，负载均衡配置一般都需要同时配置反向代理，通过反向代理跳转到负载均衡。而Nginx目前支持自带3种负载均衡策略，还有2种常用的第三方策略。

**1、RR（默认）**

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

简单配置

```
    upstream test {
        server localhost:8080;
        server localhost:8081;
    }
    server {
        listen       81;                                                         
        server_name  localhost;                                               
        client_max_body_size 1024M;
        location / {
            proxy_pass http://test;
            proxy_set_header Host $host:$server_port;
        }
    }
```
负载均衡的核心代码为
```
    upstream test {
        server localhost:8080;
        server localhost:8081;
    }
```
这里我配置了2台服务器，当然实际上是一台，只是端口不一样而已，而8081的服务器是不存在的,也就是说访问不到，但是我们访问[http://localhost](http://localhost/)的时候,也不会有问题，会默认跳转到[http://localhost:8080](http://localhost:8080/)具体是因为Nginx会自动判断服务器的状态，如果服务器处于不能访问（服务器挂了），就不会跳转到这台服务器，所以也避免了一台服务器挂了影响使用的情况，由于Nginx默认是RR策略，所以我们不需要其他更多的设置。
**2、权重**

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。 例如

```
    upstream test {
        server localhost:8080 weight=9;
        server localhost:8081 weight=1;
    }
```
那么10次一般只会有1次会访问到8081，而有9次会访问到8080
**3、ip_hash**

上面的2种方式都有一个问题，那就是下一个请求来的时候请求可能分发到另外一个服务器，当我们的程序不是无状态的时候（采用了session保存数据），这时候就有一个很大的很问题了，比如把登录信息保存到了session中，那么跳转到另外一台服务器的时候就需要重新登录了，所以很多时候我们需要一个客户只访问一个服务器，那么就需要用ip*hash了，ip*hash的每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

```
    upstream test {
        ip_hash;
        server localhost:8080;
        server localhost:8081;
    }
```
**4、fair（第三方）**
按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```
    upstream backend { 
        fair; 
        server localhost:8080;
        server localhost:8081;
    } 
```
**5、url_hash（第三方）**
按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。 在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法

```
    upstream backend { 
        hash $request_uri; 
        hash_method crc32; 
        server localhost:8080;
        server localhost:8081;
    } 
```
以上5种负载均衡各自适用不同情况下使用，所以可以根据实际情况选择使用哪种策略模式,不过fair和url_hash需要安装第三方模块才能使用，由于本文主要介绍Nginx能做的事情，所以Nginx安装第三方模块不会再本文介绍
### ***HTTP服务器***

Nginx本身也是一个静态资源的服务器，当只有静态资源的时候，就可以使用Nginx来做服务器，同时现在也很流行动静分离，就可以通过Nginx来实现，首先看看Nginx做静态资源服务器

```
    server {
        listen       80;                                                         
        server_name  localhost;                                               
        client_max_body_size 1024M;

        location / {
               root   e:\wwwroot;
               index  index.html;
           }
    }
```
这样如果访问[http://localhost](http://localhost/)就会默认访问到E盘wwwroot目录下面的index.html，如果一个网站只是静态页面的话，那么就可以通过这种方式来实现部署。
**动静分离**

动静分离是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作，这就是网站静态化处理的核心思路

```
upstream test{  
       server localhost:8080;  
       server localhost:8081;  
    }   
    server {  
        listen       80;  
        server_name  localhost;  
        location / {  
            root   e:\wwwroot;  
            index  index.html;  
        }  
        # 所有静态请求都由nginx处理，存放目录为html  
        location ~ \.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {  
            root    e:\wwwroot;  
        }  
        # 所有动态请求都转发给tomcat处理  
        location ~ \.(jsp|do)$ {  
            proxy_pass  http://test;  
        }  
        error_page   500 502 503 504  /50x.html;  
        location = /50x.html {  
            root   e:\wwwroot;  
        }  
    }  
```
这样我们就可以吧HTML以及图片和css以及js放到wwwroot目录下，而tomcat只负责处理jsp和请求，例如当我们后缀为gif的时候，Nginx默认会从wwwroot获取到当前请求的动态图文件返回，当然这里的静态文件跟Nginx是同一台服务器，我们也可以在另外一台服务器，然后通过反向代理和负载均衡配置过去就好了，只要搞清楚了最基本的流程，很多配置就很简单了，另外localtion后面其实是一个正则表达式，所以非常灵活
## 蓝绿部署

### 蓝绿？

蓝绿部署，英文名Blue Green Deployment，是一种可以保证系统在不间断提供服务的情况下上线的部署方式。

如何保证系统不间断提供服务呢？


* 蓝绿部署的模型中包含两个集群，在没有上线的正常情况下，集群A和集群B的代码版本是一致的，并且同时对外提供服务。
* 在系统升级的时候下，我们首先把一个集群（比如集群A）从负载列表中摘除，进行新版本的部署。集群B仍然继续提供服务。
* 当集群A升级完毕，我们把负载均衡重新指向集群A，再把集群B从负载列表中摘除，进行新版本的部署。集群A重新提供服务。
* 最后，当集群B也升级完成，我们把集群B也恢复到负载列表当中。这个时候，两个集群的版本都已经升级，并且对外的服务几乎没有间断过。
### 热切换

通过修改nginx.conf配置，然后nginx -s reload

