最近在装mysql练练手，遇到了好多坑，够写好几篇博客了哈哈
其中一个就是flush privileges; 的时候遇到了Table ‘mysql.servers’ doesn’t exist
在这里插入图片描述

然后我就去百度查，答案全都是一模一样的转来转去，表不存在嘛创建一个就好

use mysql;
CREATE TABLE `servers` (
`Server_name` char(64) NOT NULL,
`Host` char(64) NOT NULL,
`Db` char(64) NOT NULL,
`Username` char(64) NOT NULL,
`Password` char(64) NOT NULL,
`Port` int(4) DEFAULT NULL,
`Socket` char(64) DEFAULT NULL,
`Wrapper` char(64) NOT NULL,
`Owner` char(64) NOT NULL,
PRIMARY KEY (`Server_name`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13

但是我执行后还是报错Table ‘mysql.servers’ doesn’t exist
在这里插入图片描述
然后我去库里show tables;

发现已经有这张表了

所以我们现在的正确做法是把表删了重新建

drop table if exists mysql.servers;

    1

然后再执行上面的那个建表sql，
建成功后刷新权限就完成了