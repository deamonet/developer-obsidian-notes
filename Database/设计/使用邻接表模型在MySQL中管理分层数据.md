[BEGTUT.COM](https://www.begtut.com)

[Beginner Tutorials](https://www.begtut.com)

轻松上手,快乐学习!

使用邻接表模型在MySQL中管理分层数据

简介：在本教程中，您将学习如何使用邻接表模型在MySQL中管理分层数据。

邻接表模型简介

分层数据无处不在。它可以是博客类别，产品层次结构或组织结构。

在MySQL中有许多方法可以管理分层数据，而邻接表模型可能是最简单的解决方案。由于其简单性，邻接列表模型是开发人员和数据库管理员非常喜欢的选择。

在邻接列表模型中，每个节点都有一个指向其父节点的指针。顶层节点没有父节点。请参阅以下电子产品类别：

![](data:image/png;base64,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)

在使用邻接表模型之前，您应该熟悉一些术语：

-   Electronics是顶级节点或根节点。
-   Laptops, Cameras & photo, Phones & Accessories节点是Electronics节点的子代。反之亦然，电子节点Laptops, Cameras & photo, Phones & Accessories是节点的父节点。
-   叶子节点是没有孩子如节点Laptops，PC，Android，iOS，等，而非叶节点是具有至少一个子代。
-   节点的子孙称为子孙。节点的父母，祖父母等也称为祖先。

为了模拟这一类的树，我们可以创建一个命名的表category有三列：id，title，和parent_id如下：

CREATETABLEcategory (  
id int(10) unsigned NOTNULLAUTO_INCREMENT,  
title varchar(255) NOTNULL,  
parent_id int(10) unsigned DEFAULTNULL,  
PRIMARYKEY(id),  
FOREIGNKEY(parent_id) REFERENCEScategory (id)  
ONDELETECASCADEONUPDATECASCADE  
);

表中的每一行都是该id列标识的树中的一个节点。parent_id列是category表本身的外键。它的作用类似于指向该id列的指针  。

INSERTINTOcategory(title,parent_id)  
VALUES('Electronics',NULL);

要插入非根节点，只需将其设置  parent_id为其父节点的ID。例如，所述parent_id的Laptop & PC，Cameras & Photos和Phone & Accessories节点被设置为1：

INSERTINTOcategory(title,parent_id)  
VALUES('Laptops & PC',1);  
INSERTINTOcategory(title,parent_id)  
VALUES('Laptops',2);  
INSERTINTOcategory(title,parent_id)  
VALUES('PC',2);  
INSERTINTOcategory(title,parent_id)  
VALUES('Cameras & photo',1);  
INSERTINTOcategory(title,parent_id)  
VALUES('Camera',5);  
INSERTINTOcategory(title,parent_id)  
VALUES('Phones & Accessories',1);  
INSERTINTOcategory(title,parent_id)  
VALUES('Smartphones',7);  
INSERTINTOcategory(title,parent_id)  
VALUES('Android',8);  
INSERTINTOcategory(title,parent_id)  
VALUES('iOS',8);  
INSERTINTOcategory(title,parent_id)  
VALUES('Other Smartphones',8);  
INSERTINTOcategory(title,parent_id)  
VALUES('Batteries',7);  
INSERTINTOcategory(title,parent_id)  
VALUES('Headsets',7);  
INSERTINTOcategory(title,parent_id)  
VALUES('Screen Protectors',7);

寻找根节点

根节点是没有父节点的节点。换句话说，它parent_id是NULL：

SELECT  
    id, titleFROM  
    categoryWHERE  
    parent_id ISNULL;

运行结果：

+----+-------------+  
| id | title       |  
+----+-------------+  
|  1 | Electronics |  
+----+-------------+  
1 row in set (0.00 sec)

查找节点的直接子代

以下查询获取根节点的直接子代：

SELECT  
    id, titleFROM  
    categoryWHERE  
    parent_id = 1;

运行结果：

+----+----------------------+  
| id | title                |  
+----+----------------------+  
|  2 | Laptops & PC         |  
|  5 | Cameras & photo      |  
|  7 | Phones & Accessories |  
+----+----------------------+  
3 rows in set (0.00 sec)

查找叶节点

叶节点是没有子节点的节点。

SELECT  
    c1.id, c1.titleFROM  
    category c1        LEFTJOIN  
    category c2 ONc2.parent_id = c1.id  
WHERE  
    c2.id ISNULL;

运行结果：

+----+-------------------+  
| id | title             |  
+----+-------------------+  
|  3 | Laptops           |  
|  4 | PC                |  
|  6 | Camera            |  
|  9 | Android           |  
| 10 | iOS               |  
| 11 | Other Smartphones |  
| 12 | Batteries         |  
| 13 | Headsets          |  
| 14 | Screen Protectors |  
+----+-------------------+  
9 rows in set (0.00 sec)

完整查询整棵树

以下递归公用表表达式（CTE）检索整个类别树。请注意，自MySQL 8.0起，CTE功能已可用

WITHRECURSIVE category_path (id, title, path) AS  
(  
SELECTid, title, title aspath    FROMcategory    WHEREparent_id ISNULL  
UNIONALL  
SELECTc.id, c.title, CONCAT(cp.path, ' > ', c.title)  
FROMcategory_path AScp JOINcategory ASc      ONcp.id = c.parent_id  
)  
SELECT* FROMcategory_path  
ORDERBYpath;

运行结果：

+------+----------------------+----------------------------------------------------------------------+  
| id   | title                | path                                                                 |  
+------+----------------------+----------------------------------------------------------------------+  
|    1 | Electronics          | Electronics                                                          |  
|    5 | Cameras & photo      | Electronics > Cameras & photo                                        |  
|    6 | Camera               | Electronics > Cameras & photo > Camera                               |  
|    2 | Laptops & PC         | Electronics > Laptops & PC                                           |  
|    3 | Laptops              | Electronics > Laptops & PC > Laptops                                 |  
|    4 | PC                   | Electronics > Laptops & PC > PC                                      |  
|    7 | Phones & Accessories | Electronics > Phones & Accessories                                   |  
|   12 | Batteries            | Electronics > Phones & Accessories > Batteries                       |  
|   13 | Headsets             | Electronics > Phones & Accessories > Headsets                        |  
|   14 | Screen Protectors    | Electronics > Phones & Accessories > Screen Protectors               |  
|    8 | Smartphones          | Electronics > Phones & Accessories > Smartphones                     |  
|    9 | Android              | Electronics > Phones & Accessories > Smartphones > Android           |  
|   10 | iOS                  | Electronics > Phones & Accessories > Smartphones > iOS               |  
|   11 | Other Smartphones    | Electronics > Phones & Accessories > Smartphones > Other Smartphones |  
+------+----------------------+----------------------------------------------------------------------+  
14 rows in set (0.00 sec)

查询子树

下面的查询得到Phone & Accessories的子树，其id为7。

WITHRECURSIVE category_path (id, title, path) AS  
(  
SELECTid, title, title aspath    FROMcategory    WHEREparent_id = 7  
UNIONALL  
SELECTc.id, c.title, CONCAT(cp.path, ' > ', c.title)  
FROMcategory_path AScp JOINcategory ASc      ONcp.id = c.parent_id  
)  
SELECT* FROMcategory_path  
ORDERBYpath;

运行结果：

+------+-------------------+---------------------------------+  
| id   | title             | path                            |  
+------+-------------------+---------------------------------+  
|   12 | Batteries         | Batteries                       |  
|   13 | Headsets          | Headsets                        |  
|   14 | Screen Protectors | Screen Protectors               |  
|    8 | Smartphones       | Smartphones                     |  
|    9 | Android           | Smartphones > Android           |  
|   10 | iOS               | Smartphones > iOS               |  
|   11 | Other Smartphones | Smartphones > Other Smartphones |  
+------+-------------------+---------------------------------+  
7 rows in set (0.00 sec)

查询单个路径

要查询从下到上的单个路径，例如从iOS到Electronics，请使用以下语句：

WITHRECURSIVE category_path (id, title, parent_id) AS  
(  
SELECTid, title, parent_id    FROMcategory    WHEREid = 10-- child node  
UNIONALL  
SELECTc.id, c.title, c.parent_id    FROMcategory_path AScp JOINcategory ASc      ONcp.parent_id = c.id  
)  
SELECT* FROMcategory_path;

运行结果：

+------+----------------------+-----------+  
| id   | title                | parent_id |  
+------+----------------------+-----------+  
|   10 | iOS                  |         8 |  
|    8 | Smartphones          |         7 |  
|    7 | Phones & Accessories |         1 |  
|    1 | Electronics          |      NULL |  
+------+----------------------+-----------+  
4 rows in set (0.00 sec)

计算每个节点的级别

假设根节点的级别为0，下面的每个节点的级别等于其父节点的级别加1。

WITHRECURSIVE category_path (id, title, lvl) AS  
(  
SELECTid, title, 0lvl    FROMcategory    WHEREparent_id ISNULL  
UNIONALL  
SELECTc.id, c.title,cp.lvl + 1  
FROMcategory_path AScp JOINcategory ASc      ONcp.id = c.parent_id  
)  
SELECT* FROMcategory_path  
ORDERBYlvl;

运行结果：

+------+----------------------+------+  
| id   | title                | lvl  |  
+------+----------------------+------+  
|    1 | Electronics          |    0 |  
|    2 | Laptops & PC         |    1 |  
|    5 | Cameras & photo      |    1 |  
|    7 | Phones & Accessories |    1 |  
|    3 | Laptops              |    2 |  
|    4 | PC                   |    2 |  
|    6 | Camera               |    2 |  
|    8 | Smartphones          |    2 |  
|   12 | Batteries            |    2 |  
|   13 | Headsets             |    2 |  
|   14 | Screen Protectors    |    2 |  
|    9 | Android              |    3 |  
|   10 | iOS                  |    3 |  
|   11 | Other Smartphones    |    3 |  
+------+----------------------+------+  
14 rows in set (0.00 sec)

删除节点及其后代

要删除节点及其后代，只需删除节点本身，所有后代将通过DELETE CASCADE外键约束的来自动删除。

例如，要删除Laptops & PC节点及其子女（Laptops，PC），可以使用如下语句：

DELETEFROMcategory  
WHERE  
    id = 2;

删除节点并提升其后代

删除非叶子节点并提升其后代：

-   首先，parent_id将节点的直接子节点的更新为id新的父节点的。
-   然后，删除该节点。

例如，要删除Smartphones节点并提升其子节点（例如Android，）iOS，请执行以下操作Other Smartphones：

首先，更新的parent_id所有直属子项Smartphones：

UPDATEcategory  
SET  
    parent_id = 7-- Phones & Accessories  
WHERE  
    parent_id = 5; -- Smartphones

其次，删除Smartphones节点：

DELETEFROMcategory  
WHERE  
    id = 8;

这两个语句都应该包装在一个事务中：

BEGIN; UPDATEcategory  
SET  
    parent_id=7WHERE  
    parent_id = 5; DELETEFROMcategory  
WHERE  
    id = 8; COMMIT;

移动子树

要移动子树，刚更新的parent_id子树的顶部节点。例如，要将Cameras & photo用作的子代Phone and Accessories，请使用以下语句：

UPDATEcategory  
SET  
    parent_id = 7WHERE  
    id = 5;

在本教程中，您学习了如何使用邻接表模型来管理MySQL中的分层数据。

相关教程

-   [MySQL CTE简介](http://www.begtut.com/mysql/mysql-cte.html)
-   [MySQL递归CTE权威指南](http://www.begtut.com/mysql/mysql-recursive-cte.html)

新手教程所有内容，包括文字、图片、音频、视频、软件、程序、以及网页版式设计等均在网上搜集。

新手教程提供的内容仅用于个人学习、研究或欣赏。我们不保证内容的正确性。通过使用本站内容随之而来的风险与本站无关

访问者可将本网站提供的内容或服务用于个人学习、研究或欣赏，以及其他非商业性或非盈利性用途，但同时应遵守著作权法及其他相关法律的规定，不得侵犯本网站及相关权利人的合法权利。

本网站内容原作者如不愿意在本网站刊登内容，请及时通知本站，予以删除。

Copyright © 2019 新手教程 begtut.com All Rights Reserved. [皖ICP备19011202号](http://www.beian.miit.gov.cn)

From <[https://www.begtut.com/mysql/mysql-adjacency-list-tree.html](https://www.begtut.com/mysql/mysql-adjacency-list-tree.html)>