# 数据库脚本工单管理工具Yearning

[database](https://felord.cn/categories/database/) [database](https://felord.cn/tags/database/) 2021/06/11

平常我们线上执行的SQL脚本都是很粗犷的。呼叫一下DBA或者运维，把脚本发过去，然后告诉他在哪个环境执行。然后双方沟通不畅，测试环境的脚本执行到生产了！脚本写的有问题执行错了却没有回滚脚本！或者每个人都有执行SQL脚本的权利，出事之后互相甩锅！等等一系列问题都是胖哥遇到过的。

迫切需要一个工具能够解决上面的问题。经过调研使用了名叫**Yearning**的SQL审计工具。经过两星期的试用，都交口称赞。所以特来安利一下这个工具。

## [](https://felord.cn/yearning.html#Yearning-SQL审计平台 "Yearning SQL审计平台")Yearning SQL审计平台

**Yearning** 是面向中小型企业的轻量级MySQL SQL语句审核平台，提供查询审计，SQL审核，权限控制，自定义审核流程等功能。规范了SQL脚本执行的流程，降低了数据损坏丢失的风险。安装非常简单，可以到中文文档 `https://guide.yearning.io/`了解，这里就不多说了，接下来主要谈谈个人的使用心得。

## [](https://felord.cn/yearning.html#使用心得分享 "使用心得分享")使用心得分享

**Yearning**部署好后，你可以将需要管理的MySQL数据源配置进去。

### [](https://felord.cn/yearning.html#角色帐号 "角色帐号")角色帐号

然后就是创建和分配帐号了，除了自带的超级管理员外，我们要创建两种帐号：

Yearning的用户角色分别为:提交人，操作人，超级管理员(仅admin用户) 三类。

**提交人帐号**: 用来提交的SQL工单,查询工单的功能，分给普通开发者用来提交SQL脚本工单。

**操作人帐号**: 除了有提交人帐号拥有的功能外，还有审核并执行SQL工单的功能，这种帐号一般分给运维或者DBA使用。

### [](https://felord.cn/yearning.html#工单 "工单")工单

工单能够规范SQL脚本的执行流程，将执行的过程记录清楚，作为后面复盘和~~甩锅背锅~~的依据。

这时候你在给DBA发脚本，他会让你老老实实提交工单，白纸黑字写清楚脚本的基本信息。

![提交SQL工单](media/提交SQL工单.png)

提交完了，DBA审核你的脚本是否合规，做出批准和驳回的决定 。

![SQL工单审核](media/SQL工单审核.png)

提交人还可以查询自己的提交记录、审核结果、执行情况。

![查询我的工单](media/查询我的工单.png)

看到了吧，一切清清楚楚，明明白白！规范了流程，降低了沟通成本，并对执行的过程记录在案。还能自动生成回滚语句以防不测。

最关键的是颜值也非常高！

![](media/20210527233632.png)

### [](https://felord.cn/yearning.html#注意事项 "注意事项")注意事项

**Yearning** 目前兼容99%的MySQL标准SQL语法，**目前不支持跨库DML语句回滚，也不支持存储过程和触发器，好像外键也不支持**。

## [](https://felord.cn/yearning.html#总结 "总结")总结

**Yearning**可以规范中小团队MySQL的SQL审计管理。如果你的应用很多，或者开发团队已经初具规模，你可以去试一试**Yearning**。好了今天的分享就到这里，多多关注：**码农小胖哥** ，获取更多能够帮助你开发和管理的效率工具。

![关注一下吧！](media/关注一下吧！.bmp)

转载声明： 本站已签约维权机构，商业转载请联系作者获得授权，非商业转载请注明出处链接。 © [felord.cn](https://felord.cn/yearning.html)

评论系统未开启，无法评论！

### 文章目录

1. [Yearning SQL审计平台](https://felord.cn/yearning.html#Yearning-SQL%E5%AE%A1%E8%AE%A1%E5%B9%B3%E5%8F%B0)
2. [使用心得分享](https://felord.cn/yearning.html#%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%E5%88%86%E4%BA%AB)
    1. [角色帐号](https://felord.cn/yearning.html#%E8%A7%92%E8%89%B2%E5%B8%90%E5%8F%B7)
    2. [工单](https://felord.cn/yearning.html#%E5%B7%A5%E5%8D%95)
    3. [注意事项](https://felord.cn/yearning.html#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)
3. [总结](https://felord.cn/yearning.html#%E6%80%BB%E7%BB%93)

访问量:   |   访客数:

Copyright © 2019-2022   [**felord.cn**](https://felord.cn/) 版权所有   [豫ICP备19038867号-1](http://beian.miit.gov.cn/)  本站云存储由[![](media/upyun.svg)](https://www.upyun.com/?utm_source=lianmeng&utm_medium=referral)提供CDN加速/云存储服务