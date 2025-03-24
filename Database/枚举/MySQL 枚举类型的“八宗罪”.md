

[![](media/4fe5ee3a310941e5020ade1a62efa5c1~tplv-t2oaga2asx-no-mark!100!100!100!100.jpg)](https://juejin.cn/user/4441682706178717)

[红烧不是清蒸![lv-6](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-6.b69935b.png "创作等级")](https://juejin.cn/user/4441682706178717)

2018年03月09日 15:01 ·  阅读 11446

原文链接： [www.zcfy.cc](https://link.juejin.cn/?target=http%3A%2F%2Fwww.zcfy.cc%2Farticle%2Fa-href-title-komlenic-com-komlenic-com-a)

> 原文出处 [8 Reasons Why MySQL's ENUM Data Type Is Evil](https://link.juejin.cn/?target=http%3A%2F%2Fkomlenic.com%2F244%2F8-reasons-why-mysqls-enum-data-type-is-evil%2F "http://komlenic.com/244/8-reasons-why-mysqls-enum-data-type-is-evil/")

# MySQL 枚举类型的“八宗罪”

2011年 3月 2日 周三

MySQL的 [枚举（ENUM）类型](https://link.juejin.cn/?target=http%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.5%2Fen%2Fenum.html "http://dev.mysql.com/doc/refman/5.5/en/enum.html") 是程序员群体中的一个讨论热点。乍一看，我们可以通过枚举类型，很好地将记录值限制在允许范围内。一个典型的例子是，一个具有字段名称为“大陆板块”的数据表：每一个国家位于一个大陆板块，而这些大陆板块不太可能经常变化。当然，或许一天北美板块会与亚洲板块碰撞形成_北美亚_,但即便你的数据库能够延续使用到那个时候，起码你也不需要研讨怎么去重构你的数据表，那将是当时的开发者要做的工作。

言归正传。如果，使用ENUM是唯一 一个，能够代表某一个国家属于哪个大陆板块的选择，那我们大可进行下一步，去争辩诸如NoSQL的优劣、Git和SVN孰强孰弱、你喜欢的框架有哪些缺点这些其他的问题。但这里有一个普遍适用于实现枚举的最佳实践：

![Comparison of ENUM vs Reference Table](media/Comparison_of_ENUM_vs_Reference_Table.jpg)

[维基百科](https://link.juejin.cn/?target=http%3A%2F%2Fwikipedia.org%2Fwiki%2FReference_table "http://wikipedia.org/wiki/Reference_table") 是这样描述关系表的:

> …这是一种将可知的枚举数据分离出来的表。例如，一个关系型数据库的仓库数据，仓库里面的“物件”有可能会有一个“状态”的字段记录已经声明的值，例如：“已售，预定，售罄”。在极简的数据库设计当中，这些值都会在独立的关系表“状态”中存储，以此满足范式（[database normalization](https://link.juejin.cn/?target=http%3A%2F%2Fwikipedia.org%2Fwiki%2FDatabase_normalisation "http://wikipedia.org/wiki/Database_normalisation")）。

所以，关系表也可以满足枚举的实现。下面就来看看，ENUM的”八宗罪“到底是什么：

### 1. 数据被错误对待

男、女；先生、夫人、小姐；非洲、亚洲，等等。这些人们使用作为ENUM类型字段的短词称为_数据_。当你使用一个ENUM类型字段, 技术上看，是你将数据抽离出来 (对应到实际数据表时), 放到一个独立的地位（一种数据库的元数据，具有精确定义字段）。 这不同与约束数据类型，如我们通常的做法：数值型字段只能存储整型数据，或者日期型字段不能为空——这些都没有问题，而且还十分重要。使用ENUM类型字段时，我们实际上是保存_部分数据_ 去作为 _这个数据模型_的一个特征信息。简而言之， ENUM类型字段破坏了范式要求。这也许看起来十分“学院派”或“迂腐陈旧”，但这正是以下各种“罪行”的源头。

### 2. 更改ENUM类型字段，代价很昂贵

永恒不变的是， 每次你创建ENUM类型字段的时候都说：“这个字段不可能变的”。人类普遍欠缺顾全大局的能力，预测上更是糟糕，其如研发部的新产品线、贵司新的航运方案、北美板块碰撞亚洲板块。

使用ALTER TABLE去修改整个数据表的ENUM类型字段，是十分耗费资源的。如果将`ENUM('red', 'blue', 'black')` 改为 `ENUM('red', 'blue', 'white')`, MySQL 需要重构整个数据表，并且检索 _所有数据_去检查 `'black'`这个无效值。 MySQL 是真的蠢，它确实会在你每次_增加_一个新的ENUM值时都这么做的！（传言未来会处理ENUM类型字段的效率问题，但我对其受重视程度深表怀疑。）

全表重构在小型数据表中可能没有那么痛苦，但在海量数据的情况下可能会导致资源被锁死_很长很长_一段时间。如果你使用关系表去替代ENUM类型字段，改变枚举集合只不过是使用`INSERT`、`UPDATE`和`DELETE`，对比来看真是滑稽。

很重要的一点，当更改ENUM类型字段的枚举集合时，MySQL会转换任意已有但不存在于新的枚举集合中的记录值为`''`（空的字符串）。使用关系表，在更改和删除枚举集合时会灵活很多（下面会提到）。

### 3. 几乎无法给关联数据添加额外的属性

![Adding related info to a reference table](media/Adding_related_info_to_a_reference_table.jpg)

至今都没有一个可以更加_明智地_改变ENUM类型字段的方法，这也是我们的常态。在我们的“国家、大陆板块”例子中, 更改“国土面积”会出现什么情况？我们没有预料到这个属性， 但也要既来之则安之。使用关系表设计，我们可以轻易地拓展“大陆板块”这个数据表，各种方式为其增加我们想要的数据和字段 。ENUM？快别说了。

另一种极妙的灵活性体现在关系表的拓展便捷性上。一个简单的标记位字段即可表示这个“枚举值”是否可用。所以，当你的公司不打算销售黑色的装饰品了，你只需在“黑色”所对应的_is_discontinued_字段中做个标记即可。而且你依然可以查询到已售的颜色（译者：指的是，ENUM的修改会导致原有，而现在已经没有的值变为空字符串，数据失去了部分特征），_同时_你那些黑色装饰品的订单依然可统可计哦！ENUM，你要不要试试？

### 4. 获取ENUM全部可能值，很麻烦

一个很常见的需求是，将数据库中存在的数据显示在可拖拽列表中，例如：

选择颜色：

红 蓝 黑

如果这些数值存储在一个名为‘colors’的数据表里，你所要做的仅仅是：`SELECT * FROM colors`，这样即可动态地令数据地显示在可拖拽列表中。你可以添加或者改变color关系表中的颜色，并且，你那酷炫订单的颜色可选项会自动更新，真了不起。 （译：此处所举例子，应等同于：“通过后台管理，可以限定前端用户某类型数据的可选项。”这样的功能。）

回到ENUM上：你要如何获取全部的枚举值？你当然可以使用ENUM值搭配DISTINCT去查询（译：即是查询ENUM值互相不相同的数据，等于利用DISTINCT的唯一性去查询ENUM），但这样也只会返回_确实使用过，并存在于数据表ENUM字段可选值中_的ENUM值，而不是所有可能的值。你也可以查询INFORMATION_SCHEMA然后通过代码解析返回的数据，去找到你想要的ENUM的所有值，但这完全是多此一举。事实上，我依然没有发现，有任何兼顾了优雅与原生的SQL方式，可以获取ENUM类型字段的所有值。

### 5. ENUM类型字段所提供的优化有限

通常使用ENUM的正当理由，不外乎“优化”二字，譬如，性能提升，简化模型与高可读性。

那我们从性能上看。你可以在未优化的数据库中做很多匪夷所思而夸张的事，但是大多情况是，在数据达到一定规模前，都不会出现影响性能的情况，并且通常我们的产品远未达到那个尺度规模。有一点需要注意的是，因为数据库开发者们都热衷于令自己的设计可以达到_完备的范式_，并且只会在遇到性能问题时才会考虑_反范式_。如果你担心使用关系表会导致变慢，可以在同一基准下测试不同方式下的表现，再进行考虑。切勿先入为主地认为关联查询会成为瓶颈，可能有时并非如此。(可参照 [evidence to support that ENUM isn't always _appreciably_ faster than alternatives](https://link.juejin.cn/?target=http%3A%2F%2Fwww.mysqlperformanceblog.com%2F2008%2F01%2F24%2Fenum-fields-vs-varchar-vs-int-joined-table-what-is-faster%2F "http://www.mysqlperformanceblog.com/2008/01/24/enum-fields-vs-varchar-vs-int-joined-table-what-is-faster/").)

另一个关于ENUM优化方式的说法是，ENUM可以有效减少数据库中的数据表外键。不可置否，使用外键相当于是将很多不同的盒子以线相连，而且在大型系统中，范式设计已可降低对人类的理解能力界限、复杂型查询的要求。但是，我们为什么会设计模型，为什么要将模型抽象化以便我们能够理解它。去试试做一个新数据模型图或者ER图，并且忽略一些小细节和关系表。有时候使用ENUM确实如_看上去那般简单_，但事实上你在心里需要想着一个隐式的关系表，所以并没有_看上去那般简单_。

### 6. ENUM值在其他数据表中不可直接复用

当你（在数据表中）创建了一个带值的ENUM字段，在其他数据表中无法直接复用这个ENUM。而当有了关系表，相同应用形式下，可以在其他多个数据表中复用。当改变关系表中的一个数据，其他多个数据表也会得到响应。

![A reference table can easily be linked to multiple tables](media/A_reference_table_can_easily_be_linked_to_multiple_tables.jpg)

ENUM类型字段的分离，将使你能在多个数据表中复用相同的ENUM值（需要保持一致性）。

### 7. ENUM类型字段有显然陷阱

假设你设置了一个字段“color”`ENUM('blue', 'black', 'red')` ，这时你想`INSERT`一行数据，但“color”字段是 `'purple'`， MySQL 会将不合法的值变为 `''` （空字符串）。 处理上没问题, 但如果我们使用的是带外键的关系表, 那么我们的数据能因健壮性而更加可靠。

同样，MySQL 会为ENUM值关联枚举索引，并且在使用中会错误地调用到索引而不是ENUM值，反之亦然。

想象一下：

```sql
CREATE TABLE test (foobar ENUM('0', '1', '2'));

mysql> INSERT INTO test VALUES ('1'), (1);
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM test;
+--------+
| foobar |
+--------+
| 1      |
| 0      |
+--------+
2 rows in set (0.00 sec)

复制代码
```

我们插入了 '1' (**字符串**)，并且不小心插入了 1 (**没有引号，数值型**)。 MySQL 会将我们地数值型数据当作是枚举索引去处理（并没有错，但会令人混淆），根据索引可知，ENUM字段的第一个值为 0 。（译：枚举索引由 1 开始）

### 8. ENUM 的移植性不佳

ENUM类型不是SQL标准，属于MySQL，而其他DBMS不一定有原生的支持。 [PostgreSQL](https://link.juejin.cn/?target=http%3A%2F%2Fwww.postgresql.org%2F "http://www.postgresql.org/"), [MariaDB](https://link.juejin.cn/?target=http%3A%2F%2Fmariadb.org%2F "http://mariadb.org/"),与[Drizzle](https://link.juejin.cn/?target=http%3A%2F%2Flaunchpad.net%2Fdrizzle "http://launchpad.net/drizzle") (后面那两个就MySQL的分支), 我只知道这三个是支持的ENUM的。如果某个人打算将数据库迁移, 那么他就要花费更多的步骤去处理你那些“精妙”的ENUM字段了，相信他会“更爱你”。如果（那个人）是你, 你可会发现自己当时真是“聪明够了”。通常来说，数据库迁移不会经常发生，并且，由于所有人都会假设迁移数据库的过程中，必然要出乱子，因此成为“第八宗罪”。

## 几时适合使用ENUM:

### 1. 当你需要存储的是准确、不变的值时

大陆板块就是最好的例子，定义十分准确。另一个常用例子是称谓：先生、夫人、小姐，或者是扑克的花色：方块、梅花、红心、黑桃。但是，即便是这些例子，有时也需要去拓展值的范围（例如有人需要你称呼“陈医生”而不是“陈先生”的时候，或者你的扑克游戏里面需要用到小丑牌）。

### 2. 你永远不需要存储额外的关联信息

用回扑克牌的例子。扑克游戏老少咸宜，依赖的规则是梅花和黑桃为黑色，方块和红心为红色（例如，尤克牌）。如果我们需要为花色关联额外的信息，例如颜色，那将如何？如果我们使用关系表，那我们只需要在关系表中新增字段即可，小事一桩。如果我们使用ENUM去表示花色，那我们就很难去准确的表示花色于颜色的关联了，如此我们只能在应用层上去达成这种关联。

### 3. ENUM值的数量大于_2_个并少于_20_个

如果你的ENUM值只有两个，你_完全可以_将ENUM换成更加高效的TINYINT(1)或者更更高效的BIT(1)（MySQL5.0.3及以上）。例如: `gender ENUM('male', 'female')` 可以变换为: `is_male BIT(1)`. 当你只有两个选项时，完全能以布尔值 `true`/`false`，结合字段名字中的“is”关键词来区分。至于20个的上限设定，没错，ENUM事实上可以保存多达65535个值，但求你千万别试。超过二十个值会变得很累赘，超过50个必然难于管理与使用。

## 如果你无论如何都要用ENUM:

### 1. ENUM值千万不要使用数值型

ENUM定义为_字符型_数据是有原因的。并不是说你使用_数值型_字段类型去存储数字是错误的，但有充足的证据显示，MySQL内部机制使用数字去引用索引（参考 上面的第七条）。反正不要在ENUM中存储数字，OK？

### 2. 考虑使用严格模式

启用严格模式，至少在你插入一个不存在的ENUM值时会报告错误。否则，只会简单地出现一个警告，继而该值被设置为一个空字符串`""`（枚举索引为0）。抄笔记：如果你设置了IGNORE，错误依然会被忽略。

## 结论

从开发、维护的角度去做有意义的事，性能问题出现时再考虑优化——普遍而言，使用关系表抑或是使用ENUM类型，争议不断。

> 性能瓶颈（这个概念）被滥用已是不争事实。 开发者们浪费了大量的时间去思考它、担心它，（例如）非关键代码上的运行速度。这些对效率的苛求，给调试与维护造成了很大的负面影响。我们理应忽略那小部分的效率，就拿（达到）_97%_（效率）而言，过早的优化是万恶之源。
> 
> 虽说我们不应放弃优化那_3%_（的效率），但一个优秀的开发者不应为此而盲目自满（译：指在开发意识上，对于效率的高度追求），而应意识到要理智对待关键代码。但这也应在理解代码的前提下进行。- [Donald Knuth](https://link.juejin.cn/?target=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FDonald_Knuth "http://en.wikipedia.org/wiki/Donald_Knuth")

---

[komlenic.com](https://link.juejin.cn/?target=http%3A%2F%2Fkomlenic.com%2F "http://komlenic.com/") is the weblog/playground of [Chris Komlenic](https://link.juejin.cn/?target=http%3A%2F%2Fkomlenic.com%2F248%2Fabout-chris-komlenic-and-komlenic-com%2F "http://komlenic.com/248/about-chris-komlenic-and-komlenic-com/"), a full stack developer and generalist living in central Pennsylvania.

分类：

[后端](https://juejin.cn/backend)

标签：

[MySQL](https://juejin.cn/tag/MySQL)[数据库](https://juejin.cn/tag/%E6%95%B0%E6%8D%AE%E5%BA%93)