
crontab命令里一定要使用全局路径，当前路径以及变量都是不可取的。

一个典型的配置如下

```shell
*/15 * * * * /etc/profile;/bin/sh /solax/scripts/storm-monitor.sh >> /solax/scripts/storm-monitor.log 2>&1
```


# crontab使用教程

发表于 2019-11-18

crontab是一个定时任务守护进行，可以精确到分钟，大多用于定时提醒、运行脚本等对时间精度要求不高的场景。

## [](https://lovethegirl.github.io/2019/11/18/crontab/#crontab文件的分析 "crontab文件的分析")crontab文件的分析

```
日志文件目录: /var/log/cron*
crontab文件: /etc/crontab，文件的内容如下：

```

```
SHELL=/bin/bashPATH=/sbin:/bin:/usr/sbin:/usr/binMAILTO=root# For details see man 4 crontabs# Example of job definition:# .---------------- minute (0 - 59)# |  .------------- hour (0 - 23)# |  |  .---------- day of month (1 - 31)# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat# |  |  |  |  |# *  *  *  *  * user-name  command to be executed

第一行SHELL变量指定了系统要使用哪个shell，这里是bash

第二行PATH变量指定了系统执行命令的路径

第三行MAILTO变量指定了crond的任务执行信息将通过电子邮件发送给root用户；如果MAILTO变量的值为空，则表示不发送任务执行信息给用户

星号（*）：代表所有可能的值，如month字段为星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。

第一个*代表minute,范围为0 - 59
第二个*代表hour，范围为0 - 23
第三个*代表day of mouth,范围为 1 -31
第四个*代表mouth，范围为 1 - 12
第五个*代表day of week,范围为0 - 6

逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”

中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”

正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。
```

## [](https://lovethegirl.github.io/2019/11/18/crontab/#常见的命令参数 "常见的命令参数")常见的命令参数

|   |
|---|
|usage:  crontab [-u user] file  <br>      crontab [-u user] [ -e \| -l \| -r ]  <br>              (default operation is replace, per 1003.2)  <br>      -e      (edit user's crontab)  <br>      -l      (list user's crontab)  <br>      -r      (delete user's crontab)  <br>      -i      (prompt before deleting user's crontab)  <br>      -s      (selinux context)|

## [](https://lovethegirl.github.io/2019/11/18/crontab/#crontab的日志路径 "crontab的日志路径")crontab的日志路径

```
crontab的日志是按照时间的顺序排列的，日志路径为 /var/log/cron*
/var/log/cron只会记录是否执行了某些计划的脚本，但是具体执行是否正确以及脚本执行过程中的一些信息则linux会每次都发邮件到该用户下。
```

|   |
|---|
|Nov 12 14:45:01 ceph-node2 CROND[62088]: (root) CMD (/usr/bin/echo `date` > /root/test.txt )  <br>Nov 12 14:46:01 ceph-node2 CROND[62110]: (root) CMD (/usr/bin/echo `date` > /root/test.txt )  <br>Nov 12 14:47:01 ceph-node2 CROND[62130]: (root) CMD (/usr/bin/echo `date` > /root/test.txt )  <br>Nov 12 14:48:01 ceph-node2 CROND[62151]: (root) CMD (/usr/bin/echo `date` > /root/test.txt )  <br>Nov 12 14:49:01 ceph-node2 CROND[62171]: (root) CMD (/usr/bin/echo `date` > /root/test.txt )  <br>Nov 12 14:50:01 ceph-node2 CROND[62193]: (root) CMD (/usr/lib64/sa/sa1 1 1)  <br>Nov 12 14:50:01 ceph-node2 CROND[62194]: (root) CMD (/usr/bin/echo `date` > /root/test.txt )  <br>Nov 12 14:51:01 ceph-node2 CROND[62215]: (root) CMD (/usr/bin/echo `date` > /root/test.txt )  <br>Nov 12 14:52:01 ceph-node2 CROND[62312]: (root) CMD (/usr/bin/echo `date` > /root/test.txt )|

## [](https://lovethegirl.github.io/2019/11/18/crontab/#常用命令 "常用命令")常用命令

### [](https://lovethegirl.github.io/2019/11/18/crontab/#crontab安装以及启动服务 "crontab安装以及启动服务")crontab安装以及启动服务

|   |
|---|
|//安装crontab  <br>yum install crontabs  <br>  <br>//crontab的服务操作说明  <br>/sbin/service crond start //启动服务  <br>  <br>/sbin/service crond stop //关闭服务  <br>  <br>/sbin/service crond restart //重启服务  <br>  <br>/sbin/service crond reload //重新载入配置  <br>  <br>//查看crontab服务状态：  <br>	  <br>  service crond status  <br>  <br>//手动启动crontab服务：  <br> service crond status|

### [](https://lovethegirl.github.io/2019/11/18/crontab/#编辑crontab服务 "编辑crontab服务")编辑crontab服务

|   |
|---|
|//编辑定时任务  <br>  crontab –e  <br>  ==》vim /var/spool/cron/root  <br>  <br>// 每隔2分钟输出时间到文件  <br>  */2 * * * * echo `date` >> $HOME>test.txt  <br>  <br>//crontab -r 删除定时任务  <br>  <br>==> 从/var/spool/cron目录中删除用户的crontab文件  <br>==> 如果不指定用户，则默认删除当前用户的crontab文件  <br>crontab –i  在删除用户的crontab文件时给确认提示  <br>  <br>//备份crontab文件  <br>crontab -l > $HOME/mycron|

### [](https://lovethegirl.github.io/2019/11/18/crontab/#crontab常用实例 "crontab常用实例")crontab常用实例

```
//每小时执行/etc/cron.hourly目录内的脚本
0 * * * * root run-parts /etc/cron.hourly

//每隔2分钟同步一次互联网时间
echo "*/2 * * * * /usr/bin/ntpstat time.windows.com >/dev/null 2>&1" >> /var/spool/cron/root

//每天3-5,17-20每隔30分钟执行一次脚本
echo "*/30 [3-5],[17-20] * * * /bin/sh /home/omc/h.sh >/dev/null 2>&1" >> /var/spool/cron/root
```