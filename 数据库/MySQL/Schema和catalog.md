在学习数据库时，会遇到一个让人迷糊的Schema的概念。实际上，schema就是数据库对象的集合，这个集合包含了各种对象如：表、视图、存储过程、索引等。

如果把database看作是一个仓库，仓库很多房间（schema），一个schema代表一个房间，table可以看作是每个房间中的储物柜，user是每个schema的主人，有操作数据库中每个房间的权利，就是说每个数据库映射user有每个schema（房间）的钥匙。

作者：garvey.lian 链接：https://www.zhihu.com/question/20355738/answer/230743815


作者：夏天 链接：https://www.zhihu.com/question/20355738/answer/111087933

对于mysql，schema和database可以理解为等价的.

As defined in the MySQL Glossary:

In MySQL, physically, a schema is synonymous with a database. You can substitute the keyword SCHEMA instead of DATABASE in MySQL SQL syntax, for example using CREATE SCHEMA instead of CREATE DATABASE.

Some other database products draw a distinction. For example, in the Oracle Database product, a schema represents only a part of a database: the tables and other objects owned by a single user.

[Difference Between Schema / Database in MySQL](https://link.zhihu.com/?target=http%3A//stackoverflow.com/questions/11618277/difference-between-schema-database-in-mysql)

多用用Google搜索吧，英文资料多一点，准确一点。

[编辑于 2016-07-14 14:48](http://www.zhihu.com/question/20355738/answer/111087933)