[log_fdw](https://github.com/aws/postgresql-logfdw) is a [Foreign Data Wrapper](https://www.postgresql.org/docs/current/ddl-foreign-data.html) that allows to run SQL queries against Postgres log files stored in file system.

To install `log_fdw` extension refer to it's [installation instructions](https://github.com/aws/postgresql-logfdw?tab=readme-ov-file#quick-install-instructions).

Enable both plain text and CSV log output setting the following configuration parameters:

```sql
ALTER SYSTEM SET logging_collector='on'
ALTER SYSTEM SET log_filename='postgresql-%a.log'
ALTER SYSTEM SET log_destination='stderr,csvlog'
```

Logging verbosity can be configured using the following configuration parameters ([reference](https://www.postgresql.org/docs/current/runtime-config-logging.html#RUNTIME-CONFIG-LOGGING-WHAT)):

 - `log_checkpoints`
 - `log_connections`
 - `log_duration`
 - `log_error_verbosity`
 - `log_line_prefix`
 - `log_lock_waits`
 - `log_statement`

The following Babelfish-specific configuration parameter allows to log details about TDS connections (can be set to values from `0` to `3`):

 - `babelfishpg_tds.tds_debug_log_level`

Create the extension inside the `postgres` database (note: Babelfish physical database cannot be used to create this extension inside it):

```sql
CREATE EXTENSION log_fdw
```

List available log files:

```sql
SELECT * FROM list_postgres_log_files()
```

|**file_name**|**file_size_bytes**|
|-------------|-------------------|
|postgresql-Tue.log|1423|
|postgresql-Tue.csv|2491|

Create FDW server required to read the files:

```sql
CREATE SERVER log_fdw_server FOREIGN DATA WRAPPER log_fdw
```

Crete foreign tables for both plain-text and CSV files:

```sql
SELECT * FROM create_foreign_table_for_log_file('postgresql_tue_log','log_fdw_server','postgresql-Tue.log')
SELECT * FROM create_foreign_table_for_log_file('postgresql_tue_csv','log_fdw_server','postgresql-Tue.csv')
```

Run queries against these tables that will read live log records from file system:

```sql
SELECT * FROM postgresql_tue_log
```

|**log_entry**|
|-------------|
|2024-07-02 19:43:28.646 IST [45806] LOG:  starting PostgreSQL 16.3 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 13.3.1 20240522 (Red Hat 13.3.1-1), 64-bit|
|2024-07-02 19:43:28.646 IST [45806] LOG:  listening on IPv4 address 0.0.0.0, port 5432|

```sql
SELECT * FROM postgresql_tue_csv
```

|**log_time**|**user_name**|**database_name**|**process_id**|**connection_from**|**session_id**|**session_line_num**|**command_tag**|**session_start_time**|**virtual_transaction_id**|**transaction_id**|**error_severity**|**sql_state_code**|**message**|**detail**|**hint**|**internal_query**|**internal_query_pos**|**context**|**query**|**query_pos**|**location**|**application_name**|**backend_type**|**leader_pid**|**query_id**|
|------------|-------------|-----------------|--------------|-------------------|--------------|--------------------|---------------|----------------------|--------------------------|------------------|------------------|------------------|-----------|----------|--------|------------------|----------------------|-----------|---------|-------------|------------|--------------------|----------------|--------------|------------|
|2024-07-02 18:43:28.646+01|NULL|NULL|45806|NULL|66844a50.b2ee|1|NULL|2024-07-02 18:43:28+01|NULL|0|LOG|00000|starting PostgreSQL 16.3 on x86_64-pc-linux-gnu| compiled by gcc (GCC) 13.3.1 20240522 (Red Hat 13.3.1-1)| 64-bit|NULL|NULL|NULL|NULL|NULL|NULL|NULL|NULL|postmaster|NULL|0|
|2024-07-02 18:43:28.646+01|NULL|NULL|45806|NULL|66844a50.b2ee|2|NULL|2024-07-02 18:43:28+01|NULL|0|LOG|00000|listening on IPv4 address |0.0.0.0| port 5432|NULL|NULL|NULL|NULL|NULL|NULL|NULL|NULL|postmaster|NULL|0|

For example, query all `ERROR` logs records written during last hour:

```sql
SELECT * FROM postgresql_tue_csv
WHERE error_severity = 'ERROR'
AND log_time > NOW() - INTERVAL '1 hour'
ORDER BY log_time DESC
```

Because log records are read straight from file system, for repetead queries to large number of records it may be more performant to copy the records into ordinary DB table first and create appropriate indices.