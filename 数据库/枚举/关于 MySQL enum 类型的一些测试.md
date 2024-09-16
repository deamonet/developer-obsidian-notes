 [![](media/100x100.jpg) leo 的个人博客](https://learnku.com/blog/leo108) /  6796 /  26 / 创建于 5年前

## 背景：

在开发项目时通常会遇到一些状态字段，例如订单的状态有 `待支付`、`已支付`、`已关闭`、`已退款` 等，我以前做的项目都是把这些状态用数字存在数据库中，然后在 php 代码中用常量来维护一份映射表，例如：

```php
const STATUS_PENDING = 0;
const STATUS_PAID = 1;
const STATUS_CLOSED = 2;
const STATUS_REFUNDED = 3;
```

但是在实际使用过程中发现并不是那么好用，由于各种原因（追查 bug、临时的统计需求等）我们常常需要登录到 mysql 服务器里手动执行一些 sql 查询，由于许多表都有状态字段，写 sql 时必须对照的 php 代码里的映射关系来写，一不小心还有可能将不同表的状态数字弄混导致大问题。

于是我在新项目中准备使用 mysql 的 enum 类型来存储各种状态，在使用过程中发现如果在 Laravel 的 migration 文件中对使用了 enum 类型的表做变更（即使是变更非 enum 类型的字段）都会报错

```php
[Doctrine\DBAL\DBALException]
  Unknown database type enum requested, Doctrine\DBAL\Platforms\MySQL57Platform may not support it.
```

搜索了一下，发现是 [doctrine 不支持 mysql 的 enum](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/cookbook/mysql-enums.html)，该文中列举了 enum 的 3 个缺点：

1.  新增 enum 值的时候需要重建整个表，当数据量大的时候可能需要耗费数小时。
2.  enum 值的排序规则是按创建表结构时指定的顺序，而非字面值的大小。
3.  依赖 mysql 对 enum 值的校验并不是非常必要，在默认配置下插入非法值最终会变成空值。

根据新项目的实际情况，不太可能出现需要对状态字段做排序的需求，即使有我们可以在设计表结构的时候就定好顺序，因此缺点 2 可以忽略；而缺点 3 则可以通过代码规范、插入 / 更新前校验等方式来规避；至于缺点 1，我们需要做一些测试。

## 测试准备

首先创建一个表：

```php
CREATE TABLE `enum_tests` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `status` enum('pending','success','closed') COLLATE utf8mb4_unicode_ci NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

然后插入 100W 条数据：

```php
$count = 1000000;
$bulk  = 1000;
$data  = [];
foreach (['pending', 'success', 'closed'] as $status) {
    $data[$status] = [];
    for ($i = 0; $i < $bulk; $i++) {
        $data[$status][] = ['status' => $status];
    }
}

for ($i = 0; $i < $count; $i += $bulk) {
    $status = array_random(['pending', 'success', 'closed']);
    EnumTest::insert($data[$status]);
}
```

## 测试过程

### 测试 1

在 enum 值列表最后添加一个值 `refunded`

```php
ALTER TABLE `enum_tests` CHANGE `status` `status` ENUM('pending','success','closed','refunded') CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL;
```

输出：

```php
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

结论：在末尾追加 enum 值时几乎没有成本。

### 测试 2：

删除刚刚添加的值 `refunded`

```php
ALTER TABLE `enum_tests` CHANGE `status` `status` ENUM('pending','success','closed') CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL;
```

输出：

```php
Query OK, 1000000 rows affected (5.93 sec)
Records: 1000000  Duplicates: 0  Warnings: 0
```

结论：删除一个没有用过的 enum 值仍需全表扫描，成本较高，但还在可接受范围内。

### 测试 3：

将 `refunded` 插入到值列表中间而非末尾

```php
ALTER TABLE `enum_tests` CHANGE `status` `status` ENUM('pending','success','refunded', 'closed') CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL;
```

输出：

```php
Query OK, 1000000 rows affected (6.00 sec)
Records: 1000000  Duplicates: 0  Warnings: 0
```

结论：在原 enum 值列表中间新增值需要全表扫描并更新，成本较高。

### 测试 4：

删除值列表中间的值

```php
ALTER TABLE `enum_tests` CHANGE `status` `status` ENUM('pending','success','closed') CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL;
```

输出：

```php
Query OK, 1000000 rows affected (4.23 sec)
Records: 1000000  Duplicates: 0  Warnings: 0
```

结论：需全表扫描，成本较高。

### 测试 5：

给 `status` 字段添加索引后再执行上述测试

```php
ALTER TABLE `enum_tests` ADD INDEX(`status`);
```

发现测试 2-4 的耗时反而有所增加，应该是同时需要更新索引导致的。

## 结语：

对于我的新项目来说只会出现新增 enum 值的情况，即使将来有个别状态废弃不用也不需要去调整 enum 的值列表，因此决定在项目中引入 enum 类型作为存储状态的数据类型。

> 本作品采用[《CC 协议》](https://learnku.com/docs/guide/cc4.0/6589)，转载必须注明作者和本文链接