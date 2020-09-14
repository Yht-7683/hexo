---
title: redis的高可用、主从与缓存的使用
date: 2020-09-14 14:53:04
tags: Redis
excerpt: 转载5、redis的高可用、主从与缓存的使用
---


## [2_redis的五大数据结构.xmind](https://uploader.shimo.im/f/h4TbcU17cTA7y2fs.xmind)[3_redis的高可用.xmind](https://uploader.shimo.im/f/MXRhSoojr4wOZqaV.xmind)[_课程5_redis简介_高可用_预习_.xmind](https://uploader.shimo.im/f/VZS9Mr2HPgcZRfMy.xmind)[【课程5】redis简介、高可用（预习）.xmind](https://uploader.shimo.im/f/7MkBBPBElvAgYmLP.xmind)[1_redis简介.xmind](https://uploader.shimo.im/f/SdPF9O42LHQormxr.xmind)

为什么需要缓存框架

那么我们第一节课中已经说明了一个网站演变的过程中，用户量的增加引起了并发量提高，如果不做处理，则频繁的查询数据库，结果是页面显示的慢，服务器、数据库不堪重负。如果网站页面所展示的数据的更新不是特别频繁，想提高页面显示的速度，减轻服务器的负担，此时应该考虑使用缓存。

所以，缓存是介于应用程序和物理数据源之间，其作用是为了降低应用程序对物理数据源访问的频次，从而提高了应用的运行性能。缓存内的数据是对物理数据源中的数据的复制，应用程序在运行时从缓存读写数据，在特定的时刻或事件会同步缓存和物理数据源的数据。

使用缓存有一个原则：**越高层次的缓存效果越好**。 推荐使用页面缓存。

## redis简介

redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)和zset(有序集合)、Hash(哈希类型的映射表)。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。

在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，Python，Ruby，Perl，PHP客户端，使用很方便。

### 特性

速度快


