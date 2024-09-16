KB: 42520更新时间：2022-06-06 10:15[提交缺陷](https://xing.aliyun.com/submit?documentId=42520&website=cn&language=zh)

[产品详情](https://www.aliyun.com/product/ecs)

[相关技术圈](https://developer.aliyun.com/group/ecs/)

[我的收藏](https://help.aliyun.com/my_favorites.html)

## 概述

本文主要介绍忘记MySQL数据库的root密码时如何重置密码。

## 详细信息

### 修改Linux系统中MySQL数据库的root密码

如果忘记了MySQL数据库root用户的密码，可以通过修改配置文件，登录时跳过密码，然后在数据库里面修改密码。一般数据库默认的用户为root。

1.  执行如下命令，编辑MySQL数据库的my.cnf配置文件。
    
    vim /etc/my.cnf
    
    > **说明**：my.cnf配置文件的路径以实际环境为准。
    
2.  在[mysqld]字段下新增如下内容，然后保存退出。
    
    skip-grant-tables
    
3.  执行如下命令，重启MySQL服务。
    
    /etc/init.d/mysqld restart
    
    > **说明**：MySQL启动脚本路径以实际环境为准。
    
4.  执行如下命令，登录数据库。
    
    /usr/bin/mysql
    
    > **说明**：MySQL命令路径以实际环境为准。
    
5.  依次执行如下SQL语句，更新密码。
    
    USE mysql;  
    UPDATE user SET authentication_string = password ('[$Password]') WHERE User = 'root';  
    flush privileges;  
    quit
    
    > **说明**：[$Password]为新密码，不建议新密码为“123456”，此密码太简单，密码需要满足密码复杂性要求，需要大小写字母和数字组合，最小长度为8位，根据此密码策略，设置密码。
    
6.  再次编辑`/etc/my.cnf`配置文件，删除或者注释第2步添加的skip-grant-tables。  
    ![](media/b56cd6f5-c681-415a-8d58-df9a3e78ed10.png)
7.  执行如下命令，重启MySQL服务。
    
    /etc/init.d/mysqld restart
    
8.  使用新密码登录数据库，确认能正常登录。

### 修改Windows系统中MySQL数据库的root密码

本文的操作系统以“Windows Server 2008 R2 标准版 SP1 64位中文版”为例，MySQL版本以”mysql Ver 14.12 Distrib 5.0.87, for Win32 (ia32)”为例，其他版本的操作方法类型。

1.  登录终端，切换至MySQL的bin目录。
    
    > **说明**：MySQL服务的bin目录以实际环境为准。
    
2.  执行如下命令，停止MySQL服务。
    
    net stop mysql
    
    系统显示类似如下。  
    ![QQ???20150501120749.png](media/QQ!20150501120749.png)
3.  执行如下命令，以安全模式启动MySQL服务。
    
    mysqld-nt.exe --skip-grant-tables
    
    系统显示类型如下，注意此终端不能关闭。  
    
    ![](media/71fcdc13-b871-4078-86a2-4103f0a9f8db.png)
    
4.  登录另一个终端，执行如下命令，登录MySQL数据库，在提示输入密码时直接回车即可。  
    
    mysql -uroot -p
    
5.  依次执行如下SQL语句，更新密码，完成后退出。
    
    use mysql;  
    UPDATE user SET authentication_string = password ('[$Password]') WHERE User = 'root';  
    flush privileges;  
    exit
    
    系统显示类型如下。  
    ![](media/08c16721-4ea6-47ff-a15d-3a809cbf1add.png)
6.  打开任务管理器，关闭MySQL相关进程。  
    ![QQ???20150501121404.png](media/QQ!20150501121404.png)
7.  执行如下命令，启动MySQL服务。
    
    net start mysql
    
    系统显示类型如下。  
    ![QQ???20150501121611.png](media/QQ!20150501121611.png)
8.  执行如下命令，使用新密码登录数据库。确认能正常登录数据库说明修改成功。
    
    mysql -uroot -p[$Password]