mysql查看最近执行的sql语句，默认情况下mysql是不会记录最近执行sql语句的，需要手动开启才能记录。另外最近执行sql语句有两种方式输出，要么是table，要么是文件。

查看mysql是否开启sql记录以及输出方式的脚本如下：

show variables like '%log_output%'; -- 查看输出方式
show variables like '%general_log%'; -- 查看是否开启

开启和关闭日志记录的脚本：

```sql
set GLOBAL general_log=on;

set GLOBAL general_log=off;
```
日志输出方式如下：

```sql
set GLOBAL log_output='table';


set GLOBAL log_output='file'; 
```


mysql.general_log 表格记录了所有SQL记录，记录SQL语句的数据类型是BLOB