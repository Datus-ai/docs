---
title: "Installation"
description: "Set up the Datus Agent on your system in just a few minutes."
---

# Installation

Follow these steps to install the Datus Agent locally and set up your environment.

## Step 1: Prepare Your Environment

### Install Python 3.12

Datus requires a Python 3.12 environment. Choose your preferred method below:

=== "Conda"

    ```bash
    conda create -n datus python=3.12
    conda activate datus
    ```

=== "virtualenv"

    ```bash
    virtualenv datus --python=python3.12
    source datus/bin/activate
    ```

=== "uv"

    ```bash
    uv venv datus --python 3.12
    source datus/bin/activate
    ```

### Install the Datus Agent

=== "Stable Release"

    ```bash
    pip install datus-agent
    ```

=== "Beta Release"

    ```bash
    pip install --no-deps -i https://test.pypi.org/simple/ datus-agent
    ```

## Step 2: Initialize the Configuration

### First-Time Setup

Run the initialization command:

```bash
datus-agent init
```

### Customize Your Environment

The setup process will guide you through configuring various components:

#### 1. LLM Configuration

Configure your preferred LLM provider:

```bash
- Which LLM provider? [openai/deepseek/claude/kimi/qwen] (deepseek): kimi
- Enter your API key:
- Enter your base URL (https://api.deepseek.com):
- Enter your model name (deepseek-chat):
→ Testing LLM connectivity...
✔ LLM model test successful
```

#### 2. Namespace Setup

Set up the database connection. For example, `duckdb-demo.duckdb` is a demo database provided with the project:



```bash
- Namespace name: duckdb-demo
- Database type [sqlite/duckdb/snowflake/mysql/postgresql/starrocks] (starrocks):duckdb
- Connection string: ~/.datus/sample/duckdb-demo.duckdb
→ Testing database connectivity...
✔ Database connection test successful
```

!!! tip "Demo Database Connection"
    Datus provides a pre-configured demo DuckDB database for testing and learning purposes. 
    
    **Connection string:** `~/.datus/sample/duckdb-demo.duckdb`
    
    This demo database contains sample tables and data that you can use to explore Datus features without setting up your own database. It's automatically available after running `datus-agent init`.

#### 3. Workspace Configuration

Configure the workspace root directory (where your SQL files are stored):

```bash
- Workspace path (～/.datus/workspace):
✔ Workspace directory created
```

#### 4. Knowledge Base (Optional)

Initialize the knowledge base for metadata and SQL history:

```bash
- Initialize vector DB for metadata? [y/n] (n): y
→ Initializing metadata knowledge base...
→ Processed 17 tables with 17 sample records
✔ Metadata knowledge base initialized
- Initialize SQL history from workspace? [y/n] (n):
```

#### 5. Configuration Summary

After setup, you'll see a summary like this:

| Setting   | Value                        |
|-----------|------------------------------|
| LLM       | kimi (kimi-k2-turbo-preview) |
| Namespace | duckdb-demo                  |
| Workspace | /Users/username/.datus/workspace |

You're now ready to run `datus-cli --namespace duckdb-demo`

## Step 3: Launch the Datus Agent

### Run Datus CLI

Start the Datus CLI with your configured namespace:

```bash
datus-cli --namespace duckdb-demo
```

You should see output similar to:

```
Datus - AI-powered SQL command-line interface
Type '.help' for a list of commands or '.exit' to quit.

Namespace duckdb-demo selected
Connected to duckdb using database duckdb
Context: Current: database: duckdb
Type SQL statements or use ! @ . commands to interact.
Datus>
```

## Next Steps

Follow the [Quickstart guide](./Quickstart.md) to begin your experience with Datus Agent.