# [How to store tags for a blog article in SQL database?](https://stackoverflow.com/questions/59350887/how-to-store-tags-for-a-blog-article-in-sql-database)


Asked 2 years, 11 months ago Modified [2 years, 11 months ago](https://stackoverflow.com/questions/59350887/how-to-store-tags-for-a-blog-article-in-sql-database?lastactivity "2019-12-16 05:34:59Z") Viewed 1k times

I currently am working on developing a blogging website. For this application we're using MySQL as the database. For this application,I created a blog table which contains the following properties:

-   id
-   blog_title
-   content
-   user_id
-   updated_at
-   upvotes

I want to add **tags** to this table. What is the recommended way of adding tags to this table so that in the application I can search for articles/blogs based on **tags** ?

-   [mysql](https://stackoverflow.com/questions/tagged/mysql "show questions tagged 'mysql'")
-   [sql](https://stackoverflow.com/questions/tagged/sql "show questions tagged 'sql'")
-   [blogs](https://stackoverflow.com/questions/tagged/blogs)

asked Dec 16, 2019 at 5:18


## Answer

It is common to use many-to-many relationship for tags. In your case it can be a couple of tables:

tags

-   id
-   name

tag_blog

-   tag_id (foreign key to tags)
-   article_id (foreign key to articles)

You can set combination of (tag_id, article_id) as primary key. In this case it will be guaranteed that tag can be mentioned only once for the given article.






# [How does a tagging article database look like?](https://dba.stackexchange.com/questions/72986/how-does-a-tagging-article-database-look-like)

[Ask Question](https://dba.stackexchange.com/questions/ask)

Asked 8 years, 4 months ago

Modified [8 years, 4 months ago](https://dba.stackexchange.com/questions/72986/how-does-a-tagging-article-database-look-like?lastactivity "2014-08-02 12:51:45Z")

Viewed 840 times

0

[](https://dba.stackexchange.com/posts/72986/timeline)

I have a article website and each article has some tags. How should be my database structure?

This is my article table (_article_):

```sql
article_id (key)  |  article_content  |  article_title  |  article_lang_id
```

This is the list of my tags _(tag_list):_

```sql
tag_list_id (key)  |  tag_list_name  |  tag_list_description
```

And this is my tag table _(tag_link)_:

```sql
  tag_id (key)   |   article_id (f-key)  |   tag_list_id (f-key)
```

I echo the article content with php, how can I echo (I mean showing tag bellow content) tags bellow the content?

This is my question:

**1- How can I show each content tags near content?**

(each content can has less than 5 tags)

**2-What is the MYSQL query that I can run to select my tags with content?** (is there any professional MySQL orders to select content with tags form tag_link and then select the name of tag from tag_list and then show it near article, can I do it with MySQL orders like join, having, on and several other? How?)

**3-Is my database structure for tagging true?**

(I can do this with php variables and ... , but I need to do it by MySQL)

-   [mysql](https://dba.stackexchange.com/questions/tagged/mysql "show questions tagged 'mysql'")
-   [php](https://dba.stackexchange.com/questions/tagged/php "show questions tagged 'php'")

[Share](https://dba.stackexchange.com/q/72986 "Short permalink to this question")

[Improve this question](https://dba.stackexchange.com/posts/72986/edit)

Follow

asked Aug 2, 2014 at 8:23

[

![Mohammad's user avatar](media/Mohammad's_user_avatar.jpg)

](https://dba.stackexchange.com/users/44860/mohammad)

[Mohammad](https://dba.stackexchange.com/users/44860/mohammad)

311 bronze badge

[Add a comment](https://dba.stackexchange.com/questions/72986/how-does-a-tagging-article-database-look-like# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 1 Answer

Sorted by:

                                              Highest score (default)                                                                   Date modified (newest first)                                                                   Date created (oldest first)                              

1

[](https://dba.stackexchange.com/posts/72996/timeline)

1 & 2:use inner join

```sql
SELECT column_name(s)
FROM table1
INNER JOIN table2
ON table1.column_name=table2.column_name;
```

3: yes your structure is true