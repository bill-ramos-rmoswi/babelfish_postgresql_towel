# Optimizing String Splitting in SQL Server and Babelfish: A Performance Journey
Written by Bill Ramos @bill_ramos_rmoswi   
This blog post was written with the help of Claude.ai and validated by me.   

## Introduction

As database technologies evolve, it's crucial to stay updated with the latest features that can enhance your query performance. In this post, we'll explore string splitting operations in SQL Server and Babelfish for PostgreSQL, comparing different approaches and their performance implications. I'll also provide practical guidance on how to refactor your existing code to take advantage of these optimizations.

## The Evolution of String Splitting in SQL Server

Prior to SQL Server 2016, developers often created their own string-splitting functions using various patterns. While these custom functions got the job done, they often came with a performance cost. Let's look at three common patterns:

1. Looping through characters
2. Using XML
3. Employing a recursive CTE

Here's an example of a custom string-splitting function using a loop:

```sql
CREATE FUNCTION dbo.split_string_loop
(
    @String NVARCHAR(MAX),
    @Delimiter CHAR(1)
)
RETURNS @Result TABLE
(
    [data] NVARCHAR(MAX)
)
AS
BEGIN
    DECLARE @StartIndex INT = 1
    DECLARE @EndIndex INT

    WHILE @StartIndex <= LEN(@String)
    BEGIN
        SET @EndIndex = CHARINDEX(@Delimiter, @String, @StartIndex)
        
        IF @EndIndex = 0
            SET @EndIndex = LEN(@String) + 1
        
        INSERT INTO @Result ([data])
        VALUES (SUBSTRING(@String, @StartIndex, @EndIndex - @StartIndex))
        
        SET @StartIndex = @EndIndex + 1
    END

    RETURN
END
```

Each of these methods has its drawbacks in terms of performance, especially when dealing with large datasets or frequent calls.

## Enter STRING_SPLIT()

With SQL Server 2016, Microsoft introduced the native `STRING_SPLIT()` function. This built-in function offers a more efficient way to split strings, potentially boosting your query performance.

Instead of:

```sql
SELECT [data] FROM [dbo].[split_string_loop]('1,2', ',');
```

You can now use:

```sql
SELECT [value] FROM STRING_SPLIT('1,2', ',');
```

## Performance Comparison: UDF vs Native STRING_SPLIT()

