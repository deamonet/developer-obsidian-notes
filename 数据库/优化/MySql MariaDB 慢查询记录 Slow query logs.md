The slow query log consists of SQL statements that take more than [`long_query_time`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_long_query_time) seconds to execute and require at least [`min_examined_row_limit`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_min_examined_row_limit) rows to be examined. The slow query log can be used to find queries that take a long time to execute and are therefore candidates for optimization. However, examining a long slow query log can be a time-consuming task. To make this easier, you can use the [**mysqldumpslow**](https://dev.mysql.com/doc/refman/5.7/en/mysqldumpslow.html "4.6.8 mysqldumpslow — Summarize Slow Query Log Files") command to process a slow query log file and summarize its contents. See [Section 4.6.8, “mysqldumpslow — Summarize Slow Query Log Files”](https://dev.mysql.com/doc/refman/5.7/en/mysqldumpslow.html "4.6.8 mysqldumpslow — Summarize Slow Query Log Files").


The slow query log is a record of SQL queries that took a long time to perform.

Note that, if your queries contain user's passwords, the slow query log may contain passwords too. Thus, it should be protected.

The number of rows affected by the slow query are also recorded in the slow query log.

# mysql数据库开启慢查询日志

__ 老马吃草 __ 2021-08-08 PM __ 497℃ __ 0条

## 首先我们要知道为什么要开启 **mysql慢查询日志**

　开启 **慢查询日志** 它能记录下所有执行超过 **long_query_time** 时间的SQL语句,能够帮助开发人员,运维人员快速的找到执行慢的SQL语句,方便我们对这些SQL语句进行进一步[优化](https://www.lmcc.top/tag/optimize/ "优化"),从而增强[数据库](https://www.lmcc.top/tag/database/ "数据库")性能。

　今天我们要讲的主角就是 **mysql慢查询日志**。

## mysql开启慢[日志](https://www.lmcc.top/tag/log/ "日志")版本要高于mysql5.6以上

```
SELECT VERSION();  #查询版本号
# 或者
show variables like '%version%' #查询版本号
```

![ktb46vpr.png](https://cdn.lmcc.top/usr/uploads/2021/09/4017486269.png "ktb46vpr.png")

## mysql **慢查询日志** 相关参数说明

```
slow_query_log #慢查询开启状态,ON开启,OFF关闭
slow_query_log_file #慢查询日志存放的位置（这个目录需要MySQL的运行帐号的可写权限,一般设置为MySQL的数据存放目录）
long_query_time #查询超过多少秒才记录,默认10s,查询命令 SHOW VARIABLES LIKE 'long_query_time';
log_queries_not_using_indexes = 1 #表明记录没有使用索引的 SQL 语句
```

## 首先查看mysql的[慢查询](https://www.lmcc.top/tag/slow-query/ "慢查询")状态是否开启

　如果出现 **log_slow_queries** 状态为 **OFF** ,说明当前 **mysql** [数据库](https://www.lmcc.top/category/database.html "数据库")没有开启慢查询

```
show variables like '%query%';
```

![ktb3wk7i.png](https://cdn.lmcc.top/usr/uploads/2021/09/1876149631.png "ktb3wk7i.png")

## [配置](https://www.lmcc.top/tag/config/ "配置")开启mysql慢查询日志

　找到 **mysql** 目录下的 **my.cnf** 文件并编辑,在 **[mysqld]** 字段下加入一下配置信息:

```
long_query_time=3 #表示记录查询超过1s的sql
slow_launch_time=1 #表示如果建立线程花费了比这个值更长的时间,slow_launch_threads计数器将增加
slow_query_log=ON #开启慢查询日志
slow_query_log_file=/var/lib/mysql/slow_queries.log #慢查询日志记录文件
```

![ktb4d66k.png](https://cdn.lmcc.top/usr/uploads/2021/09/1143398171.png "ktb4d66k.png")  
　慢查询的日志记录文件对于mysql用户权限必须可写入,配置完成后保存配置信息然后重新启动 **mysql** 服务即可

## 查询mysql慢日志超出时间 **long_query_time**

```
show variables like 'long_query_time';
```

![ktb4l3uc.png](https://cdn.lmcc.top/usr/uploads/2021/09/3618229227.png "ktb4l3uc.png")

## 查询mysql慢日志记录位置 **slow_query**

```
show variables like 'slow_query%';
```

![ktb4np71.png](https://cdn.lmcc.top/usr/uploads/2021/09/2001004100.png "ktb4np71.png")

## 通过执行一下sql测试记录sql **慢查询日志**

```
select sleep(5);
```

## 然后通过慢日志路径下的 **mysql-slow.log** 查看是有一下记录

```
# Time: 210908 14:38:51
# User@Host: typecho_loc_dh_t[typecho_loc_dh_t] @  [192.168.146.1]  Id:   259
# Query_time: 5.003445  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1631083131;
select sleep(5);
```

![ktb4ttye.png](https://cdn.lmcc.top/usr/uploads/2021/09/1084446406.png "ktb4ttye.png")

## 最后我们通过 **mysqladmin** 命令将无效的 **慢查询日志** 清理掉

```
mysqladmin -uroot -p flush-logs
```

　执行该命令后,命令行会提示输入密码,输入正确密码后,将执行删除操作。新的慢查询日志会直接覆盖旧的查询日志,不需要再手动删除。  
　数据库管理员也可以手工删除慢查询日志,删除之后需要重新启动 **mysql** 服务。

　关于怎么开启 **mysql慢查询日志** 的具体[教程](https://www.lmcc.top/tag/tutorial/ "教程")老马已经记录分享完毕了有兴趣的小伙伴可以下去自行测试下吧,总的来说 **mysql慢查询日志** 还是很有用的,特别是对一些大项目,重量级的项目至关重要

　其中常见数据库日志记录了用户对数据库的各种操作及数据库发生的各种事件,能帮助数据库管理员追踪,分析问题,**mysql** 提供了 **错误日志**,**二进制日志**,**查询日志**,**慢查询日志** 。

__标签: [教程](https://www.lmcc.top/tag/tutorial/), [MySql](https://www.lmcc.top/tag/mysql/), [数据库](https://www.lmcc.top/tag/database/), [日志](https://www.lmcc.top/tag/log/), [慢查询](https://www.lmcc.top/tag/slow-query/)

非特殊说明，本博所有文章均为博主原创

版权属于：老马吃草

本文链接：[https://www.lmcc.top/articles/123.html](https://www.lmcc.top/articles/123.html)

作品采用：《[署名-非商业性使用-相同方式共享 4.0 国际 (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)》许可协议授权

如若转载，请注明出处：[https://www.lmcc.top/articles/123.html](https://www.lmcc.top/articles/123.html)