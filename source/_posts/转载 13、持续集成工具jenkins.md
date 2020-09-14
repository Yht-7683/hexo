---
title: 持续集成工具jenkins
date: 2020-09-14 15:00:04
tags: jenkins
excerpt: 转载 13、持续集成工具jenkins
---

[课程13预习_持续集成工具jenkins.xmind](https://uploader.shimo.im/f/tHJoVGKfXjw43TZY.xmind)[1_持续集成工具jenkins.xmind](https://uploader.shimo.im/f/cj0TWPCixFMdZaRP.xmind)[2_持续集成的概念.xmind](https://uploader.shimo.im/f/TYRmVrm7VD8t4Ykk.xmind)程序员开发应用，开发后需要提交svn，然后从svn拉取代码，进行构建，发布到tomcat中，发布，然后看呈现效果，这样的工作是频繁反复的在进行的，浪费了程序员的大量时间，那么能不能把这些工作自动化呢，只需要程序员更新代码到svn，然后自动的构建，发布，呈现效果，当然是可以的，通过jenkins来实现。

## 什么是持续集成

互联网软件的开发和发布，已经形成了一套标准流程，最重要的组成部分就是持续集成（Continuous integration，简称CI）。

持续集成指的是，频繁地（一天多次）将代码集成到主干。

它的好处主要有两个。

（1）快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易。

（2）防止分支大幅偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成。

持续集成的目的，就是让产品可以快速迭代，同时还能保持高质量。它的核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。

这种做法的核心思想在于：既然事实上难以做到事先完全了解完整的、正确的需求，那么就干脆一小块一小块的做，并且加快交付的速度和频率，使得交付物尽早在下个环节得到验证。早发现问题早返工。

**持续集成**

![图片](https://images-cdn.shimo.im/737vd8eXqlQXQ34d/image.png!thumbnail)

持续集成强调开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。

**持续交付**

![图片](https://images-cdn.shimo.im/YmelRpypaccvrMrH/image.png!thumbnail)

持续交付在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境的「类生产环境」（*production-like environments*）中。比如，我们完成单元测试后，可以把代码部署到连接数据库的 Staging 环境中更多的测试。如果代码没有问题，可以继续手动部署到生产环境中。

**持续部署**

![图片](https://images-cdn.shimo.im/3b6FqXOzKrcVwbeI/image.png!thumbnail)

持续部署则是在持续交付的基础上，把部署到生产环境的过程自动化。

### 总结就是

**持续**，就是说每完成一个完整的部分，就向下个环节交付，发现问题可以马上调整。是的问题不会放大到其他部分和后面的环节。

**集成，**是指软件个人研发的部分向软件整体部分交付，以便尽早发现个人开发部分的问题；

**部署，**是代码尽快向可运行的开发/测试节交付，以便尽早测试；

**交付，**是指研发尽快向客户交付，以便尽早发现生产环境中存在的问题。

1）核心价值体现在：

a、持续集成中的任何一个环节都是自动完成的，无需太多的人工干预，有利于减少重复过程以节省时间、费用和工作量；

b、持续集成保障了每个时间点上团队成员提交的代码是能成功集成的。换言之，任何时间点都能第一时间发现软件的集成问题，使任意时间发布可部署的软件成为了可能；

c、持续集成还能利于软件本身的发展趋势，这点在需求不明确或是频繁性变更的情景中尤其重要，持续集成的质量能帮助团队进行有效决策，同时建立团队对开发产品的信心。

2）持续集成系统的组成

由此可见，一个完整的构建系统必须包括：

A、一个自动构建过程，包括自动编译、分发、部署和测试等。

B、 一个代码存储库，即需要版本控制软件来保障代码的可维护性，同时作为构建过程的素材库。

C、一个持续集成服务器。本文中介绍的 Jenkins/Jenkins 就是一个配置简单和使用方便的持续集成服务器。

## 什么是jenkins

Jenkins是一个开源的持续集成的服务器，Jenkins开源帮助我们自动构建各类项目。Jenkins强大的插件式，使得Jenkins可以集成很多软件，可能帮助我们持续集成我们的工程项目。

Jenkins对于maven工程完整的编译和发布流程如下：

1、Jenkins从SVN上拉取代码到指定的编译机器上。

2、在编译机器上触发编译命令或脚本。

3、编译得到的结果文件。

4、把结果文件传到指定的服务器上。

5、重启服务

## jenkins环境安装

### 安装java环境

1、本地下载**jdk8.tar.gz**包，然后通过xshell和xftp工具上传到**/opt**【可自定义】目录。

2、使用**tar -zxvf jdk8.tar.gz**解压文件。

3、打开**/etc/profile**文件中配置java环境

```
export JAVA_HOME=/opt/jdk1.8.0_161
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
使用命令**source /etc/profile**让环境重新加载，输入**java -version**坚持是否配置成功
![图片](https://images-cdn.shimo.im/YBpMwlwsPOsXbsgX/image.png!thumbnail)

### 安装maven环境

1、查看地址

```
https://maven.apache.org/download.cgi
```
2、复制要下载的链接地址，使用linux的wget命令下载。切换到**/opt**目录，直接下载：
```
wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.tar.gz
```
3、使用**tar -zxvf apache-maven-3.5.3-bin.tar.gz**解压文件得到**apache-maven-3.5.3**
4、**/etc/profile**配置环境，并使用命令**source /etc/profile**让环境重新加载

```
export MAVEN_HOME=/opt/apache-maven-3.5.3
export PATH=${MAVEN_HOME}/bin:$PATH
```
5、检验是否配置成功。
![图片](https://images-cdn.shimo.im/p63TOsQeH8Y1mEhs/image.png!thumbnail)

maven的配置setting.xml最好修改镜像源：

```
<mirror>  
   <id>alimaven</id>  
   <name>aliyun maven</name>  
   <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
   <mirrorOf>central</mirrorOf>          
 </mirror>  
```
### 安装git环境

按照以下命名安装：

```
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
wget https://www.kernel.org/pub/software/scm/git/git-1.8.3.1.tar.gz
tar xzf git-1.8.3.1.tar.gz
cd git-1.8.3.1
make prefix=/usr/local/git all
make prefix=/usr/local/git install
echo "export PATH=$PATH:/usr/local/git/bin" >>/etc/bashrc
source /etc/bashrc
git --version
```
### mysql安装


* [http://www.runoob.com/mysql/mysql-install.html](http://www.runoob.com/mysql/mysql-install.html)

查看默认密码：

```
grep 'temporary password' /var/log/mysqld.log
```
![图片](https://images-cdn.shimo.im/kqghiXLI6E8MuSWe/image.png!thumbnail)

修改密码：

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'admin';
```
## 安装jenkins

### yum的安装方式

```
yum install -y java-1.8.0-openjdk && \wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo && \rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key &&\yum clean all && yum makecache && \yum install -y jenkins && \systemctl start jenkins
```
rpm -ql jenkins#查看jenkins安装相关目录
安装目录/var/lib/jenkins

配置文件 /etc/sysconfig/jenkins

日志目录 /var/log/jenkins

详细可见：[https://blog.csdn.net/qq_26848099/article/details/78901240](https://blog.csdn.net/qq_26848099/article/details/78901240)

### 下载jenkins.war

直接下载war包，windows和linux环境通用。linux可以通过wget命令直接下载到指定目录。windows直接把链接复制到浏览器链接栏即可下载。

```
wget http://mirrors.shu.edu.cn/jenkins/war-stable/2.121.1/jenkins.war
```
下载完了之后可以在当前目录看到jenkins.war包，表示已经下载完成。
新版本看这里：[http://mirrors.jenkins-ci.org/war/latest/](http://mirrors.jenkins-ci.org/war/latest/)

### 运行命令

jenkins.war包可以有两种运行方式：


* 第一种是把war包放到tomcat的webapps目录下运行；
* 第二种是直接通过java -jar来启动项目，因为jenkins.war包里面已经内置了jetty服务器，可以直接启动，通过--httpPort来指定启动端口，添加&表示以服务形式启动。
```
java -jar jenkins.war --httpPort=8081 &
```
### 安装过程

打开链接地址[http://你的ip](http://xn--ip-0p3cm89l/):8081,

![图片](https://images-cdn.shimo.im/NO68eFVOhVAZIDnm/image.png!thumbnail)

获取秘钥：

```
cat /root/.jenkins/secrets/initialAdminPassword
```
新手最好直接选择安装推荐的插件就可以，熟悉以后下次就可以自定义插件安装即可。
![图片](https://images-cdn.shimo.im/vgryHTJFXaYceKDu/image.png!thumbnail)

正在下载推荐的额插件

![图片](https://images-cdn.shimo.im/v9HGSdXQtWIefB2H/image.png!thumbnail)

插件安装完毕，跳转到创建管理员。填入用户名密码。

![图片](https://images-cdn.shimo.im/ilnAHgiUfLw9EtjT/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/CAe28rIy690lalvh/image.png!thumbnail)

jenkins安装成功。

![图片](https://images-cdn.shimo.im/FiW7CM9XlaIgvpaF/image.png!thumbnail)

### jenkins插件管理

本次课程主要讲发布maven项目到tomcat中。所以还需要安装以下两个插件：

打开管理插件页面：

![图片](https://images-cdn.shimo.im/IwxBZQ8YGN82fCrg/image.png!thumbnail)


* 一个构建maven项目插件

![图片](https://images-cdn.shimo.im/3CEjJga1gjcodKoq/image.png!thumbnail)


* 一个发布到tomcat的插件

![图片](https://images-cdn.shimo.im/Adnn9dv6Amoall7F/image.png!thumbnail)

选择，直接安装！

### jenkins全局配置

系统管理--》全局工具配置，需要把服务器的jdk、maven、git等环境配置好。

![图片](https://images-cdn.shimo.im/9wN9RLhN7mwldckM/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/iLxsNwO5RSMx6p3P/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/3QGDKxTcJcwFoRt5/image.png!thumbnail)

## 构建jar项目

### 共同配置


* 新建一个maven项目。

![图片](https://images-cdn.shimo.im/oLMSkWX6YpcM6YZ0/image.png!thumbnail)


* 配置git或者svn地址，jenkins会自动从远程仓库拉去最新代码。

![图片](https://images-cdn.shimo.im/bDKekdg9XZck8i7e/image.png!thumbnail)

如果是私有项目的话，需要加入用户名密码

![图片](https://images-cdn.shimo.im/UGtFRnKb2yEKsDt1/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/8i0cRSO7EOQzck7Y/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/kg6FIMSZLbMk2GEF/image.png!thumbnail)


* 开始maven打包构建的命令，可以去掉test测试
```
clean install -Dmaven.test.skip=true -Ptest
```
![图片](https://images-cdn.shimo.im/V86jrVzTrY8SwM4w/image.png!thumbnail)

### jenkins与应用服务器同一台机器


* 经过上一个步骤之后，jenkins会在默认路径/root/.jenkins/workspace/上看到打包的项目，/root/.jenkins/workspace/homework/target目录下有打包的jar文件。spring-boot-homework-0.0.1-SNAPSHOT.jar。因此我们打包完成之后的工作就是要替换原来的jar文件，然后重启项目。

![图片](https://images-cdn.shimo.im/ADu38UgvWQkizAeZ/image.png!thumbnail)

这里我们使用shell脚本来完成。


1. DIR表示存放jar文件、和启动项目的文件夹。JAEFILE是指jar包的名称。
2. 接下来的工作其实就是杀掉原来的spring-boot-homework-0.0.1-SNAPSHOT.jar进程
3. 备份原来的jar包
4. 把新生成的jar覆盖原来的
5. 然后java -jar启动项目

* BUILD_ID=dontKillMe表示jenkins不杀掉衍生的线程
* nohup表示不用提示

shell脚本如下：这个脚本有缺陷，只做测试使用

```
echo '开始启动项目~~~~~~~~~'
DATE=$(date +%Y%m%d_%H%M)
export JAVA_HOME PATH CLASSPATH
JAVA_HOME=/opt/jdk1.8.0_181
PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH
DIR=/opt/jar
JARFILE=spring-boot-homework-0.0.1-SNAPSHOT.jar
if [ ! -d $DIR/backup ];then
   mkdir -p $DIR/backup
fi
cd $DIR
ps -ef | grep $JARFILE | grep -v grep | awk '{print $2}' | xargs kill -9
mv $JARFILE backup/$JARFILE$DATE
mv -f /root/.jenkins/workspace/homework/target/$JARFILE .
BUILD_ID=dontKillMe nohup java -jar $JARFILE > out.log &
if [ $? = 0 ];then
        sleep 30
        tail -n 50 out.log
fi
cd backup/
ls -lt|awk 'NR>5{print $NF}'|xargs rm -rf
echo '结束启动项目~~~~'
```
致此，项目就会自动部署，只需要点击构建按钮，jenkins就会自动拉去最新代码，然后备份、重启项目。
回顾一下流程：


* 首先创建maven项目
* 配置项目的git仓库地址
* 配置maven构建命令（打包命令）
* 打包文成之后执行shell实现切换jar，重新部署项目。
### jenkins与应用服务器不同台机器

上面的过程是当jenkins与应用服务器同一台的时候才能直接复制过去，当服务器不同的时候就不能这样做了，可以通过ssh或者scp来传输jar包过去，然后再执行shell脚本。


* 配置ssh免密登录

为了方便演示，我这里用的还是同一台机器，不过不再使用复制的方式，而是通过ssh吧文件传输给自己。

在配置之前，输入ssh 127.0.0.1，会提示要输入密码。

![图片](https://images-cdn.shimo.im/jdRj8HDlY9gUOZyG/image.png!thumbnail)

ssh的配置可使用密钥，也可以使用密码，这里我们使用密钥来配置，在配置之前先配置好jenkins服务器和应用服务器的密钥认证。

**应用服务器**上生成密钥对，使用ssh-keygen -t rsa命令。输入下面命令**一直回车**，一个矩形图形出现就说明成功，在~/.ssh/下会有私钥id_rsa和公钥id_rsa.pub

```
ssh-keygen -t rsa
```
![图片](https://images-cdn.shimo.im/xtRo41Z9EaQnAWOg/image.png!thumbnail)

![图片](https://images-cdn.shimo.im/FfPeify1fM46Fjp2/image.png!thumbnail)

然后把生成的公钥推动到jenkins服务器上，加入47.106.38.101是应用服务器的ip，因为这里用的jenkins和应用服务器同一台，所以ip是一样的。

```
ssh-copy-id -i ~/.ssh/id_rsa.pub 47.106.38.101
```
第一次需要验证密码，输入应用服务器的登录免密。出现以下提示，说明成功了。
![图片](https://images-cdn.shimo.im/qHeyNv76nlQI1tqQ/image.png!thumbnail)

在**应用服务器**上重启ssh服务，

```
service sshd restart
```
这时候再次ssh 47.106.38.101或者ssh 127.0.0.1就会直接登录成功，不再需要免密。
![图片](https://images-cdn.shimo.im/Lit845DATbQjBiHH/image.png!thumbnail)

致此，免密登录配置完成~

总结一下上面的内容，就是应用服务器上配置密钥对，然后把公钥发送到jenkins服务器上，这样jenkins就有公钥可以加密连接到应用服务器了。

接下来的工作就是上次jar文件到应用服务器，然后执行shell实现重启应用。

这里使用一种比较简单的方法，直接替换之前的命令行就行。

![图片](https://images-cdn.shimo.im/jFn8PHADnH4DujCj/image.png!thumbnail)命令行的意思是通过scp命令把jar上次到远程服务器的指定目录，然后通过ssh免密登录指定远程的脚本。

```
cd target/
scp spring-boot-homework-0.0.1-SNAPSHOT.jar 127.0.0.1:/opt/jar/target/
ssh 127.0.0.1 "cd /opt/jar;  ./republish.sh;"
```

远程脚本如下：**和之前的差不多，改了一下jar的目录而已**。

```
DATE=$(date +%Y%m%d_%H:%M:%S)
export JAVA_HOME PATH CLASSPATH
JAVA_HOME=/opt/jdk1.8.0_181
PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH
DIR=/opt/jar
JARFILE=spring-boot-homework-0.0.1-SNAPSHOT.jar
if [ ! -d $DIR/backup ];then
   mkdir -p $DIR/backup
fi
cd $DIR
ps -ef | grep $JARFILE | grep -v grep | awk '{print $2}' | xargs kill -9
mv $JARFILE backup/$JARFILE$DATE
mv -f /opt/jar/target/$JARFILE .
java -jar $JARFILE > out.log &
if [ $? = 0 ];then
        sleep 30
        tail -n 50 out.log
fi
cd backup/
ls -lt|awk 'NR>5{print $NF}'|xargs rm -rf
```

***另一种方法，还没测试成功。（暂时放弃）---------------------------------------------------------***

配置jenkins的ssh传输

首先安装publish over ssh插件

![图片](https://images-cdn.shimo.im/PftHCxZQNeMMl163/image.png!thumbnail)

安装之后，点击系统管理 > 系统设置

选择 Publish over SSH


* Passphrase 不用设置
* Path to key 写上生成的ssh路径：/root/.ssh/id_rsa

下面的SSH Servers是重点


* Name 随意起名代表这个服务，待会要根据它来选则
* Hostname 配置应用服务器的地址
* Username 配置linux登陆用户名
* Remote Directory 不填

![图片](https://images-cdn.shimo.im/LNes0Dc1HnsmP0PJ/image.png!thumbnail)

点击test configuration之后，出现success，表示成功。


* 新建一个maven构建项目home_server_split。复制之前的项目配置。

![图片](https://images-cdn.shimo.im/7X3VVA3SSKU1Rd4h/image.png!thumbnail)

大部分的配置都是和同一台机器的一样的，需要修改的地方是打包完成之后的操作。

![图片](https://images-cdn.shimo.im/KgTYjmru1msjObuY/image.png!thumbnail)


* Source files配置:target/xxx-0.0.1-SNAPSHOT.jar 项目jar包名
* Remove prefix:target/
* Remote directory:代码应用服务器的目录地址，
* Exec command： 应用服务器对应的脚本。

***此种方法，还没测试成功。（暂时放弃）---------------------------------------------------------***

## 构建war项目（知道一下就行）

与发包jar项目的前面相同，不同的是打包构建之后的步骤不一样，之前是使用shell来完成项目的重新部署，现在打成war之后我们需要做的是替换原来的war包

springboot打包成war包请参考


* [https://blog.csdn.net/zhoucheng05_13/article/details/77915294](https://blog.csdn.net/zhoucheng05_13/article/details/77915294)
### tomcat设置容器账号密码

因为jenkins把项目打包成war包之后需要把项目发布到tomcat的webapp目录下运行，所以需要给tomcat这是发布的账号密码。


* tomcat7在**conf/tomcat-users.xml**下添加如下代码
```
<role rolename="tomcat"/>
<role rolename="manager-script"/>
<role rolename="manager"/>
  <role rolename="admin"/>
  <role rolename="admin-gui"/>
  <role rolename="manager-gui"/>
  <user username="admin" password="123456" roles="admin,admin-gui,manager-gui,manager-script,manager-gui"/>
```
* tomcat8和tomcat9还需要在打开webapps/manager/META-INF/context.xml，把一行代码注释掉，如图：
* 参考：[https://my.oschina.net/xwzj/blog/900463](https://my.oschina.net/xwzj/blog/900463)

![图片](https://images-cdn.shimo.im/fORVH4i1198AtuWk/image.png!thumbnail)


* WAR/EAR files:是war包的相对路径，如target/xxx.war
* content path:Tomcat的发布路径,即项目的上下文，用于访问项目。如[http://localhost:8080/heo](http://localhost:8080/heo)，heo
* 即为content path。
* contaners :发布到的容器，主要可发布到tomcat、jboss、GlassFish
* deploy on failure：发生错误的时候是否发布到tomcat
## 码云Hook自动构建


* [https://gitee.com/oschina/Gitee-Jenkins-Plugin](https://gitee.com/oschina/Gitee-Jenkins-Plugin)

这里要利用的其实是码云的webhook功能。

![图片](https://images-cdn.shimo.im/ulMEFxMiB1cPXgsj/image.png!thumbnail)


1. **首先jenkins安装gitee插件，实现自动部署插件**

![图片](https://images-cdn.shimo.im/JryUeNB6040m76CT/image.png!thumbnail)


1. **添加码云链接配置**

* 前往 Jenkins -> Manage Jenkins -> Configure System -> Gitee Configuration -> Gitee connections
* 在 Connection name 中输入 Gitee 或者你想要的名字
* Gitee host URL 中输入码云完整 URL地址：[https://gitee.com](https://gitee.com/)（码云私有化客户输入部署的域名）
* Credentials 中如还未配置码云 APIV5 私人令牌，点击 Add - > Jenkins
    * Domain 选择 Global credentials
    * Kind 选择 Gitee API Token
    * Scope 选择你需要的范围
    * Gitee API Token 输入你的码云私人令牌，获取地址：[https://gitee.com/profile/personal_access_tokens](https://gitee.com/profile/personal_access_tokens)

![图片](https://images-cdn.shimo.im/pVSLE1bIP1kCXoSH/image.png!thumbnail)

生成的令牌：

![图片](https://images-cdn.shimo.im/AOuLZlZfPlwJoprY/image.png!thumbnail)


    * ID, Descripiton 中输入你想要的 ID 和描述即可。

![图片](https://images-cdn.shimo.im/nfScZO8HVBwSGjVE/image.png!thumbnail)


* Credentials 选择配置好的 Gitee APIV5 Token
* 点击 Advanced ，可配置是否忽略 SSL 错误（适您的Jenkins环境是否支持），并可设置链接测超时时间（适您的网络环境而定）
* 点击 Test Connection 测试链接是否成功，如失败请检查以上 3，5，6 步骤。

点击测试成功，标识gitee配置完成。

![图片](https://images-cdn.shimo.im/A0DZqOOIeIIPZNjv/image.png!thumbnail)

配置了gitee的链接之后，接下来就是使用。

分为以下几个步骤：

1、打开x项目的配置，然后Gitee链接处。选择刚才配置的链接id

![图片](https://images-cdn.shimo.im/KuPjKGIUOx4pWEzL/image.png!thumbnail)

2、配置构建触发器：

![图片](https://images-cdn.shimo.im/MsMjNg8nSIoJn66F/image.png!thumbnail)

生成hook密码。

![图片](https://images-cdn.shimo.im/MbWmhObtsgcqY6n8/image.png!thumbnail)

3、打开gitee上的项目，打开管理界面的webhooks。

url的配置需要注意一下，jenkins上建议的地址是*Gitee webhook 触发构建，需要在 Gitee webhook 中填写 URL:*[http://47.106.38.101:8080/project/homework](http://47.106.38.101:8080/project/homework)*。*但是我的jenkins端口其实是8081，所以需要改回来。然后把jenkins上生成的gitee webhook密码复制过来。

![图片](https://images-cdn.shimo.im/RsWaxKasRjYkR0XQ/image.png!thumbnail)

点击测试，会发现，jenkins会自动开始构建完成项目发布了。也就是说，以后只要发布代码到git上，jenkins就会自动构建了，完成了自动化过程。

![图片](https://images-cdn.shimo.im/isiOHZvwzn4B8lZk/image.png!thumbnail)


