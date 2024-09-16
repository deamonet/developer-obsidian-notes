
本指南由 Deamonet 在金贝能源实习期间撰写

# Mycat2 配置详细指南及实用技巧

mycat 官方网站：http://www.mycat.org.cn/

mycat Github 项目地址：[GitHub - MyCATApache/Mycat2: MySQL Proxy using Java NIO based on Sharding SQL,Calcite ,simple and fast](https://github.com/MyCATApache/Mycat2)

mycat 文档地址：[Mycat2权威指南 · Yuque](https://www.yuque.com/ccazhw/ml3nkf?)

## 介绍

mycat 是一个数据库抽象层，用于 JDBC 应用与后台数据库之间，其本身也正是模拟一台 mysql 数据库。

mycat 结构上实现了跨服务器的分库分表，成为了多台数据库实例的统一前端；功能上避免了数据库前台服务的代码修改，降低了业务数量上的扩张成本，提高了超大表格的检索性能；业务上减少了业务开发跟数据库运维之间的职能交叉。

## 安装与启动

到如下网址下载相关文件：需要下载的文件有：1.2x-release和install-template。前者是mycat2核心java源代码生成的可执行jar包；后者是mycat2在各系统上运行所需的兼容文件。

1.      解压install-template，前述jar包置于lib子文件夹内，转移到安装路径。

2.      编辑conf文件夹中prototypeDs.datasource.json文件，提供MySQL JDBC URL[[2]](#_ftn2)及账户密码。

3.      如有必要，修改sever.json，内含有监听IP和端口、模拟MySQL版本。

4.      给bin文件夹内的wrapper及mycat二进制文件赋予可执行权限，即可执行mycat。[[3]](#_ftn3)

mycat程序对应的参数有start, stop, restart, console等，分别对应启动/停止/重启/前台启动。[[4]](#_ftn4)

---

[[1]](#_ftnref1) 该网址可能被DNS污染。

[[2]](#_ftnref2) JDBC URL中是否写数据库名，并不重要，也没区别。

[[3]](#_ftnref3) 如果使用Windows系统，则改执行mycat.bat文件，并且要用install参数以安装mycat服务。

[[4]](#_ftnref4) 前台启动会输出实时报错信息。

## 概念

逻辑库与物理库：前者是mycat模拟出的数据库，后者是真实的数据库，通常逻辑库都会对应物理库。物理库用于存储数据。逻辑库和物理库的名字是否相同均可。

单表与分片表：前者是不需要分片的表，后者是需要分片的表。对应配置文件中的名字是normal和sharding。

数据源和集群：前者指向一个真实存在的mysql服务作为mycat储存后端，同时可根据读写的用途区分出两个同源的数据源；后者有一个或多个前者组成，用于增加系统冗余，保持系统稳定；同时划分各数据源的用途，包括读写分离，主从备份等。配置文件中出现targetName字段，二者均可填入。

原型库：原型库的存在是为了解决分库分表所带来多表查询当中出现的问题[[1]](#_ftn1)；或者在安装启动阶段提供初始的数据源配置。

全局表、广播表、ER表：本文档中暂时用不到。这都是为了处理分库分表之后多表查询存在的问题。

分片算法：给分片表分片的依据。默认的MOD_HASH会根据指定的字段把数据均匀分配到各个物理表中。除此之外，还要多种分片算法，以及可以自定义分片算法。

---

[[1]](#_ftnref1) 详情请看官方文档：什么是原型库？https://www.yuque.com/ccazhw/ml3nkf/zvkguf

## 单表配置



## 数据分布

在确定分库分表数量之前最后，看一下数据的分布，重点在于如何使得分表之后的数据可以均匀分布在每一个区间。

例如如果分片键是id

```sql
SELECT count(*) as u FROM `sys_user` GROUP BY MOD(id, 2); # 用2取模

SELECT count(*) as u FROM `sys_user` GROUP BY MOD(id, 3); # 用3取模
```

对于phoebus库里的sys_user，可以发现，其中99%的数都是偶数，也就是说，所有按偶数次分表都不均匀，从实践来看，用奇数次分表是均匀的。

结果如下：

表：sys_user

```sql
SELECT count(*) as u FROM `sys_user` GROUP BY MOD(id, 3);

1671
1651
1612
```

表：dev_inverter

```sql
SELECT count(*) as u FROM `dev_inverter` GROUP BY MOD(userId, 3);

1306
2826
1664
2007
```

表：dev_wifi

```sql
SELECT count(*) as u FROM `dev_wifi` GROUP BY MOD(id, 3);

406526
406523
406521
```

表：stt_yield_daily202211

```sql
select count(*) from stt_yield_daily_202211 GROUP BY mod(userId, 3);

67710
22736
33422
```

表dev_inverter对3取模，结果却有四行，这是因为存在空值，常理来说，这一字段不改存在空值，因为这都是用户才会生成的。

从数学上看，这个问题就是对于任意的原始分布，如何转换成一个均匀的分组结果。而且不仅是当前的分布，还要考虑到以后的分布，或许最简单的做法就是重新设置一个连续递增的分片键，而不是用原始的数据分布。

至于有没有其他合适并且简单，跟取模一样直接计算出分组结果而无需考虑数字本身大小的方法，还需要更深入的研究。

配置的重点在于如何设置，文档上并未给出详细说明，只能实践得出结果。

## 空值处理

如果分片键是空值，会怎么样，文档中对此毫无说明。github上issues也找不到对应解释。通过之后的实践可知，空值会分配到表编号为0的表。另外创建一个分表可能更有道理，不过考虑到正常业务的情况下，空值应该是少数情况。不创建分表也是可以理解的。（虽然在这个表里空值不少）。

不过在partition那一节里，分片键为空的数据根本不能插入。

## mappingFormat的局限

mappingFormat是一种自动分配物理分表的配置描述。简单的分片配置都是用mappingFormat，不过没想到的是连奇数分片都不能胜任。下面做一个验证。

另外可以注意的一点就是，mappingFormat中可以写入java代码。

把function一项修改成：

```json
"function":{
    "properties":{
    "dbNum":3,
    "mappingFormat":"c${targetIndex}/db${dbIndex}/inverter${index}",
    "tableNum":1,
    "tableMethod":"mod_hash(userId)",
    "storeNum":2,
    "dbMethod":"mod_hash(userId)"
    },
    "ranges":{}
},
"partition":{}
```

最后得到的储存节点结果是：

```json
 /*+mycat:showTopology{"schemaName":"phoebus","tableName":"dev_inverter"}*/;
+------------+------------+-----------+---------+------------+-------+
| targetName | schemaName | tableName | dbIndex | tableIndex | index |
+------------+------------+-----------+---------+------------+-------+
| c0         | db0        | inverter0 | 0       | 0          | 0     |
| c1         | db1        | inverter1 | 1       | 0          | 1     |
| c2         | db2        | inverter2 | 2       | 0          | 2     |
+------------+------------+-----------+---------+------------+-------+
3 rows in set (0.00 sec)
```

而实际上，并未配置c2集群，此时输入查询命令，会报java空指针错误，如果把配置中的c${targetIndex}改为c0，此错误不再发生，说明的确是自动配置的错误。

```shell
mysql> select * from phoebus.dev_inverter;
ERROR: 
java.lang.NullPointerException

mysql> select * from phoebus.sys_user;
ERROR: 
java.lang.NullPointerExceptionon
```

文档中对于mappingFormat并没有特别详细的说明，各种入门的文章恰好也都是偶数的分片，这样一个问题居然毫无提示。这些index都只做了简单的解释，targetIndex也都不作详细说明，产生误解也难怪了。

## 精细化的分片控制 Partition

partition配置存储节点,在分片算法无法使用的时候就扫描这些存储节点,所以分片算法无法正确配置的时候仍然可以查询,但是可能插入报错。

mappingFormat的配置优先级比Partition更高，如果配置了mappingFormat，Partition配置就会失效；下文第二种配置中的 data比第一种的names配置优先级更高。

partition从示例中看一共有两种写法：

第一种

```json
"partition": {
    "targetNames":"c$0-1",//集群或者数据源
    "schemaNames":"db1_$0-1",//物理库
    "tableNames":"t1_$0-1"//物理表
}
```

第二种

```json
"partition": {
    "data":[
        ["c0","db1","t2","0","0","0"],
        ["c1","db1","t2","1","1","1"]
    ]
}
```

其中["c0","db1","t2","0","0","0"]分别表示 集群/数据源, 物理库, 物理表, 分库下标, 分表下标,全局分区 的下标，而这一整个表示一个物理分表。

如果把配置文件改成：

```json
"function":{
    "properties":{
        "tableMethod":"mod_hash(userId)",
    },
    "ranges":{}
},
"partition":{
    "targetName":"c$0-1",
    "dbName":"d$0-1",
    "tableName":"inverter$0-1"

}
```

结果是退回到默认的配置

```shell
mysql> /*+mycat:showTopology{"schemaName":"phoebus","tableName":"dev_inverter"}*/;
+------------+------------+----------------+---------+------------+-------+
| targetName | schemaName | tableName      | dbIndex | tableIndex | index |
+------------+------------+----------------+---------+------------+-------+
| prototype  | phoebus_0  | dev_inverter_0 | 0       | 0          | 0     |
+------------+------------+----------------+---------+------------+-------+
1 row in set (0.00 sec)
```

正确使用Partition的方法是，如下配置：

```json
        "function":{
            "clazz":"io.mycat.router.mycat1xfunction.PartitionByMod",
            "properties":{
                "count":"3",
                "columnName":"userId"
            },
            "ranges":{}
        },
        "partition":{
            "schemaNames":"db$0-1",
            "tableNames":"inverter$0-1",
            "targetNames":"c$0-1"
        },
```

结果如下，显然总分表数量是所有数量相乘的结果。

```shell
mysql> /*+mycat:showTopology{"schemaName":"phoebus","tableName":"dev_inverter"}*/;
+------------+------------+-----------+---------+------------+-------+
| targetName | schemaName | tableName | dbIndex | tableIndex | index |
+------------+------------+-----------+---------+------------+-------+
| c0         | db0        | inverter0 | NULL    | NULL       | NULL  |
| c0         | db0        | inverter1 | NULL    | NULL       | NULL  |
| c0         | db1        | inverter0 | NULL    | NULL       | NULL  |
| c0         | db1        | inverter1 | NULL    | NULL       | NULL  |
| c1         | db0        | inverter0 | NULL    | NULL       | NULL  |
| c1         | db0        | inverter1 | NULL    | NULL       | NULL  |
| c1         | db1        | inverter0 | NULL    | NULL       | NULL  |
| c1         | db1        | inverter1 | NULL    | NULL       | NULL  |
+------------+------------+-----------+---------+------------+-------+
8 rows in set (0.00 sec)
```

第二种方法的配置：

```json
"function":{
 "clazz":"io.mycat.router.mycat1xfunction.PartitionByMod",
 "properties":{
 "count":"3",
 "columnName":"userId"
 },
 "ranges":{}
 },
"partition": {
    "data":[
        ["c0","db1","inverter0","0","0","0"],
        ["c1","db1","inverter1","1","1","1"]
 }
```

结果

```shell
mysql> /*+mycat:showTopology{"schemaName":"phoebus","tableName":"dev_inverter"}*/;
+------------+------------+-----------+---------+------------+-------+
| targetName | schemaName | tableName | dbIndex | tableIndex | index |
+------------+------------+-----------+---------+------------+-------+
| c0         | db0        | inverter0 | 0       | 0          | 0     |
| c1         | db1        | inverter1 | 1       | 1          | 1     |
+------------+------------+-----------+---------+------------+-------+
2 rows in set (0.01 sec)
```

在上述配置当中，如果count与最终的分表数量不一致，则插入数据时会报错，这一点需要注意。实际上count就是取模的值

```shell
ERROR 1105 (HY000): t.tbl 分片算法越界 分片值:{4}
```

## 其他报错信息及原因

```shell
ERROR: 
java.lang.IllegalArgumentException: 路由计算返回结果个数为2
```

似乎是因为分片键确定错误，指向表中不存在的字段。

```shell
ERROR: 
java.lang.IllegalArgumentException: columnValue:null Please eliminate any quote and non number within it.
```

似乎是因为分片键为空

## 其他常用的注解命令

1. 重置配置：/*+ mycat:resetConfig{} */;

2. 展示分片表物理储存分表：/*+ mycat:showTopology{"schemaName":"db1", "tableName":"normal"} */;

3. 展示表格详细信息：/+ mycat:showTables{"schemaName":"mysql"}/

## 性能测试

用python mysql connector 链接物理数据库跟mycat数据库，写一个python脚本计算每条语句用时

其中几张表都只有几千条数据，stt_yield_daily_202211有十几万条数据

其他的分片表由于userId存在空值，尤其是dev_wifi 90%以上都是空值，所以没办法做性能比较。

此时Mycat2使用的四台服务器两主两从。其中一台性能更好的主服务器同时运行了Mycat2实例，它对应的从服务器上有较多应用。

```sql
select * from sys_user a inner join stt_yield_daily_202211 b on a.id=b.userId;
```

第一次测试结果（测量用时，单位都是秒）

```shell
    |mysql  |mycat
3   |26.94  |5.148
```

第二次测试结果

```shell
    |mysql  |mycat
3   |5.525  |5.043
```

第三次测试结果

```shell
    |mysql  |mycat
3   |5.539  |5.146
```

平均来看的话，用mycat的确性能会高一点。

## 问题

在往mycat2里插入数据时，主从的数据存在差异，而且是每一张表都有数据行数的差错，而且主从上的谁多谁少不固定，不过大致上相当。另外，从mysql实例来看，并没有明显的错误。
