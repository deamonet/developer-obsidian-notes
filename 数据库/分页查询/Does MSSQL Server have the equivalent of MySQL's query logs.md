SQL server keeps statistical information about all queries in various tables. You can use the following code to determine what the longest running query is (from the `sys.dm_exec_query_stats` table).

You should run the following DBCC commands:

This DBCC command clears the server cache and restarts logging of the query running time:

```
DBCC FREEPROCCACHE
```

Run this query to find the longest running query:

```
SELECT DISTINCT TOP 10
 t.TEXT QueryName,
 s.execution_count AS ExecutionCount,
 s.max_elapsed_time AS MaxElapsedTime,
 ISNULL(s.total_elapsed_time / s.execution_count, 0) AS AvgElapsedTime,
 s.creation_time AS LogCreatedOn,
 ISNULL(s.execution_count / DATEDIFF(s, s.creation_time, GETDATE()), 0) AS FrequencyPerSec
 FROM sys.dm_exec_query_stats s
 CROSS APPLY sys.dm_exec_sql_text( s.sql_handle ) t
 ORDER BY
 s.max_elapsed_time DESC
 GO 
```

You should also take a look at the _Technet_ magazine article [Optimizing SQL Server Query Performance](http://technet.microsoft.com/en-us/magazine/2007.11.sqlquery.aspx), which has a query for determining which query is the most expensive read I/O query, as well as guidance on how to look at execution plans and other optimizations.