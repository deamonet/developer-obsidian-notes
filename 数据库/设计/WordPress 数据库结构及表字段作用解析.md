

2020年11月1日 2730点热度 0人点赞 2条评论

首先分别来先看看 WordPress 所有的数据表都是干什么用的吧，一下是 WordPress 完整的 12 张数据表，当然刚安装好的时候一般是只有 11 张表的，如果还有其它相关的数据库，那么可能就不是 WordPress 本身的数据库，有可能是某些主题或插件需要而创建的，所以一下肯定子凡要介绍的还是 WordPress 本身的数据库表和字段了。

1.  wp_commentmeta：存储 Akismet 或手工审核的评论是否为垃圾评论的判断结果；
2.  wp_comments：存储评论信息，如评论内容、评论所属文章、评论人昵称、邮箱、URL 等；
3.  wp_links：存储友情链接信息，如友链名称、URL、打开方式、描述、是否可见等；
4.  wp_options：存储 WordPress 系统默认及后台系统选项、插件及主题配置信息，包括网站标题、副标题、当前主题等等；
5.  wp_postmeta：存储文章的一些相关信息，如文章附件图片的 alt 信息、文章所在分类的 URL 以及文章自定义的自定义字段，其中可能就有文章访问次数等；
6.  wp_posts：存储文章信息，包括文章标题、正文、摘要、作者、发布时间、访问密码、评论数、修改时间、文章地址等；
7.  wp_termmeta：存储对菜单分类的更多设置，属于开发性功能居多，例如分类目录的缩略图、颜色标识等；
8.  wp_terms：存储菜单分类、标签分类名称及 URL 信息；
9.  wp_term_relationships：存储文章和分类、标签的相互对应关系；
10.  wp_term_taxonomy：存储分类和标签的描述信息、父子关系、所属包含的文章数等；
11.  wp_usermeta：存储用户的姓名、昵称、权限等信息；
12.  wp_users：存储用户名、密码、昵称、邮箱、注册时间等信息；

## 1. wp_commentmeta

-   meta_id：自增唯一 ID
-   comment_id：评论 ID
-   meta_key：键名
-   meta_value：键值

## 2. wp_comments

-   comment_ID：自增唯一 ID
-   comment_post_ID：对应文章 ID
-   comment_author：评论者
-   comment_author_email：评论者邮箱
-   comment_author_url：评论者网址
-   comment_author_IP：评论者 IP
-   comment_date：评论时间
-   comment_date_gmt：评论时间（GMT+0 时间）
-   comment_content：评论正文
-   comment_karma：未知
-   comment_approved：评论是否被批准
-   comment_agent：评论者的 USER AGENT
-   comment_type：评论类型(pingback/普通)
-   comment_parent：父评论 ID
-   user_id：评论者用户 ID（未登录用户的评论则为空

## 3. wp_links

-   link_id：自增唯一 ID
-   link_url：链接 URL
-   link_name：链接标题
-   link_image：链接图片
-   link_target：链接打开方式
-   link_description：链接描述
-   link_visible：是否可见（Y/N）
-   link_owner：添加者用户 ID
-   link_rating：评分等级
-   link_updated：未知
-   link_rel：XFN 关系
-   link_notes：XFN 注释
-   link_rss：链接 RSS 地址

## 4. wp_options

-   option_id：自增唯一 ID
-   blog_id：博客 ID，用于多用户博客，默认 0
-   option_name：键名
-   option_value：键值
-   autoload：WordPress 加载时自动载入（yes/no）

## 5. wp_postmeta

-   meta_id：自增唯一 ID
-   post_id：对应文章 ID
-   meta_key：键名
-   meta_value：键值

## 6. wp_posts

-   ID：自增唯一 ID
-   post_author：对应作者 ID
-   post_date：发布时间
-   post_date_gmt：发布时间（GMT+0 时间）
-   post_content：正文
-   post_title：标题
-   post_excerpt：摘录
-   post_status：文章状态（publish/auto-draft/inherit 等）
-   comment_status：评论状态（open/closed）
-   ping_status：PING 状态（open/closed）
-   post_password：文章密码
-   post_name：文章缩略名
-   to_ping：未知
-   pinged：已经 PING 过的链接
-   post_modified：修改时间
-   post_modified_gmt：修改时间（GMT+0 时间）
-   post_content_filtered：未知
-   post_parent：父文章，主要用于 PAGE
-   guid：唯一标识符（短链接）
-   menu_order：排序 ID
-   post_type：文章类型（post/page 等）
-   post_mime_type：MIME 类型
-   comment_count：评论总数

## 7. wp_termmeta

-   meta_id：自增唯一 ID
-   term_id：分类 ID
-   meta_key：键名
-   meta_value：键值

## 8. wp_terms

-   term_id：分类 ID
-   name：分类名
-   slug：缩略名
-   term_group：分组

## 9. wp_term_relationships

-   object_id：对应文章 ID/链接 ID
-   term_taxonomy_id：对应自定义分类 ID
-   term_order：排序

## 10. wp_term_taxonomy

-   term_taxonomy_id：自定义分类 ID
-   term_id：分类 ID
-   taxonomy：分类(category/post_tag)
-   description：分类描述
-   parent：所属父分类 ID
-   count：文章数统计

## 11. wp_usermeta

-   umeta_id：自增唯一 ID
-   user_id：对应用户 ID
-   meta_key：键名
-   meta_value：键值

## 12. wp_users

-   ID：自增唯一 ID
-   user_login：登录名
-   user_pass：密码
-   user_nicename：昵称
-   user_email：Email
-   user_url：网址
-   user_registered：注册时间
-   user_activation_key：激活码
-   user_status：用户状态
-   display_name：显示名称