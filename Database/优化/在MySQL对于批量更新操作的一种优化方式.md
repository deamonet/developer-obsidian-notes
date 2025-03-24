在MySQL对于批量更新操作的一种优化方式

[万里数据库](https://www.modb.pro/u/431641) 2021-06-09

2937

引言

批量更新数据，不同于这种 update a=a+1 where pk > 500，而是需要对每一行进行单独更新 update a=1 where pk=1;update a=12 where pk=7;… 这样连续多行update语句的场景，是少见的。

可以说是偶然也是一种必然，在GreatDB 5.0的开发过程中，我们需要对多语句批量update的场景进行优化。

两种多行更新操作的耗时对比

在我们对表做多行更新的时候通常会遇到以下两种情况

1.单语句批量更新（update a=a+1 where pk > 500）

2.多语句批量更新（update a=1 where pk=1;update a=12 where pk=7;…）

下面我们进行实际操作比较两种场景，在更新相同行数时所消耗的时间。

数据准备

数据库版本：MySQL 8.0.23

t1表，建表语句以及准备初始数据1000行

create  database if not exists test;  
use test  
##建表  
create table t1(c1 int primary key,c2 int);  
##创建存储过程用于生成初始数据  
DROP PROCEDURE IF EXISTS insdata;  
DELIMITER $$  
CREATE PROCEDURE insdata(IN beg INT, IN end INT) BEGIN  
 WHILE beg <= end  
 DO  
 INSERT INTO test.t1 values (beg, end);  
SET beg = beg+1;  
END WHILE;  
END $$  
DELIMITER ;  
##插入初始数据1000行  
call insdata(1,1000);

1.单语句批量更新

更新语句

update  t1 set c2=10 where c1 <=1000;

执行结果

mysql> update  t1 set c2=10 where c1 <=1000;  
Query OK, 1000 rows affected (0.02 sec)  
Rows matched: 1000  Changed: 1000  Warnings: 0

2.多语句批量更新

以下脚本用于生成1000行update语句，更新c2的值等于1000以内的随机数

#!/bin/bash

for i in {1..1000}  
do  
        echo "update t1 set c2=$((RANDOM%1000+1)) where c1=$i;" >> update.sql  
done

生成sql语句如下

update t1 set c2=292 where c1=1;  
update t1 set c2=475 where c1=2;  
update t1 set c2=470 where c1=3;  
update t1 set c2=68 where c1=4;  
update t1 set c2=819 where c1=5;

... ....  
update t1 set c2=970  where c1=1000;

因为source /ssd/tmp/tmp/1000/update.sql;执行结果如下，执行时间不易统计：

Query OK, 1 row affected (0.00 sec)  
Rows matched: 1  Changed: 1  Warnings: 0

Query OK, 1 row affected (0.01 sec)  
Rows matched: 1  Changed: 1  Warnings: 0

Query OK, 1 row affected (0.00 sec)  
Rows matched: 1  Changed: 1  Warnings: 0

Query OK, 1 row affected (0.00 sec)  
Rows matched: 1  Changed: 1  Warnings: 0

Query OK, 1 row affected (0.01 sec)  
Rows matched: 1  Changed: 1  Warnings: 0

Query OK, 1 row affected (0.00 sec)  
Rows matched: 1  Changed: 1  Warnings: 0

Query OK, 1 row affected (0.01 sec)  
Rows matched: 1  Changed: 1  Warnings: 0

所以利用Linux时间戳进行统计：

#!/bin/bash

start_time=`date +%s%3N`  
/ssd/tmp/mysql/bin/mysql -h127.0.0.1 -uroot -P3316 -pabc123 -e "use test;source /ssd/tmp/tmp/1000/update.sql;"  
end_time=`date +%s%3N`  
echo "执行时间为："$(($end_time-$start_time))"ms"

执行结果：

[root@computer-42 test]# bash update.sh  
mysql: [Warning] Using a password on the command line interface can be insecure.  
执行时间为：4246ms

执行所用时间为：4246ms=4.246 sec

结果比较

单语句批量更新

多语句批量更新

0.02sec

4.246sec

总结

由上述例子我们可以看到，同样是更新1000行数据。单语句批量更新与多语句批量更新的执行效率差距很大。

而产生这种巨大差异的原因，除了1000行sql语句本身的网络与语句解析开销外，影响性能的地方主要是以下几个方面：

1.如果会话是auto_commit=1，每次执行update语句后都要执行commit操作。commit操作耗费时间较久，会产生两次磁盘同步（写binlog和写redo日志）。在进行比对测试时，尽量将多个语句放到一个事务内，保证只提交一次事务。

2.向后端发送多语句时，后端每处理一个语句均会向client返回一个response包，进行一次交互。如果多语句使用一个事务的话，网络io交互应该是影响性能的主要方面。之前在性能测试时发现网卡驱动占用cpu很高。

我们的目标是希望在更新1000行时，第二种场景的耗时能够减少到一秒以内。

对第二种场景的优化

接下来我们来探索对更新表中多行为不同值时，如何提高它的执行效率。

简单分析

从执行的update语句本身来说，两种场景所用的表结构都进行了最大程度的简化，update语句也十分简单，且where条件为主键，理论上已经没有优化的空间。

如果从其他方面来考虑，根据上述原因分析会有这样三个优化思路：

1、减少执行语句的解析时间来提高执行效率

2、减少commit操作对性能的影响，尽量将多个语句放到一个事务内，保证只提交一次事务。

3、将多条语句合并成一条来提高执行效率

方案一：使用prepare语句，减小解析时间

以下脚本用于生成prepare执行语句

#!/bin/bash

echo "prepare pr1 from 'update test.t1 set c2=? where c1=?';" > prepare.sql  
for i in {1..1000}  
do  
echo "set @a=$((RANDOM%1000+1)),@b=$i;" >>prepare.sql  
echo "execute pr1 using @a,@b;" >> prepare.sql  
done  
echo "deallocate prepare pr1;" >> prepare.sql

生成语句如下

prepare pr1 from 'update test.t1 set c2=? where c1=?';  
set @a=276,@b=1;  
execute pr1 using @a,@b;  
set @a=341,@b=2;  
execute pr1 using @a,@b;  
set @a=803,@b=3;  
execute pr1 using @a,@b;  
... ...  
set @a=582,@b=1000;  
execute pr1 using @a,@b;  
deallocate prepare pr1;

执行语句

#!/bin/bash

start_time=`date +%s%3N`  
/ssd/tmp/mysql/bin/mysql -h127.0.0.1 -uroot -P3316 -pabc123 -e "use test;source /ssd/tmp/tmp/test/prepare.sql;"  
end_time=`date +%s%3N`  
echo "执行时间为："$(($end_time-$start_time))"ms"

执行结果：

[root@computer-42 test]# bash prepare_update_id.sh  
mysql: [Warning] Using a password on the command line interface can be insecure.  
执行时间为：4518ms

与优化前相比

优化前

优化后

提升效率

4.246 sec

4.518 sec

无提升

很遗憾，执行总耗时反而增加了。

这里笔者有一点推测是由于原本一条update语句，被拆分成了两条语句：

set @a=276,@b=1;  
execute pr1 using @a,@b;

这样在MySQL客户端和MySQL进程之间的通讯次数增加了，所以增加了总耗时。

因为prepare预处理语句执行时只能使用用户变量传递，以下执行语句会报错

mysql> execute pr1 using 210,5;  
ERROR 1064 (42000): You have an error in your SQL syntax;  
check the manual that corresponds to your MySQL server version  
for the right syntax to use near '210,5' at line 1

所以无法在语法方面将两条语句重新合并，笔者便使用了以下另外一种执行方式

执行语句

#!/bin/bash

start_time=`date +%s%3N`  
/ssd/tmp/mysql/bin/mysql -h127.0.0.1 -uroot -P3316 -pabc123  <<EOF  
use test;  
DROP PROCEDURE IF EXISTS pre_update;  
DELIMITER $$  
CREATE PROCEDURE pre_update(IN beg INT, IN end INT) BEGIN  
 prepare pr1 from 'update test.t1 set c2=? where c1=?';  
 WHILE beg <= end  
 DO  
 set  @a=beg+1,@b=beg;  
 execute pr1 using @a,@b;  
 SET beg = beg+1;  
END WHILE;  
deallocate prepare pr1;  
END $$  
DELIMITER ;  
call pre_update(1,1000);  
EOF  
end_time=`date +%s%3N`  
echo "执行时间为："$(($end_time-$start_time))"ms"

执行结果：

[root@computer-42 test]# bash prepare_update_id.sh  
mysql: [Warning] Using a password on the command line interface can be insecure.  
执行时间为：3862ms

与优化前相比：

优化前

优化后

提升效率

4.246 sec

3.862 sec

9.94%

这样的优化幅度符合prepare语句的理论预期，但仍旧不够理想。

方案二：多个update语句放到一个事务内执行，最终commit一次

以下脚本用于生成1000行update语句在一个事务内，更新c2的值等于1000以内的随机数

#!/bin/bash  
echo "begin;" > update.sql  
for i in {1..1000}  
do  
        echo "update t1 set c2=$((RANDOM%1000+1)) where c1=$i;" >> update.sql  
done  
echo "commit;" >> update.sql

生成sql语句如下

begin;  
update t1 set c2=279 where c1=1;  
update t1 set c2=425 where c1=2;  
update t1 set c2=72 where c1=3;  
update t1 set c2=599 where c1=4;  
update t1 set c2=161 where c1=5;

... ....  
update t1 set c2=775  where c1=1000;  
commit;

执行时间统计的方法，同上

[root@computer-42 test]# bash update.sh  
mysql: [Warning] Using a password on the command line interface can be insecure.  
执行时间为：194ms

执行时间为194ms=0.194sec

与优化前相比：

优化前

优化后

提升效率

4.246 sec

0.194sec

20.89倍

可以看出多次commit操作对性能的影响还是很大的。

方案三：使用特殊SQL语法，将多个update语句合并

合并多条update语句

在这里我们引入一种并不常用的MySQL语法：

1）优化前：

update多行执行语句类似“update xxx; update xxx;update xxx;… …”

2）优化后：

改成先把要更新的语句拼成一个视图（结果集表)，然后用结果集表和源表进行关联更新。这种更新方式有个隐式限制“按主键或唯一索引关联更新”。

