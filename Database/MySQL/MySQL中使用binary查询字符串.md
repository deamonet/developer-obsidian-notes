# MySQL中使用binary查询字符串

[#mysql](https://fitzix.github.io/tags/mysql/)

Jun 7, 2015

今天，在做老师布置的实验作业时候遇到一个关于MySQL字符串比较问题。场景是这样的，就是需要在数据库中查询记录而进行字符串比较时，需要对字符串进行大小写区分比较，这样在默认情况下进行操作可能不会达到你想要的效果。比如下面实例：

```
SELECT *　FROM user where username='maratrix';
SELECT * FROM user where username='MARATRIX';
```

经测试，发现上面两条SQL语句的执行结果是一样的，这个结果并不是我们想要的，怎么解决？

### 问题探究

在我看来，做技术有一点要清楚的是：**_解决问题一定要抓住问题的本源，从而游刃有余_**。下面，我们看看这个小问题是咋回事的呢？

通过看MySQL手册可以知道，默认情况下，对MySQL数据库中的字段进行查询或者排序都是不区分大小写的。

但是在有些应用中，需要进行区分大小写的操作，咋办？

**答：使用BINARY操作符**。

> BINARY操作符将后面的字符串抛给一个二进制字符串。这是一种简单的方式来促使逐字节而不是逐字符的进行列比较。这使得比较区分大小写，即使该列不被定义为 BINARY或 BLOB。

> BINARY影响整个比较；它可以在任何操作数前被给定，而产生相同的结果。

### 解决方法

1、第一种是在创建表结构时候使用binary属性来定义字段：

```
create table if not exists user(
    id int unsigned primary key auto_increment,
    name varchar(32) binary,
)engine=myisam;
```

或者在表结构创建好后使用alter来添加字段binary属性

```
alter table user modify name varchar(32) binary ;
```

2、第二种方法是在sql语句中使用bianry来进行区分大小写操作：

```
SELECT * FROM user where name=binary 'maratrix';
或者
SELECT * FROM user where binary name='maratrix';
```

进过测试发现，使用`SELECT * FROM user where name=binary 'maratrix';`效率更高点，原因是将binary放在字符串前会使用索引（假设该字段存在索引），而将binary放在字段前面将不会使用索引，即使索引存在也不会使用。

### 注意

在一些语境中，假如你将一个编入索引的列派给BINARY, MySQL 将不能有效使用这个索引。

[PREV](https://fitzix.github.io/2015/06/12/shuffle_array_note/)[NEXT](https://fitzix.github.io/2015/06/04/how_to_use_session_without_cookie/)

未找到相关的 [Issues](https://github.com/fitzix/fitzix.github.io/issues) 进行评论

请联系 @fitzix 初始化创建

© 2015 - 2019 [fitzix](https://github.com/fitzix), powered by [Hexo](https://hexo.io/) and [hexo-theme-apollo](https://github.com/fitzix/hexo-theme-apollo).