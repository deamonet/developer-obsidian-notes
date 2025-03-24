What does it mean if the Mysql query:

```sql
SHOW PROCESSLIST;
```

returns "Sending data" in the State column?

I imagine it means the query has been executed and MySQL is sending “result” Data to the client but I'm wondering why its taking so much time (up to an hour).

Thank you.



299

[](https://stackoverflow.com/posts/10347999/timeline)

This is quite a misleading status. It should be called "reading and filtering data".

This means that `MySQL` has some data stored on the disk (or in memory) which is yet to be read and sent over. It may be the table itself, an index, a temporary table, a sorted output etc.

If you have a 1M records table (without an index) of which you need only one record, `MySQL` will still output the status as "sending data" while scanning the table, despite the fact it has not sent anything yet.

> _MySQL 8.0.17 and later_: This state is no longer indicated separately, but rather is included in the _Executing_ state.

[edited Sep 15, 2021 at 12:06](https://stackoverflow.com/posts/10347999/revisions "show all edits to this post")
answered Apr 27, 2012 at 9:17