UPDATE t1 m, (  
    SELECT 1 AS c1, 2 AS c2  
    UNION ALL  
    SELECT 2, 2  
    UNION ALL  
    SELECT 3, 3  
    ... ...  
    UNION ALL  
    SELECT n, 2  
  ) r  
SET m.c1 = r.c1, m.c2 = r.c2  
WHERE m.c1 = r.c1;

3）具体的例子：

###建表  
create table t1(c1 int primary key,c2 int);  
###插入5行数据  
insert into t1 values(1,1),(2,1),(3,1),(4,1),(5,1);  
select  * from t1;  
###更新c2为c1+1  
UPDATE t1 m, (  
  SELECT 1 AS c1, 2 AS c2  
  UNION ALL  
  SELECT 2, 3  
  UNION ALL  
  SELECT 3, 4  
  UNION ALL  
  SELECT 4, 5  
  UNION ALL  
  SELECT 5, 6  
 ) r  
SET m.c1 = r.c1, m.c2 = r.c2  
WHERE m.c1 = r.c1;  
###查询结果  
select * from t1;

执行结果：

mysql> create table t1(c1 int primary key,c2 int);  
  Query OK, 0 rows affected (0.03 sec)

mysql> insert into t1 values(1,1),(2,1),(3,1),(4,1),(5,1);  
  Query OK, 5 rows affected (0.00 sec)  
  Records: 5  Duplicates: 0  Warnings: 0

