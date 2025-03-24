# [INSERT with SELECT](https://stackoverflow.com/questions/5391344/insert-with-select)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 13 years, 8 months ago

Modified [3 years, 1 month ago](https://stackoverflow.com/questions/5391344/insert-with-select?lastactivity "2021-10-06 13:11:34Z")

Viewed 477k times

353

[](https://stackoverflow.com/posts/5391344/timeline)

I have a query that inserts using a `SELECT` statement:

```sql
INSERT INTO courses (name, location, gid) 
SELECT name, location, gid 
FROM courses 
WHERE cid = $cid
```

Is it possible to only select "name, location" for the insert, and set `gid` to something else in the query?

- [mysql](https://stackoverflow.com/questions/tagged/mysql "show questions tagged 'mysql'")
- [sql](https://stackoverflow.com/questions/tagged/sql "show questions tagged 'sql'")
- [insert-select](https://stackoverflow.com/questions/tagged/insert-select "show questions tagged 'insert-select'")



Yes, absolutely, but check your syntax.

```sql
INSERT INTO courses (name, location, gid)
SELECT name, location, 1
FROM   courses
WHERE  cid = 2
```

You can put a constant of the same type as `gid` in its place, not just 1, of course. And, I just made up the `cid` value.