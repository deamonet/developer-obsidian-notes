-   [![博客园Logo](https://common.cnblogs.com/logo.svg)](https://www.cnblogs.com/ "开发者的网上家园")
-   [首页](https://www.cnblogs.com/)
-   [新闻](https://news.cnblogs.com/)
-   [博问](https://q.cnblogs.com/)
-   [专区](https://brands.cnblogs.com/)
-   [闪存](https://ing.cnblogs.com/)
-   [班级](https://edu.cnblogs.com/)

-   

-   -   ![搜索](https://common.cnblogs.com/images/blog/search.svg)
        
        所有博客
    -   ![搜索](https://common.cnblogs.com/images/blog/search.svg)
        
        当前博客
    
-   [注册](https://account.cnblogs.com/signup) 登录

[满格](https://www.cnblogs.com/lijiaman/)

-   [博客园](https://www.cnblogs.com/)
-   [首页](https://www.cnblogs.com/lijiaman/)
-   [新随笔](https://i.cnblogs.com/EditPosts.aspx?opt=1)
-   [联系](https://msg.cnblogs.com/send/gegeman)

-   [管理](https://i.cnblogs.com/)

随笔 - 197  文章 - 0  评论 - 41  阅读 - 60万

# [在一台Linux服务器上安装多个MySQL实例（一）--使用mysqld_multi方式](https://www.cnblogs.com/lijiaman/p/12587630.html)

[![image](https://img2020.cnblogs.com/blog/823295/202003/823295-20200328153036979-1548275162.png "image")](https://img2020.cnblogs.com/blog/823295/202003/823295-20200328153036664-113480935.png)

**（一）MySQL多实例概述**

实例是进程与内存的一个概述，所谓MySQL多实例，就是在服务器上启动多个相同的MySQL进程，运行在不同的端口（如3306，3307，3308），通过不同的端口对外提供服务。

由于MySQL在一个实例下面可以创建多个数据库，所以通常在一台服务器上只要安装一个MySQL实例即可满足使用。但在实际使用中，因为服务器硬件资源充足，或者业务需要（比如在一台服务器上创建开发数据库和测试数据库），往往会在一台服务器上创建多个实例。

**（二）MySQL部署多实例的方法**

MySQL多实例部署主要有以下两种方式：

-   使用官方自带的mysqld_multi来配置管理，特点是使用同一份MySQL配置文件，这种方式属于集中式管理，管理起来较为方便；
-   使用单独的MySQL配置文件来单独配置实例，这种方式逻辑简单，数据库之间没有关联。

本文将对第一种方式进行环境搭建学习。

**（三）**实验环境****

操作系统   ：CentOS Linux release 7.4.1708 (Core)

数据库版本：5.7.24-log

预计划安装4个MySQL实例，规划信息为：

**实例1**

**实例2**

**实例3**

**实例4**

basedir=/usr/local/mysql

datadir=/mysql/3306/data

port=3306

socket=/tmp/mysql_3306.sock

basedir=/usr/local/mysql

datadir=/mysql/3307/data

port=3307

socket=/tmp/mysql_3307.sock

basedir=/usr/local/mysql

datadir=/mysql/3308/data

port=3308

socket=/tmp/mysql_3308.sock

basedir=/usr/local/mysql

datadir=/mysql/3309/data

port=3309

socket=/tmp/mysql_3309.sock

**（四）实验过程**

（4.1）在安装MySQL之前，需要卸载服务器自带的MySQL包和MySQL数据库分支mariadb的包

[root@masterdb ~]# rpm -qa|grep mysql
[root@masterdb ~]# rpm -qa |grep mariadb
mariadb-libs-5.5.56-2.el7.x86_64

[root@masterdb ~]# rpm -e mariadb-libs-5.5.56-2.el7.x86_64 --nodeps

（4.2）依赖包安装

MySQL对libaio 库有依赖性。如果未在本地安装该库，则数据目录初始化和随后的服务器启动步骤将失败 、

# install library

[root@mysql mysql]# yum install libaio

对于MySQL 5.7.19和更高版本：通用Linux版本中增加了对非统一内存访问（NUMA）的支持，该版本现在对libnuma库具有依赖性 。

# install library

 
[root@mysql mysql]# yum install libnuma

（4.3）创建用户和用户组

[root@masterdb ~]# groupadd mysql
[root@masterdb ~]# useradd -r -g mysql -s /bin/false mysql

（4.4）解压安装包

[root@masterdb ~]# cd /usr/local/
[root@masterdb local]# tar xzvf /root/mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz

# 修改解压文件名，与前面定义的basedir相同
[root@masterdb local]# mv mysql-5.7.24-linux-glibc2.12-x86_64/ mysql

最终解压结果如下：

![复制代码](media/复制代码.gif)

[root@masterdb mysql]# ls -l
 total 36
 drwxr-xr-x  2 root root   4096 Mar 28 13:48 bin
 -rw-r--r--  1 7161 31415 17987 Oct  4  2018 COPYING
 drwxr-xr-x  2 root root     55 Mar 28 13:48 docs
 drwxr-xr-x  3 root root   4096 Mar 28 13:48 include
 drwxr-xr-x  5 root root    230 Mar 28 13:48 lib
 drwxr-xr-x  4 root root     30 Mar 28 13:48 man
 -rw-r--r--  1 7161 31415  2478 Oct  4  2018 README
 drwxr-xr-x 28 root root   4096 Mar 28 13:48 share
 drwxr-xr-x  2 root root     90 Mar 28 13:48 support-files

![复制代码](media/复制代码.gif)

（4.5）创建数据文件存放路径

![复制代码](media/复制代码.gif)

[root@masterdb mysql]# mkdir -p /mysql/{3306,3307,3308,3309}/data
[root@masterdb mysql]# chown -R mysql:mysql /mysql 
[root@masterdb mysql]# cd /mysql
[root@masterdb mysql]# tree
.
├── 3306
│   └── data
├── 3307
│   └── data
├── 3308
│   └── data
└── 3309
    └── data

![复制代码](media/复制代码.gif)

（4.6）创建MySQL参数配置文件  

![复制代码](media/复制代码.gif)

[root@masterdb mysql]# vim /etc/my.cnf 

[mysqld]
user=mysql
basedir = /usr/local/mysql

[mysqld_multi]
mysqld=/usr/local/mysql/bin/mysqld_safe
mysqladmin=/usr/local/mysql/bin/mysqladmin
log=/usr/local/mysql/mysqld_multi.log

[mysqld3306]
mysqld=mysqld
mysqladmin=mysqladmin
datadir=/mysql/3306/data
port=3306
server_id=3306
socket=/tmp/mysql_3306.sock
log-error = /mysql/3306/error_3306.log

[mysqld3307]
mysqld=mysqld
mysqladmin=mysqladmin
datadir=/mysql/3307/data
port=3307
server_id=3307
socket=/tmp/mysql_3307.sock
log-error=/mysql/3307/error_3307.log

[mysqld3308]
mysqld=mysqld
mysqladmin=mysqladmin
datadir=/mysql/3308/data
port=3308
server_id=3308
socket=/tmp/mysql_3308.sock
log-error=/mysql/3308/error_3308.log

[mysqld3309]
mysqld=mysqld
mysqladmin=mysqladmin
datadir=/mysql/3309/data
port=3309
server_id=3309
socket=/tmp/mysql_3309.sock
log-error = /mysql/3309/error_3309.log

![复制代码](media/复制代码.gif)

（4.7）初始化数据库

注意，初始化实例的最后一行记录了root的初始密码

![复制代码](media/复制代码.gif)

# 初始化3306实例
[root@masterdb mysql]# /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --initialize --basedir=/usr/local/mysql/ --datadir=/mysql/3306/data
2020-03-28T06:10:28.484174Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2020-03-28T06:10:28.689102Z 0 [Warning] InnoDB: New log files created, LSN=45790
2020-03-28T06:10:28.723881Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2020-03-28T06:10:28.781205Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: d29ad574-70ba-11ea-a38f-000c29fb6200.
2020-03-28T06:10:28.782195Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2020-03-28T06:10:28.783078Z 1 [Note] A temporary password is generated for root@localhost: YuJ6Bi=PtqCJ

# 初始化3307实例 
[root@masterdb mysql]# /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --initialize --basedir=/usr/local/mysql/ --datadir=/mysql/3307/data
2020-03-28T06:10:45.598676Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2020-03-28T06:10:45.793277Z 0 [Warning] InnoDB: New log files created, LSN=45790
2020-03-28T06:10:45.829673Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2020-03-28T06:10:45.886255Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: dcccdb2f-70ba-11ea-a565-000c29fb6200.
2020-03-28T06:10:45.887571Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2020-03-28T06:10:45.890477Z 1 [Note] A temporary password is generated for root@localhost: &s)nYg.e4qx#
[root@masterdb mysql]# 

# 初始化3308实例
[root@masterdb mysql]# /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --initialize --basedir=/usr/local/mysql/ --datadir=/mysql/3308/data
2020-03-28T06:10:55.237714Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2020-03-28T06:10:55.442794Z 0 [Warning] InnoDB: New log files created, LSN=45790
2020-03-28T06:10:55.479012Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2020-03-28T06:10:55.534839Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: e28d1d57-70ba-11ea-a5c4-000c29fb6200.
2020-03-28T06:10:55.535622Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2020-03-28T06:10:55.536387Z 1 [Note] A temporary password is generated for root@localhost: Mz<kr!vsh1yj
[root@masterdb mysql]# 

# 初始化3309实例
[root@masterdb mysql]# /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --initialize --basedir=/usr/local/mysql/ --datadir=/mysql/3309/data
2020-03-28T06:11:05.644331Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2020-03-28T06:11:05.840498Z 0 [Warning] InnoDB: New log files created, LSN=45790
2020-03-28T06:11:05.879941Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2020-03-28T06:11:05.936262Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: e8c03ed2-70ba-11ea-a8fb-000c29fb6200.
2020-03-28T06:11:05.937179Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2020-03-28T06:11:05.937877Z 1 [Note] A temporary password is generated for root@localhost: K.KLa30i-sv3

![复制代码](media/复制代码.gif)

（4.8）设置环境变量

添加了环境变量，操作系统才能够自己找到mysql、mysqld_multi等命令的位置

![复制代码](media/复制代码.gif)

[root@masterdb mysql]# vim /etc/profile
# 在文件末尾添加下面信息
export PATH=/usr/local/mysql/bin:$PATH

#使环境变量生效
[root@masterdb mysql]# source /etc/profile

![复制代码](media/复制代码.gif)

（4.9）使用mysqld_multi管理多实例

![复制代码](media/复制代码.gif)

# 使用mysqld_multi启动3306端口的实例
[root@masterdb mysql]# mysqld_multi start 3306

# 使用mysqld_multi启动全部实例
[root@masterdb mysql]# mysqld_multi start

# 使用mysqld_multi查看实例状态
[root@masterdb mysql]# mysqld_multi report
Reporting MySQL servers
MySQL server from group: mysqld3306 is running
MySQL server from group: mysqld3307 is running
MySQL server from group: mysqld3308 is running
MySQL server from group: mysqld3309 is running

![复制代码](media/复制代码.gif)

使用mysqld_multi关闭实例较为麻烦，需要配置密码，因此如何关闭各个实例，见后面章节：**（六）关闭多实例数据库 。**

**（五）访问多实例数据库**

（5.1）登录MySQL数据库

在安装完成并启动数据库后，需要去访问各个MySQL实例，这里非常有意思，经常会发现无法连接到数据库上，我们不妨看一下几种连接方式：

连接方式一：使用服务器IP地址，无法连接。这里还是比较好理解的，MySQL创建完成后，数据库账号[root@localhost](mailto:root@localhost)只允许本地连接，参数“-h”后面用服务器IP被认为了远程连接，因此无法登陆

[root@masterdb mysql]# mysql -uoot -p -h192.168.10.11 -P3306
Enter password: 
ERROR 1130 (HY000): Host 'masterdb' is not allowed to connect to this MySQL server

连接方式二：使用localhost访问数据库，无法连接。我觉得有些匪夷所思，可以看到，MySQL实例使用的socket文件不对

[root@masterdb mysql]# mysql -uroot -p -hlocalhost -P3306
Enter password: 
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

连接方式三：使用127.0.0.1访问数据库，可以连接。有些难以理解，理论上127.0.0.1和localhost是对应的，127.0.0.1可以访问数据库，但是localhost却无法访问

![复制代码](media/复制代码.gif)

[root@masterdb mysql]# mysql -uroot -p -h127.0.0.1 -P3306
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.7.24 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> exit
Bye

![复制代码](media/复制代码.gif)

连接方式四：使用socket文件连接，可以正常访问

![复制代码](media/复制代码.gif)

[root@masterdb mysql]# mysql -S /tmp/mysql_3306.sock -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.24

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

![复制代码](media/复制代码.gif)

（5.2）修改数据库[root@localhost](mailto:root@localhost)密码

初次登陆MySQL数据库，需要修改root密码，否则无法正常使用

![复制代码](media/复制代码.gif)

[root@masterdb mysql]# mysql -S /tmp/mysql_3306.sock -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.24

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

-- 无法查询
mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.

-- 修改root@localhost用户的密码
mysql> alter user  root@localhost identified by '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye

![复制代码](media/复制代码.gif)

**（六）关闭多实例数据库**

（6.1）直接使用mysqld_multi来关闭实例

使用mysqld_multi关闭多实例数据库目前来看比较麻烦，需要在my.cnf文件的[mysqld_multi]模块里面配置用户密码，并且各个数据库的用户密码都需要相同，否则无法关闭。

[![image](https://img2020.cnblogs.com/blog/823295/202003/823295-20200328153037617-105938576.png "image")](https://img2020.cnblogs.com/blog/823295/202003/823295-20200328153037364-292329049.png)

我们可以看一下使用mysqld_multi来关闭数据库实例的日志：

![复制代码](media/复制代码.gif)

[root@masterdb mysql]# cat /usr/local/mysql/mysqld_multi.log 

# 当执行：mysqld_multi report时，显示所有数据库均在运行
Reporting MySQL servers
MySQL server from group: mysqld3306 is running
MySQL server from group: mysqld3307 is running
MySQL server from group: mysqld3308 is running
MySQL server from group: mysqld3309 is running
mysqld_multi log file version 2.16; run: Sat Mar 28 14:55:16 2020

# 当执行：mysqld_multi stopt时，mysqld_multi会调用mysqladmin去关闭数据库，使用的是[mysqld_multi]里面配置的账号密码，此时3306的密码是正确的，

# 其它都是错误的，因此3306关闭成功，而其它端口的实例因为密码错误而连接数据库失败，自然没有关闭数据库
Stopping MySQL servers

mysqladmin: [Warning] Using a password on the command line interface can be insecure.
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
mysqladmin: connect to server at 'localhost' failed
error: 'Access denied for user 'root'@'localhost' (using password: YES)'
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
mysqladmin: connect to server at 'localhost' failed
error: 'Access denied for user 'root'@'localhost' (using password: YES)'
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
mysqladmin: connect to server at 'localhost' failed
error: 'Access denied for user 'root'@'localhost' (using password: YES)'
mysqld_multi log file version 2.16; run: Sat Mar 28 14:55:21 2020

# 结果：仅仅关闭了密码正确的3306端口数据库
Reporting MySQL servers
MySQL server from group: mysqld3306 is not running
MySQL server from group: mysqld3307 is running
MySQL server from group: mysqld3308 is running
MySQL server from group: mysqld3309 is running
mysqld_multi log file version 2.16; run: Sat Mar 28 14:58:07 2020

![复制代码](media/复制代码.gif)

既然知道了mysqld_multi是调用mysqladmin来关闭数据库的，那最好的办法还是直接使用mysqladmin来关闭各个数据库了，下面演示使用mysqladmin来关闭数据库实例。

（6.2）使用mysqladmin来关闭实例

![复制代码](media/复制代码.gif)

[root@masterdb mysql]# mysqld_multi report
Reporting MySQL servers
MySQL server from group: mysqld3306 is running
MySQL server from group: mysqld3307 is running
MySQL server from group: mysqld3308 is running
MySQL server from group: mysqld3309 is running
[root@masterdb mysql]# 
[root@masterdb mysql]# 
[root@masterdb mysql]# cd
[root@masterdb ~]# mysqladmin -h127.0.0.1 -uroot -p -P3306 shutdown 
Enter password: 
[root@masterdb ~]# 
[root@masterdb ~]# mysqladmin -h127.0.0.1 -uroot -p -P3307 shutdown 
Enter password: 
[root@masterdb ~]# mysqld_multi report
Reporting MySQL servers
MySQL server from group: mysqld3306 is not running
MySQL server from group: mysqld3307 is not running
MySQL server from group: mysqld3308 is running
MySQL server from group: mysqld3309 is running

![复制代码](media/复制代码.gif)

最终关闭了3306和3307数据库。

【结束】

**相关文档集合：**

**1.在一台Linux服务器上安装多个MySQL实例（一）--使用mysqld_multi方式**

[2.在一台Linux服务器上安装多个MySQL实例（二）--使用单独的MySQL配置文件](https://www.cnblogs.com/lijiaman/p/12588095.html)

分类: [--200 MySQL](https://www.cnblogs.com/lijiaman/category/1448653.html), [----220 MySQL安装](https://www.cnblogs.com/lijiaman/category/1683035.html)

好文要顶 关注我 收藏该文 ![](media/icon_weibo_24.png) ![](media/wechat.png)

[![](media/20170618002435.png)](https://home.cnblogs.com/u/lijiaman/)

[gegeman](https://home.cnblogs.com/u/lijiaman/)  
[粉丝 - 92](https://home.cnblogs.com/u/lijiaman/followers/) [关注 - 15](https://home.cnblogs.com/u/lijiaman/followees/)  

+加关注

4

0

[«](https://www.cnblogs.com/lijiaman/p/12324033.html) 上一篇： [MySQL复制(四)--多源(主)复制](https://www.cnblogs.com/lijiaman/p/12324033.html "发布于 2020-02-17 22:19")  
[»](https://www.cnblogs.com/lijiaman/p/12588095.html) 下一篇： [在一台Linux服务器上安装多个MySQL实例（二）--使用单独的MySQL配置文件](https://www.cnblogs.com/lijiaman/p/12588095.html "发布于 2020-03-28 16:47")

posted @ 2020-03-28 15:31  [gegeman](https://www.cnblogs.com/lijiaman/)  阅读(12481)  评论(1)  [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=12587630)  收藏  举报

刷新评论[刷新页面](https://www.cnblogs.com/lijiaman/p/12587630.html#)[返回顶部](https://www.cnblogs.com/lijiaman/p/12587630.html#top)

登录后才能查看或发表评论，立即 登录 或者 [逛逛](https://www.cnblogs.com/) 博客园首页

[【推荐】阿里云2核2G云服务器低至99元/年，百款云产品优惠享不停](https://click.aliyun.com/m/1000369037/)  
[【推荐】技术大牛都在关注的音视频&元宇宙社区，前沿技术在线学！](https://brands.cnblogs.com/zego)  

**编辑推荐：**  
· [关于如何编写好金融科技客户端 SDK 的思考](https://www.cnblogs.com/1992monkey/p/17213598.html)  
· [ASP.NET Core Web API 接口限流](https://www.cnblogs.com/s0611163/p/17199379.html)  
· [生产环境 Java 应用服务内存泄漏分析与解决](https://www.cnblogs.com/cgli/p/17201943.html)  
· [我的十年编程路 2019年篇](https://www.cnblogs.com/wenhanxiao/p/17197082.html)  
· [现代图片性能优化 - 图片资源的容错及可访问性处理](https://www.cnblogs.com/coco1s/p/17202234.html)  

**阅读排行：**  
· [Linux系统下祼机安装mysql8.0和docker mysql 8.0 性能差异对比~](https://www.cnblogs.com/netcore3/p/17215527.html)  
· [.NET中委托性能的演变](https://www.cnblogs.com/InCerry/p/the-evolution-of-delegate-performance-in-net-c8f23572b8b1.html)  
· [我的十年编程路 2021年篇](https://www.cnblogs.com/wenhanxiao/p/17213750.html)  
· [gRPC之.Net6中的初步使用介绍](https://www.cnblogs.com/qubernet/p/17210812.html)  
· [给我一块画布，我可以造一个全新的跨端UI](https://www.cnblogs.com/BaiCai/p/17214620.html)  

### 公告

昵称： [gegeman](https://home.cnblogs.com/u/lijiaman/)  
园龄： [7年5个月](https://home.cnblogs.com/u/lijiaman/ "入园时间：2015-10-15")  
粉丝： [92](https://home.cnblogs.com/u/lijiaman/followers/)  
关注： [15](https://home.cnblogs.com/u/lijiaman/followees/)

+加关注

<

2023年3月

>

日

一

二

三

四

五

六

26

27

28

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

1

2

3

4

5

6

7

8

### [随笔分类](https://www.cnblogs.com/lijiaman/post-categories) (222)

-   [--100 Oracle(59)](https://www.cnblogs.com/lijiaman/category/922180.html)
-   [----120 Oracle RAC(7)](https://www.cnblogs.com/lijiaman/category/979023.html)
-   [----130 Oracle安装与升级(4)](https://www.cnblogs.com/lijiaman/category/1471150.html)
-   [----140 Oracle数据库调优(13)](https://www.cnblogs.com/lijiaman/category/1543122.html)
-   [----150 DataGuard(9)](https://www.cnblogs.com/lijiaman/category/1437573.html)
-   [----160 Oracle备份与恢复(12)](https://www.cnblogs.com/lijiaman/category/1020310.html)
-   [----170 GoldenGate(2)](https://www.cnblogs.com/lijiaman/category/1537228.html)
-   [--200 MySQL(22)](https://www.cnblogs.com/lijiaman/category/1448653.html)
-   [----210 MySQL高可用(12)](https://www.cnblogs.com/lijiaman/category/1658422.html)
-   [----220 MySQL安装(3)](https://www.cnblogs.com/lijiaman/category/1683035.html)
-   [----240 MySQL性能优化(3)](https://www.cnblogs.com/lijiaman/category/1840931.html)
-   [----260 MySQL备份与恢复(6)](https://www.cnblogs.com/lijiaman/category/1749077.html)
-   [--400 MongoDB(22)](https://www.cnblogs.com/lijiaman/category/1775666.html)
-   [--500 PostgreSQL(3)](https://www.cnblogs.com/lijiaman/category/2199705.html)
-   [--600 ELK Stack(2)](https://www.cnblogs.com/lijiaman/category/1867842.html)
-   [--700 SQL Server(4)](https://www.cnblogs.com/lijiaman/category/1631223.html)
-   [hadoop(1)](https://www.cnblogs.com/lijiaman/category/1193681.html)
-   [java(2)](https://www.cnblogs.com/lijiaman/category/882899.html)
-   [Linux/UNIX(24)](https://www.cnblogs.com/lijiaman/category/931673.html)
-   [php(5)](https://www.cnblogs.com/lijiaman/category/870102.html)
-   [windows(1)](https://www.cnblogs.com/lijiaman/category/945177.html)
-   [zabbix(2)](https://www.cnblogs.com/lijiaman/category/1400489.html)
-   [虚拟机(4)](https://www.cnblogs.com/lijiaman/category/1470966.html)

### 相册 (1)

-   [picture(1)](https://www.cnblogs.com/lijiaman/gallery/1266936.html)

### [阅读排行榜](https://www.cnblogs.com/lijiaman/most-viewed)

-   [1. oracle client安装与配置(52010)](https://www.cnblogs.com/lijiaman/p/6391396.html)
-   [2. oracle执行计划（二）----如何查看执行计划(48401)](https://www.cnblogs.com/lijiaman/p/11488979.html)
-   [3. Oracle 11gR2 RAC + ASM + Grid安装(33952)](https://www.cnblogs.com/lijiaman/p/7673795.html)
-   [4. [Oracle]理解undo表空间(33579)](https://www.cnblogs.com/lijiaman/p/7617351.html)
-   [5. [Oracle]Oracle数据库任何用户密码都能以sysdba角色登入(19871)](https://www.cnblogs.com/lijiaman/p/6195523.html)
-   [6. Oracle分区表删除分区引发错误ORA-01502: 索引或这类索引的分区处于不可用状态(17211)](https://www.cnblogs.com/lijiaman/p/9277149.html)
-   [7. Linux 物理卷(PV)、逻辑卷(LV)、卷组(VG)管理(12535)](https://www.cnblogs.com/lijiaman/p/12885649.html)
-   [8. 在一台Linux服务器上安装多个MySQL实例（一）--使用mysqld_multi方式(12473)](https://www.cnblogs.com/lijiaman/p/12587630.html)
-   [9. 【oracle】Enterprise Manager 无法连接到数据库实例。下面列出了组件的状态---个人解决方案(12338)](https://www.cnblogs.com/lijiaman/p/6160087.html)
-   [10. [Oracle]约束（constraint）(10170)](https://www.cnblogs.com/lijiaman/p/7225831.html)

### [评论排行榜](https://www.cnblogs.com/lijiaman/most-commented)

-   [1. keepalived+MySQL实现高可用(6)](https://www.cnblogs.com/lijiaman/p/13430668.html)
-   [2. 【Linux资源管理】一款优秀的linux监控工具——nmon(5)](https://www.cnblogs.com/lijiaman/p/9466614.html)
-   [3. Oracle分区表删除分区引发错误ORA-01502: 索引或这类索引的分区处于不可用状态(3)](https://www.cnblogs.com/lijiaman/p/9277149.html)

### [最新评论](https://www.cnblogs.com/lijiaman/comments)

-   [1. Re:Linux 物理卷(PV)、逻辑卷(LV)、卷组(VG)管理](https://www.cnblogs.com/lijiaman/p/12885649.html)
-   感谢博主分享
    
-   --kone~
-   [2. Re:MySQL复合索引探究](https://www.cnblogs.com/lijiaman/p/14364171.html)
-   学习了！
    
-   --于仁杰
-   [3. Re:centos7.4下离线安装CDH5.7](https://www.cnblogs.com/lijiaman/p/8733316.html)
-   @硅谷少年 感谢指正。 看报错是访问数据库受限，因为MySQL是使用 【"用户@IP" 或 "用户@主机名"】的方式访问数据库的，估计9.10这步创建数据库账号的时候这样创建就没问题了 。在MySQL...
-   --gegeman

Copyright © 2023 gegeman  
Powered by .NET 7.0 on Kubernetes