mysql> select * from t1;  
  +----+------+  
  | c1 | c2   |  
  +----+------+  
  |  1 |    1 |  
  |  2 |    1 |  
  |  3 |    1 |  
  |  4 |    1 |  
  |  5 |    1 |  
  +----+------+  
  5 rows in set (0.00 sec)

mysql> update  t1 m,(select 1 as c1,2 as c2 union all select 2,3 union all select 3,4 union all select 4,5 union all select 5,6 ) r set m.c1=r.c1,m.c2=r.c2  where m.c1=r.c1;  
Query OK, 5 rows affected (0.01 sec)  
  Rows matched: 5  Changed: 5  Warnings: 0

mysql> select * from t1;  
  +----+------+  
  | c1 | c2   |  
+----+------+  
  |  1 |    2 |  
  |  2 |    3 |  
  |  3 |    4 |  
  |  4 |    5 |  
  |  5 |    6 |  
  +----+------+  
  5 rows in set (0.00 sec)

4）更进一步的证明

在这里笔者选择通过观察语句执行生成的binlog，来证明优化方式的正确性。

首先是未经优化的语句:

begin;  
update t1 set c2=2 where c1=1;  
update t1 set c2=3 where c1=2;  
update t1 set c2=4 where c1=3;  
update t1 set c2=5 where c1=4;  
update t1 set c2=6 where c1=5;  
commit;

