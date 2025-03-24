发布于2019-04-03 11:07:00阅读 119.2K0

## [mysql](https://cloud.tencent.com/product/cdb?from=10680)启动报错

今天启动mysql又一次报错：The server quit without updating PID file！记得上次出现这个问题的时候，尝试了一些常规的方法，未果，所以索性重新进行安装。但是，相同的问题今天又出现了！！！OH， my god！恰巧今天时间充裕，尝试各种办法，终于皇天不负有心人，经过一个小时的奋战后，终于让我给搞定，整个过程是这样的！

### 解决过程

首先，按着自己思路去查看日志文件，相信大家能看到的最多的内容就是Innodb，这是什么玩意儿？其实日志中的信息基本没啥用，就不浪费太多时间了。

然后，使用最直接的办法——百度。相信很多人搜到的结果都是以下几项：

-   可能是/usr/local/mysql/data/mysql.pid文件没有写的权限 解决方法 ：给予权限，执行 “chown -R mysql:mysql /var/data” “chmod -R 755 /usr/local/mysql/data” 然后重新启动mysqld！
-   可能进程里已经存在mysql进程 解决方法：用命令“ps -ef|grep mysqld”查看是否有mysqld进程，如果有使用“kill -9 进程号”杀死，然后重新启动mysqld！
-   可能是第二次在机器上安装mysql，有残余数据影响了服务的启动。 解决方法：去mysql的数据目录/data看看，如果存在mysql-bin.index，就赶快把它删除掉吧，它就是罪魁祸首了。
-   mysql在启动时没有指定配置文件时会使用/etc/my.cnf配置文件，请打开这个文件查看在[mysqld]节下有没有指定数据目录(datadir)。 解决方法：请在[mysqld]下设置这一行：datadir = /usr/local/mysql/data
-   skip-federated字段问题 解决方法：检查一下/etc/my.cnf文件中有没有没被注释掉的skip-federated字段，如果有就立即注释掉吧。
-   错误日志目录不存在 解决方法：使用“chown” “chmod”命令赋予mysql所有者及权限
-   selinux惹的祸，如果是centos系统，默认会开启selinux 解决方法：关闭它，打开/etc/selinux/config，把SELINUX=enforcing改为SELINUX=disabled后存盘退出重启机器试试

