This folder covers the essentials for assessing your SQL Server solutions and things to consider with planning your modernization / migration project with Babelfish.

It all starts with understanding if Babelfish even makes sense. 

## Where Babelfish may be a non-starter or not a good fit
Here are the some things that should give you pause when considering a migration to Babelfish for PostgreSQL.
+ __You don't have access to the application source code.__ You will need to make some application changes to use Babelfish with existing SQL Server solutions. 
+ __You application absolutely relies on the limitations or unsupported features.__ AWS has a topic for [Babelfish limitations](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/babelfish-limitations.html). There are often other ways to workaround the limitations, but you will need to refactor parts of your application.
+ __Your application requires less than a few millisecond latency query execution when opening a new connection.__ This is a current limitation with Babelfish on Aurora as of version 16.2 and earlier. Hopefully it will get addressed in the near future. If not, you may need to take a hybrid approach of modifying your application code for this scenario to use a native PostgreSQL connection instead of using T-SQL with TDS.
+ __Your application, database, and operations are committed to making the project a success.__ I've seen failed migration attempts with Babelfish initiated by the database team or the application team where they lacked alignment.

## Babelfih Compass assessments
There is lots to cover here. However, you can get started by referring to these two blog posts:
+ [Migrate SQL Server to Babelfish for Aurora PostgreSQL using the Compass tool and AWS DMS](https://aws.amazon.com/blogs/database/migrate-sql-server-to-babelfish-for-aurora-postgresql-using-the-compass-tool-and-aws-dms/)
+ [Client-side T-SQL assessment for SQL Server to Babelfish for Aurora PostgreSQL migration](https://aws.amazon.com/blogs/database/client-side-t-sql-assessment-for-sql-server-to-babelfish-for-aurora-postgresql-migration/)

Of course, there is the GitHib repro for [Babelfish Compass](https://github.com/babelfish-for-postgresql/babelfish_compass) which gets updated with every release of Aurora PostgreSQL.

## Coming soon: Babelfish Compass command line assessment tips