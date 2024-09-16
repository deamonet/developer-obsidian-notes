80 人赞同了该文章

最近在复习数据库索引部分，看到了 fulltext，也即全文索引，虽然全文索引在平时的业务中用到的不多，但是感觉它有点儿意思，所以花了点时间研究一下，特此记录。

## 引入

## 概念

通过数值比较、范围过滤等就可以完成绝大多数我们需要的查询，但是，如果希望通过关键字的匹配来进行查询过滤，那么就需要基于相似度的查询，而不是原来的精确数值比较。全文索引就是为这种场景设计的。

你可能会说，用 like + % 就可以实现模糊匹配了，为什么还要全文索引？like + % 在文本比较少时是合适的，但是对于大量的文本数据检索，是不可想象的。全文索引在大量的数据面前，能比 like + % 快 N 倍，速度不是一个数量级，但是全文索引可能存在精度问题。

你可能没有注意过全文索引，不过至少应该对一种全文索引技术比较熟悉：各种的搜索引擎。虽然搜索引擎的索引对象是超大量的数据，并且通常其背后都不是关系型数据库，不过全文索引的基本原理是一样的。

## 版本支持

开始之前，先说一下全文索引的版本、存储引擎、数据类型的支持情况

1.  MySQL 5.6 以前的版本，只有 MyISAM 存储引擎支持全文索引；
2.  MySQL 5.6 及以后的版本，MyISAM 和 InnoDB 存储引擎均支持全文索引;
3.  只有字段的数据类型为 char、varchar、text 及其系列才可以建全文索引。

测试或使用全文索引时，要先看一下自己的 MySQL 版本、存储引擎和数据类型是否支持全文索引。

## 操作全文索引

索引的操作随便一搜都是，这里还是再啰嗦一遍。

## 创建

1.创建表时创建全文索引

```sql
create table fulltext_test (
    id int(11) NOT NULL AUTO_INCREMENT,
    content text NOT NULL,
    tag varchar(255),
    PRIMARY KEY (id),
    FULLTEXT KEY content_tag_fulltext(content,tag)  // 创建联合全文索引列
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

2.在已存在的表上创建全文索引

```sql
create fulltext index content_tag_fulltext
    on fulltext_test(content,tag);
```

3.通过 SQL 语句 ALTER TABLE 创建全文索引

```sql
alter table fulltext_test
    add fulltext index content_tag_fulltext(content,tag);
```

## 修改

修改个 O，直接删掉重建。

## 删除

1.直接使用 DROP INDEX 删除全文索引

```sql
drop index content_tag_fulltext
    on fulltext_test;
```

2.通过 SQL 语句 ALTER TABLE 删除全文索引

```sql
alter table fulltext_test
    drop index content_tag_fulltext;
```

## 使用全文索引

和常用的模糊匹配使用 like + % 不同，全文索引有自己的语法格式，使用 match 和 against 关键字，比如

```sql
select * from fulltext_test 
    where match(content,tag) against('xxx xxx');
```

**注意：** match() 函数中指定的列必须和全文索引中指定的列完全相同，否则就会报错，无法使用全文索引，这是因为全文索引不会记录关键字来自哪一列。如果想要对某一列使用全文索引，请单独为该列创建全文索引。

## 测试全文索引

## 添加测试数据

有了上面的知识，就可以测试一下全文索引了。

首先创建测试表，插入测试数据

```sql
create table test (
    id int(11) unsigned not null auto_increment,
    content text not null,
    primary key(id),
    fulltext key content_index(content)
) engine=MyISAM default charset=utf8;

insert into test (content) values ('a'),('b'),('c');
insert into test (content) values ('aa'),('bb'),('cc');
insert into test (content) values ('aaa'),('bbb'),('ccc');
insert into test (content) values ('aaaa'),('bbbb'),('cccc');
```

按照全文索引的使用语法执行下面查询

```sql
select * from test where match(content) against('a');
select * from test where match(content) against('aa');
select * from test where match(content) against('aaa');
```

根据我们的惯性思维，应该会显示 4 条记录才对，然而结果是 1 条记录也没有，只有在执行下面的查询时

```sql
select * from test where match(content) against('aaaa');
```

才会搜到 _aaaa_ 这 1 条记录。

为什么？这个问题有很多原因，其中最常见的就是 **最小搜索长度** 导致的。另外插一句，使用全文索引时，测试表里至少要有 4 条以上的记录，否则，会出现意想不到的结果。

MySQL 中的全文索引，有两个变量，最小搜索长度和最大搜索长度，对于长度小于最小搜索长度和大于最大搜索长度的词语，都不会被索引。通俗点就是说，想对一个词语使用全文索引搜索，那么这个词语的长度必须在以上两个变量的区间内。

这两个的默认值可以使用以下命令查看

```sql
show variables like '%ft%';
```

可以看到这两个变量在 MyISAM 和 InnoDB 两种存储引擎下的变量名和默认值

```sql
// MyISAM
ft_min_word_len = 4;
ft_max_word_len = 84;

