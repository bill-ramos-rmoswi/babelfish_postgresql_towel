# SQL Server to Aurora PostgreSQL & Babelfish Transition Resource Guide

**Purpose**: Comprehensive resource collection for helping customers maintain and optimize their applications after migration to Aurora PostgreSQL, addressing the needs of Developers, DBAs, and Operations teams.

---

## 📋 Table of Contents

1. [Migration & Assessment Resources](#migration--assessment-resources)
2. [Post-Migration Operations & Monitoring](#post-migration-operations--monitoring)
3. [Performance Troubleshooting](#performance-troubleshooting)
4. [Developer Transition Guides](#developer-transition-guides)
5. [DBA Transition Guides](#dba-transition-guides)
6. [Babelfish-Specific Resources](#babelfish-specific-resources)
7. [Tools & Utilities](#tools--utilities)
8. [Expert Perspectives](#expert-perspectives)
9. [Best Practices & Patterns](#best-practices--patterns)

---

## 🎯 Migration & Assessment Resources

### AWS and Open-Source Official Documentation

**Using Babelfish for Aurora PostgreSQL**
- https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/babelfish.html
- Architecture overview, dual-endpoint (TDS/PostgreSQL) access, and core capabilities

**Babelfish for PostgreSQL accelerates your journey to migrate SQL Server applications to PostgreSQL**
- https://babelfishpg.org/
- Official Babelfish homepage and documentation portal for the open-source PostgreSQL extension that understands SQL Server T-SQL and wire protocol

### AWS Blog Posts - Migration Process

**Migrate SQL Server to Babelfish for Aurora PostgreSQL using the Compass tool and AWS DMS**
- https://aws.amazon.com/blogs/database/migrate-sql-server-to-babelfish-for-aurora-postgresql-using-the-compass-tool-and-aws-dms/
- Step-by-step migration using Babelfish Compass and AWS DMS, includes Northwind sample database example

**Migrate Microsoft SQL Server to Babelfish for Aurora PostgreSQL with minimal downtime using AWS DMS**
- https://www.amazonaws.cn/en/blog-selection/migrate-microsoft-sql-server-to-babelfish-for-aurora-postgresql-with-minimal-downtime-using-aws-dms/
- Minimal downtime approach using CDC for continuous replication

**Migrate SQL Server databases to Babelfish for Aurora PostgreSQL using change tracking with a linked server**
- https://aws.amazon.com/blogs/database/migrate-sql-server-databases-to-babelfish-for-aurora-postgresql-using-change-tracking-with-a-linked-server/
- Alternative approach for SQL Server Web Edition which doesn't support CDC

### Community Resources

**Using Babelfish to migrate to PostgreSQL**
- https://babelfishpg.org/docs/usage/migration/
- Official Babelfish documentation on migration strategies

---

## 🔧 Post-Migration Operations & Monitoring

### Aurora PostgreSQL Operations

**Best practices with Amazon Aurora PostgreSQL**
- https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html
- Avoiding slow performance, automatic restart, and failover scenarios

**Essential concepts for Aurora PostgreSQL tuning**
- https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Tuning.concepts.html
- Memory management, wait events, process architecture, WAL mechanism

**Aurora PostgreSQL Performance Optimization Techniques (PDF)**
- https://pages.awscloud.com/rs/112-TZM-766/images/DMWQ1D4S3T2 - Amazon Aurora Performance Optimization Techniques.pdf
- Performance tuning, extensions (pg_repack, pg_partman, pg_hint_plan), monitoring tools

### Monitoring Solutions

**Monitor the health of Amazon Aurora PostgreSQL instances in large-scale deployments**
- https://aws.amazon.com/blogs/database/monitor-the-health-of-amazon-aurora-postgresql-instances-in-large-scale-deployments/
- Open-source solution for fleet-wide monitoring with CloudWatch integration
- GitHub: https://github.com/aws-samples/monitoring-aurora-postgresql-health-large-scale-deployments

**Amazon Aurora Postgres Advanced Monitoring** (???)
- https://github.com/awslabs/amazon-aurora-postgres-monitoring
- Serverless Lambda-based monitoring with custom CloudWatch dashboards and metrics

**An Open Source Performance Monitoring Tool for Aurora PostgreSQL (Intuit AWR)** 
**NOTE** Not tested with Babelfish as of October 14, 2025

- https://medium.com/intuit-engineering/an-open-source-performance-monitoring-tool-for-aurora-postgresql-58fb8be21f64
- Automatic Workload Repository (AWR) for Aurora PostgreSQL with historical statistics

**Setting Up Database Monitoring for Aurora managed Postgres (Datadog)**
**NOTE** Not tested with Babelfish as of October 14, 2025

- https://docs.datadoghq.com/database_monitoring/setup_postgres/aurora/
- Third-party monitoring with deep query metrics and explain plans. Focuses on PostgreSQL queries, but should work with Babelfish on the TDS port

---

## 🚨 Performance Troubleshooting

### AWS Troubleshooting Guides

**Troubleshoot PostgreSQL performance issues**
- https://repost.aws/knowledge-center/rds-aurora-postgresql-performance-issues
- Slow queries, resource bottlenecks, query plan management, autovacuum tuning

**Resolve high CPU use for Amazon RDS or Aurora PostgreSQL**
- https://repost.aws/knowledge-center/rds-aurora-postgresql-high-cpu
- Using pg_stat_statements, pg_stat_activity, Enhanced Monitoring, Performance Insights

**Troubleshoot Amazon Aurora PostgreSQL Performance Issues by Scenario (Part 1)**
- https://medium.com/@oleksii.bebych/troubleshoot-amazon-aurora-postgresql-performance-issues-by-scenario-part-1-fe9daed97a32
- MultiXacts, idle connections, memory consumption issues, wait events

### Query Planning Resources

**Babelfish Query Planning Documentation**
- https://babelfishpg.org/docs/usage/query_planning/
- Understanding how Babelfish handles query plans compared to SQL Server

**Transition a pivot query that includes dynamic columns from SQL Server to PostgreSQL**
- https://aws.amazon.com/blogs/database/transition-a-pivot-query-that-includes-dynamic-columns-from-sql-server-to-postgresql/
- Handling PIVOT operations and dynamic columns in PostgreSQL

---

## 👨‍💻 Developer Transition Guides

### Language & Code Conversion

**Migrate SQL Server to Amazon Aurora PostgreSQL using best practices and lessons learned from the field**
- https://aws.amazon.com/blogs/database/migrate-sql-server-to-amazon-aurora-postgresql-using-best-practices-and-lessons-learned-from-the-field/
- Entity Framework Core migration, embedded queries, stored procedure conversion

**Best practices for migrating SQL Server MERGE statements to Babelfish for Aurora PostgreSQL**
- https://aws.amazon.com/blogs/database/best-practices-for-migrating-sql-server-merge-statements-to-babelfish-for-aurora-postgresql/
- Using Babelfish Compass -rewrite flag for automatic MERGE conversion

### Getting Started Courses

**Babelfish for AWS Aurora PostgreSQL - Hands on Learning (Udemy)**
- https://www.udemy.com/course/babelfish-for-aws-aurora-postgresql-hands-on-learning/
- Comprehensive video course on Babelfish migration and operations

---

## 👨‍🔧 DBA Transition Guides

### PostgreSQL Administration Basics

**PostgreSQL DBA Roadmap**
- https://roadmap.sh/postgresql-dba
- Community-driven learning path for PostgreSQL DBAs

**PostgreSQL Database Administration (DBA) for Beginners (Udemy)**
- https://www.udemy.com/course/postgresql-database-administration-dba-for-beginners/
- Foundational course covering installations, queries, table management

**PostgreSQL DBA Guide (Medium)**
- https://s0dade.medium.com/postgresql-dba-870c6b9d979b
- Step-by-step guide covering RDBMS concepts, indexes, query optimization

**PostgreSQL: Documentation: Part III. Server Administration**
- https://www.postgresql.org/docs/current/admin.html
- Official PostgreSQL documentation for administrators

### SQL Server to PostgreSQL Differences

**PostgreSQL for the SQL Server DBA: The First Four Settings to Check**
- https://www.softwareandbooz.com/postgresql-to-sql-server-the-first-four-settings/
- Critical memory settings: shared_buffers, work_mem, maintenance_work_mem, effective_cache_size

**SQL to SQL: A Practical Guide from a DBA**
- https://peter-whyte.com/2024/12/sql-to-sql-a-practical-guide/
- Practical tips for SQL Server DBAs working with PostgreSQL

---

## 🐟 Babelfish-Specific Resources

### Core Documentation

**Babelfish Official Documentation**
- https://babelfishpg.org/docs/usage/migration/
- Official migration guide and usage documentation

**Babelfish Compass Tool**
- https://github.com/babelfish-for-postgresql/babelfish_compass
- Compatibility assessment tool for Babelfish migrations

**Deep dive into Babelfish Compass**
- https://aws.amazon.com/blogs/database/deep-dive-into-babelfish-compass/
- Detailed explanation of assessment reports and migration planning

### Differences & Limitations

**Differences between Babelfish for Aurora PostgreSQL and SQL Server**
- Included in AWS documentation - understanding what's supported vs. not supported

---

## 🛠️ Tools & Utilities

### Migration Tools

**DBConvert**
- https://dbconvert.com/
- Commercial tool for SQL Server ↔ PostgreSQL with bidirectional sync

**pgloader**
- Open-source command-line tool for data migration

**Flyway (Redgate)**
- https://www.red-gate.com/products/flyway/community/
- Database migration engine supporting 50+ databases including SQL Server and PostgreSQL

### PostgreSQL Management Tools

**pgAdmin**
- Standard GUI tool for PostgreSQL management

**Redgate SQL Tools**
- https://www.red-gate.com/hub/product-learning/flyway/how-can-redgate-tools-help-with-a-cloud-migration
- SQL Monitor, Flyway Desktop, SQL Prompt work with PostgreSQL/Aurora

### Performance Analysis Tools

**pgBadger**
- https://github.com/darold/pgbadger
- PostgreSQL log analyzer with detailed reports and graphs

**pg_stat_statements**
- Native PostgreSQL extension for query performance tracking

**PGPerfStatsSnapper**
- Periodic collection of PostgreSQL performance statistics

---

## 🎓 Expert Perspectives

### Brent Ozar Articles

**Two Important Differences Between SQL Server and PostgreSQL**
- https://www.brentozar.com/archive/2018/08/two-important-differences-between-sql-server-and-postgresql/
- CTEs as optimization fences, lack of variables/batch processing

**As a SQL Server DBA, Postgres Backups Surprised Me**
- https://www.brentozar.com/archive/2024/08/as-a-sql-server-dba-postgres-backups-surprised-me/
- pg_dump vs pg_basebackup, third-party backup tools (Barman, PGBackrest)

**The Real Problem with SQL Server's Licensing Costs**
- https://www.brentozar.com/archive/2023/11/the-real-problem-with-sql-servers-licensing-costs/
- Discussion of migration motivations and PostgreSQL adoption

**SQL Server Features I'd Like To See, PostgreSQL Edition**
- https://www.brentozar.com/archive/2015/10/sql-server-features-id-like-to-see-postgresql-edition/
- Unlogged tables, generate_series, and other PostgreSQL advantages

**Announcing Google Managed PostgreSQL (and why SQL Server DBAs should care)**
- https://www.brentozar.com/archive/2017/03/announcing-google-managed-postgresql-sql-server-dbas-care/
- Cloud platform considerations for SQL Server DBAs

**Brent Ozar on Babelfish (Pluralsight Interview)**
- https://www.pluralsight.com/resources/blog/cloud/ozar-whats-the-future-of-microsoft-sql-server
- Discussion of Babelfish as competitive pressure and migration accelerator

**Smart Postgres**
- https://smartpostgres.com/about/
- Brent Ozar's PostgreSQL-focused resource site

**PostgreSQL Archives - Brent Ozar Unlimited**
- https://www.brentozar.com/archive/category/postgresql/
- Collection of PostgreSQL articles from SQL Server perspective

### Redgate Perspectives

**Getting Started with Flyway Migrations on PostgreSQL**
- https://www.red-gate.com/hub/product-learning/flyway/getting-started-with-flyway-migrations-on-postgresql
- Phil Factor's guide to using Flyway with PostgreSQL for SQL Server developers

**Redgate Tools for Migrating Databases To The Cloud**
- https://www.red-gate.com/hub/product-learning/flyway/how-can-redgate-tools-help-with-a-cloud-migration
- SQL Monitor, Flyway, and other tools for cloud migrations

**Getting an Overview of Changes to a PostgreSQL Database using Flyway**
- https://www.red-gate.com/hub/product-learning/flyway/getting-an-overview-of-changes-to-a-postgresql-database-using-flyway
- Branch-based development workflows with Flyway and PostgreSQL

---

## ✅ Best Practices & Patterns

### Code Conversion Patterns

**T-SQL to PL/pgSQL Conversions**
- Variables and batch processing differences
- WAITFOR alternatives (pg_sleep)
- MERGE statement workarounds
- Dynamic SQL and temp tables

### Performance Optimization

**Memory Configuration**
- shared_buffers: 25% of RAM (vs SQL Server's dynamic allocation)
- work_mem: Per-operation memory (watch for multiplication across connections)
- maintenance_work_mem: For vacuum and index operations
- effective_cache_size: Query planner hint (doesn't actually allocate memory)

**Autovacuum Tuning**
- Understanding dead tuples and bloat
- Adjusting autovacuum thresholds for high-traffic tables
- Monitoring vacuum operations with pg_stat_user_tables

**Query Plan Management**
- Aurora PostgreSQL query plan management (apg_plan_mgmt)
- Using EXPLAIN and auto_explain
- Index optimization strategies

### Application Architecture

**Dual-Endpoint Strategy**
- Legacy apps → TDS endpoint (port 1433) with T-SQL
- New development → PostgreSQL endpoint (port 5432) with PL/pgSQL
- Gradual migration approach

**Connection Pooling**
- PgBouncer for connection management
- Differences from SQL Server connection handling

**Schema Naming**
- Single-database mode: Direct schema mapping
- Multiple-database mode: dbname_schemaname pattern

---

## 🔗 Additional Resources

### Caylent Resources

**Caylent Accelerate™ with Database Evolution**
- https://caylent.com/blog/caylent-launches-database-evolution-powered-by-sql-polyglot-and-aws-generative-ai-to-accelerate-database-migrations
- AI-powered solution automating up to 70% of migration process
- **Note**: Check for polyglot-sandbox GitHub repository content (access may require Caylent credentials)

### AWS Training & Certification

- AWS Database Migration workshops
- Aurora PostgreSQL deep dive courses
- Babelfish hands-on labs

### Community Forums

- PostgreSQL mailing lists
- AWS Database Forum
- Stack Overflow (postgresql, amazon-aurora, babelfish tags)
- Reddit: r/PostgreSQL, r/aws

---

## 📊 Success Metrics & Case Studies

### Documented Success Stories

**CDL (Insurance Technology)**
- 94% code compatibility with Babelfish
- 93% cost reduction
- 3 months of engineering time saved
- Leveraged Aurora Serverless v2 for auto-scaling

**Diligent (GRC SaaS)**
- Thousands of developer hours saved
- Minimal code changes required
- Eliminated SQL Server licensing costs
- Aurora Serverless v2 integration

---

## 🎯 Building Your Offering

### Recommended Structure for Customer Training

**Week 1: Foundations**
- PostgreSQL architecture for SQL Server DBAs
- Key differences in memory management and processes
- Basic operations and command-line tools

**Week 2: Operations**
- Monitoring and alerting setup
- Backup and recovery strategies
- Performance baselining

**Week 3: Troubleshooting**
- Reading PostgreSQL logs
- Query performance analysis
- Wait event interpretation

**Week 4: Advanced Topics**
- Query plan management
- Vacuum tuning
- Index optimization

### Ongoing Support Services

**Monthly Health Checks**
- Review key metrics
- Identify performance regressions
- Proactive optimization recommendations

**Quarterly Deep Dives**
- Application-specific tuning
- Cost optimization analysis
- Feature utilization review

**Emergency Response**
- 24/7 on-call support for critical issues
- Performance troubleshooting
- Disaster recovery assistance

---

## 📝 Notes

This resource guide is organized for building a comprehensive post-migration support offering. The content covers:

✅ **Migration Phase**: Assessment, planning, and execution resources  
✅ **Operations Phase**: Day-to-day management and monitoring  
✅ **Optimization Phase**: Performance tuning and cost optimization  
✅ **Emergency Response**: Troubleshooting and issue resolution  

**Next Steps:**
1. Review Caylent's polyglot-sandbox repository for internal resources
2. Develop hands-on labs using these resources
3. Create customer-specific runbooks based on their application patterns
4. Build monitoring dashboards using the tools referenced
5. Establish escalation procedures for complex issues

**Last Updated**: October 2025
