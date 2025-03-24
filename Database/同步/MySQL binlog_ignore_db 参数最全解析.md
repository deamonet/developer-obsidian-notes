**前言：**

经过前面文章学习，我们知道 binlog 会记录数据库所有执行的 DDL 和 DML 语句（除了数据查询语句select、show等）。注意默认情况下会记录所有库的操作，那么如果我们有另类需求，比如说只让某个库记录 binglog 或排除某个库记录 binlog ，是否支持此类需求呢？本篇文章我们一起来看下。

#### 1. binlog_do_db 与 binlog_ignore_db

当数据库实例开启 binlog 时，我们执行 show master status 命令，会看到有 Binlog_Do_DB 与 Binlog_Ignore_DB 选项。

```sql
mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000009 |   282838 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
```

默认情况下，这两个选项为空，那么这两个参数有何作用？是否如同其字面意思一个只让某个库记录 binglog 一个排除某个库记录 binlog 呢？笔者查阅官方文档，简单说明下这两个参数的作用：

-   **binlog_do_db**：此参数表示只记录指定数据库的二进制日志，默认全部记录。
-   **binlog_ignore_db**：此参数表示不记录指定的数据库的二进制日志。

这两个参数为互斥关系，一般只选择其一设置，只能在启动命令行中或配置文件中加入。指定多个数据库要分行写入，举例如下：

```sql
# 指定 db1 db2 记录binlog
[mysqld]
binlog_do_db = db1
binlog_do_db = db2

# 不让 db3 db4 记录binlog
[mysqld]
binlog_ignore_db = db3
binlog_ignore_db = db4
```

此外，这二者参数具体作用与否还与 binlog 格式有关系，在某些情况下 binlog 格式设置为 STATEMENT 或 ROW 会有不同的效果。在实际应用中 binlog_ignore_db 用途更广泛些，比如说某个库的数据不太重要，为了减轻服务器写入压力，我们可能不让该库记录 binlog 。网上也有文章说设置 binlog_ignore_db 会导致从库同步错误，那么设置该参数到底有什么效果呢，下面我们来具体实验下。

#### 2. binlog_ignore_db 具体效果

首先说明下，我的测试数据库实例是 5.7.23 社区版本，共有 testdb、logdb 两个业务库，我们设置 logdb 不记录 binlog ，下面来具体实验下：

```sql
# binlog 为 ROW 格式 

# 1.不使用 use db
mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |      154 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
mysql> select database();
+------------+
| database() |
+------------+
| NULL       |
+------------+
1 row in set (0.00 sec)
mysql> CREATE TABLE testdb.`test_tb1` ( id int , name varchar(30) ) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.06 sec)

mysql> insert into testdb.test_tb1 values (1001,'sdfde');
Query OK, 1 row affected (0.01 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |      653 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE logdb.`log_tb1` ( id int , name varchar(30) ) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.05 sec)

mysql> insert into logdb.log_tb1 values (1001,'sdfde');
Query OK, 1 row affected (0.00 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |      883 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
mysql> insert into logdb.log_tb1 values (1002,'sdsdfde'); 
Query OK, 1 row affected (0.01 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |      883 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+

mysql> alter table logdb.log_tb1 add column c3 varchar(20);   
Query OK, 0 rows affected (0.12 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     1070 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
# 结论：其他库记录正常 logdb库会记录DDL 不记录DML 

# 2.使用 use testdb跨库
mysql> use testdb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select database();
+------------+
| database() |
+------------+
| testdb     |
+------------+
1 row in set (0.00 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     1070 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE `test_tb2` ( id int , name varchar(30) ) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.05 sec)

mysql> insert into test_tb2 values (1001,'sdfde');
Query OK, 1 row affected (0.04 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     1574 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE logdb.`log_tb2` ( id int , name varchar(30) ) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.05 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     1810 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> insert into logdb.log_tb2 values (1001,'sdfde');
Query OK, 1 row affected (0.01 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     1810 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
# 结论：同样logdb库会记录DDL 不记录DML 

# 3.使用 use logdb跨库
mysql> use logdb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select database();
+------------+
| database() |
+------------+
| logdb      |
+------------+
1 row in set (0.00 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     1810 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE testdb.`test_tb3` ( id int , name varchar(30) ) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.23 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     1810 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> insert into testdb.test_tb3 values (1001,'sdfde');
Query OK, 1 row affected (0.02 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     2081 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE `log_tb3` ( id int , name varchar(30) ) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.05 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     2081 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> insert into log_tb3 values (1001,'sdfde');
Query OK, 1 row affected (0.02 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     2081 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
# 结论：logdb都不记录 同时不记录其他库的DDL

# 4.每次操作都进入此库 不跨库
mysql> use testdb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     2081 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE `test_tb4` ( id int , name varchar(30) ) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.05 sec)

mysql> insert into test_tb4 values (1001,'sdfde');
Query OK, 1 row affected (0.01 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     2585 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> use logdb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> CREATE TABLE `log_tb4` ( id int , name varchar(30) ) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.04 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     2585 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> insert into log_tb4 values (1001,'sdfde');
Query OK, 1 row affected (0.01 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |     2585 |              | logdb            |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
# 结论：其他库全部记录 logdb全不记录
```

同样的，将 binlog 格式设置为 STATEMENT ，再次进行测试，这里不再赘述测试过程，总结下 STATEMENT 格式下的实验结果：

-   未选择任何数据库进行操作，所有都会记录。
-   选择testdb，对testdb和logdb分别进行操作，所有库都会记录。
-   选择logdb，对testdb和logdb分别进行操作，所有库都不会记录。
-   选择某个库并只对当前库进行操作，则记录正常，不会记录logdb。

看了这么多实验数据，你是否眼花缭乱了呢，下面我们以思维导图的形式总结如下：

![](media/1607001921790-f3c7e703-6629-4f5b-867b-9dae6b170b22.jpeg.jpg)

这么看来 binlog_ignore_db 参数的效果确实和诸多因素有关，特别是有从库的情况下，主库要特别小心使用此参数，很容易产生主从同步错误。不过，按照严格标准只对当前数据库进行操作，则不会产生问题。这也告诉我们要严格按照标准来，只赋予业务账号某个单库的权限，也能避免各种问题发生。

**总结：**

不清楚各位读者是否对这种介绍参数的文章感兴趣呢？可能这些是数据库运维人员比较关注的吧。本篇文章主要介绍关于 binlog 的 binlog_ignore_db 参数的具体作用，可能本篇文章实验环境还不够考虑周全，有兴趣的同学可以参考下官方文档，有助于对该参数有更深入的了解。

![wx_blog.png](media/wx_blog.png)

作者：[MySQL技术](https://www.cnblogs.com/kunjian/)  
出处：[https://www.cnblogs.com/kunjian/](https://www.cnblogs.com/kunjian/)  
本文版权归作者和博客园共有，欢迎转载，但未经作者同意必须保留此段声明，且在文章页面明显位置给出原文连接，否则保留追究法律责任的权利。  
如果文中有什么错误，欢迎指出。以免更多的人被误导。有需要沟通的，可以站内私信，文章留言，或者关注『MySQL技术』公众号私信我。一定尽力回答。  

分类: [MySQL](https://www.cnblogs.com/mysqljs/category/1473977.html)