## SQL FULL OUTER JOIN 关键字

FULL OUTER JOIN 关键字只要左表（table1）和右表（table2）其中一个表中存在匹配，则返回行.

FULL OUTER JOIN 关键字结合了 LEFT JOIN 和 RIGHT JOIN 的结果。

### SQL FULL OUTER JOIN 语法

SELECT _column_name(s)_  
FROM _table1_  
FULL OUTER JOIN _table2_  
ON _table1.column_name_=_table2.column_name_;

![SQL FULL OUTER JOIN](media/SQL_FULL_OUTER_JOIN.gif)

---

---

## 演示数据库

在本教程中，我们将使用 RUNOOB 样本数据库。

下面是选自 "Websites" 表的数据：

+----+--------------+---------------------------+-------+---------+
| id | name         | url                       | alexa | country |
+----+--------------+---------------------------+-------+---------+
| 1  | Google       | https://www.google.cm/    | 1     | USA     |
| 2  | 淘宝          | https://www.taobao.com/   | 13    | CN      |
| 3  | 菜鸟教程      | http://www.runoob.com/    | 4689  | CN      |
| 4  | 微博          | http://weibo.com/         | 20    | CN      |
| 5  | Facebook     | https://www.facebook.com/ | 3     | USA     |
| 7  | stackoverflow | http://stackoverflow.com/ |   0 | IND     |
+----+---------------+---------------------------+-------+---------+

下面是 "access_log" 网站访问记录表的数据：

+-----+---------+-------+------------+
| aid | site_id | count | date       |
+-----+---------+-------+------------+
|   1 |       1 |    45 | 2016-05-10 |
|   2 |       3 |   100 | 2016-05-13 |
|   3 |       1 |   230 | 2016-05-14 |
|   4 |       2 |    10 | 2016-05-14 |
|   5 |       5 |   205 | 2016-05-14 |
|   6 |       4 |    13 | 2016-05-15 |
|   7 |       3 |   220 | 2016-05-15 |
|   8 |       5 |   545 | 2016-05-16 |
|   9 |       3 |   201 | 2016-05-17 |
+-----+---------+-------+------------+
9 rows in set (0.00 sec)

---

## SQL FULL OUTER JOIN 实例

下面的 SQL 语句选取所有网站访问记录。

MySQL中不支持 FULL OUTER JOIN，你可以在 SQL Server 测试以下实例。

## 实例

SELECT Websites.name, access_log.count, access_log.date  
FROM Websites  
FULL OUTER JOIN access_log  
ON Websites.id=access_log.site_id  
ORDER BY access_log.count DESC;

**注释：**FULL OUTER JOIN 关键字返回左表（Websites）和右表（access_log）中所有的行。如果 "Websites" 表中的行在 "access_log" 中没有匹配或者 "access_log" 表中的行在 "Websites" 表中没有匹配，也会列出这些行。

[](https://www.runoob.com/sql/sql-join-right.html) [SQL RIGHT JOIN 关键字](https://www.runoob.com/sql/sql-join-right.html "SQL RIGHT JOIN 关键字")

[SQL UNION 操作符](https://www.runoob.com/sql/sql-union.html "SQL UNION 操作符") [](https://www.runoob.com/sql/sql-union.html)

## 

1 篇笔记 写笔记

1.     Serlucus
    
      857***973@qq.com
    
    781
    
    A inner join B 取交集。
    
    A left join B 取 A 全部，B 没有对应的值为 null。
    
    A right join B 取 B 全部 A 没有对应的值为 null。
    
    A full outer join B 取并集，彼此没有对应的值为 null。
    
    对应条件在 **on** 后面填写。
    
    Serlucus
    
       Serlucus
    
      857***973@qq.com
    
    2年前 (2020-10-22)