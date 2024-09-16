 2015-05-01

 7,815

一个正常运行的 WordPress 站点主要由以下三部分组成：

1.  安装的 WordPress 核心数据
2.  存放在`wp-content`目录的主题、插件和上传的多媒体内容
3.  数据库，我们在 WordPress 仪表盘添加的一切内容都在数据库里面保存

很多 WordPress 用户可能从来没有关心过 WordPress 数据库，甚至没有意识到，他们发表的文章、页面，WordPress 站点中的评论，在仪表盘中创建的菜单这一切都在 [WordPress 数据库](https://www.wpzhiku.com/tag/godeep/)中保存着，用户访问我们的网站时，由 WordPress 程序从数据库一读取内容，然后填充到我们所使用的主题中，最终返回给用户。

在本文中，我将带大家详细了解以下 WordPress 数据库的方方面面，本文将包括以下几个部分。

1.  介绍
2.  数据之间的关系
3.  内容类型
4.  用户数据
5.  元数据
6.  自定义分类法，分类目录，标签和自定义分类法项目
7.  自定义分类法和文章元数据的比较
8.  选项数据表
9.  WordPress多站点数据

通过本文，我们可以了解到 WordPress 不同的内容都是在 WordPress 的什么数据表中存储着的，以及各种数据是如何关联起来的。WordPress 中的内容类型。

数据表是用来存储内容的，再离间数据表之前，我们先来了解一下 WordPress 中的数据类型，然后再看 WordPress 数据表就比较好理解了。在 WordPress 中，有以下一些内容类型：

-   文章
-   页面
-   自定义文章类型
-   附件
-   连接
-   导航菜单项目（在数据库中以文章的方式保存）

以上的内容类型有一些附加的数据：

-   分类目录
-   标签
-   自定义分类法和分类
-   文章元数据

除了这些，Wordpress 中还有一些其他类型的数据：

-   小工具
-   选项
-   用户
-   站点 (在多站点一)
-   硬编码内容 (在主题或插件中添加的)
-   调用的站外数据

以上所有的内容类型都在数据库一的某个数据表一保存着，他们可能自己就是一条完整的数据，也可能是另外一个数据的一部分，也有可能链接到了其他的数据表，如关于文章的数据可能会被链接到关于用户的数据，这样我们就知道某篇文章是谁发表的了。

## WordPress数据库结构

WordPress 使用相互关联的数据库表来最小化数据的存储量 – 这产生了一对多的关系。也就是说，一个用户可以有很多文章，要想知道某篇文章是由哪个作者发布的，只需要分类给这篇文章一个作者 ID 就可以了。这样的设计节省了很多数据库空间，而且为 WordPress 带来了很大的灵活性，我们可以随意修改用户数据，而不用修改以前发布过的文章。

下面的图表是从 WordPress 官方文档一转载过来的，该表很清楚的为大家展示了数据表是怎么关联的。

[![working-with-data-in-wordpress-introduction-database-tables](media/working-with-data-in-wordpress-introduction-database-tables.jpg)](https://www.wpzhiku.com/wp-content/uploads/2015/03/working-with-data-in-wordpress-introduction-database-tables.jpg)

大多数数据表通过一个字段链接到了一个或多个数据表，这个字段一般是这个数据表的唯一数据，熟悉数据库的人肯定知道，这个数据就是 ID，如某个文章的 post_id，这些链接关系如下表：

数据表

存储的数据

关联到

`wp_posts`

文章、页面、附件、版本、导航菜单项目

`wp_postmeta` (通过`post_id`关联)  
`wp_term_relationships`(通过`post_id`关联)

`wp_postmeta`

每个文章的元数据

`wp_posts` (通过 `post_id`关联)

`wp_comments`

评论

`wp_posts` (通过 `post_id` 关联)

`wp_commentmeta`

评论元数据

`wp_comments` (通过`comment_id` 关联)

`wp_term_relationships`

文章和自定义分类法之间的关系

`wp_posts` (通过 `post_id` 关联)  
`wp_term_taxonomy` (通过`term_taxonomy_id` 关联)

`wp_term_taxonomy`

自定义分类法（包括默认的分类目录和标签）

`wp_term_relationships`(通过 `term_taxonomy_id`关联)

`wp_terms`

关联到分类法中的分类目录，标签和自定义分类项目

`wp_term_taxonomy` (通过`term_id` 关联)

`wp_links`

博客连接（已弃用，可以不用考虑）

`wp_term_relationships`(通过 `link_id` 关联)

`wp_users`

用户

`wp_posts` (通过`post_author` 关联)

`wp_user_meta`

每个用户的元数据

`wp_users` (通过 `user_id` 关联)

`wp_options`

网站设置和选项 (通过 WordPress 后台、主题、或插件设置)

独立的，不与其他任何数据表关联

开始之前，有一些需要注意的事项需要告诉大家。

-   数据表默认有一个`wp_` 的前缀，我们可以再安装时或通过配置文件修改这个前缀。
-   最核心的数据表是`wp_posts数据表，一般这个数据表一存储的内容是最多的，这个数据表把几乎所有的内容都关联到了一起。`
-   只有一个数据表是独立的，和其他数据表没有关联，那就是`wp_options` 数据表，这个数据表是一个key-value结构的数据表，存储着关于WordPress的一些设置和缓存的数据。
-   有两个数据表存储这关于自定义文章分类法的信息，再后面我们会详细解释。
-   `wp_users` 和 `wp_comments` 数据表没有关联，即使可以指定一条评论是哪个用户发表的，但是这两个数据表没有实际的关联，关键是通过文章进行的。
-   多站点有一些额外的数据表，这不是本文讨论的内容，以后有机会我们再详细讨论。

## 链接内容到数据表

了解了 WordPress 内容结构和数据表，现在是把他们对应起来的时候了下，表显示了每个数据表被用来存储哪种类型的内容。

内容类型

数据表

文章

`wp_posts`

页面

`wp_posts`

自定义文章类型

`wp_posts`

附件

`wp_posts`

链接

`wp_links`

导航菜单

`wp_posts`

分类目录

`wp_terms`

标签

`wp_terms`

自定义分类法

`wp_term_taxonomy`

分类项目

`wp_terms`

文章元数据

`wp_post_meta`

小工具

`wp_options`

设置选项

`wp_options`

用户

`wp_users`

文章正文

`wp_posts` (如果添加了文章)  
`wp_options` (如果添加了小工具)  
主题和插件文件 (硬编码的)

其他内容

`wp_posts` (如果添加了文章)  
`wp_options` (如果添加了小工具)  
主题和插件文件 (硬编码的)

你可能会注意到并不是所有的数据表都在上表中出现了，在这些没有出现的数据中，并不存储实际的数据，而是作为辅助性的数据表，存储着一些关于这些数据的关系。

## 总结

通过本文，希望你能对 WordPress 数据类型以及 WordPress 是如何存储这些数据的有了一个比较深入的了解。在本系列下一篇文章中，我将为大家介绍 WordPress 数据之间的关系，以及数据表之间是如何关联的。

**标签：**[WordPress数据](https://www.wpzhiku.com/tag/wordpress%e6%95%b0%e6%8d%ae/)[WordPress数据库](https://www.wpzhiku.com/tag/wordpress%e6%95%b0%e6%8d%ae%e5%ba%93/)[数据库](https://www.wpzhiku.com/tag/%e6%95%b0%e6%8d%ae%e5%ba%93/)[深入理解](https://www.wpzhiku.com/tag/godeep/)


| 信息 类型  | 涉及数据表及关联关系                                                                                                                                                   |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 用户信息   | 数据表：wp_users、wp_usermeta，关联关系：wp_users.ID->wp_usermeta.user_id                                                                                               |
| 分类信息   | 数据表：wp_terms、wp_term_taxonomy关联关系：wp_terms.term_id->wp_term_taxonomy.term_id                                                                                 |
| 链接信息   | 数据表：wp_links、wp_term_relationships、wp_terms、wp_term_taxonomy、 wp_users、wp_usermeta关联关系：                                                                      |
|        |                                                                                                                                                              |
|        | 一，确定链接所属分类 （1）wp_links.link_id->wp_term_relationships.object_id， （2）wp_term_relationships.term_taxonomy_id->wp_term_taxonomy.term_taxonomy_id                |
|        | （该关系还要取决与wp_term_taxonomy表中的taxonomy分类类型为“link_category”） （3）wp_terms.term_id->wp_term_taxonom.term_id                                                       |
|        | 二、确定链接所有者 （4）wp_links.link_owner->wp_users.ID（5）wp_users.ID->wp_usermeta.user_id                                                                             |
| 文章信息   | 数据表：wp_posts、wp_postmeta、wp_comments、wp_term_relationships、wp_terms、 wp_term_taxonomy、wp_users、wp_usermeta关联关系：一、确定文章信息 （1）wp_posts.ID->wp_postsmeta.post_id |
|        |                                                                                                                                                              |
|        | 二、确定文章评论 （2）wp_posts.ID->wp_comments.comment_post_id                                                                                                         |
|        | 三、确定文章评论的作者 （3）wp_comments.comment_author->wp_users.ID                                                                                                       |
|        | （4）wp_users.ID->wp_usermeta.user_id                                                                                                                          |
|        | 四、确定文章所属分类                                                                                                                                                   |
|        | （5）wp_posts.ID->wp_term_relationships.object_id，                                                                                                             |
|        | （6）wp_term_relationships.term_taxonomy_id->wp_term_taxonomy.term_taxonomy_id                                                                                 |
|        | （该关系还要取决与wp_term_taxonomy表中的taxonomy分类类型为“category”或者“tag”）                                                                                                  |
|        | （7）wp_terms->term_id->wp_term_taxonomy                                                                                                                       |
|        | 五、确定文章作者                                                                                                                                                     |
|        | （8）wp_posts.author->wp_users.ID;                                                                                                                             |
|        | （9）wp_users.ID->wp_usermeta.user_id                                                                                                                          |
| 文章评论信息 | 数据表：wp_comments、wp_posts、wp_users、wp_usermeta关联关系：                                                                                                           |
|        | 一、确定评论的文章 （1）wp_comments.comment_post_id->wp_posts.ID                                                                                                        |
|        | 二、确定评论的作者 （2）wp_comments.comment_author->wp_users.ID（3）wp_users.ID->wp_usermeta.user_id                                                                      |
| 基本配置信息 | 数据表：wp_options没有关联关系                                                                                                                                         |
