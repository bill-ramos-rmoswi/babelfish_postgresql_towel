# Anti-Pattern - Non-SARGable values with BETWEEN clause
To optimize a T-SQL `WHERE` clause that uses a `BETWEEN` clause for `datetime` types, you can follow these steps:

1. **Use appropriate data types**: Ensure that the `datetime` columns involved in the `BETWEEN` clause are using the appropriate data type, such as `datetime2` or `datetimeoffset`, which offer better precision and range compared to the older `datetime` data type.

2. **Create appropriate indexes**: Create indexes on the `datetime` columns used in the `BETWEEN` clause. Indexes will significantly improve the performance of the query, as the query optimizer can use them to quickly locate the relevant rows.

   ```sql
   CREATE INDEX IX_TableName_DateTimeColumn ON TableName (DateTimeColumn);
   ```

3. **Use inclusive `BETWEEN` clauses**: When possible, use inclusive `BETWEEN` clauses (e.g., `BETWEEN '2023-01-01' AND '2023-12-31'`) instead of exclusive `BETWEEN` clauses (e.g., `> '2023-01-01' AND < '2023-12-31'`). Inclusive `BETWEEN` clauses are generally more efficient, as the query optimizer can better leverage the indexes.

4. **Avoid functions in the `WHERE` clause**: Avoid using functions, such as `CONVERT` or `CAST`, on the `datetime` columns in the `WHERE` clause. These functions can prevent the query optimizer from using the indexes effectively. If you currently use `ISNULL`, replace it with `COALESCE` or pre-compute.

   ```sql
   -- Good
   WHERE DateTimeColumn BETWEEN '2023-01-01' AND '2023-12-31'

   -- Bad
   WHERE CONVERT(DATE, DateTimeColumn) BETWEEN '2023-01-01' AND '2023-12-31'

   -- Bad
   DECLARE @begindate DATE = NULL;
   DECLARE @enddate DATE = NULL;
   -- some app code that might change the variables above
   WHERE DateTimeColumn BETWEEN ISNULL(@begindate, CAST( DATEADD( d, GETDATE(), -30) AS DATE)
                            AND ISNULL(@enddate, CAST( GETDATE() AS DATE)
   -- Better
   DECLARE @begindate DATE = CAST( DATEADD( d, GETDATE(), -30) AS DATE);
   DECLARE @enddate DATE = CAST( GETDATE() AS DATE);
   -- some app code that might change the variables above
   WHERE DateTimeColumn BETWEEN @begindate AND @enddate
   ```

5. **Use appropriate data types for parameters**: If you're passing parameters to the `BETWEEN` clause, ensure that the parameter data types match the data types of the columns in the table. This will allow the query optimizer to use the indexes effectively.

6. **Monitor and analyze query plans**: Regularly monitor and analyze the query plans for your T-SQL statements that use `BETWEEN` clauses on `datetime` types. This will help you identify any areas for further optimization, such as missing indexes or suboptimal query structures.

By following these optimization techniques, you can improve the performance of your T-SQL queries that use `BETWEEN` clauses on `datetime` types.