......  
### UPDATE `test`.`t1`  
### WHERE  
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */  
### SET  
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=2 /* INT meta=0 nullable=1 is_null=0 */  
......  
### UPDATE `test`.`t1`  
### WHERE  
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */  
### SET  
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=3 /* INT meta=0 nullable=1 is_null=0 */  
......  
### UPDATE `test`.`t1`  
### WHERE  
###   @1=3 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */  
### SET  
###   @1=3 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=4 /* INT meta=0 nullable=1 is_null=0 */  
......  
### UPDATE `test`.`t1`  
### WHERE  
###   @1=4 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */  
### SET  
###   @1=4 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=5 /* INT meta=0 nullable=1 is_null=0 */  
......  
### UPDATE `test`.`t1`  
### WHERE  
###   @1=5 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */  
### SET  
###   @1=5 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=6 /* INT meta=0 nullable=1 is_null=0 */

…

然后是优化后的语句：

UPDATE t1 m, (  
  SELECT 1 AS c1, 2 AS c2  
  UNION ALL  
  SELECT 2, 3  
  UNION ALL  
  SELECT 3, 4  
  UNION ALL  
  SELECT 4, 5  
  UNION ALL  
  SELECT 5, 6  
 ) r  
SET m.c1 = r.c1, m.c2 = r.c2  
WHERE m.c1 = r.c1;

### UPDATE `test`.`t1`  
### WHERE  
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */  
### SET  
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=2 /* INT meta=0 nullable=1 is_null=0 */  
### UPDATE `test`.`t1`  
### WHERE  
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */  
### SET  
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=3 /* INT meta=0 nullable=1 is_null=0 */  
### UPDATE `test`.`t1`  
### WHERE  
###   @1=3 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */  
### SET  
###   @1=3 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=4 /* INT meta=0 nullable=1 is_null=0 */  
### UPDATE `test`.`t1`  
### WHERE  
###   @1=4 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */  
### SET  
###   @1=4 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=5 /* INT meta=0 nullable=1 is_null=0 */  
### UPDATE `test`.`t1`  
### WHERE  
###   @1=5 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */  
### SET  
###   @1=5 /* INT meta=0 nullable=0 is_null=0 */  
###   @2=6 /* INT meta=0 nullable=1 is_null=0 */