* 数据存在内存中
* c语言编写,5W行代码
* 单线程数据模型
* [https://baijiahao.baidu.com/s?id=1624003934114185747&wfr=spider&for=pc](https://baijiahao.baidu.com/s?id=1624003934114185747&wfr=spider&for=pc)

持久化

多种数据结构

支持多种编辑语言

功能丰富

简单

主从复制

## redis的安装

redis-server -- port 6380

### linux安装


* 下载链接：[https://redis.io/download](https://redis.io/download)
```
#解压安装
$ wget http://download.redis.io/releases/redis-5.0.4.tar.gz
$ tar xzf redis-5.0.4.tar.gz
$ cd redis-5.0.4
$ make
#启动
$ cd src
$ ./redis-server redis.conf
#试用
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```
安装过程中遇到以下错误说明gcc-c++环境未安装，直接运行yum -y install gcc-c++先安装环境：
```
[root@localhost redis-5.0.4]# make
cd src && make all
make[1]: 进入目录“/opt/redis-5.0.4/src”
    CC adlist.o
/bin/sh: cc: 未找到命令
make[1]: *** [adlist.o] 错误 127
make[1]: 离开目录“/opt/redis-5.0.4/src”
make: *** [all] 错误 2
```
遇到一下错误运行make distclean
```
[root@bogon redis-3.2.0]# make
cd src && make all
make[1]: 进入目录“/usr/local/redis-3.2.0/src”
CC adlist.o
In file included from adlist.c:34:0:
zmalloc.h:50:31: 致命错误：jemalloc/jemalloc.h：没有那个文件或目录
#include <jemalloc/jemalloc.h>                              ^
编译中断。
make[1]: *** [adlist.o] 错误 1
make[1]: 离开目录“/usr/local/redis-3.2.0/src”
make: *** [all] 错误 2
```
解决“jemalloc/jemalloc.h：没有那个文件或目录“问题，在进行编译（因为上次编译失败，有残留的文件）。直接运行make distclean命令。
### windows安装


* 下载链接：[https://github.com/MicrosoftArchive/redis/releases](https://github.com/MicrosoftArchive/redis/releases)

![图片](https://images-cdn.shimo.im/TBpPOKgqXBIKLiQj/image.png!thumbnail)

解压之后切换到安装目录运行**redis-server.exe redis.windows.conf**

### 客户端


* [https://pan.baidu.com/s/1wdE8Xxku0ghM6xP77O0k9w](https://pan.baidu.com/s/1wdE8Xxku0ghM6xP77O0k9w)

![图片](https://images-cdn.shimo.im/Pk5fcsPdaKAi6DeS/image.png!thumbnail)

## redis命令


* 请参考一下地址：[http://www.runoob.com/redis/redis-commands.html](http://www.runoob.com/redis/redis-commands.html)
* [http://doc.redisfans.com/](http://doc.redisfans.com/)
### scan命令

**SCAN cursor [MATCH pattern] [COUNT count]**


* [SCAN](http://doc.redisfans.com/key/scan.html#scan)命令用于迭代当前数据库中的数据库键。
* [SSCAN](http://doc.redisfans.com/set/sscan.html#sscan)命令用于迭代集合键中的元素。
* [HSCAN](http://doc.redisfans.com/hash/hscan.html#hscan)命令用于迭代哈希键中的键值对。
* [ZSCAN](http://doc.redisfans.com/sorted_set/zscan.html#zscan)命令用于迭代有序集合中的元素（包括元素成员和元素分值）。

一个基于游标的迭代器（cursor based iterator）：[SCAN](http://doc.redisfans.com/key/scan.html#scan)命令每次被调用之后， 都会向用户返回一个新的游标， 用户在下次迭代时需要使用这个新游标作为[SCAN](http://doc.redisfans.com/key/scan.html#scan)命令的游标参数， 以此来延续之前的迭代过程。

当[SCAN](http://doc.redisfans.com/key/scan.html#scan)命令的游标参数被设置为 0 时， 服务器将开始一次新的迭代， 而当服务器向用户返回值为 0 的游标时， 表示迭代已结束。

COUNT 参数的默认值为 10 。

![图片](https://uploader.shimo.im/f/JTPKCLHkjlQSFxet.png!thumbnail)

又比如：

![图片](https://uploader.shimo.im/f/OrI9nz9noBwY5beg.png!thumbnail)

### Info命令

查看redis服务运行信息，分为 9 大块，每个块都有非常多的参数，这 9 个块分别是


* Server 服务器运行的环境参数
* Clients 客户端相关信息
* Memory 服务器运行内存统计数据
* Persistence 持久化信息
* Stats 通用统计数据
* Replication 主从复制相关信息
* CPU CPU 使用情况
* Cluster 集群信息
* KeySpace 键值对统计数量信息
## redis的配置


* 参考[https://www.cnblogs.com/tmpt/p/redis_conf_detail_annotation.html](https://www.cnblogs.com/tmpt/p/redis_conf_detail_annotation.html)
```
# redis version 2.8.19
 
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
 
# 只用最新的，所以可以放到最后。
# include /path/to/local.conf
# include /path/to/other.conf
 
################################  通用 #####################################
 
# 是否后台执行
daemonize yes
 
# 后台执行的pid文件
# pidfile /var/run/redis.pid
 
# 0的话，不接受TCP连接
port 6379
 
# TCP listen() backlog. 虽然listen有两个参数int listen(int s, int backlog); 但是第二个参数会被/proc/sys/net/core/somaxconn覆盖。比如nginx设置的是511，但是也会被这个覆盖成默认的128,
# 所以要/etc/sysctl.conf中添加net.core.somaxconn = 2048 然后 sysctl -p ，就是说，如果软件设置大于linux配置，就是linux配置，软件设置小于linux，就用软件的，就是用最小的那个。
tcp-backlog 511
 
# 绑定IP请求来源
# bind 192.168.1.100 10.0.0.1
 
# 在空闲多少秒后关闭链接（0是禁用此功能）
timeout 0
 
# TCP keepalive
# Linux内核里，下边的值以秒记，相当于tcp_keepalive_time，要用两倍的这个时间才能杀死（画外音，也就是probes*intvl=如下的值了，详见EverNote搜索“linux 在线服务器优化配置”）
# 设成60比较好
tcp-keepalive 0
 
# 日志记录等级debug》verbose》notice（生产环境）》warning
loglevel notice
 
# 日志名。空字符串意味着输出到 标准输出。后台运行的redis标准输出是/dev/null。（画外音，所以要设置个文件名）
logfile ""
 
# 是否把log记到系统日志里。标示是什么？
# syslog-enabled no
# syslog-ident redis
 
#设置db的数量，默认db是0，你可以用SELECT <dbid> dbid在0到下边的值-1；
databases 16
 
################################ 快照 ################################
 
# 保存时间间隔，更新数量。如果1个key更新了，15min保存一次。10个key更新了，5分钟保存一次，10000个key更新了，每1分钟保存一次。主动调用SAVE()会阻塞所有客户端！一般是BGSAVE异步的。
save 900 1
save 300 10
save 60 10000
 
# 如果最后一次的后台保存RDB snapshot出错，redis就会拒绝所有写请求。这样也相当于一个报警吧。等后台保存继续工作后，redis就允许写了。
# 如果你自己配置好了redis的持久化进程的监控，你可以关闭下边：
stop-writes-on-bgsave-error yes
 
# 是否压缩dump后的   .rdb 数据库？默认压缩。会省硬盘，但耗CPU。
rdbcompression no
 
# 是否校验rdb快照？CRC64校验值会放在文件尾部。会导致10%性能下降。关闭后，校验值用0填充
rdbchecksum yes
 
# DB名称
dbfilename dump.rdb
 
# 工作目录
# DB会写入这个目录，以上边的名字。“仅追加文件”也会存在这个目录。注意：这里必须是目录名，不能是文件名！
dir ./
################################# 复制集 #################################
 
# 主从复制。用slaveof去复制另一份redis。
# 1）redis复制是异步的。但你可以让主redis拒绝写请求，当少于某个个数的从redis在线。
# 2）如果复制进程暂停了一小会儿，slave可以进行重新进行部分同步，你可以设置一下复制backlog大小
# 3）是自动的，无需干预。
 
# slaveof <masterip> <masterport>
 
# 如果主机需要鉴权，则需要配置密码
# masterauth <master-password>
 
# 当slave和master断了，会有两种情况：
# 1）默认：slave-serve-stale-data yes 这时，slave接受请求并返回老数据
# 2）如果是no了，则对任何命令都返回SYNC with master in progress，INFO和SLAVEOF命令除外！
slave-serve-stale-data yes
 
# 2.6之后，redis默认slave都是read-only的，但是slave默认可以执行所有管理员命令。CONFIG,DEBUG等。你可以用rename-command去重命名危险的命令，隐藏他们。
slave-read-only yes
 
# 复制集同步策略：磁盘或者socket
# 新slave连接或者老slave重新连接时候不能只接收不同，得做一个全同步。需要一个新的RDB文件dump出来，然后从master传到slave。可以有两种情况：
# 1）基于硬盘（disk-backed）：master创建一个新进程dump RDB，完事儿之后由父进程（即主进程）增量传给slaves。
# 2）基于socket（diskless）：master创建一个新进程直接dump RDB到slave的socket，不经过主进程，不经过硬盘。
# 基于硬盘的话，RDB文件创建后，一旦创建完毕，可以同时服务更多的slave。基于socket的话， 新slave来了后，得排队（如果超出了repl-diskless-sync-delay还没来），完事儿一个再进行下一个。
# 当用diskless的时候，master等待一个repl-diskless-sync-delay的秒数，如果没slave来的话，就直接传，后来的得排队等了。否则就可以一起传。
# disk较慢，并且网络较快的时候，可以用diskless。（默认用disk-based）
repl-diskless-sync no
# 设置成0的话，传输开始ASAP
repl-diskless-sync-delay 5
 
# Slave发送ping给master。默认10s
# repl-ping-slave-period 10
 
# 超时时间，包括从master看slave，从slave看master，要大于上边的repl-ping-slave-period
# repl-timeout 60
 
# SYNC完毕后，在slave的socket里关闭TCP_NODELAY。
# 如果是yes,reids发送少量的TCP包给slave，但可能导致最高40ms的数据延迟。
# 如果是no，那可能在复制的时候，会消耗 少量带宽。
# 默认我们是为了低延迟优化而设置成no，如果主从之间有很多网络跳跃。那设置成yes吧。
repl-disable-tcp-nodelay no
 
# 复制集后台backlog大小
# 越大，slave可以丢失的时间就越长。
# repl-backlog-size 1mb
 
# 多久释放backlog，当确认master不再需要slave的时候，多久释放。0是永远不释放。
# repl-backlog-ttl 3600
 
# 当master不可用，Sentinel会根据slave的优先级选举一个master。最低的优先级的slave，当选master。而配置成0，永远不会被选举。（必须≥0）。默认是100
slave-priority 100
 
# slave小于几个，网络lag大于几秒的时候，master停止接受write请求。默认对slave数目无限制，给0。网络延迟给10s
# min-slaves-to-write 3
min-slaves-max-lag 10
 
################################## 安全 ###################################
# 多数情况下无需密码鉴别slave。同时，由于redis处理速度太快，所以爆破速率可达150K/S。10万/S。所以如果你要设置密码，必须设置超强的密码。
# requirepass foobared <--这就是密码
 
# 命令重命名
# 在一个shared环境里，可以对危险的命令，比如CONFIG，进行重命名：也可以用空字符串，达到完全屏蔽此命令的目的。
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
# rename-command CONFIG ""
# 记录进AOF或者传给slave的重命名操作可能会引发问题哦~。
 
################################### 限制 ####################################
# 设置最大client连接数。默认10000一万个。如果redis没法控制最大文件数。则给到最低32.
# maxclients 10000
 
# 如果redis用内存超过了设置的限制，第一，开始用maxmemory-policy配置的策略往外删数据，如果配置成了noeviction。所有write都会拒绝，比如set，lpush等。所有读请求可以接受。
# 主要用在把redis用在LRU缓存，或者用在一个内存吃紧又不能删除的策略上。
# 如果你有slave，你应该把最大内存别设置的太大，留一些系统内存给slave output buffers（如果是noeviction策略，就无需这样设置了）
# maxmemory <bytes>
 
# 内存策略。
# volatile-lru ->用LRU删除设置了ttl的key
# allkeys-lru ->用LRU删除任何key
# volatile-random ->随机删除有ttl的key
# allkeys-random ->随机删除任何key
# volatile-ttl ->删除即将ttl到期的key
# noeviction ->不删，有write的时候报错。
# 如下操作会返回错误
#       set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
# 默认是
# maxmemory-policy volatile-lru
 
# LRU和最小TTL并不是最精确的，但是差不多了也。默认redis每次取3个key然后取最符合删除策略的删除。你可以配置这个数。越低，删除的东西就会越多。比如设置100个，就能删百分之一。
# maxmemory-samples 3
 
############################## AOF ###############################
# 默认redis异步的dump数据到disk。但如果断电了，那么就会丢失一部分数据了（根据save的配置）。
# AOF提供更好模式。比如用默认的AOF，redis只丢失最近一秒的数据（断电情况），或者最后一个write操作（redis自身错误，os正常）。每个write操作写一次AOF。
# 当AOF文件太大了，redis会自动重写一个aof文件出来。
# AOF和RDB持久化可以同时启用。redis会优先读AOF恢复数据。
# Please check http://redis.io/topics/persistence for more information
appendonly no
 
# 默认文件名
appendfilename "appendonly.aof"
 
# fsync()三种：
# no：让OS托管，这样更快。
# always：每次write都刷到log，慢，最安全。
# everysec：每秒一次flush。（默认）
# http://antirez.com/post/redis-persistence-demystified.html
# appendfsync always
appendfsync everysec
# appendfsync no
 
# 当fsync为always或者everysec，当一个bgsave或者AOF rewrite线程正在耗费大量I/0，redis可能会在fsync上阻塞很久。发生之后就无法fix，即使是另一个线程跑fsync，也会阻塞我们同步的write方法。
# 如下方法可以解决这个问题：当bgsave()或bgrewriteaof()在跑，主进程的fsync()就无法调用。也就是当子进程在save，那段时光相当于redis是appendaof no的。也就是有可能会丢失最多30s的log。
# 所以如果你有lag问题，把下边改成yes，否则就用no。yes意思是暂停aof，拒绝主进程的这次fsync。no是redis是排队的，不会被prevent了，但主进程是阻塞的。
no-appendfsync-on-rewrite no
 
# 自动重写AOF
# 当AOF文件大小到一定比例，就自动隐式调用BGREWRITEAOF
# 过程：redis记住最后一次rewrite时aof文件大小（重启后没rewrite的话，就是启动时AOF文件的大小），如果现在AOF大小和上次的比例达到特定值就重写。也要指定最小AOF大小，防止到2倍：1M的时候也重写。
# 把percentage改成0，就是禁用重写。
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
 
# AOF文件可能在尾部是不完整的（上次system关闭有问题，尤其是mount ext4文件系统时没有加上data=ordered选项。只会发生在os死时，redis自己死不会不完整）。那redis重启时load进内存的时候就有问题了。
# 发生的时候，可以选择redis启动报错，或者load尽量多正常的数据。
# 如果aof-load-truncated是yes，会自动发布一个log给客户端然后load（默认）。如果是no，用户必须手动redis-check-aof修复AOF文件才可以。
aof-load-truncated yes
 
 
################################ LUA SCRIPTING  ###############################
# 如果达到最大时间限制（毫秒），redis会记个log，然后返回error。
# 当一个脚本超过了最大时限。只有SCRIPT KILL和SHUTDOWN NOSAVE可以用。第一个可以杀没有调write命令的东西。要是已经调用了write，只能用第二个命令杀。
# 设置成0或者负值，时限就无限。
lua-time-limit 5000
 
################################## SLOW LOG ###################################
# 线程阻塞不能服务其他请求的时间长度。两个参数：第一个是时长（以微秒为单位！，是毫秒的千分之一。）。第二个是log的size，超过了，就会删除之前的log。
# 1000000是一秒。负值是所有请求都记log！下边是0.10S。100毫秒。
slowlog-log-slower-than 10000
 
# log长度的设置值是没限制。但是需要内存。
slowlog-max-len 128
 
################################ LATENCY MONITOR ##############################
# 用LATENCY打印redis实例在跑命令时的耗时图表。
# 只记录大于等于下边设置的值的操作。0的话，就是关闭监视。可以动态开启。直接运行CONFIG SET latency-monitor-threshold <milliseconds>
latency-monitor-threshold 0
 
############################# Event notification ##############################
# 可以通知pub/sub客户端关于key空间的变化。http://redis.io/topics/notifications
# 比如如果开着开关。一个client进行了DEL操作在“foo”key上在database0上。两个消息将会发布通过 pub/sub
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
# 大部分人不需要这个功能，并且还需要一定开销，所以默认关闭。
notify-keyspace-events ""
 
############################### ADVANCED CONFIG ###############################
# hash结构存储，小数据量的用数组，大数据量用map（encoding保存结构信息）
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
 
# list同上。
list-max-ziplist-entries 512
list-max-ziplist-value 64
 
# Set在一种情况下会用特殊encoding：整个set是string组成，但是突然需要变成64位带符号整数且是10为根。。不懂。
set-max-intset-entries 512
 
# zset同set
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
 
# HyperLogLog 不懂。大于16000完全不可接受！当CPU很顶得住的话，给10000可以。默认给3000.
hll-sparse-max-bytes 3000
 
# Active rehashing 越多次的操作进入了正在进行rehash的table，越多的rehash步骤需要执行。如果redis是空闲的，那么rehash操作是永远没法停止的，越多的内存也被消耗了。
# 默认就用yes就行 了如果你想释放内存ASAP。
activerehashing yes
 
# client output buffer限制，可以用来强制关闭传输缓慢的客户端（比如redis pub的东西有比较慢的client无法及时sub）
# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
# class可以为以下：
#
# normal -> normal clients including MONITOR clients
# slave  -> slave clients
# pubsub -> clients subscribed to at least one pubsub channel or pattern
# 当hard限制到了会立即被关闭客户端。如果soft限制到了，会等soft秒。
# 比如硬限制是32m，soft是16m，10secs。到32m就立即断，或者在16m以上停止了10secs。
# 设置成0就是关闭。
client-output-buffer-limit normal 0 0 0 
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
 
# redis内部调度（进行关闭timeout的客户端，删除过期key等等）频率，越大则调度频率越高。设置成100以上会对CPU造成大压力除非你对线上实时性要求很高。可以在1~500之间。
hz 10
 
# 当child进程在rewrite AOF文件，如果这个选项是yes，那么这个file每32MB会写fsync()。这个是保证增量写硬盘而防止写硬盘时I/O突增。
aof-rewrite-incremental-fsync yes
```
## redis单线程

为什么这么快？


* 纯内存操作
* 非阻塞IO
* 避免线程切换和竞态消耗

如何使用？


* 拒绝长（慢）命令
    * keys、flushall。。
* 一次只运行一个命令
## redis存储机制


* [https://www.cnblogs.com/yyjie/p/7487937.html](https://www.cnblogs.com/yyjie/p/7487937.html)

Redis存储机制分成两种Snapshot和AOF。无论是那种机制,Redis都是将数据存储在内存中。

### Snapshot工作原理

是将数据先存储在内存，然后当数据累计达到某些设定的伐值的时候，就会触发一次DUMP操作，将变化的数据一次性写入数据文件（RDB文件）。

![图片](http://img.blog.csdn.net/20170210143107403?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSFVYVTk4MTU5ODQzNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### AOF 工作原理

是将数据也是先存在内存，但是在存储的时候会使用调用fsync来完成对本次写操作的日志记录，这个日志揭露文件其实是一个基于Redis网络交互协议的文本文件。AOF调用fsync也不是说全部都是无阻塞的，在某些系统上可能出现fsync阻塞进程的情况，对于这种情况可以通过配置修改，但默认情况不要修改。AOF最关键的配置就是关于调用fsync追加日志文件的平率，有两种预设频率，always每次记录进来都添加，everysecond 每秒添加一次。两个配置各有所长后面分析。由于是采用日志追加的方式来持久话数据，所以引出了第二个日志的概念：rewrite. 后面介绍它的由来。

![图片](http://img.blog.csdn.net/20170210143206336?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSFVYVTk4MTU5ODQzNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### 存储模式性能和安全比较

**1.性能**

Snapshot方式的性能是要明显高于AOF方式的，原因有两点：

采用2进制方式存储数据，数据文件比较小，加载快速。

存储的时候是按照配置中的save策略来存储，每次都是聚合很多数据批量存储，写入的效率很好，而AOF则一般都是工作在实时存储或者准实时模式下。相对来说存储的频率高，效率却偏低。

**2.数据安全**

AOF数据安全性高于Snapshot存储，原因：

Snapshot存储是基于累计批量的思想，也就是说在允许的情况下，累计的数据越多那么写入效率也就越高，但数据的累计是靠时间的积累完成的，那么如果在长时间数据不写入RDB，但Redis又遇到了崩溃，那么没有写入的数据就无法恢复了，但是AOF方式偏偏相反，根据AOF配置的存储频率的策略可以做到最少的数据丢失和较高的数据恢复能力。

说完了性能和安全，这里不得不提的就是在Redis中的Rewrite的功能，AOF的存储是按照记录日志的方式去工作的，那么成千上万的数据插入必然导致日志文件的扩大，Redis这个时候会根据配置合理触发Rewrite操作，所谓Rewrite就是将日志文件中的所有数据都重新写到另外一个新的日志文件中，但是不同的是，对于老日志文件中对于Key的多次操作，只保留最终的值的那次操作记录到日志文件中，从而缩小日志文件的大小。这里有两个配置需要注意：

auto-aof-rewrite-percentage 100 （当前写入日志文件的大小占到初始日志文件大小的某个百分比时触发Rewrite）

auto-aof-rewrite-min-size 64mb （本次Rewrite最小的写入数据良）

两个条件需要同时满足。

## redis的数据结构、使用场景

![图片](https://images-cdn.shimo.im/7do43jkNXXIobvYV/image.png!thumbnail)

![图片](https://uploader.shimo.im/f/bKLB0GR4iUMSqsxa.png!thumbnail)

### 字符串

缓存、计数器、分布式锁、分布式ID、、

![图片](https://uploader.shimo.im/f/nEUlBPzOKpIy2wm7.png!thumbnail)

### HASH

存储用户信息、用户主页访问量

![图片](https://images-cdn.shimo.im/cH10C0fSUkIoimcI/image.png!thumbnail)

![图片](https://uploader.shimo.im/f/ZrBnyYjNdBEpjAZh.png!thumbnail)

### LIST

微博关注人微博时间轴列表

### SET

赞、踩、标签

### ZSET

排行榜

## redis的主从模式

### 主从配置

主从机器一般是分开的两台机器，这里为了演示方便，使用了一台机器，所以配置文件中，处理端口的区分之外，还需要注意日志文件，数据库文件，pid文件等的区分。

### 启动redis命令

redis-server redis.conf

*直接自定加载的配置文件即可*

**主机配置**

复制一份redis.conf成redis-6379.conf。修改配置：

```
#演示方便，开放ip连接
bind 0.0.0.0
#后台运行
daemonize yes
#pid文件
pidfile /var/run/redis_6379.pid
dbfilename dump-6379.rdb
#日志文件
logfile "6379.log"
```
**从机配置**

复制一份redis.conf成redis-6380.conf。修改配置：

```
bind 0.0.0.0
port 6380
daemonize yes
pidfile /var/run/redis_6380.pid
dbfilename dump-6380.rdb
logfile "6380.log"
#slaveof表示作为从库的配置
slaveof 127.0.0.1 6379
#从库只能读操作
slave-read-only yes
```
### 查看redis的信息

redis-cli -p 6379 info replication


* 其中6379表示指定端口
* replication表示主从的信息

查看主库信息：

```
[root@izwz95t3hfncu7j54o7zefz src]# ./redis-cli -p 6379 info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=1183,lag=0
master_replid:32125185597f85a2155d187ccd57168cf6681395
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1183
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1183
```
查看从库信息：
```
[root@izwz95t3hfncu7j54o7zefz src]# ./redis-cli -p 6380 info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:1239
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:32125185597f85a2155d187ccd57168cf6681395
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1239
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1239
```
查看配置效果：复制成功！！

```
[root@izwz95t3hfncu7j54o7zefz src]# ./redis-cli -p 6379 set hello helloworld
OK
[root@izwz95t3hfncu7j54o7zefz src]# ./redis-cli -p 6380 get hello
"helloworld"
```
## redis的高可用

### sentinel启动命令

redis-sentinel sentinel.conf

### sentinel机制

![图片](https://images-cdn.shimo.im/Ooe0kYLSfIkTVhnX/image.png!thumbnail)

过程：


1. 多个sentinel发现并确认master有问题
2. 选举出一个sentinel作为领导
3. 选出一个slave作为master
4. 通知其余slave成为新的master的slave
5. 通知客户端主从变化
6. 等待老的master复活成为新的master的slave
### **创建sentinel.conf配置文件**


1. port26379
2. daemonize yes
3. # sentinel announce-ip <ip>
4. # sentinel announce-port <port>
5. dir/opt/redis-5.0.4/data
6. logfile "26379.log"
7. sentinel monitor mymaster 127.0.0.16379 2
8. # sentinel auth-pass <master-name> <password>
9. sentinel down-after-milliseconds mymaster30000
10. sentinel parallel-syncs mymaster1
11. sentinel failover-timeout mymaster180000
12. # sentinel notification-script <master-name> <script-path>

附课程的配置文件：

[redis-6379.conf](https://uploader.shimo.im/f/HnSnIQQSGDQotki4.conf)[redis-6380.conf](https://uploader.shimo.im/f/lbgQQRyBkbM8wrCP.conf)[redis-6381.conf](https://uploader.shimo.im/f/ZzZpySSG0KcfHrDY.conf)[sentinel-26379.conf](https://uploader.shimo.im/f/JPXbq1UE21cUWroZ.conf)[sentinel-26380.conf](https://uploader.shimo.im/f/7dbdAEXL5Z45P3fG.conf)[sentinel-26381.conf](https://uploader.shimo.im/f/E2rCkoUgVOkHZwTC.conf)

**配置文件说明：**

**1.****port**:当前Sentinel服务运行的端口

**2.****dir**: Sentinel服务运行时使用的临时文件

**3.****sentinel monitor master001****192.168****.****110.101 6379 2**:Sentinel去监视一个名为master001的主redis实例，这个主实例的IP地址为本机地址192.168.110.101，端口号为6379，而将这个主实例判断为失效至少需要2个 Sentinel进程的同意，只要同意Sentinel的数量不达标，自动failover就不会执行

**4.****sentinel down****-****after****-****milliseconds master001****30000**:指定了Sentinel认为Redis实例已经失效所需的毫秒数。当实例超过该时间没有返回PING，或者直接返回错误，那么Sentinel将这个实例标记为主观下线。只有一个 Sentinel进程将实例标记为主观下线并不一定会引起实例的自动故障迁移：只有在足够数量的Sentinel都将一个实例标记为主观下线之后，实例才会被标记为客观下线，这时自动故障迁移才会执行

**5.****sentinel parallel****-****syncs master001****1**：指定了在执行故障转移时，最多可以有多少个从Redis实例在同步新的主实例，在从Redis实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长

**6.****sentinel failover****-****timeout master001****180000**：如果在该时间（ms）内未能完成failover操作，则认为该failover失败

**7.****sentinel notification-script <master-name> <script-path>**：指定sentinel检测到该监控的redis实例指向的实例异常时，调用的报警脚本。该配置项可选，但是很常用

### 三个定时任务

（1）每隔10s每个sentinel会对master节点和slave节点执行info命令

作用就是发现slave节点，并且确认主从关系，因为redis-Sentinel节点启动的时候是知道

master节点的，只是没有配置相应的slave节点的信息

（2）每隔两秒，sentinel都会通过master节点内部的channel来交换信息（基于发布订阅）

作用是通过master节点的频道来交互每个Sentinel对master节点的判断信息

（3） 每隔一秒每个sentinel对其他的redis节点（master，slave，sentinel）执行ping操作，对于master来说，若超过30s内没有回复，就对该master进行主观下线并询问其他的Sentinel节点是否可以客观下线

### 故障转移过程


* [https://www.cnblogs.com/Xrinehart/p/3502213.html](https://www.cnblogs.com/Xrinehart/p/3502213.html)

**1. 领导者选举**

作用：选举出一个sentenel节点作为领导者去进行故障转移操作。

过程：

1). 每个做主观下线的sentinel节点向其他sentinel节点发送上面那条命令，要求将它设置为领导者。

2). 收到命令的sentinel节点如果还没有同意过其他的sentinel发送的命令（还未投过票），那么就会同意，否则拒绝。

3). 如果该sentinel节点发现自己的票数已经过半且达到了quorum的值，就会成为领导者。

4). 如果这个过程出现多个sentinel成为领导者，则会等待一段时间重新选举。

![图片](https://uploader.shimo.im/f/TUnoZjIooiAkPa1f.png!thumbnail)

**2. 选出新的master节点**

redis sentinel会选一个合适的slave来升级为master，那么，如何选择一个合适的slave呢？顺序如下：

1). 选择slave-priority最高的slave节点（默认是相同）。

2). 选择复制偏移量最大的节点。

3). 如果以上两个条件都不满足，选runId最小的（启动最早的）。

**3. 更改slave节点的master节点**

当选举出新的master节点后，会将其余的节点变更为新的master节点的slave节点，如果原有的master节点重新上线，成为新的master节点的slave节点。

**4. 通知客户端**

当所有节点配置结束后，sentinel会通知客户端节点变更信息。

**5. 客户端连接新的master节点**

客户端收到节点信息后，会连接新的master节点。

![图片](https://uploader.shimo.im/f/NuZFhubnubM60KPs.png!thumbnail)

