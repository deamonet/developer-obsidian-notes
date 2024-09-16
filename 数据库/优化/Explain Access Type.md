#### EXPLAIN Join Types

The `type` column of [`EXPLAIN`](https://dev.mysql.com/doc/refman/8.0/en/explain.html "13.8.2 EXPLAIN Statement") output describes how tables are joined. In JSON-formatted output, these are found as values of the `access_type` property. The following list describes the join types, ordered from the best type to the worst:

-   [`system`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_system)
    
    The table has only one row (= system table). This is a special case of the [`const`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_const) join type.
    
-   [`const`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_const)
    
    The table has at most one matching row, which is read at the start of the query. Because there is only one row, values from the column in this row can be regarded as constants by the rest of the optimizer. [`const`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_const) tables are very fast because they are read only once.
    
    [`const`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_const) is used when you compare all parts of a `PRIMARY KEY` or `UNIQUE` index to constant values. In the following queries, _`tbl_name`_ can be used as a [`const`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_const) table:
    
    ```sql
    SELECT * FROM tbl_name WHERE primary_key=1;
    
    SELECT * FROM tbl_name
      WHERE primary_key_part1=1 AND primary_key_part2=2;
    ```
    
-   [`eq_ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_eq_ref)
    
    One row is read from this table for each combination of rows from the previous tables. Other than the [`system`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_system) and [`const`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_const) types, this is the best possible join type. It is used when all parts of an index are used by the join and the index is a `PRIMARY KEY` or `UNIQUE NOT NULL` index.
    
    [`eq_ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_eq_ref) can be used for indexed columns that are compared using the `=` operator. The comparison value can be a constant or an expression that uses columns from tables that are read before this table. In the following examples, MySQL can use an [`eq_ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_eq_ref) join to process _`ref_table`_:
    
    ```sql
    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column=other_table.column;
    
    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column_part1=other_table.column
      AND ref_table.key_column_part2=1;
    ```
    
-   [`ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_ref)
    
    All rows with matching index values are read from this table for each combination of rows from the previous tables. [`ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_ref) is used if the join uses only a leftmost prefix of the key or if the key is not a `PRIMARY KEY` or `UNIQUE` index (in other words, if the join cannot select a single row based on the key value). If the key that is used matches only a few rows, this is a good join type.
    
    [`ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_ref) can be used for indexed columns that are compared using the `=` or `<=>` operator. In the following examples, MySQL can use a [`ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_ref) join to process _`ref_table`_:
    
    ```sql
    SELECT * FROM ref_table WHERE key_column=expr;
    
    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column=other_table.column;
    
    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column_part1=other_table.column
      AND ref_table.key_column_part2=1;
    ```
    
-   [`fulltext`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_fulltext)
    
    The join is performed using a `FULLTEXT` index.
    
-   [`ref_or_null`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_ref_or_null)
    
    This join type is like [`ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_ref), but with the addition that MySQL does an extra search for rows that contain `NULL` values. This join type optimization is used most often in resolving subqueries. In the following examples, MySQL can use a [`ref_or_null`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_ref_or_null) join to process _`ref_table`_:
    
    ```sql
    SELECT * FROM ref_table
      WHERE key_column=expr OR key_column IS NULL;
    ```
    
    See [Section 8.2.1.15, “IS NULL Optimization”](https://dev.mysql.com/doc/refman/8.0/en/is-null-optimization.html "8.2.1.15 IS NULL Optimization").
    
-   [`index_merge`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_index_merge)
    
    This join type indicates that the Index Merge optimization is used. In this case, the `key` column in the output row contains a list of indexes used, and `key_len` contains a list of the longest key parts for the indexes used. For more information, see [Section 8.2.1.3, “Index Merge Optimization”](https://dev.mysql.com/doc/refman/8.0/en/index-merge-optimization.html "8.2.1.3 Index Merge Optimization").
    
-   [`unique_subquery`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_unique_subquery)
    
    This type replaces [`eq_ref`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_eq_ref) for some `IN` subqueries of the following form:
    
    ```sql
    value IN (SELECT primary_key FROM single_table WHERE some_expr)
    ```
    
    [`unique_subquery`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_unique_subquery) is just an index lookup function that replaces the subquery completely for better efficiency.
    
-   [`index_subquery`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_index_subquery)
    
    This join type is similar to [`unique_subquery`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_unique_subquery). It replaces `IN` subqueries, but it works for nonunique indexes in subqueries of the following form:
    
    ```sql
    value IN (SELECT key_column FROM single_table WHERE some_expr)
    ```
    
-   [`range`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_range)
    
    Only rows that are in a given range are retrieved, using an index to select the rows. The `key` column in the output row indicates which index is used. The `key_len` contains the longest key part that was used. The `ref` column is `NULL` for this type.
    
    [`range`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_range) can be used when a key column is compared to a constant using any of the [`=`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_equal), [`<>`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_not-equal), [`>`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_greater-than), [`>=`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_greater-than-or-equal), [`<`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_less-than), [`<=`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_less-than-or-equal), [`IS NULL`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is-null), [`<=>`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_equal-to), [`BETWEEN`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_between), [`LIKE`](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#operator_like), or [`IN()`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_in) operators:
    
    ```sql
    SELECT * FROM tbl_name
      WHERE key_column = 10;
    
    SELECT * FROM tbl_name
      WHERE key_column BETWEEN 10 and 20;
    
    SELECT * FROM tbl_name
      WHERE key_column IN (10,20,30);
    
    SELECT * FROM tbl_name
      WHERE key_part1 = 10 AND key_part2 IN (10,20,30);
    ```
    
-   [`index`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_index)
    
    The `index` join type is the same as [`ALL`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_all), except that the index tree is scanned. This occurs two ways:
    
    -   If the index is a covering index for the queries and can be used to satisfy all data required from the table, only the index tree is scanned. In this case, the `Extra` column says `Using index`. An index-only scan usually is faster than [`ALL`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_all) because the size of the index usually is smaller than the table data.
        
    -   A full table scan is performed using reads from the index to look up data rows in index order. `Uses index` does not appear in the `Extra` column.
        
    
    MySQL can use this join type when the query uses only columns that are part of a single index.
    
-   [`ALL`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_all)
    
    A full table scan is done for each combination of rows from the previous tables. This is normally not good if the table is the first table not marked [`const`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_const), and usually _very_ bad in all other cases. Normally, you can avoid [`ALL`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#jointype_all) by adding indexes that enable row retrieval from the table based on constant values or column values from earlier tables.