可以看到，优化前后binlog中记录的SQL语句是一致的。这也说明了我们优化后语句与原执行语句是等效的。

5）从语法角度的分析

UPDATE t1 m, --被更新的t1表设置别名为m  
(  
  SELECT 1 AS c1, 2 AS c2  
  UNION ALL  
  SELECT 2, 3  
  UNION ALL  
  SELECT 3, 4  
  UNION ALL  
  SELECT 4, 5  
  UNION ALL  
  SELECT 5, 6  
) r --通过子查询构建的临时表r  
SET m.c1 = r.c1, m.c2 = r.c2  
WHERE m.c1 = r.c1;

将子查询临时表r单独拿出来，我们看一下执行结果：

mysql> select 1 as c1,2 as c2 union all select 2,3 union all select 3,4 union all select 4,5 union all select 5,6;  
+----+----+  
| c1 | c2 |  
+----+----+  
|  1 |  2 |  
|  2 |  3 |  
|  3 |  4 |  
|  4 |  5 |  
|  5 |  6 |  
+----+----+  
5 rows in set (0.00 sec)

可以看到，这就是我们想要更新的那部分数据，在更新之后的样子。通过t1表与r表进行join update，就可以将t1表中相应的那部分数据，更新成我们想要的样子，完成了使用一条语句完成多行更新的操作。

6）看一下执行计划

以下为explain执行计划，使用了嵌套循环连接，外循环表t1 as m根据条件m.c1=r.c1过滤出5条数据，每更新一行数据需要扫描一次内循环表r，共循环5次：

如果光看执行计划，似乎这条语句的执行效率不是很高，所以我们接下来真正执行一下。

7）实践检验

以下脚本用于生成优化后update语句，更新c2的值等于1000以内的随机数

#!/bin/bash

echo "update t1 as m,(select 1 as c1,2 as c2 " >> update-union-all.sql

for j in {2..1000}  
do  
        echo "union all select $j,$((RANDOM%1000+1))" >> update-union-all.sql

done

echo ") as r set m.c2=r.c2 where m.c1=r.c1" >> update-union-all.sql

生成SQL语句如下

update t1 as m,(select 1 as c1,2 as c2  
union all select 2,644  
union all select 3,322  
union all select 4,660  
union all select 5,857  
union all select 6,752  
... ...  
union all select 999,225  
union all select 1000,77  
) as r set m.c2=r.c2 where m.c1=r.c1

执行语句

#!/bin/bash

start_time=`date +%s%3N`  
/ssd/tmp/mysql/bin/mysql -h127.0.0.1 -uroot -P3316 -pabc123 -e \  
"use test;source /ssd/tmp/tmp/1000/update-union-all.sql;"  
end_time=`date +%s%3N`  
echo "执行时间为："$(($end_time-$start_time))"ms"

执行结果：

[root@computer-42 test]# bash update-union-all.sh  
mysql: [Warning] Using a password on the command line interface can be insecure.  
执行时间为：58ms

与优化前相比:

优化前

优化后

提升效率

4.246 sec

0.058 sec

72.21倍

|

多次测试对比结果如下：

总结

根据以上理论分析与实际验证，我们找到了一种对批量更新场景的优化方式。

