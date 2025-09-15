Beginning with version 3.0 Babelfish supports writing server logs in JSON format.

To enable JSON logs it is necessary to set the following configuration parameter:

```sql
ALTER SYSTEM SET log_destination = 'jsonlog'
```

Multiple log destinations can be enabled at the same time:

```sql
ALTER SYSTEM SET log_destination = 'stderr,jsonlog'
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

Log file with JSON records can be loaded into DB table like this, with each record represented as a single `jsonb` value:

```sql
CREATE TABLE log_tuesday(j jsonb)
```

Loading can be done without any additional tools/scripts by connecting on Postgres port (default value `5432`) and using [COPY](https://www.postgresql.org/docs/16/sql-copy.html) SQL command like this:

```sql
COPY log_tuesday FROM '/absolute/path/to/data/log/postgresql-Tue.json' csv quote e'\x01' delimiter e'\x02'
```

`csv` format is used because there is (yet) no native support for JSON format in `COPY` command. But treating JSON log file as CSV and specifying non-existing quote and delimiter values we can read each JSON records as a single 'jsonb' field.

On Windows path to log file with backslashes can be used as is:

```sql
COPY log_tuesday FROM 'C:\Program Files\WiltonDB Software\wiltondb3.3\data\log\postgresql-Tue.json' csv quote e'\x01' delimiter e'\x02'
```

After records are loaded into DB table we can use [Postgres JSON funtions](https://www.postgresql.org/docs/16/functions-json.html) to query
particular JSON fields, for example:

```sql
SELECT j['pid'], j->>'message' FROM log_tuesday
WHERE j->>'remote_host' = '192.168.178.58'
```

|pid|msg|
|---|---|
|8332|Unmapped error found. Code: 1088, Message: 'MERGE' is not currently supported in Babelfish, File: tsqlIface.cpp, Line: 2761, Context: babelfishpg_tsql|
|8332|'MERGE' is not currently supported in Babelfish|

Line endings embedded in JSON strings can be replaced with actual line endings, for example, query all `ERROR` logs records written during last hour:

```sql
SELECT j['pid'], replace(j->>'message', '\r\n', e'\n') FROM log_tuesday
WHERE j->>'error_severity' = 'ERROR'
AND cast(j->>'timestamp' AS timestamp) > now() - INTERVAL '1 hour'
```