To demonstrate the performance benefits of using the native STRING_SPLIT() function, I used the AdventureWorksDW2022 sample database. You can download this database from the [official Microsoft GitHub repository](https://github.com/Microsoft/sql-server-samples/releases/tag/adventureworks).

I set up a scenario using the FactResellerSales table, adding a new column called SalesTerritoryTags and populating it with test data. Then, I compared the performance of a user-defined function (UDF) for string splitting against the native STRING_SPLIT() function.

The results showed a significant performance difference:

- UDF version:
  - CPU time = 1404 ms, elapsed time = 293 ms
  - Worktable: 471,244 logical reads
  - FactResellerSales: 3,476 logical reads, Scan count 9

- Built-in STRING_SPLIT() version:
  - CPU time = 110 ms, elapsed time = 199 ms
  - FactResellerSales: 3,301 logical reads, Scan count 1

The built-in STRING_SPLIT() function outperforms the UDF for several reasons:

1. Optimization: It's highly optimized by Microsoft for this specific task.
2. Reduced I/O: It requires significantly fewer I/O operations.
3. No Worktable: It doesn't need to create and use a temporary worktable.
4. Fewer Scans: It scans the main table only once.
5. Lower CPU Usage: It uses more efficient algorithms and possibly takes advantage of lower-level optimizations.
6. Scalability: The performance gap would likely widen with larger datasets.

## Refactoring Your Code: A GenAI Approach for SQL Server

One challenge in adopting STRING_SPLIT() is that it returns a column named `[value]`, while many custom functions return a column named `[data]`. This discrepancy can make refactoring existing code a daunting task.

To help with this transition, I can leverage GenAI to generate a prompt for refactoring T-SQL code. Here's an example prompt:

```
Generate a T-SQL script to refactor code using custom string-splitting functions to use the native STRING_SPLIT() function. The script should:

1. Replace calls to custom functions (like split_string_loop, split_string_xml, split_string_cte) with STRING_SPLIT().
2. Change references to the 'data' column to 'value'.
3. Handle cases where the split function is used in subqueries or as part of larger queries.
4. Maintain the original query logic and structure as much as possible.
5. Include comments explaining the changes made.

Example input:
SELECT @FinalInvoice = COALESCE(@FinalInvoice + ', ', '') + CAST(EntityId AS VARCHAR(MAX))                  
FROM tblTagApplied WITH (NOLOCK)                  
WHERE Entity = 'Invoice' AND TagListId IN (                  
    SELECT Data                  
    FROM Split_String_Loop(@prmInvoiceTag, ',') 
)

Please provide the refactored T-SQL code.
```

Using this prompt with a GenAI tool can help you quickly generate refactored code. Here's an example of what the output might look like:

```sql
-- Refactored query using STRING_SPLIT()
SELECT @FinalInvoice = COALESCE(@FinalInvoice + ', ', '') + CAST(EntityId AS VARCHAR(MAX))                  
FROM tblTagApplied WITH (NOLOCK)                  
WHERE Entity = 'Invoice' AND TagListId IN (                  
    SELECT value -- Changed 'Data' to 'value'
    FROM STRING_SPLIT(@prmInvoiceTag, ',') -- Replaced Split_String_Loop with STRING_SPLIT
)

-- Comments:
-- 1. Replaced Split_String_Loop(@prmInvoiceTag, ',') with STRING_SPLIT(@prmInvoiceTag, ',')
-- 2. Changed 'Data' to 'value' in the subquery SELECT clause
-- 3. The rest of the query structure remains unchanged
```

## Using STRING_SPLIT in Babelfish for PostgreSQL

While SQL Server's native STRING_SPLIT() function offers significant performance improvements, users of Babelfish for PostgreSQL have additional options to consider. As of Babelfish version 4.2, the implementation of sys.string_split is an improvement over a custom UDF, but it still uses a UDF under the hood. However, PostgreSQL provides a native function called string_to_table that can offer even better performance.

Here's how you can use string_to_table in a Babelfish T-SQL query:

```sql
DECLARE @SalesTerritoryTags VARCHAR(100) = '1,2,3,4,5';

SELECT COUNT(*) AS SalesCount
FROM FactResellerSales f
WHERE EXISTS (
    SELECT 1 
    FROM string_to_table(f.SalesTerritoryTags, ',' COLLATE "C") AS s(value)
    WHERE s.value IN (
        SELECT unnest.value
        FROM string_to_table(@SalesTerritoryTags, ',' COLLATE "C") AS unnest(value)
    )
);
```

Note the use of `COLLATE "C"` with the delimiter. This is necessary because Babelfish requires explicit collation for string comparisons when calling native PostgreSQL functions.

In this query, `unnest` is an alias for the table returned by `string_to_table`. The `UNNEST` function in PostgreSQL is used to expand an array into a set of rows. In this case, `string_to_table` is returning a set of values, which is similar to using `UNNEST` on an array.

For more information on these PostgreSQL functions, you can refer to:
- [STRING_TO_TABLE documentation](https://www.postgresql.org/docs/current/functions-string.html#FUNCTIONS-STRING-OTHER)
- [UNNEST documentation](https://www.postgresql.org/docs/current/functions-array.html#ARRAY-FUNCTIONS-TABLE)

## Refactoring Your Code: A GenAI Approach for Babelfish

To help refactor existing code for use with Babelfish, you can use the following prompt with a GenAI tool:

```
Generate a T-SQL script compatible with Babelfish for PostgreSQL to refactor code using custom string-splitting functions to use the PostgreSQL native string_to_table function. The script should:

1. Replace calls to custom functions (like split_string_loop, split_string_xml, split_string_cte) with string_to_table().
2. Change references to the 'data' column to 'value'.
3. Handle cases where the split function is used in subqueries or as part of larger queries.
4. Add the necessary COLLATE "C" clause to string literals used as delimiters.
5. Maintain the original query logic and structure as much as possible.
6. Include comments explaining the changes made.

Example input:
SELECT @FinalInvoice = COALESCE(@FinalInvoice + ', ', '') + CAST(EntityId AS VARCHAR(MAX))                  
FROM tblTagApplied WITH (NOLOCK)                  
WHERE Entity = 'Invoice' AND TagListId IN (                  
    SELECT Data                  
    FROM Split_String_Loop(@prmInvoiceTag, ',') 
)

Please provide the refactored T-SQL code compatible with Babelfish for PostgreSQL.
```

This revised prompt will help generate code that takes advantage of PostgreSQL's native string-splitting capabilities while maintaining compatibility with Babelfish's T-SQL interface.

## Performance Comparison: Babelfish STRING_SPLIT vs Native PostgreSQL string_to_table

My performance tests revealed significant differences between Babelfish's STRING_SPLIT implementation and PostgreSQL's native string_to_table function:

1. Babelfish STRING_SPLIT:
   - CPU time: 0.828125 s user, 0.015625 s system
   - Elapsed time: 2.440177 s

2. PostgreSQL native string_to_table:
   - CPU time: 0.015625 s user, 0.000000 s system
   - Elapsed time: 0.103192 s

Key observations:

1. CPU Usage: The native PostgreSQL function uses significantly less CPU time (0.015625 s) than the Babelfish implementation (0.828125 s). This indicates that string_to_table is much more efficiently implemented and optimized for PostgreSQL.

2. Elapsed Time: The native function completes in about 0.103 seconds, while the Babelfish implementation takes about 2.44 seconds. This is a dramatic difference in execution time, with the native function being roughly 23 times faster.

3. System Resources: The native function uses no measurable system time, while the Babelfish implementation uses 0.015625 s. This suggests that the native function requires fewer system resources and operations.

These metrics clearly show that the native PostgreSQL string_to_table function is much more efficient than the current Babelfish implementation of sys.string_split. This performance difference is likely due to the native function being implemented in C as part of the core PostgreSQL engine, while the Babelfish version is implemented as a user-defined function, which incurs additional overhead.

## Considerations When Using STRING_SPLIT() or string_to_table

While these native functions offer significant performance improvements, keep in mind:

1. Compatibility: STRING_SPLIT() is only available in SQL Server 2016 and later versions.
2. Ordering: In SQL Server, the order of elements is not guaranteed unless you use the `ordinal` option (available from SQL Server 2022).
3. Complex Scenarios: These functions may not be suitable for extremely complex string-splitting scenarios that require custom logic.
4. Babelfish Specifics: When using string_to_table in Babelfish, remember to use the COLLATE "C" clause with string literals used as delimiters.

## Conclusion

Keeping up with the latest database features is crucial for maintaining high-performing queries. The introduction of STRING_SPLIT() in SQL Server 2016 and the availability of PostgreSQL's native string_to_table function in Babelfish offer significant opportunities to improve code that relies on string-splitting operations.

When optimizing your database queries:

1. Replace custom string-splitting UDFs with native functions when possible.
2. In SQL Server, use STRING_SPLIT() for better performance.
3. In Babelfish, consider using PostgreSQL's string_to_table for optimal performance.
4. Leverage GenAI tools to assist in refactoring your existing code.
5. Regularly review your code for opportunities to leverage new database features.
6. Test performance with your specific data and queries, as results may vary based on data size and query complexity.

By embracing these optimized string-splitting methods, you can potentially see substantial performance improvements, especially in frequently executed queries or those dealing with large datasets. The dramatic performance differences I observed (up to 23 times faster in some cases) highlight the importance of using these native functions.

Moreover, using GenAI prompts for code refactoring can make the transition process smoother and less error-prone than manual search-and-replace methods. This approach not only saves time but also reduces the risk of introducing bugs during the refactoring process.

Remember, database optimization is an ongoing process. Regularly reviewing your code for opportunities to leverage new SQL Server or PostgreSQL features can lead to significant performance gains over time. Stay curious, keep learning, and don't hesitate to embrace new tools and techniques that can make your databases run faster and more efficiently.

By following these guidelines and utilizing the provided GenAI prompts, you can streamline your code refactoring process and significantly improve your database performance, whether you're using SQL Server or Babelfish for PostgreSQL.
