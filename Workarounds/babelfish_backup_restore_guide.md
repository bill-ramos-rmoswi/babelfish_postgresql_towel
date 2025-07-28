# How to Backup and Restore Babelfish T-SQL Databases

Babelfish has specific versions of `pg_dump` to correctly handle Babelfish database roles and structures. These specialized tools are `bbf_dumpall` and `bbf_dump`, which include additional code to correctly backup and restore databases using standard PostgreSQL restore tools.

**Important:** You can only restore a database on a Babelfish cluster that is the same major and minor version as the source cluster, and the target database must not exist.

For detailed documentation, see the [Babelfish dump and restore wiki](https://github.com/babelfish-for-postgresql/babelfish-for-postgresql/wiki/Babelfish-dump-and-restore).

## Prerequisites

The easiest way to use the `bbf_dump` commands is to launch an Amazon EC2 instance with Amazon Linux 2023 (AL2023) using the Amazon EC2 console, as the tools come pre-built. Otherwise, you would need to compile them from source.

For development, I highly recommend using **Cursor IDE** with the **Remote - SSH extension** to connect to your EC2 instance. This provides a full IDE experience with integrated chat capabilities to interact with files and run scripts directly against the EC2 instance.

## Setup Development Environment

### 1. Connect to EC2 with Cursor Remote-SSH

1. Install Cursor IDE from [cursor.com](https://cursor.com)
2. Install the "Remote - SSH" extension
3. Configure SSH connection to your EC2 instance
4. Connect and enjoy a full IDE experience on your remote server

### 2. Configure Environment Variables

Create a configuration script to manage your database connection parameters securely:

```bash
# Create environment setup script
cat > ~/db_config.sh << 'EOF'
#!/bin/bash
# PostgreSQL/Babelfish connection parameters
export PSQL_EDITOR="code --wait"
export PGPORT="5432"
export PGDATABASE="babelfish_db"
export PGUSER="your_username"
export PGPASSWORD="your_password"
export PGHOST="your-babelfish-cluster.cluster-xxxxxx.region.rds.amazonaws.com"

# Target cluster for restore (if different)
export TARGET_PGHOST="your-target-cluster.cluster-xxxxxx.region.rds.amazonaws.com"
EOF

# Make executable and source
chmod +x ~/db_config.sh
source ~/db_config.sh
```

### 3. Create Backup Script

```bash
#!/bin/bash
# backup_babelfish.sh - Backup Babelfish T-SQL Database

show_help() {
    cat << EOF
backup_babelfish.sh - Backup Babelfish T-SQL Database

USAGE:
    $(basename "$0") [OPTIONS] BBF_DATABASE_NAME

DESCRIPTION:
    Creates a backup of a Babelfish T-SQL database using bbf_dumpall and bbf_dump.
    Files are organized in a structured directory format under \$HOME/bbf_backups/.

ARGUMENTS:
    BBF_DATABASE_NAME   Name of the Babelfish T-SQL database to backup

OPTIONS:
    -h, --help          Show this help message and exit
    -t, --time          Show timing information for backup operations

DIRECTORY STRUCTURE:
    \$HOME/bbf_backups/
    ├── database_name/
    │   └── YYYY-MM-DD_HHMM/
    │       ├── database_name.tar   (bbf_dump output)
    │       └── database_name.pgsql (bbf_dumpall output)

ENVIRONMENT VARIABLES:
    PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD

EXAMPLES:
    $(basename "$0") northwind
    $(basename "$0") --time rainbowtreecarev3
    $(basename "$0") -h

EXIT CODES:
    0    Success
    1    General error
    2    Missing required arguments
    3    Database connection failed
    4    Database does not exist

EOF
}

# Parse command line arguments
USE_TIME=false
BBF_DATABASE_NAME=""

while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            show_help
            exit 0
            ;;
        -t|--time)
            USE_TIME=true
            shift
            ;;
        -*)
            echo "Error: Unknown option $1" >&2
            show_help
            exit 2
            ;;
        *)
            if [[ -z "$BBF_DATABASE_NAME" ]]; then
                BBF_DATABASE_NAME="$1"
            else
                echo "Error: Multiple database names specified" >&2
                show_help
                exit 2
            fi
            shift
            ;;
    esac
done

# Validate required arguments
if [[ -z "$BBF_DATABASE_NAME" ]]; then
    echo "Error: BBF_DATABASE_NAME is required" >&2
    show_help
    exit 2
fi

# Source environment variables
if [[ -f ~/db_config.sh ]]; then
    source ~/db_config.sh
else
    echo "Warning: ~/db_config.sh not found. Ensure environment variables are set."
fi

# Validate required environment variables
if [[ -z "$PGHOST" || -z "$PGPORT" || -z "$PGDATABASE" || -z "$PGUSER" ]]; then
    echo "Error: Required environment variables not set (PGHOST, PGPORT, PGDATABASE, PGUSER)" >&2
    show_help
    exit 2
fi

# Configuration
BACKUP_DATE=$(date +%Y-%m-%d_%H%M)
BACKUP_BASE_DIR="$HOME/bbf_backups"
BACKUP_DIR="${BACKUP_BASE_DIR}/${BBF_DATABASE_NAME}/${BACKUP_DATE}"
ROLES_FILE="${BACKUP_DIR}/${BBF_DATABASE_NAME}.pgsql"
DB_FILE="${BACKUP_DIR}/${BBF_DATABASE_NAME}.tar"

# Time command setup
TIME_CMD=""
if [[ "$USE_TIME" == true ]]; then
    TIME_CMD="time"
fi

echo "Starting backup of Babelfish database: $BBF_DATABASE_NAME"
echo "Backup directory: $BACKUP_DIR"
echo "Target host: $PGHOST:$PGPORT"

# Test database connectivity
echo "Testing database connection..."
if ! psql --host="$PGHOST" --port="$PGPORT" --dbname="$PGDATABASE" --username="$PGUSER" -c "SELECT 1;" >/dev/null 2>&1; then
    echo "✗ Cannot connect to database server" >&2
    echo ""
    show_help
    exit 3
fi

# Check if BBF database exists
echo "Checking if database '$BBF_DATABASE_NAME' exists..."
DB_EXISTS=$(psql --host="$PGHOST" --port="$PGPORT" --dbname="$PGDATABASE" --username="$PGUSER" -t -c "SELECT name FROM sys.databases WHERE name = '$BBF_DATABASE_NAME';" 2>/dev/null | tr -d ' ')

if [[ -z "$DB_EXISTS" ]]; then
    echo "✗ Database '$BBF_DATABASE_NAME' does not exist" >&2
    echo ""
    echo "Available databases:"
    psql --host="$PGHOST" --port="$PGPORT" --dbname="$PGDATABASE" --username="$PGUSER" -c "SELECT name AS \"Available databases\" FROM sys.databases WHERE database_id > 4;" 2>/dev/null
    exit 4
fi

echo "✓ Database '$BBF_DATABASE_NAME' found"

# Create backup directory
mkdir -p "$BACKUP_DIR"
if [[ $? -ne 0 ]]; then
    echo "✗ Failed to create backup directory: $BACKUP_DIR" >&2
    exit 1
fi

# Step 1: Backup T-SQL roles with bbf_dumpall
echo ""
echo "Step 1: Backing up T-SQL roles and schema..."
echo "Output file: $ROLES_FILE"

$TIME_CMD bbf_dumpall --database "$PGDATABASE" \
                      --host="$PGHOST" \
                      --port="$PGPORT" \
                      --username "$PGUSER" \
                      --bbf-database-name="$BBF_DATABASE_NAME" \
                      --roles-only \
                      --quote-all-identifiers \
                      --verbose \
                      --no-role-passwords \
                      -f "$ROLES_FILE"

if [[ $? -eq 0 ]]; then
    echo "✓ Roles backup completed: $ROLES_FILE"
    echo "  File size: $(du -h "$ROLES_FILE" | cut -f1)"
else
    echo "✗ Roles backup failed" >&2
    exit 1
fi

# Step 2: Backup database contents with bbf_dump
echo ""
echo "Step 2: Backing up database contents..."
echo "Output file: $DB_FILE"

$TIME_CMD bbf_dump --dbname="$PGDATABASE" \
                   --host="$PGHOST" \
                   --port="$PGPORT" \
                   --username "$PGUSER" \
                   --bbf-database-name="$BBF_DATABASE_NAME" \
                   --quote-all-identifiers \
                   --verbose \
                   --file="$DB_FILE" \
                   --format=tar

if [[ $? -eq 0 ]]; then
    echo "✓ Database backup completed: $DB_FILE"
    echo "  File size: $(du -h "$DB_FILE" | cut -f1)"
else
    echo "✗ Database backup failed" >&2
    exit 1
fi

# Summary
echo ""
echo "=== Backup Summary ==="
echo "Database: $BBF_DATABASE_NAME"
echo "Backup directory: $BACKUP_DIR"
echo "Files created:"
echo "  - Roles/Schema: $ROLES_FILE ($(du -h "$ROLES_FILE" | cut -f1))"
echo "  - Data/Objects: $DB_FILE ($(du -h "$DB_FILE" | cut -f1))"
echo "Total backup size: $(du -sh "$BACKUP_DIR" | cut -f1)"
echo ""
echo "✓ Backup completed successfully"
```

### 4. Create Restore Script

```bash
#!/bin/bash
# restore_babelfish.sh - Restore Babelfish T-SQL Database

show_help() {
    cat << EOF
restore_babelfish.sh - Restore Babelfish T-SQL Database

USAGE:
    $(basename "$0") [OPTIONS] BBF_DATABASE_NAME

DESCRIPTION:
    Restores a Babelfish T-SQL database from backup files. If -r and -d options
    are not specified, the script will automatically find the latest backup
    for the specified database name.

ARGUMENTS:
    BBF_DATABASE_NAME   Name of the Babelfish T-SQL database to restore

OPTIONS:
    -h, --help              Show this help message and exit
    -r, --roles-file FILE   Path to roles pgsql file (optional)
    -d, --database-file FILE Path to database tar file (optional)
    -t, --time              Show timing information for restore operations
    --target-host HOST      Target host (uses TARGET_PGHOST env var if not specified)

DIRECTORY STRUCTURE (for auto-discovery):
    \$HOME/bbf_backups/
    ├── database_name/
    │   └── YYYY-MM-DD_HHMM/
    │       ├── database_name.tar   (bbf_dump output)
    │       └── database_name.pgsql (bbf_dumpall output)

ENVIRONMENT VARIABLES:
    PGPORT, PGDATABASE, PGUSER, PGPASSWORD, TARGET_PGHOST

EXAMPLES:
    $(basename "$0") northwind
    $(basename "$0") --time --target-host=target.cluster.amazonaws.com northwind
    $(basename "$0") -r backup.pgsql -d backup.tar northwind

EXIT CODES:
    0    Success
    1    General error
    2    Missing required arguments
    3    Database connection failed
    4    Database already exists
    5    Backup files not found
    6    Version mismatch error

EOF
}

# Parse command line arguments
USE_TIME=false
BBF_DATABASE_NAME=""
ROLES_FILE=""
DB_FILE=""
RESTORE_HOST=""

while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            show_help
            exit 0
            ;;
        -r|--roles-file)
            ROLES_FILE="$2"
            shift 2
            ;;
        -d|--database-file)
            DB_FILE="$2"
            shift 2
            ;;
        -t|--time)
            USE_TIME=true
            shift
            ;;
        --target-host)
            RESTORE_HOST="$2"
            shift 2
            ;;
        -*)
            echo "Error: Unknown option $1" >&2
            show_help
            exit 2
            ;;
        *)
            if [[ -z "$BBF_DATABASE_NAME" ]]; then
                BBF_DATABASE_NAME="$1"
            else
                echo "Error: Multiple database names specified" >&2
                show_help
                exit 2
            fi
            shift
            ;;
    esac
done

# Validate required arguments
if [[ -z "$BBF_DATABASE_NAME" ]]; then
    echo "Error: BBF_DATABASE_NAME is required" >&2
    show_help
    exit 2
fi

# Source environment variables
if [[ -f ~/db_config.sh ]]; then
    source ~/db_config.sh
else
    echo "Warning: ~/db_config.sh not found. Ensure environment variables are set."
fi

# Set target host
RESTORE_HOST=${RESTORE_HOST:-$TARGET_PGHOST}
if [[ -z "$RESTORE_HOST" ]]; then
    echo "Error: Target host not specified. Use --target-host option or set TARGET_PGHOST environment variable" >&2
    show_help
    exit 2
fi

# Validate required environment variables
if [[ -z "$PGPORT" || -z "$PGDATABASE" || -z "$PGUSER" ]]; then
    echo "Error: Required environment variables not set (PGPORT, PGDATABASE, PGUSER)" >&2
    show_help
    exit 2
fi

# Auto-discover backup files if not specified
if [[ -z "$ROLES_FILE" || -z "$DB_FILE" ]]; then
    BACKUP_BASE_DIR="$HOME/bbf_backups"
    DB_BACKUP_DIR="$BACKUP_BASE_DIR/$BBF_DATABASE_NAME"
    
    if [[ ! -d "$DB_BACKUP_DIR" ]]; then
        echo "Error: No backup directory found for database '$BBF_DATABASE_NAME'" >&2
        echo "Expected: $DB_BACKUP_DIR" >&2
        exit 5
    fi
    
    # Find latest backup (newest date_time folder)
    LATEST_BACKUP=$(find "$DB_BACKUP_DIR" -maxdepth 1 -type d -name "????-??-??_????" | sort -r | head -n1)
    
    if [[ -z "$LATEST_BACKUP" ]]; then
        echo "Error: No backup folders found in $DB_BACKUP_DIR" >&2
        exit 5
    fi
    
    ROLES_FILE="$LATEST_BACKUP/${BBF_DATABASE_NAME}.pgsql"
    DB_FILE="$LATEST_BACKUP/${BBF_DATABASE_NAME}.tar"
    
    echo "Auto-discovered backup files:"
    echo "  Backup date: $(basename "$LATEST_BACKUP")"
    echo "  Roles file: $ROLES_FILE"
    echo "  Database file: $DB_FILE"
fi

# Verify files exist
if [[ ! -f "$ROLES_FILE" ]]; then
    echo "Error: Roles file not found: $ROLES_FILE" >&2
    exit 5
fi

if [[ ! -f "$DB_FILE" ]]; then
    echo "Error: Database file not found: $DB_FILE" >&2
    exit 5
fi

# Time command setup
TIME_CMD=""
if [[ "$USE_TIME" == true ]]; then
    TIME_CMD="time"
fi

echo "Starting restore of Babelfish database: $BBF_DATABASE_NAME"
echo "Target host: $RESTORE_HOST:$PGPORT"
echo "Roles file: $ROLES_FILE ($(du -h "$ROLES_FILE" | cut -f1))"
echo "Database file: $DB_FILE ($(du -h "$DB_FILE" | cut -f1))"
echo ""

# Test target database connectivity
echo "Testing target database connection..."
if ! psql --host="$RESTORE_HOST" --port="$PGPORT" --dbname="$PGDATABASE" --username="$PGUSER" -c "SELECT 1;" >/dev/null 2>&1; then
    echo "✗ Cannot connect to target database server" >&2
    echo ""
    show_help
    exit 3
fi

# Check if target is a Babelfish cluster and if database already exists
echo "Checking target cluster and database status..."
DB_CHECK_RESULT=$(psql --host="$RESTORE_HOST" --port="$PGPORT" --dbname="$PGDATABASE" --username="$PGUSER" -t -c "SELECT name FROM sys.databases WHERE name = '$BBF_DATABASE_NAME';" 2>&1)

if echo "$DB_CHECK_RESULT" | grep -q "relation.*sys.databases.*does not exist"; then
    echo "✗ Target server is not a Babelfish cluster (sys.databases not found)" >&2
    exit 3
elif echo "$DB_CHECK_RESULT" | grep -q "ERROR\|FATAL"; then
    echo "✗ Error checking target database: $DB_CHECK_RESULT" >&2
    exit 3
fi

# Clean the result and check if database exists
DB_EXISTS=$(echo "$DB_CHECK_RESULT" | tr -d ' \n')
if [[ -n "$DB_EXISTS" ]]; then
    echo "✗ Database '$BBF_DATABASE_NAME' already exists on target cluster" >&2
    echo ""
    echo "Current user databases on target:"
    psql --host="$RESTORE_HOST" --port="$PGPORT" --dbname="$PGDATABASE" --username="$PGUSER" -c "SELECT name AS \"Current user databases\" FROM sys.databases WHERE database_id > 4;" 2>/dev/null
    exit 4
fi

echo "✓ Target cluster is Babelfish-enabled"
echo "✓ Database '$BBF_DATABASE_NAME' does not exist on target (good)"

# Step 1: Apply roles to target cluster
echo ""
echo "Step 1: Applying T-SQL roles to target cluster..."

if [[ "$USE_TIME" == true ]]; then
    RESTORE_OUTPUT=$(time psql --host="$RESTORE_HOST" \
                              --port="$PGPORT" \
                              --dbname="$PGDATABASE" \
                              --username="$PGUSER" \
                              --single-transaction \
                              --file "$ROLES_FILE" 2>&1)
else
    RESTORE_OUTPUT=$(psql --host="$RESTORE_HOST" \
                          --port="$PGPORT" \
                          --dbname="$PGDATABASE" \
                          --username="$PGUSER" \
                          --single-transaction \
                          --file "$ROLES_FILE" 2>&1)
fi

RESTORE_EXIT_CODE=$?

# Check for version mismatch errors
if echo "$RESTORE_OUTPUT" | grep -q "RAISE"; then
    echo "✗ Version mismatch or compatibility error during restore:" >&2
    echo "$RESTORE_OUTPUT" >&2
    exit 6
elif [[ $RESTORE_EXIT_CODE -ne 0 ]]; then
    echo "✗ Failed to apply roles" >&2
    echo "$RESTORE_OUTPUT" >&2
    exit 1
else
    echo "✓ Roles applied successfully"
fi

# Step 2: Restore database objects and data
echo ""
echo "Step 2: Restoring database objects and data..."

if [[ "$USE_TIME" == true ]]; then
    RESTORE_OUTPUT=$(time pg_restore --host="$RESTORE_HOST" \
                                    --port="$PGPORT" \
                                    -d "$PGDATABASE" \
                                    -U "$PGUSER" \
                                    --verbose \
                                    "$DB_FILE" 2>&1)
else
    RESTORE_OUTPUT=$(pg_restore --host="$RESTORE_HOST" \
                               --port="$PGPORT" \
                               -d "$PGDATABASE" \
                               -U "$PGUSER" \
                               --verbose \
                               "$DB_FILE" 2>&1)
fi

RESTORE_EXIT_CODE=$?

# Check for version mismatch errors
if echo "$RESTORE_OUTPUT" | grep -q "RAISE"; then
    echo "✗ Version mismatch or compatibility error during restore:" >&2
    echo "$RESTORE_OUTPUT" >&2
    exit 6
elif [[ $RESTORE_EXIT_CODE -ne 0 ]]; then
    echo "✗ Database restore failed" >&2
    echo "$RESTORE_OUTPUT" >&2
    exit 1
else
    echo "✓ Database restore completed successfully"
fi

# Final verification
echo ""
echo "Verifying restored database..."
VERIFY_RESULT=$(psql --host="$RESTORE_HOST" --port="$PGPORT" --dbname="$PGDATABASE" --username="$PGUSER" -t -c "SELECT name FROM sys.databases WHERE name = '$BBF_DATABASE_NAME';" 2>/dev/null | tr -d ' ')

if [[ "$VERIFY_RESULT" == "$BBF_DATABASE_NAME" ]]; then
    echo "✓ Database '$BBF_DATABASE_NAME' successfully created on target cluster"
else
    echo "⚠ Warning: Could not verify database creation"
fi

# Summary
echo ""
echo "=== Restore Summary ==="
echo "Database: $BBF_DATABASE_NAME"
echo "Target host: $RESTORE_HOST"
echo "Source files:"
echo "  - Roles: $ROLES_FILE"
echo "  - Data: $DB_FILE"
echo "Status: SUCCESS"
echo ""
echo "✓ Restore completed successfully"
```

## Usage Examples

### Backup a Database

```bash
# Make scripts executable
chmod +x backup_babelfish.sh restore_babelfish.sh

# Basic backup
./backup_babelfish.sh northwind

# Backup with timing information
./backup_babelfish.sh --time rainbowtreecarev3

# Show help
./backup_babelfish.sh --help
```

**Expected directory structure after backup:**
```
$HOME/bbf_backups/
├── northwind/
│   └── 2025-07-27_1430/
│       ├── northwind.tar
│       └── northwind.pgsql
└── rainbowtreecarev3/
    └── 2025-07-27_1445/
        ├── rainbowtreecarev3.tar
        └── rainbowtreecarev3.pgsql
```

### Restore a Database

```bash
# Auto-discover latest backup (recommended)
./restore_babelfish.sh northwind

# Auto-discover with timing info and specific target
./restore_babelfish.sh --time --target-host=target-cluster.amazonaws.com northwind

# Manual file specification
./restore_babelfish.sh -r /path/to/roles.pgsql -d /path/to/database.tar northwind

# Show help
./restore_babelfish.sh --help
```

### List Available Backups

```bash
# View backup structure
find $HOME/bbf_backups -type f -name "*.tar" -o -name "*.pgsql" | sort

# View backup sizes
du -sh $HOME/bbf_backups/*/
```

## Manual Commands (Alternative)

If you prefer running commands manually:

### Backup Commands

```bash
# Source environment
source ~/db_config.sh

# Backup roles
bbf_dumpall --database $PGDATABASE \
            --host=$PGHOST \
            --port=$PGPORT \
            --username $PGUSER \
            --bbf-database-name=northwind \
            --roles-only \
            --quote-all-identifiers \
            --verbose \
            --no-role-passwords \
            -f northwind-roles-$(date +%Y%m%d).sql

# Backup database
bbf_dump --dbname=$PGDATABASE \
         --host=$PGHOST \
         --port=$PGPORT \
         --username $PGUSER \
         --bbf-database-name=northwind \
         --quote-all-identifiers \
         --verbose \
         --file=northwind-db-$(date +%Y%m%d).tar \
         --format=tar
```

### Restore Commands

```bash
# Apply roles
psql --host=$TARGET_PGHOST \
     --port=$PGPORT \
     --dbname=$PGDATABASE \
     --username $PGUSER \
     --single-transaction \
     --file northwind-roles-20240127.sql

# Restore database
pg_restore --host=$TARGET_PGHOST \
           --port=$PGPORT \
           -d $PGDATABASE \
           -U $PGUSER \
           --verbose \
           northwind-db-20240127.tar
```

## Security Best Practices

1. **Store credentials securely**: Never hardcode passwords in scripts. Use environment variables or AWS Secrets Manager.

2. **Secure backup files**: Ensure backup files are stored with appropriate permissions and consider encryption for sensitive data.

3. **Network security**: Use VPC security groups to restrict database access to authorized EC2 instances only.

4. **Audit access**: Enable CloudTrail and RDS logging to track database access and operations.

## Troubleshooting

- **Version compatibility**: Ensure source and target Babelfish clusters are the same major.minor version
- **Database exists error**: The target database must not exist before restore
- **Permission errors**: Verify the user has sufficient privileges for backup/restore operations
- **Network connectivity**: Ensure EC2 instance can reach both source and target RDS clusters

## Conclusion

Using Cursor IDE with Remote-SSH provides an excellent development experience for managing Babelfish database operations. The combination of environment variables, reusable scripts, and visual file management makes database backup and restore operations both secure and efficient.

This approach allows you to maintain multiple environment configurations, automate routine backup tasks, and leverage Cursor's AI chat capabilities to assist with script development and troubleshooting.
