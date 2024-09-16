类似 MySQL 里的分页，须结合 Order By 使用

-- Syntax for SQL Server and Azure SQL Database  
  
ORDER BY order_by_expression  
    [ COLLATE collation_name ]   
    [ ASC | DESC ]   
    [ ,...n ]   
[ <offset_fetch> ]  
  
<offset_fetch> ::=  
{   
    OFFSET { integer_constant | offset_row_count_expression } { ROW | ROWS }  
    [  
      FETCH { FIRST | NEXT } {integer_constant | fetch_row_count_expression } { ROW | ROWS } ONLY  
    ]  
}


[ORDER BY 子句 (Transact-SQL) - SQL Server | Microsoft Learn](https://learn.microsoft.com/zh-cn/sql/t-sql/queries/select-order-by-clause-transact-sql?view=sql-server-ver16)