2022-03-12 64

**简介：** 【mysql】ENUM类型

# ENUM类型

-   ENUM类型也叫作枚举类型，ENUM类型的取值范围需要在定义字段时进行指定。设置字段值时，**ENUM类型只允许从成员中选取单个值，不能一次选取多个值。**
-   其所需要的存储空间由定义ENUM类型时指定的成员个数决定。

文本字符串类型

长度

长度范围

占用的存储空间

ENUM

L

1 <= L <= 65535

1或2个字节

-   当ENUM类型包含1～255个成员时，需要1个字节的存储空间；
-   当ENUM类型包含256～65535个成员时，需要2个字节的存储空间。
-   ENUM类型的成员个数的上限为65535个。

举例：

创建表如下：

```
CREATE TABLE test_enum(
season ENUM('春','夏','秋','冬','unknow')
);
```

添加数据：

```
INSERT INTO test_enum
VALUES('春'),('秋');

INSERT INTO test_enum
VALUES('UNKNOW');
```

-   忽略大小写

![在这里插入图片描述](https://img-blog.csdnimg.cn/6762049a328e46f0af092718d680e1e7.png "在这里插入图片描述")

-   当添加个没有定义的数值时，就会报错

![在这里插入图片描述](https://img-blog.csdnimg.cn/8d3e3ef108fb46ddb3c6b8259c560c96.png "在这里插入图片描述")

-   当添加多个定义的值，也会报错

![在这里插入图片描述](https://img-blog.csdnimg.cn/c5d8923ff8484629b3ae038d7c4f1017.png "在这里插入图片描述")

-   可以使用索引进行枚举元素的调用，下标从 1 开始

```
# 允许按照角标的方式获取指定索引位置的枚举值
INSERT INTO test_enum
VALUES('1'),(3);

SELECT * FROM test_enum;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/664974976ebb4c9094e403668148ac06.png "在这里插入图片描述")

-   没有限制非空的情况下，可以添加`null`值

```
# 当ENUM类型的字段没有声明为NOT NULL时，插入NULL也是有效的
INSERT INTO test_enum
VALUES(NULL);
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/618eb9b56c44418fb815dddea4c04093.png "在这里插入图片描述")

**版权声明：**本文内容由阿里云实名注册用户自发贡献，版权归原作者所有，阿里云开发者社区不拥有其著作权，亦不承担相应法律责任。具体规则请查看《[阿里云开发者社区用户服务协议](https://developer.aliyun.com/article/768092)》和《[阿里云开发者社区知识产权保护指引](https://developer.aliyun.com/article/768093)》。如果您发现本社区中有涉嫌抄袭的内容，填写[侵权投诉表单](https://yida.alibaba-inc.com/o/right)进行举报，一经查实，本社区将立刻删除涉嫌侵权内容。

[关系型数据库](https://developer.aliyun.com/label/sc/article_de-3-100067) [MySQL](https://developer.aliyun.com/label/sc/article_de-3-100210) [索引](https://developer.aliyun.com/label/sc/article_de-3-100070)