// InnoDB
innodb_ft_min_token_size = 3;
innodb_ft_max_token_size = 84;
```

可以看到最小搜索长度 MyISAM 引擎下默认是 4，InnoDB 引擎下是 3，也即，MySQL 的全文索引只会对长度大于等于 4 或者 3 的词语建立索引，而刚刚搜索的只有 _aaaa_ 的长度大于等于 4。

## 配置最小搜索长度

全文索引的相关参数都无法进行动态修改，必须通过修改 MySQL 的配置文件来完成。修改最小搜索长度的值为 1，首先打开 MySQL 的配置文件 /etc/my.cnf，在 [mysqld] 的下面追加以下内容

```text
[mysqld]
innodb_ft_min_token_size = 1
ft_min_word_len = 1
```

然后重启 MySQL 服务器，并修复全文索引。注意，修改完参数以后，一定要修复下索引，不然参数不会生效。

两种修复方式，可以使用下面的命令修复

```sql
repair table test quick;
```

或者直接删掉重新建立索引，再次执行上面的查询，_a、aa、aaa_ 就都可以查出来了。

但是，这里还有一个问题，搜索关键字 _a_ 时，为什么 _aa、aaa、aaaa_ 没有出现结果中，讲这个问题之前，先说说两种全文索引。

## 两种全文索引

## 自然语言的全文索引

默认情况下，或者使用 in natural language mode 修饰符时，match() 函数对文本集合执行自然语言搜索，上面的例子都是自然语言的全文索引。

自然语言搜索引擎将计算每一个文档对象和查询的相关度。这里，相关度是基于匹配的关键词的个数，以及关键词在文档中出现的次数。在整个索引中出现次数越少的词语，匹配时的相关度就越高。相反，非常常见的单词将不会被搜索，如果一个词语的在超过 50% 的记录中都出现了，那么自然语言的搜索将不会搜索这类词语。上面提到的，测试表中必须有 4 条以上的记录，就是这个原因。

这个机制也比较好理解，比如说，一个数据表存储的是一篇篇的文章，文章中的常见词、语气词等等，出现的肯定比较多，搜索这些词语就没什么意义了，需要搜索的是那些文章中有特殊意义的词，这样才能把文章区分开。

## 布尔全文索引

在布尔搜索中，我们可以在查询中自定义某个被搜索的词语的相关性，当编写一个布尔搜索查询时，可以通过一些前缀修饰符来定制搜索。

MySQL 内置的修饰符，上面查询最小搜索长度时，搜索结果 ft_boolean___syntax 变量的值就是内置的修饰符，下面简单解释几个，更多修饰符的作用可以查手册

-   **+** 必须包含该词
-   **-** 必须不包含该词
-   **>** 提高该词的相关性，查询的结果靠前
-   **<** 降低该词的相关性，查询的结果靠后
-   **(*)星号** 通配符，只能接在词后面

对于上面提到的问题，可以使用布尔全文索引查询来解决，使用下面的命令，_a、aa、aaa、aaaa_ 就都被查询出来了。

```sql
select * test where match(content) against('a*' in boolean mode);
```

## 总结

好了，差不多写完了，又到了总结的时候。

MySQL 的全文索引最开始仅支持英语，因为英语的词与词之间有空格，使用空格作为分词的分隔符是很方便的。亚洲文字，比如汉语、日语、汉语等，是没有空格的，这就造成了一定的限制。不过 MySQL 5.7.6 开始，引入了一个 ngram 全文分析器来解决这个问题，并且对 MyISAM 和 InnoDB 引擎都有效。

事实上，MyISAM 存储引擎对全文索引的支持有很多的限制，例如表级别锁对性能的影响、数据文件的崩溃、崩溃后的恢复等，这使得 MyISAM 的全文索引对于很多的应用场景并不适合。所以，多数情况下的建议是使用别的解决方案，例如 Sphinx、Lucene 等等第三方的插件，亦或是使用 InnoDB 存储引擎的全文索引。

## 几个注意点

1.  使用全文索引前，搞清楚版本支持情况；
2.  全文索引比 like + % 快 N 倍，但是可能存在精度问题；
3.  如果需要全文索引的是大量数据，建议先添加数据，再创建索引；
4.  对于中文，可以使用 MySQL 5.7.6 之后的版本，或者第三方插件。

**参考文章**

[mysql全文索引__简介](http://blog.51cto.com/imysqldba/1618465)

[MySQL 官方参考手册](https://dev.mysql.com/doc/refman/5.7/en/fulltext-search.html)  
高性能 MySQL（第三版）

> 本文原始链接：[MySQL 之全文索引](http://mrzhouxiaofei.com/2018/04/14/MySQL%20%E4%B9%8B%E5%85%A8%E6%96%87%E7%B4%A2%E5%BC%95/)

编辑于 2018-04-14 21:59

「真诚赞赏，手留余香」

赞赏

还没有人赞赏，快来当第一个赞赏的人吧！

[

MySQL

](https://www.zhihu.com/topic/19554128)

[

全文索引

](https://www.zhihu.com/topic/19569111)

​赞同 80​​8 条评论

​分享

​喜欢​收藏​申请转载

​

![](media/v2-c1ccd457ef0518c33a4ce0a9bf1d4fa9_l.jpg.png)

评论千万条，友善第一条

  

8 条评论

默认

最新

[![XiuYuGe](media/XiuYuGe.png)](https://www.zhihu.com/people/a73cc1f804fe7325871444b21cb183c9)

[XiuYuGe](https://www.zhihu.com/people/a73cc1f804fe7325871444b21cb183c9)

布尔全文索引的SQL语句缺少 from 语句

2019-12-30

​回复​4

[![弱狗就该被强食](media/弱狗就该被强食.jpg)](https://www.zhihu.com/people/5563f514a671942cf94de47a45062943)

[弱狗就该被强食](https://www.zhihu.com/people/5563f514a671942cf94de47a45062943)

![](media/v2-4812630bc27d642f7cafcd6cdeca3d7a.jpg-1.png)

很棒的一篇文章

2020-06-21

​回复​1

[![笑看风云](media/笑看风云.jpg)](https://www.zhihu.com/people/56575cf2f73ff7776184e963479b09f5)

[笑看风云](https://www.zhihu.com/people/56575cf2f73ff7776184e963479b09f5)

没图太干了，全文索引数据怎么存储的，DOC_ID这些也得说一下吧。

2020-11-20

​回复​1

[![愚一](media/愚一.jpg)](https://www.zhihu.com/people/aa49c3bab6cc53a264b9df8a81751a23)

[愚一](https://www.zhihu.com/people/aa49c3bab6cc53a264b9df8a81751a23)

请问为什么“如果需要全文索引的是大量数据，建议先添加数据，再创建索引” 呢？

2021-05-20

​回复​赞

[![prestu](media/prestu.jpg)](https://www.zhihu.com/people/4e56c250040c35aac9f4ce9f027cfaa2)

[prestu](https://www.zhihu.com/people/4e56c250040c35aac9f4ce9f027cfaa2)

不只是全文索引这么干，其他索引也是这样，因为在插入数据时，需要不停的维护索引结构呀，会消耗很多时间的，也影响性能

2022-01-03

​回复​2

[![炼金术士](media/炼金术士.jpg)](https://www.zhihu.com/people/d68f219b85467c00b10c3709352f1be4)

[炼金术士](https://www.zhihu.com/people/d68f219b85467c00b10c3709352f1be4)

不错哦，楼主想问一下，‘全文索引的分词形式貌似不能自己添加吧？’我看‘-’，‘、’这些是可以分开的

2021-03-10

​回复​赞

[![长街](media/长街.jpg)](https://www.zhihu.com/people/f6e40958c38dffed318fd3f14e1ad32d)

[长街](https://www.zhihu.com/people/f6e40958c38dffed318fd3f14e1ad32d)

测试like和布尔全文索引结果不一致，为什么会存在这种精度问题呢？

2020-09-15

​回复​赞

[![霜之哀伤](media/霜之哀伤.jpg)](https://www.zhihu.com/people/2489c9564e298ba466941eced43c024c)

[霜之哀伤](https://www.zhihu.com/people/2489c9564e298ba466941eced43c024c)

	如果不是单词，是单词的一部分，比如订单号的后4位或者后5位，可以支持吗