
Y1ran_

已于 2022-09-18 11:35:08 修改

7243
 收藏 12
分类专栏： NoSql数据库 Mysql数据库 文章标签： 数据结构 数据 数据库 mysql 编码
版权

NoSql数据库
同时被 2 个专栏收录
11 篇文章0 订阅
订阅专栏

Mysql数据库
16 篇文章0 订阅
订阅专栏
对于MYSQL的字符集转换问题，我们通常都知道，SET NAMES UFT8作为声明
该语句作用相当于
SET character_set_client = x;
SET character_set_results = x;
SET character_set_connection = x;
 
set names utf8 是用于设置编码，可以再在建数据库的时候设置，也可以在创建表的时候设置，或只是对部分字段进行设置，而且在设置编码的时候，这些地方最好是一致的，这样能最大程度上避免数据记录出现乱码。
 
执行SET NAMES utf8的效果等同于同时设定如下：
SET character_set_client='utf8';
SET character_set_connection='utf8';
SET character_set_results='utf8';
 
另外，如果数据出现乱码可以试着用以下办法解决：
一、避免创建数据库及表出现中文乱码和查看编码方法
1、创建数据库的时候：CREATE DATABASE `test`
CHARACTER SET 'utf8'
COLLATE 'utf8_general_ci';
2、建表的时候 CREATE TABLE `database_user` (
`ID` varchar(40) NOT NULL default '',
`UserID` varchar(40) NOT NULL default '',
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
 
这3个设置好了，基本就不会出问题了,即建库和建表时都使用相同的编码格式。
如果是已经建了库和表可以通过以下方式进行查询。
1.查看默认的编码格式:
mysql> show variables like "%char%";
+--------------------------+---------------+
| Variable_name | Value |
+--------------------------+---------------+
| character_set_client | gbk |
| character_set_connection | gbk |
| character_set_database | utf8 |
| character_set_filesystem | binary |
| character_set_results | gbk |
| character_set_server | utf8 |
| character_set_system | utf8 |
+--------------------------+-------------+
注：以前2个来确定,可以使用set names utf8,set names gbk设置默认的编码格式;
乱码。

执行SET NAMES utf8的效果等同于同时设定如下：
SET character_set_client='utf8';
SET character_set_connection='utf8';
SET character_set_results='utf8';

另外，如果数据出现乱码可以试着用以下办法解决：
一、避免创建数据库及表出现中文乱码和查看编码方法
1、创建数据库的时候：CREATE DATABASE `test`
CHARACTER SET 'utf8'
COLLATE 'utf8_general_ci';
2、建表的时候 CREATE TABLE `database_user` (
`ID` varchar(40) NOT NULL default '',
`UserID` varchar(40) NOT NULL default '',
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

这3个设置好了，基本就不会出问题了,即建库和建表时都使用相同的编码格式。
如果是已经建了库和表可以通过以下方式进行查询。
1.查看默认的编码格式:
mysql> show variables like "%char%";
+--------------------------+---------------+
| Variable_name | Value |
+--------------------------+---------------+
| character_set_client | gbk |
| character_set_connection | gbk |
| character_set_database | utf8 |
| character_set_filesystem | binary |
| character_set_results | gbk |
| character_set_server | utf8 |
| character_set_system | utf8 |
+--------------------------+-------------+
注：以前2个来确定,可以使用set names utf8,set names gbk设置默认的编码格式;
此外，Windows下可通过修改my.ini内的

# CLIENT SECTION
[mysql]
default-character-set=utf8
# SERVER SECTION
[mysqld]
default-character-set=utf8
这两个字段来更改数据库的默认字符集。第一个是客户端默认的字符集，第二个是服务器端默认的字符集。假设我们把两个都设为utf8，然后在MySQL Command Line Client里面输入“show variables like “character_set_%”;”，可看到如下字符：

character_set_client latin1
character_set_connection latin1
character_set_database utf8
character_set_results latin1
character_set_server utf8
character_set_system utf8

其中的utf8随着我们上面的设置而改动。此时，要是我们通过采用UTF-8的PHP程序从数据库里读取数据，很有可能是一串“?????” 或者是其他乱码。网上查了半天，解决办法倒是简单，在连接数据库之后，读取数据之前，先执行一项查询“SET NAMES UTF8”，即在PHP里为

mysql_query("SET NAMES UTF8"); (注意大写)
即可显示正常（只要数据库里信息的字符正常）。为什么会这样？这句查询“SET NAMES UTF8”到底是什么作用？

到MySQL命令行输入“SET NAMES UTF8;”，然后执行“show variables like “character_set_%”;”，发现原来为latin1的那些变量“character_set_client”、“character_set_connection”、“character_set_results”的值全部变为utf8了，原来是这3个变量在捣蛋。查阅手册，上面那句等于：

SET character_set_client = utf8;
SET character_set_results = utf8;
SET character_set_connection = utf8;
信息输入路径：client→connection→server；
信息输出路径：server→connection→results。

换句话说，每个路径要经过3次改变字符集编码。以出现乱码的输出为例，server里utf8的数据，传入connection转为latin1，传入results转为latin1，utf-8页面又把results转过来。如果两种字符集不兼容，比如latin1和utf8，转化过程就为不可逆的，破坏性的。所以就转不回来了。

但这里要声明一点，“SET NAMES UTF8”作用只是临时的，MySQL重启后就恢复默认了。

接下来就说到MySQL在服务器上的配置问题了。岂不是我们每次对数据库读写都得加上“SET NAMES UTF8”，以保证数据传输的编码一致？能不能通过配置MySQL来达到那三个变量默认就为我们要想的字符集？手册上没说，我在网上也没找到答案。所以，从服务器配置的角度而言，是没办法省略掉那行代码的。

总结以下几点:

要保证select的返回与程序编码一致,即character_set_results与程序编码一致.

要保证程序编码与浏览器一致,即程序编码与<meta http-equiv="Content-Type" content="text/html; charset=?" />一致.