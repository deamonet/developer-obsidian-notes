笔试被问烂了：SQL的执行顺序

### 先给结论

from>join>where>group by>聚合函数>having>select>order by>limit

### 1、最先执行from table；

需要先确定从哪个表中取数据，所以最先执行from table。

### 2、join连接

用于把来自两个或多个表的行结合起来，简单补充一下连接的类型

-   自然连接（natural join）
-   内连接（inner join）：内连接查询能将左表和右表中能关联起来的数据连接后返回，返回的结果就是两个表中所有相匹配的数据。
-   外连接（outer join）：外连接分为左外连接（LEFT JOIN：即使右表中没有匹配，也从左表返回所有的行）、右外连接（RIGHT JOIN：即使左表中没有匹配，也从右表返回所有的行）、还有一个FULL JOIN(全连接)，不过MYSQL不支持全连接
-   交叉连接（cross join）即笛卡尔连接

### 3、where语句；

where语句是对条件加以限定

### 4、分组语句【group by…… having】；

group by是分组语句

having是和group by配合使用的，用来作条件限定

### 5、聚合函数；

常用的聚合函数有max，min， count，sum，聚合函数的执行在group by之后，having之前

举例：count函数查询分组后，每一组分别有多少条数据

select count(*) from user group by gender

值得注意的是：**聚合函数的执行在group by之后，having之前**

### 6、select语句；

对分组聚合完的表挑选出需要查询的数据

### 7、Distinct

distinct对数据进行去重

如果sql语句存在聚合函数，例如count、max等，会**先执行聚合函数再去重**

### 8、order by排序语句。

order by排序语句

select * from user order by id  升序排序
select * from user order by id desc 降序排序

### 9、limit

limit用于指定返回的数据条数

select * from user limit 2
从user表中查询前两条数据
该sql等同于
select * from user limit 0,2
表示从第0条开始取两条数据

limit常配合order by使用

select * from user order by id limit 3
根据id排序，选出id排序前三的数据

### 总结

**from>join>where>group by>聚合函数>having>select>order by>limit**

### 例子

select 
distinct user.name 
from user 
join vip on user.id=vip.id 
where user.id>10 
group by user.mobile 
having count(*)>2 
order by user.id
limit 3;

#### 执行顺序

1.  from user
2.  join vip on user.id=vip.id ，join是表示要关联的表，on是连接的条件
3.  where user.id>10
4.  group by user.mobile 根据user.mobile分组
5.  然后先执行count(*)在执行having，查询分组之后数量大于2的分组数据
6.  select 对分组聚合完的表挑选出需要查询的数据
7.  distinct查询出来的数据去重
8.  order by user.id 对去重后的数据排序
9.  limit 3对排序后的数据选出前面3条

[#秋招#](https://www.nowcoder.com/subject/index/002d6ce4eab1487f9cae3241b5322732)[#后端#](https://www.nowcoder.com/subject/index/ca249521e17743a7b7154ac162d38bbd)[#数据库#](https://www.nowcoder.com/subject/index/b2d8f0f7982a45a7aa7737848b8df513)[#sql#](https://www.nowcoder.com/subject/index/738bfc2745ca42f7983b957b62728dcb)[#笔试#](https://www.nowcoder.com/subject/index/5a42525969c94438a13899b5c4196665)