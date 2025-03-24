原创[eternity](https://www.modb.pro/u/485074)2022-07-18

18061

upsert对于任何使用数据库的人来说都很有用，但术语“upsert”甚至可能不会出现在DBMS的文档中！

那么什么是upsert，为什么不在文档中提及？

## SQL中的Upsert是什么？

upsert是一个portmanteau，是“update”和“insert”的组合在关系数据库的上下文中，upsert是一种数据库操作，如果表中已存在指定值，则更新现有行，如果指定值不存在，则插入新行。

例如，假设我们有一个数据库，其中一个表employees和一个id列作为主键：  
![image.png](https://oss-emcsprod-public.modb.pro/image/editor/20220716-379e5870-8ebe-4206-b0c9-579832bcef58.png)

在更改此表中的员工信息时，我们可以使用upsert。从逻辑上讲，这看起来是这样的：

- 如果表中存在员工ID，请使用新信息更新该行。
    
- 如果表中不存在员工ID，请将其添加为新行。
    

不同的RDBMS以不同的方式处理UPSERT的语法-我们稍后再讨论-但使用CockroachDB UPSERT语法，下面是两个示例SQL语句，以及如果运行每个语句，将产生的employees表：

示例#1

```
UPSERT INTO employees (id, name, email) VALUES (2, ‘Dennis’, ‘dennisp@weyland.corp’);
```

结果：  
![image.png](https://oss-emcsprod-public.modb.pro/image/editor/20220716-e6f6f2fa-a9e3-4234-b7de-7d11f1881e37.png)

在本例中，主键值2已存在于表中，因此UPSERT操作使用名称和电子邮件的新值更新了该行。

示例#2

```
UPSERT INTO employees (id, name, email) VALUES (3, ‘Ash’, ‘ash@hyperdyne.corp’);
```

结果：  
![image.png](https://oss-emcsprod-public.modb.pro/image/editor/20220716-ee181523-96b8-4abc-a81c-8007be0a314e.png)

在本例中，主键值3在表中不存在，因此UPSERT操作会在表中插入一个具有相关值的新行。

然而，这只是一个简单的例子。事实上，在许多RDBMS中，UPSERT甚至不作为命令存在！这就是为什么如果在文档中搜索所选数据库，可能找不到“upsert”的条目。  
![微信图片_20220716155028.png](https://oss-emcsprod-public.modb.pro/image/editor/20220716-bd14a09c-4918-4fbc-a5fa-a7c4452ecdaf.png)

然而，我们可以在最流行的数据库中执行upserts，所以在回到CockroachDB讨论一些细节之前，让我们先看看如何在MySQL和PostgreSQL中执行upserts。

我们将继续使用样本员工表来演示这些工作原理。

## 在MySQL中插入

UPSERT命令在MySQL中不存在，但仍然可以实现UPSERT。在MySQL的当前版本中实现UPERT的最佳方法是插入…重复密钥更新时。让我们更详细地了解一下该命令。

正如命令本身所示，插入…重复键更新将在表中插入新行，除非它在主键列中检测到重复值，在这种情况下，它将用新信息更新现有行。

因此，如果我们在示例employees表上运行以下命令…

```
INSERT INTO employees (id, name, email) VALUES (2, ‘Dennis’, ‘dennisp@weyland.corp’) ON DUPLICATE KEY UPDATE;
```

…我们将得到与上面示例#1中相同的结果。MySQL检测到主键列id中已经存在值2，因此它用新信息更新该行。

类似地，如果我们用值（4，‘Dallas’，’运行相同的命令dallas@weyland.corp，它将向具有这些值的员工中插入新行，因为示例表中不存在值4。

## 在PostgreSQL中插入

PostgreSQL也没有专用的UPSERT命令，但UPSERT可以使用插入冲突来完成。这个命令可能比INSERT复杂一点。。。在重复键上，但它也允许我们有更多的控制。

让我们先来看一下Postgres中插入冲突语句的基本结构：

```
INSERT INTO table (col1, col2, col3) 
VALUES (val1, val2, val3)
ON CONFLICT conflict_target conflict_action;
```

正如我们在上面的命令中所看到的，PostgreSQL允许我们指定两件事：

- conflict_目标，即应在何处检测冲突。
    
- conflict_操作，即在检测到冲突时应如何处理命令。
    

这使我们在如何应用我们的优势方面更有针对性。

在当前版本的PostgreSQL中，我们可以通过指定冲突目标（在本例中为id，主键列）以及在检测到冲突时要执行的操作（在本例中，更新现有行）来实现基本的upsert：

```
INSERT INTO employees (id, name, email) 
VALUES (2, ‘Dennis’, ‘dennisp@weyland.corp’)
ON CONFLICT (id) DO UPDATE;
```

运行此命令将产生与本文开头的示例#1相同的结果。PostgreSQL检测到冲突-我们试图插入一个id值为2的行，但员工中已经存在一个id为2的行-因此它使用新值对该行运行更新。

如果我们使用不会产生冲突的值运行此命令（例如，（5，‘凯恩’，'kane@weyland.corp”），它将在具有这些值的员工中插入新行。

## 在CockroachDB中插入

CockroachDB确实有一个UPSERT命令，与PostgreSQL一样，UPSERT也可以使用冲突时插入来实现。

虽然这两个命令可以获得相似的结果，但它们并不完全相同。让我们看看它们之间的区别，以及我们可能希望何时使用它们。

### 冲突时插入与插入

CockroachDB中的UPSERT命令根据主键列的唯一性执行UPSERT，并根据添加的值是否唯一执行更新或插入。

这使得使用UPSERT比在冲突时插入更简单，因为我们不需要指定冲突目标或操作。例如，对示例employees表运行以下语句…

```
UPSERT INTO employees (id, name, email) VALUES (6, ‘Lambert’, ‘lambert@weyland.corp`);
```

…结果如下表所示：  
![image.png](https://oss-emcsprod-public.modb.pro/image/editor/20220716-e5d4f9fd-91e5-41d1-aef8-9480cf9bef3a.png)

因为employees中不存在6的值，所以CockroachDB将这些值作为新行插入表中。

类似地，如果我们运行以下语句…

```
UPSERT INTO employees (id, name, email) VALUES (1, ‘Ripley’, ‘ripley@weyland.corp`);
```

我们将得到下表的结果：  
![image.png](https://oss-emcsprod-public.modb.pro/image/editor/20220716-ecf2ee90-ba58-46cd-9c21-27422feb3519.png)

因为1已经存在于id（主键列）中，所以CockroachDB用新信息更新该行。

然而，我们还可以灵活地在冲突中使用INSERT，这在某些情况下可能很有用。例如，在希望避免与主键无关的冲突的情况下，我们可以使用INSERT ON CONFLICT来处理upsert。例如，我们可以指定外键列作为冲突目标。

UPSERT和INSERT-ON-CONFLICT之间有时也存在性能差异，尽管这取决于工作负载的具体情况。

因为1已经存在于id（主键列）中，所以CockroachDB用新信息更新该行。

然而，我们还可以灵活地在冲突中使用INSERT，这在某些情况下可能很有用。例如，在希望避免与主键无关的冲突的情况下，我们可以使用INSERT ON CONFLICT来处理upsert。例如，我们可以指定外键列作为冲突目标。

UPSERT和INSERT-ON-CONFLICT之间有时也存在性能差异，尽管这取决于工作负载的具体情况。

> 原文标题：Upsert in SQL: What Is an Upsert, and When Should You Use One?  
> 原文作者：Charlie Custer  
> 原文链接：https://dzone.com/articles/upsert-in-sql-what-is-an-upsert-and-when-should-yo