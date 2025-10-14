# babelfish_postgresql_towel Support Repo for Babelfish on PostgreSQL

Bill Ramos's knowledge base for tips, tricks, and best practices for migrating SQL Server solutions to 
Amazon [Aurora PostgreSQL with Babelfish](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/babelfish.html), [Babelfish for PostgreSQL](https://babelfishpg.org/), open-source [WiltonDB for Windows](https://github.com/wiltondb/wiltondb), and PostgreSQL for SQL Server professionals.

According to the [Hitchhiker's Guide to the Galaxy Fandom page](https://hitchhikers.fandom.com/wiki/Towel), 
>"Douglas Adams got the idea for the phrase 'knowing where one's towel is' when he went travelling and found that his beach towel kept disappearing. In The Hitchhiker's Guide to the Galaxy: The Original Radio Scripts, his friends describe how he would always "mislay" his towel."

Think of this repository as your towel for all aspects of using Babelfish for migrating SQL Server applications and databases.

**Content is current as of Babelfish 5.2 on PostgreSQL 17.5**

If you have you need a towel now, please [create an Issue](https://github.com/bill-ramos-rmoswi/babelfish_postgresql_towel/issues) and we can work together to help you out!

## Table of Contents

### 📋 [Assessments](./Assessments/)
Essential guidance for evaluating SQL Server solutions and planning modernization/migration projects with Babelfish.

- **When Babelfish May Not Be a Good Fit**
  - Applications without source code access
  - Dependencies on unsupported features
  - Ultra-low latency requirements (< few milliseconds)
  - Lack of organizational alignment
- **Babelfish Compass Assessments**
  - Migration assessment tools and techniques
  - Client-side T-SQL assessment guidance
  - Links to official AWS blog posts and GitHub resources
- **Coming Soon**: Babelfish Compass command line assessment tips

### 🏆 [Best Practices](./Best_Practices/)
Performance optimization techniques and best practices for using Babelfish with PostgreSQL.

- **[BETWEEN clause with SARG](./Best_Practices/BETWEEN%20clause%20with%20SARG.md)**
  - Optimizing datetime BETWEEN clauses for better performance
  - Index creation strategies
  - Avoiding non-SARGable expressions
  - Performance monitoring techniques

- **[Embracing STRING_SPLIT versus Table-Valued Function](./Best_Practices/Embracing%20STRING_SPLIT%20versus%20Table-Valued%20Function.md)**
  - Performance comparison: UDF vs native STRING_SPLIT()
  - Migration from custom string-splitting functions
  - PostgreSQL string_to_table optimization for Babelfish
  - GenAI prompts for code refactoring
  - Detailed performance metrics and benchmarks

- **[Why Your T-SQL Functions are Probably Trying to Kill You](./Best_Practices/Why%20Your%20T-SQL%20Functions%20are%20%20Probably%20Trying%20to%20Kill%20You.md)**
  - Performance impact analysis of T-SQL functions
  - Inline calculation optimization strategies
  - Performance comparisons across SQL Server and Babelfish
  - C# developer guidance for database optimization
  - Detailed performance test results and recommendations

### 📊 [Data Migration](./Data_Migration/)
*Coming soon* - Best practices for migrating data to Babelfish

### 🛠️ [Dev Tools](./Dev_Tools/)
*Coming soon* - Best practices for using development and other tools with Babelfish

### 🔧 [Workarounds](./Workarounds/)
Solutions and workarounds for unsupported Babelfish features, including GenAI instructions tested with Claude.ai Pro.

- **[DELETE with CTE](./Workarounds/DELETE%20with%20CTE.md)**
  - Workaround for DELETE statements with CTE aliases
  - Step-by-step solution for Babelfish 4.2+ limitations
  - Practical example with duplicate data cleanup
  - Alternative JOIN-based approach for CTE deletions

- **[Babelfish Backup and Restore Guide](./Workarounds/babelfish_backup_restore_guide.md)**
  - Complete backup and restore procedures using `bbf_dump` and `bbf_dumpall`
  - Development environment setup with Cursor IDE and Remote-SSH
  - Automated backup and restore scripts with error handling
  - Environment configuration and security best practices
  - Troubleshooting guide and version compatibility requirements
  - Manual command alternatives and usage examples

## Quick Start

1. **New to Babelfish?** Start with the [Assessments](./Assessments/) section to determine if Babelfish is right for your project
2. **Performance Issues?** Check the [Best Practices](./Best_Practices/) section for optimization techniques
3. **Need a Workaround?** Browse the [Workarounds](./Workarounds/) section for solutions to common limitations
4. **Have Questions?** [Create an Issue](https://github.com/bill-ramos-rmoswi/babelfish_postgresql_towel/issues) and we'll help you out!

## Contributing

This repository welcomes contributions! If you have:
- Additional best practices or optimization techniques
- New workarounds for Babelfish limitations
- Assessment tools or methodologies
- Performance benchmarks or case studies

Please [create an Issue](https://github.com/bill-ramos-rmoswi/babelfish_postgresql_towel/issues) or submit a pull request.

## License

This project is licensed under the terms specified in the [LICENSE](./LICENSE) file.
