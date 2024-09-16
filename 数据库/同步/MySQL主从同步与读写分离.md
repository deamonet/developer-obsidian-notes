2017-12-13 1513 举报

**简介：**

MySQL主从同步  
MySQL AB复制  
1.对指定库的异地同步。  
2.MySQL主-->从复制架构的实现。  
3.MySQL服务器的只读控制。

主从：  
单向复制时，建议将从库设置为只读。

主从复制的原理：  
Master,记录数据更改操作。  
-启动binlog日志  
-设置binlog日志格式  
-设置server_id

Slave，运行2个线程。  
-Slave_IO：复制master主机binlog日志文件里的SQL到本机的relay-log文件里。  
-Slave_SQL：执行本机relay-log文件里的SQL语句，重现Master的数据操作。

构建主从同步  
1.确保数据相同  
-从库必须要有主库上的数据。

2.配置主服务器  
-启用binlog日志及设置格式，设置server_id,授权用户。

3.配置从服务器  
-设置server_id，配置为从数据库服务器。

4.测试同步效果  
-客户端连接主库，写入的数据，在访问从库的时候也能够看到。

确保数据相同  
Master服务器：  
-应包括希望同步的所有库  
-对采用MyISAM的库，可离线备份  
mysql>reset master; //重置binlog日志  
...  
#mysqldump -u root -p 123456 -B mysql test > /root/mytest.sql

Slave服务器：  
-离线导入有Master提供的备份  
-清空同名库（如果存在）  
mysql>drop database test; //先清理目标库  
#scp master:/root/mytest.sql ./ //直接scp远程拷贝  
#mysql -u root -p 123456 < /root/mytest.sql //确保与master的数据相同

配置主服务器  
1.启用binlog及允许同步：  
vim /etc/my.cnf  
[mysqld]  
log_bin=master-bin //启用binlog日志  
server_id = 10 //指定服务器ID号  
binlog_format=mixed //指定日志格式  
sync-binlog=1 //允许日志同步  
....

#systemctl start mysqld //启动服务

2.授权备份用户：  
-允许lisi从192.168.4.0/24网段访问  
-对所有库（默认不允许对单个库）有同步权限  
mysql>grant replication slave on _._ to "lisi"@"192.168.4.%" identified by "123456";

3.查看Master状态：  
-记住当前的日志文件名、偏移量位置  
mysql>show master status\G;  
......  
File: master-bin.000002 //日志文件名  
Position: 334 //偏移量  
.....

配置从服务器  
1.启用binlog及允许同步，启用只读模式。  
vim /etc/my.cnf  
[mysqld]   
log_bin=slave-bin //启用binlog日志  
server_id = 20 //指定服务器ID号  
sync-binlog=1 //允许日志同步  
read_only=1 //只读模式，同步及SUPER权限用户例外

#systemctl start mysqld //启动服务

2.指定Master相关参数  
mysql>change master to master_host="192.168.4.10",  
->master_user="lisi",  
->master_password="123456",  
->master_log_file="master-bin.000002", //日志文件  
->master_log_pos=334; //偏移量

mysql>start slave; //启动复制

Master信息会自动保存到/var/lib/mysql/master.info文件，若要更改Master信息时，应先stop slave;

3.查看Slave状态  
-确认IO线程、SQL线程都已运行  
mysql>show slave status\G  
.....  
Slave_IO_Running:Yes //IO线程已运行  
Slave_SQL_Running:Yes //SQL线程已运行

从数据库目录下多出文件：  
/var/lib/mysql/master.info //连接主服务器信息  
/var/lib/mysql/relay-log.info //中继日志信息  
/var/lib/mysql/主机名-relay-bin.xxxxxx //中继日志文件  
/var/lib/mysql/bogon-relay-bin.index //中继日志索引文件

测试主从同步配置：  
1 在主库服务器上添加访问数据的用户  
2 在客户端使用授权用户连接主库，产生的数据在从库本机也能够查看的到。

Master服务器常用选项：  
binlog_do_db= //只允许复制的库 binlog_do_db=库名1，库名2，库名n  
binlog_ignore_db= //不允许复制的库 binlog_ignore_db=库名1，库名2，库名n

Slave服务器常用选项：  
relay_log=Slave-relay-bin //指定中继日志文件名  
log_slave_updates //记录从库更新，允许链式复制（级联复制）  
replicate_do_db=库名1，库名2，库名n //仅复制指定库，其他库将被忽略（省略时复制所有库）  
replicate_ignore_db=库名1，库名2，库名n //不复制哪些库，其他库将被忽略

注：replicate_do_db和replicate_ignore_db只需选用其中一种

主从复制的结构  
单向渎职：主-->从  
链式复制：主-->从-->从  
双向复制：主<-->从  
放射式复制：从<--主-->从

```
        MySQL读写分离
```

读写分离：把客户端访问数据时的查询请求select 和写insert 给不同的数据库服务器处理。  
读写分离的原理：  
1.多台MySQL服务器  
-分别提供读、写服务，均衡流量  
-通过主从复制保持一致性  
2.由MySQL代理面向客户端  
-收到SQL写请求时，交给服务器A处理  
-收到SQL读请求时，交给服务器B处理  
-具体区分策略由服务设置

构建MySQL读写分离  
1.搭建好MySQL主从复制  
-Slave为只读

2.添加一台MySQL代理服务器  
-部署/启用 maxscale

3.客户端通过代理主机访问MySQL数据库

部署MySQL代理  
安装maxscale：  
rpm -ivh maxscale-2.1.2-1.rhel.7.x86_64.rpm  
rpm -qc maxscale  
/etc/maxscale.cnf //主配置文件  
....

修改配置文件：  
vim /etc/maxscale.cnf  
[server1] //定义数据库服务器  
type=server  
address=192.168.4.10 //master主机ip地址  
port=3306  
protocol=MySQLBackend

[server2] //定义数据库服务器  
type=server  
address=192.168.4.20 //slave主机ip地址  
port=3306  
protocol=MySQLBackend

[MySQL Monitor]  
type=monitor  
module=mysqlmon  
server=server1,server2 //定义的主、从数据库服务器列表  
user=lisi //用户名  
passwd=123456 //密码  
monitor_interval=10000

[Read-Write Service]  
type=service  
router=readwritesplit  
servers=server1,server2 //定义的主、从数据库服务器列表  
user=zhangsan //用户名  
passwd=123456 //密码  
max_slave_connections=100%

在主、从数据库服务器上创建授权用户  
mysql>grant replication slave,replication client on _._ to lisi@'%' identified by "123456" ; //创建监控用户  
mysql>grant select on mysql. _to zhangsan@'%' identified by "123456"; //创建路由用户   
mysql>grant all on_ .* to admin@'%' identified by "123456"; //创建访问用户

启动maxscale  
主要命令：  
-启动服务  
-查看端口  
-停止服务  
maxscale --config=/etc/maxscale.cnf //启动服务  
或 maxscale -f /etc/maxscale.cnf  
netstat -alntpu |grep maxscale   
pkill -9 maxscale //停止服务

读写分离服务使用的端口、管理服务使用的端口  
4006 4009

客户端测试  
登录MySQL代理:  
-mysql -h 代理的ip地址 -p 端口 -u 用户名 -p 密码

测试SQL查询、更新操作  
-可成功查询表记录  
-可成功写入数据

登录MySQL代理：  
mysql -h192.168.4.100 -p4006 -uadmin -p123456  
mysql>select @@hostname; //查看当前访问的主机名

  

     本文转自夜流璃雨 51CTO博客，原文链接：http://blog.51cto.com/13399294/2071492，如需转载请自行联系原作者