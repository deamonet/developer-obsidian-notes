-     
     发表于：[2020年3月29日](https://www.choupangxia.com/2020/03/29/mybatis-list/)
-    分类：[Mybatis](https://www.choupangxia.com/category/java/mybatis/), [Mysql](https://www.choupangxia.com/category/%e6%95%b0%e6%8d%ae%e5%ba%93/mysql/), [数据库](https://www.choupangxia.com/category/%e6%95%b0%e6%8d%ae%e5%ba%93/)
-    标签：[List参数](https://www.choupangxia.com/tag/list%e5%8f%82%e6%95%b0/), [List对象](https://www.choupangxia.com/tag/list%e5%af%b9%e8%b1%a1/), [Mybatis](https://www.choupangxia.com/tag/mybatis/)

SSM框架是JavaWeb必学的框架，虽说基本的增删改查很简单，但是当面临一些特殊情况时，有时还是会显得手足无措，此篇用来记录一些特殊场景下Mybatis框架的应用.

## 传入参数为List对象

### 1. 场景复现

首先有如下一张表：

MySQL [test]> select * from t_entry_resource;

+----+-------------+------+----------+--------+--------+---------------------+

| id | resource_id | type | title | banner | icon | add_date |

+----+-------------+------+----------+--------+--------+---------------------+

| 11 | 6 | 14 | 分类 | 1.jpg | 2.jpg | 2017-11-17 11:22:30 |

| 12 | 3 | 1 | 测试12 | 3.jpg | 4.jpg | 2017-11-17 11:22:30 |

| 13 | 653 | 1 | 测试34 | 5.jpg | 6.jpg | 2017-11-20 02:32:26 |

| 14 | 1 | 1 | 测试5 | 7.jpg | 8.jpg | 2017-11-20 02:32:51 |

| 15 | 3942 | 3 | 测试6 | 9.jpg | 10.jpg | 2017-11-20 02:34:27 |

+----+-------------+------+----------+--------+--------+---------------------+

5 rows in set (0.01 sec)

如果要根据resource_id和type来批量查询记录，该如何编写Mybatis语句？

### 2. 解决方案

直接贴出来解决方案如下所示：

Dao层接口：

List<EntryResource> findByRidAndType(List<EntryResource> entryResources);

XML语句：

<select id="findByRidAndType" resultMap="entryResource" parameterType="list">

SELECT

*

FROM

t_entry_resource a

WHERE

<foreach collection="list" index="index" item="entryResources" open="(" close=")" separator="or">

( `type`=#{entryResources.type} and resource_id=#{entryResources.resourceId} )

</foreach>

</select>

该语句利用了mybatis的foreach动态拼接SQL。