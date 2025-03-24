![](data:image/svg+xml;charset=utf-8,%3Csvg height='50' width='50' xmlns='http://www.w3.org/2000/svg' version='1.1'%3E%3C/svg%3E)

![A kitten](media/A_kitten.jpg)

蒋川

B 端数据开发，卡拉云联合创始人

最近更新 2022年02月09日

[MySQL](https://kalacloud.com/category/mysql/)[MariaDB](https://kalacloud.com/category/mariadb/)[Linux](https://kalacloud.com/category/linux/)

[![在 MySQL 中 DATETIME 和 TIMESTAMP 的区别及使用场景 - 实战讲解](https://kalacloud.com/static/e0ea97218517bc2ce0fa2b59c18ee8e7/d77b7/head.jpg "在 MySQL 中 DATETIME 和 TIMESTAMP 的区别及使用场景 - 实战讲解")](https://kalacloud.com/static/e0ea97218517bc2ce0fa2b59c18ee8e7/86be0/head.jpg)

在 MySQL 中有两种存储时间的数据类型 `DATETIME` 和 `TIMESTAMP` ，它们在数据库实际应用中，各有各的优势和劣势。本文将详细详解两个数据类型的区别，以及用实战案例说明它们的使用场景。

## [](https://kalacloud.com/blog/difference-between-mysql-datetime-and-timestamp-datatypes/#%E4%B8%80-datetime-%E5%92%8C-timestamp-%E7%9A%84%E7%9B%B8%E5%90%8C%E7%82%B9)一. `DATETIME` 和 `TIMESTAMP` 的相同点

1.  两个数据类型存储时间的格式一致。均为 `YYYY-MM-DD HH:MM:SS`
2.  两个数据类型都包含「日期」和「时间」部分。
3.  两个数据类型都可以存储微秒的小数秒（秒后6位小数秒）

## [](https://kalacloud.com/blog/difference-between-mysql-datetime-and-timestamp-datatypes/#%E4%BA%8C-datetime-%E5%92%8C-timestamp-%E7%9A%84%E5%8C%BA%E5%88%AB)二. `DATETIME` 和 `TIMESTAMP` 的区别

### 1.表示范围

-   `DATETIME`：`1000-01-01 00:00:00.000000` 到 `9999-12-31 23:59:59.999999`
-   `TIMESTAMP`：`'1970-01-01 00:00:01.000000' UTC` 到 `'2038-01-09 03:14:07.999999' UTC`

### 2.空间占用

-   `TIMESTAMP` ：占 4 个字节（小数秒+3 个字节）
-   `DATETIME`：在 MySQL 5.6.4 之前，占 8 个字节 ，之后版本，占 5 个字节。（小数秒+3 个字节）

### 3. 存入时间是否会自动转换？

-   `TIMESTAMP`：TIMESTAMP 的值是从「当前时间」转换成 UTC 时间，或者反过来转换。
-   `DATETIME`：不会做任何转换，也不会检测时区，你给什么数据，它存什么数据。

### 4.使用 `now()` 存储当前时间时，保存的实际值，是否与当前计算机时间一致？

-   `TIMESTAMP`：可能不一致。存储值会被转换成 UTC 时间值再存入数据库。
-   `DATETIME`：与当前时间是一致的。

### 5.如果存入的是 `NULL` 时，两个类型如何存储？

-   `TIMESTAMP`：会自动存储当前时间（ `now()` ）。
-   `DATETIME`：不会自动存储当前时间，会直接存入 `NULL` 值。

![](media/home-demo-66b9606d95623721268b764851017602.gif)

## 连接数据库后需要开发后台系统？


## [](https://kalacloud.com/blog/difference-between-mysql-datetime-and-timestamp-datatypes/#%E4%B8%89-%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF%E8%BE%A8%E6%9E%90)三. 使用场景辨析

在什么场景中，使用 `DATETIME` 或 `TIMESTAMP` 更合适？

### `TIMESTAMP` 使用场景：计算飞机飞行时间

一架飞机，从中国北京起飞，降落在美国纽约，计算它从北京飞往纽约的飞行时间。飞机在北京时间 2021-10-10 11:05:00 从北京起飞，在纽约时间 2021-10-10 09:50:00 降落（JL8006）。

这个场景中，如果使用 `TIMESTAMP` 来存时间，起飞和降落时间的值，都会被转换成 UTC 时间，所以它们直接相减即可获得结果。但如果使用 `DATATIME` 格式存时间，还需要进行转换，才可以完成，容易出错。

### `DATATIME` 使用场景：记录信息修改时间

如果只是记录文件修改时间，最后更新时间这种不涉及加减转换的情况，用 `DATATIME` 来存更直接，更方便，可读性高，不绕弯子，不容易出错。

## [](https://kalacloud.com/blog/difference-between-mysql-datetime-and-timestamp-datatypes/#%E5%9B%9B-timestamp--datatime-%E5%AE%9E%E6%88%98%E6%A1%88%E4%BE%8B)四. `TIMESTAMP` & `DATATIME` 实战案例

接下来我们来一起动手操作一次，只有自己敲一次代码，观察一下返回结果，才能深入理解，印象深刻。就像[卡拉云](https://kalacloud.com/)技术博客一贯的写作风格一样，本文也将附 MySQL 实战教程。

首先我们进入 MySQL sell，新建一个库：

```bash
CREATE DATABASE kalacloud_demo_database;
```

然后我们 `USE` 这个库，接下来的演示，在这个新建库中进行。

```bash
USE kalacloud_demo_database;
```

接着，我们新建一个包含`TIMESTAMP` & `DATATIME` 两个数据类型的表：

```bash
CREATE TABLE time_demo_kalacloud (`timestamp` timestamp,`datetime` datetime);
```

好，现在我们往 `time_demo_kalacloud` 插入几组数据，看一下两个数据类型的存储样式：

```bash
insert into time_demo_kalacloud values
(NULL,NULL),
(now(),now()),
('19970701171207','19970701171207');
```

使用 `SELECT` 看一下数据库中的存储结果：

```bash
select * from time_demo_kalacloud;
```

[![使用 select 查看刚刚写入数据库的三组数据](https://kalacloud.com/static/f0705ea28435dae630915c799e84f866/42e17/01-time-demo-kalacloud-check-one.png "使用 select 查看刚刚写入数据库的三组数据")](https://kalacloud.com/static/f0705ea28435dae630915c799e84f866/6a6ab/01-time-demo-kalacloud-check-one.png)

接着我们来改一下本地计算机时区，把 +8 区改为 +10 区，看看刚刚已经存储到数据库中的时间有什么变化：

```bash
set time_zone = '+10:00';
select * from time_demo_kalacloud;
```

[![使用 select 查看更改时区后三组数据的变化](https://kalacloud.com/static/5d1c95294ea1c370dd748ebdc95a8c89/42e17/02-time-demo-kalacloud-check-two.png "使用 select 查看更改时区后三组数据的变化")](https://kalacloud.com/static/5d1c95294ea1c370dd748ebdc95a8c89/470f1/02-time-demo-kalacloud-check-two.png)

请对比输入值，仔细观察以上两个输出结果时间的变化，瞬间搞明白它们的区别。

### **实战案例小结：**

1.向数据插入 `NULL` 时：`timestamp`存入了当前时间，而`datetime` 直接存了 `NULL` 本身。

2.向数据插入 `now()` 时：更改时区后，`timestamp` 随时区变化（小时+2），`datetime` 没有变化。

3.向数据库直接插入时间值：更改时区后，`timestamp` 随时区变化（小时+2），`datetime` 没有变化。

## [](https://kalacloud.com/blog/difference-between-mysql-datetime-and-timestamp-datatypes/#%E4%BA%94-%E6%80%BB%E7%BB%93)五. 总结

MySQL 有非常多相似的数据类型，灵活掌握它们细微的区别，对你在实战应用中，有极大帮助。这里顺便推荐一下，同样对你有极大帮助的新一代低代码开发工具 —— 卡拉云，不用懂前端，只要会写 SQL ，就能几分钟内作出趁手的工具，马上分享使用。

卡拉云是新一代低代码开发工具，免安装部署，可一键接入包括 MySQL 在内的常见数据库及 API。可根据自己的工作流，定制开发。无需繁琐的前端开发，只需要简单拖拽，即可快速搭建企业内部工具。**数月的开发工作量，使用卡拉云后可缩减至数天，[欢迎免费试用卡拉云](https://kalacloud.com/)。**

[![卡拉云可一键接入常见的数据库及 API](https://kalacloud.com/static/18822b2a23183deb7d11dd484a7f65aa/42e17/97-kalacloud-sql.png "卡拉云可一键接入常见的数据库及 API")](https://kalacloud.com/static/18822b2a23183deb7d11dd484a7f65aa/8ebf5/97-kalacloud-sql.png)

卡拉云可一键接入常见的数据库及 API

卡拉云可根据公司工作流需求，轻松搭建数据看板或其他内部工具，并且可一键分享给组内的小伙伴。

![使用卡拉云轻松搭建企业内部工具](media/使用卡拉云轻松搭建企业内部工具.gif)

下图为使用卡拉云在 5 分钟内搭建的「优惠券发放核销」后台，仅需要简单拖拽即可快速生成前端组件，只要会写 SQL，便可搭建一套趁手的数据库工具。**[欢迎免费试用卡拉云](https://kalacloud.com/)。**

[![使用卡拉云在 5 分钟内搭建的「优惠券发放核销」后台](https://kalacloud.com/static/34625d3adaea4ed250ff3f05b863e47c/42e17/99-kalacloud-sql-index.png "使用卡拉云在 5 分钟内搭建的「优惠券发放核销」后台")](https://kalacloud.com/static/34625d3adaea4ed250ff3f05b863e47c/5bfc8/99-kalacloud-sql-index.png)

更多数据库相关教程可访问 [卡拉云](https://kalacloud.com/) 查看。如果你还有什么疑问，欢迎一起讨论。我的微信 HiJiangChuan。

[![卡拉云联合创始人蒋川的微信](https://kalacloud.com/static/c2a43a22098d123bb577fc5498d0bd57/d77b7/wechat-jiangchuan.jpg "卡拉云联合创始人蒋川的微信")](https://kalacloud.com/static/c2a43a22098d123bb577fc5498d0bd57/86be0/wechat-jiangchuan.jpg)

有关 MySQL 教程，可继续拓展学习：

-   [如何远程连接 MySQL 数据库，阿里云腾讯云外网连接教程](https://kalacloud.com/blog/how-to-allow-remote-access-to-mysql/)
-   [如何在两台服务器之间迁移 MySQL 数据库 阿里云腾讯云迁移案例](https://kalacloud.com/blog/how-to-migrate-a-mysql-database-between-two-servers-aliyun-tencentyun/)
-   [MySQL 中如何实现 BLOB 数据类型的存取，BLOB 有哪些应用场景？](https://kalacloud.com/blog/how-to-use-the-mysql-blob-data-type-to-store-images-with-php-or-kalacloud/)
-   [MySQL 触发器的创建、使用、查看、删除教程及应用场景实战案例](https://kalacloud.com/blog/how-to-manage-and-use-mysql-database-triggers/)