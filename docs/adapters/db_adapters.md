# Database Adapters

Datus Agent supports connecting to various databases through a plugin-based adapter system. This document explains the available adapters, how to install them, and how to configure your database connections.

## Overview

Datus uses a modular adapter architecture that allows you to connect to different databases:

- **Built-in Adapters**: SQLite and DuckDB are included with the core package
- **Plugin Adapters**: Additional databases (MySQL, Snowflake, StarRocks, etc.) can be installed as separate packages

This design keeps the core package lightweight while allowing you to add support for specific databases as needed.

## Supported Databases

| Database | Package | Installation | Status |
|----------|---------|-------------|--------|
| SQLite | Built-in | Included | Ready |
| DuckDB | Built-in | Included | Ready |
| MySQL | datus-mysql | `pip install datus-mysql` | Ready |
| StarRocks | datus-starrocks | `pip install datus-starrocks` | Ready |
| Snowflake | datus-snowflake | `pip install datus-snowflake` | Ready |
| ClickZetta | datus-clickzetta | `pip install datus-clickzetta` | Ready |

## Installation

### Built-in Databases

SQLite and DuckDB are included with Datus Agent and require no additional installation.

### Plugin Adapters

Install the adapter package for your database:

```bash
# MySQL
pip install datus-mysql

# Snowflake
pip install datus-snowflake

# StarRocks
pip install datus-starrocks

# ClickZetta
pip install datus-clickzetta
```

Once installed, Datus Agent will automatically detect and load the adapter.

## Configuration

Configure your database connection in the `config.yaml` file:

### SQLite

```yaml
namespace:
  mydata:
    type: sqlite
    uri: sqlite:///path/to/database.db
```

### DuckDB

```yaml
namespace:
  analytics:
    type: duckdb
    database: /path/to/database.duckdb
```

### MySQL

```yaml
namespace:
  production:
    type: mysql
    host: localhost
    port: 3306
    username: your_username
    password: your_password
    database: your_database
```

### Snowflake

```yaml
namespace:
  warehouse:
    type: snowflake
    account: your_account
    username: your_username
    password: your_password
    warehouse: your_warehouse
    database: your_database
    schema: your_schema
    role: your_role  # optional
```

### StarRocks

```yaml
namespace:
  analytics:
    type: starrocks
    host: localhost
    port: 9030
    username: root
    password: your_password
    database: your_database
```

### ClickZetta

```yaml
namespace:
  lakehouse:
    type: clickzetta
    host: your_host
    username: your_username
    password: your_password
    database: your_workspace
```

## Multiple Database Connections

You can configure multiple databases under the same namespace:

```yaml
namespace:
  project:
    source_db:
      type: mysql
      host: source-server
      username: reader
      password: password
      database: source
    target_db:
      type: snowflake
      account: your_account
      username: writer
      password: password
      warehouse: compute_wh
      database: target
```

## Features by Adapter

### Common Features

All adapters support:

- SQL query execution (SELECT, INSERT, UPDATE, DELETE)
- DDL operations (CREATE, ALTER, DROP)
- Metadata retrieval (tables, views, schemas)
- Sample data retrieval
- Connection pooling and timeout management

### Adapter-Specific Features

#### MySQL
- INFORMATION_SCHEMA queries
- SHOW CREATE TABLE/VIEW support
- Full CRUD operations

#### Snowflake
- Multi-database and schema support
- Tables, views, and materialized views
- Arrow format for efficient data transfer
- Native SDK integration

#### StarRocks
- Multi-Catalog support
- Materialized view support
- MySQL protocol compatibility

#### ClickZetta
- Workspace and schema management
- Volume/Stage file operations
- Native SDK integration

## Troubleshooting

### Adapter Not Found

If you see an error like `Connector 'mysql' not found`, make sure you have installed the corresponding adapter package:

```bash
pip install datus-mysql
```

### Connection Issues

Check the following:

1. **Network connectivity**: Ensure you can reach the database server
2. **Credentials**: Verify username and password are correct
3. **Port**: Confirm the correct port is specified
4. **Database name**: Ensure the database exists

### Driver Dependencies

Some adapters require additional system dependencies:

- **MySQL**: Requires `pymysql` (installed automatically)
- **Snowflake**: Requires `snowflake-connector-python` (installed automatically)

## Architecture

```
datus-agent (Core)
├── Built-in Adapters
│   ├── SQLite Connector
│   └── DuckDB Connector
│
└── Plugin System (Entry Points)
    ├── datus-sqlalchemy (Base layer)
    │   ├── datus-mysql
    │   └── datus-starrocks
    │
    └── Native SDK Adapters
        ├── datus-snowflake
        └── datus-clickzetta
```

The adapter system uses Python's entry points mechanism for automatic discovery. When you install an adapter package, it registers itself with Datus Agent and becomes available for use.

## Next Steps

- [Quick Start Guide](../quick_start.md) - Get started with Datus Agent
- [Configuration Reference](../configuration.md) - Detailed configuration options
