As of version 4.2, Babelfish doesn't support using the `DELETE` statement which references a `CTE` alias. Consider the following example:
``` SQL
DROP TABLE IF EXISTS [dbo].[groupingTree]
CREATE TABLE [dbo].[groupingTree](
[GroupingId] [int] NOT NULL,
[Code] varchar(10) NOT NULL,  -- Should really be the primary key column
[Description] varchar(50) NOT NULL,
[SortOrder] [int] NOT NULL
)

INSERT INTO [dbo].[groupingTree] ([GroupingId]
,[Code]
,[Description]
,[SortOrder]) VALUES
(2, 'A', 'fd', 2),
(2, 'A', 'fd', 3),
(4, 'B', 'fd', 2),
(5, 'A', 'fd', 5)
;
```
In this scenario, data entered into the table by an application that was simply inserting values into the table with duplicate `Code` values 
and using the largest `SortOrder` value as the true value. The developer took this approach because the didn't know that having this historical values 
really slowed down queries and customer didn't need to track the old values.   
   
In order to create a primary key on `Code`, they needed to delete the latest version of the records with the largest `SortOrder` value.   

In SQL Server, you could use the following T-SQL to delete the "older" records like this:
``` SQL
;WITH cte AS (
SELECT [Code], ROW_NUMBER() OVER(PARTITION BY [Code] ORDER BY [SortOrder] DESC) AS RN
FROM [dbo].[groupingTree]
)
DELETE cte WHERE RN > 1
;
```
However, if you try the same command in Babelfish, you'll get the error:
```
Msg 33557097, Level 16, State 1, Line 21
relation "cte" does not exist
```
This is because Babelfish currently doesn't support what's needed to determine which rows to delete against the CTE. This is a limitation in PostgreSQL.
However, you can use the `CTE` for the `DLETE` statement like this.
``` SQL
-- Still use the CTE, but you need to include column names to JOIN with the DELETE
;WITH cte AS (
SELECT [Code], [SortOrder], ROW_NUMBER() OVER(PARTITION BY [Code] ORDER BY [SortOrder] DESC) AS RN
FROM [dbo].[groupingTree]
)
DELETE [dbo].[groupingTree]
FROM  [dbo].[groupingTree]  AS g
INNER JOIN cte ON g.[Code] = cte.[Code] AND g.[SortOrder] = cte.[SortOrder]
WHERE cte.RN > 1
;
```
This correctly returns the result for the table where:
``` SQL
-- Verify the result
SELECT * FROM [dbo].[groupingTree] ORDER BY [SortOrder]
;
```
Returns the desired result of:   
     
| GroupingId | Code | Description | SortOrder |
|------------|------|-------------|-----------|   
| 4          | B    | fd          | 2   |
| 5          | A    | fd          | 5   |

That's it!