[greatdb](https://www.modb.pro/tag/greatdb?type=knowledge)

「喜欢文章，快来给作者赞赏墨值吧」

【版权声明】本文为墨天轮用户原创内容，转载时必须标注文章的来源（墨天轮），文章链接，文章作者等基本信息，否则作者和墨天轮有权追究责任。如果您发现墨天轮中有涉嫌抄袭或者侵权的内容，欢迎发送邮件至：contact@modb.pro进行举报，并提供相关证据，一经查实，墨天轮将立刻删除相关内容。

评论

相关阅读

[荣誉 | 信创数据库企业排行Top3！万里数据库实力获权威机构认可](https://www.modb.pro/db/424112)

[万里数据库](https://www.modb.pro/db/424112)

[50次阅读](https://www.modb.pro/db/424112)

[2022-06-24 11:19:50](https://www.modb.pro/db/424112)

[包拯断案 | 别再让慢sql背锅@还故障一个真相](https://www.modb.pro/db/419977)

[万里数据库](https://www.modb.pro/db/419977)

[40次阅读](https://www.modb.pro/db/419977)

[2022-06-17 16:55:36](https://www.modb.pro/db/419977)

[生态 | 万里数据库与卫士通完成兼容认证 共筑网络安全生态体系](https://www.modb.pro/db/424110)

[万里数据库](https://www.modb.pro/db/424110)

[33次阅读](https://www.modb.pro/db/424110)

[2022-06-24 11:12:47](https://www.modb.pro/db/424110)

[创意信息获2家机构调研：GreatDB 数据库已在9地部署](https://www.modb.pro/db/430367)

[通讯员](https://www.modb.pro/db/430367)

[29次阅读](https://www.modb.pro/db/430367)

[2022-07-07 09:02:18](https://www.modb.pro/db/430367)

[解决方案 | GreatDB，为金融行业安全监控系统提供核心助力！](https://www.modb.pro/db/427674)

[万里数据库](https://www.modb.pro/db/427674)

[26次阅读](https://www.modb.pro/db/427674)

[2022-07-01 17:51:34](https://www.modb.pro/db/427674)

[重磅丨赛迪数据库市场研究报告，最大黑马竟是它？](https://www.modb.pro/db/431368)

[万里数据库](https://www.modb.pro/db/431368)

[23次阅读](https://www.modb.pro/db/431368)

[2022-07-08 16:08:42](https://www.modb.pro/db/431368)

[喜讯丨信息安全能力过硬！万里安全数据库通过中国软件评测中心检测认证](https://www.modb.pro/db/427673)

[万里数据库](https://www.modb.pro/db/427673)

[19次阅读](https://www.modb.pro/db/427673)

[2022-07-01 17:45:36](https://www.modb.pro/db/427673)

[技术干货 | 数据中间件如何与GreatSQL数据同步？](https://www.modb.pro/db/431370)

[万里数据库](https://www.modb.pro/db/431370)

[13次阅读](https://www.modb.pro/db/431370)

[2022-07-08 16:27:37](https://www.modb.pro/db/431370)

[万里数据库参编中国信通院多项数据库研究报告，并入围产业图谱](https://www.modb.pro/db/436338)

[万里数据库](https://www.modb.pro/db/436338)

[7次阅读](https://www.modb.pro/db/436338)

[2022-07-15 14:37:43](https://www.modb.pro/db/436338)

[荣誉 | 万里数据库荣获“数字经济创新领军企业”及数字经济案例TOP20两项殊荣](https://www.modb.pro/db/436329)

[万里数据库](https://www.modb.pro/db/436329)

[7次阅读](https://www.modb.pro/db/436329)

[2022-07-15 14:14:14](https://www.modb.pro/db/436329)

[万里数据库](https://www.modb.pro/u/431641)

关注

[123](https://www.modb.pro/u/431641)

[文章](https://www.modb.pro/u/431641)

[5](https://www.modb.pro/u/431641)

[粉丝](https://www.modb.pro/u/431641)

[24K+](https://www.modb.pro/u/431641)

[浏览量](https://www.modb.pro/u/431641)

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAIwAAACMCAYAAACuwEE+AAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNi1jMTM4IDc5LjE1OTgyNCwgMjAxNi8wOS8xNC0wMTowOTowMSAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENDIDIwMTcgKFdpbmRvd3MpIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOkQzRTY2QUY2MDE2MjExRUM4RjM1QzlCNDgxMzI5QUExIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOkQzRTY2QUY3MDE2MjExRUM4RjM1QzlCNDgxMzI5QUExIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6RDNFNjZBRjQwMTYyMTFFQzhGMzVDOUI0ODEzMjlBQTEiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6RDNFNjZBRjUwMTYyMTFFQzhGMzVDOUI0ODEzMjlBQTEiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz6VBMWoAAAkFklEQVR42uyd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)

获得了16次点赞

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAIwAAACMCAYAAACuwEE+AAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNi1jMTM4IDc5LjE1OTgyNCwgMjAxNi8wOS8xNC0wMTowOTowMSAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENDIDIwMTcgKFdpbmRvd3MpIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOkQ0NzZDMzQ4MDE2MjExRUM5Q0UzQ0VENzYyMzg4QjUyIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOkQ0NzZDMzQ5MDE2MjExRUM5Q0UzQ0VENzYyMzg4QjUyIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6RDQ3NkMzNDYwMTYyMTFFQzlDRTNDRUQ3NjIzODhCNTIiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6RDQ3NkMzNDcwMTYyMTFFQzlDRTNDRUQ3NjIzODhCNTIiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz6cKdaJAAArc0lEQVR42ux9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)

内容获得4次评论

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAIwAAACMCAYAAACuwEE+AAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNi1jMTM4IDc5LjE1OTgyNCwgMjAxNi8wOS8xNC0wMTowOTowMSAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENDIDIwMTcgKFdpbmRvd3MpIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOkQ1MzJCOENCMDE2MjExRUM5MkQ2RDZEMDQ4MTY0QzREIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOkQ1MzJCOENDMDE2MjExRUM5MkQ2RDZEMDQ4MTY0QzREIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6RDUzMkI4QzkwMTYyMTFFQzkyRDZENkQwNDgxNjRDNEQiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6RDUzMkI4Q0EwMTYyMTFFQzkyRDZENkQwNDgxNjRDNEQiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7164d1AAAo8ElEQVR42uyd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)

获得了12次收藏

TA的专栏

[GreatSQL](https://www.modb.pro/topic/51572)

收录2篇内容

热门文章

[GreatSQL MGR优化参考](https://www.modb.pro/db/51571)

[2021-04-14896浏览](https://www.modb.pro/db/51571)

[innodb_buffer_pool_size为什么无法调低至1GB以内](https://www.modb.pro/db/70179)

[2021-06-09755浏览](https://www.modb.pro/db/70179)

[技术干货 | MySQL8.0 如何快速回收膨胀的UNDO表空间](https://www.modb.pro/db/238026)

[2022-01-14728浏览](https://www.modb.pro/db/238026)

[赋能金融科技数字化转型 万里受邀2021中国金融科技应用发展研讨会](https://www.modb.pro/db/71252)

[2021-06-16597浏览](https://www.modb.pro/db/71252)

[关于互联网平台账号诽谤万里数据库的严正声明](https://www.modb.pro/db/410429)

[2022-05-30586浏览](https://www.modb.pro/db/410429)

最新文章

[万里数据库参编中国信通院多项数据库研究报告，并入围产业图谱](https://www.modb.pro/db/436338)

[2天前7浏览](https://www.modb.pro/db/436338)

[荣誉 | 万里数据库荣获“数字经济创新领军企业”及数字经济案例TOP20两项殊荣](https://www.modb.pro/db/436329)

[2天前7浏览](https://www.modb.pro/db/436329)

[技术干货 | 数据中间件如何与GreatSQL数据同步？](https://www.modb.pro/db/431370)

[2022-07-0813浏览](https://www.modb.pro/db/431370)

[重磅丨赛迪数据库市场研究报告，最大黑马竟是它？](https://www.modb.pro/db/431368)

[2022-07-0823浏览](https://www.modb.pro/db/431368)

[解决方案 | GreatDB，为金融行业安全监控系统提供核心助力！](https://www.modb.pro/db/427674)

[2022-07-0126浏览](https://www.modb.pro/db/427674)

From <[https://www.modb.pro/db/70174](https://www.modb.pro/db/70174)>