抱着试试看的态度，我把上面所有方法都尝试了一遍，结果。。。然并卵！我的所有文件的权限都没问题，所属主和所属组也没问题，也没有找到所谓的mysql-bin.index文件，日志文件也有！！！心想，what are you 弄啥嘞？？？ 继续搜索，在这里 [https://serverfault.com/questions/457337/mysql-server-quit-without-updating-pid-file](https://serverfault.com/questions/457337/mysql-server-quit-without-updating-pid-file) 看到一个成功案例，方法如下： Try moving the /var/lib/mysql/ibdata1, /var/lib/mysql/ibdata2 and so on to another directory and start the server again. It should rebuild the index files. Once it did and you verified everything is working again, you can remove the old corrupted ones. 意思是说：删除mysql的库文件下的ibdata*文件。 我毫不犹豫的照做了，然后启动mysql，当时启动界面有点不一样，我窃喜，终于搞定，可是谁知道结果却是：

```javascript
[root@localhost mysql]# /etc/init.d/mysqld start
Starting MySQL.Logging to '/data/mysql/localhost.localdomain.err'.
./usr/local/mysql/bin/mysqld_safe: 行 178:  3588 已杀死               nohup /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/data/mysql --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/data/mysql/localhost.localdomain.err --pid-file=/data/mysql/localhost.localdomain.pid --socket=/tmp/mysql.sock < /dev/null > /dev/null 2>&1
 ERROR! The server quit without updating PID file (/data/mysql/localhost.localdomain.pid).
[root@localhost mysql]# /etc/init.d/mysqld start
Starting MySQL.Logging to '/data/mysql/localhost.localdomain.err'.
.. ERROR! The server quit without updating PID file (/data/mysql/localhost.localdomain.pid).
```

总之问题还没解决！ 于是继续查找原因，尝试各种办法更改配置文件，但是仍然没搞定，最后我在此查看错误日志：

```javascript
[root@localhost mysql]# less localhost.localdomain.err  
2017-08-18 13:42:30 3905 [Note] InnoDB: Setting file ./ibdata1 size to 12 MB
2017-08-18 13:42:30 3905 [Note] InnoDB: Database physically writes the file full: wait...
2017-08-18 13:42:30 3905 [Note] InnoDB: Setting log file ./ib_logfile101 size to 48 MB
2017-08-18 13:42:35 3905 [Note] InnoDB: Setting log file ./ib_logfile1 size to 48 MB
2017-08-18 13:42:39 3905 [Note] InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
2017-08-18 13:42:39 3905 [Warning] InnoDB: Creating New log files failed, the ib_ligfile existed。
```

看到这，我突然想到删除mysql库文件/data/mysql/中的“ib_*”文件，一不做二不休，反正是在虚拟机中操作，大不了重新安装，于是我执行如下操作：

** 注意：** 执行该操作之前一定要对[数据库](https://cloud.tencent.com/solution/database?from=10680)进行备份，因为ibdata1存放的是所有数据文件，如果不小心删了库，那就惨了！！！（传说中的从删库到跑路。。。）

```javascript
[root@localhost mysql]# ls
aria_log.00000001  ib_buffer_pool  ib_logfile101              mysql
aria_log_control   ibdata1         localhost.localdomain.err  performance_schema
auto.cnf           ib_logfile0     multi-master.info          test
[root@localhost mysql]# rm -rf ib_buffer_pool  ib_logfile101 ibdata1 localhost.localdomain.err ib_logfile0
```

然后重启：

```javascript
[root@localhost mysql]# /etc/init.d/mysqld start
Starting MySQL.Logging to '/data/mysql/localhost.localdomain.err'.
....... SUCCESS! 
```

这个结果让我大吃一惊，我有点不敢相信，于是又执行下面的操作：

```javascript
[root@localhost mysql]# /etc/init.d/mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL..... SUCCESS!   

[root@localhost mysql]# ps aux |grep mysql
root      3976  0.0  0.0 113256    16 pts/0    S    13:42   0:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/data/mysql --pid-file=/data/mysql/localhost.localdomain.pid
mysql     4112  0.0 74.4 964872 366432 pts/0   Sl   13:42   0:02 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/data/mysql --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/data/mysql/localhost.localdomain.err --pid-file=/data/mysql/localhost.localdomain.pid --socket=/tmp/mysql.sock
root      4170  0.0  0.1 112664   972 pts/0    S+   14:21   0:00 grep --color=auto mysql
[root@localhost mysql]# netstat -lntp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1662/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      2050/master         
tcp6       0      0 :::3306                 :::*                    LISTEN      4112/mysqld         
tcp6       0      0 :::22                   :::*                    LISTEN      1662/sshd           
tcp6       0      0 ::1:25                  :::*                    LISTEN      2050/master         
```

到这里，我终于安心了，确实解决了，于是赶紧写下文章分析一下个人心得！不过，由于专业知识薄弱，小白没能明白其中的原理，希望有高人在评论中指点，不胜感激！！！

### 补充：

2017年8月25日补充。 今天又看到类似的报错，但是并没有影响登录使用mysql，查看日志有这样的字样“initialize buffer pool，size=128.0M”，“cannot allocate memory for the pool”，大概意思是说无法分配足够的内存供pool使用。此时想到mysql配置文件中有相关的配置，于是更改如下参数：

```javascript
innodb_buffer_pool_size = 128
#配置文件中该值默认为128M
```

将这个值调小，再次启动mysql服务，问题解决！

### 补充2：

查看日志有如下错误提示：Plugin 'InnoDB' registration as a STORAGE ENGINE failed。 解决办法：

```javascript
[root@localhost mysql]# rm -rf ib_logfile*
```

然后启动mysql，问题解决！

(adsbygoogle = window.adsbygoogle || []).push({});

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan) ，欢迎热爱写作的你一起参与！

本文分享自作者个人站点/博客：https://my.oschina.net/adailinux复制

如有侵权，请联系 cloudcommunity@tencent.com 删除。