[](https://stackoverflow.com/posts/5609657/timeline)

You need to use the `WHERE` clause. As shown in the [docs](http://dev.mysql.com/doc/refman/5.5/en/show-tables.html), you can only have a single pattern if you use `"SHOW TABLES LIKE ..."`, but you can use an expression in the WHERE clause if you use `"SHOW TABLES WHERE ..."`. Since you want an expression, you need to use the `WHERE` clause.
```sql
SHOW TABLES
FROM `<yourdbname>`
WHERE 
    `Tables_in_<yourdbname>` LIKE '%cms%'
    OR `Tables_in_<yourdbname>` LIKE '%role%';
```

