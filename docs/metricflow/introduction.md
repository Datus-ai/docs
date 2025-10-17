# Datus-MetricFlow Introduction

> **Note**: This project is forked from [dbt-labs/metricflow](https://github.com/dbt-labs/metricflow) version 0.140.0 and continues development from there. We are grateful to the dbt Labs team for their excellent work on the original MetricFlow project.

## What is MetricFlow?

MetricFlow is a semantic layer that helps you organize and manage metric definitions in code. It automatically generates clean, reusable SQL queries from your metric definitions, ensuring consistent metric outputs across your organization.

Simply put, MetricFlow allows you to:
- Define your business metrics once in a standardized way
- Query those metrics consistently across different dimensions
- Get automatically generated SQL that's optimized for your data warehouse.

## Key Features

### Metric Management
- **Centralized Definitions**: Define all metrics in one place using simple YAML configuration
- **Multiple Metric Types**: Support for simple metrics, ratios, expressions, and cumulative metrics
- **Dimension Analysis**: Break down metrics by any dimension (time, geography, product category, etc.)

### Smart Query Generation
- **Automatic SQL Generation**: Converts metric requests into optimized SQL queries
- **Multi-hop Joins**: Handles complex joins between fact tables and dimension tables
- **Time Granularity**: Aggregate metrics at different time periods (daily, weekly, monthly, etc.)

### Integration & Flexibility
- **Multiple Data Warehouses**: Works with various data platforms
- **API Integration**: Build custom integrations with downstream tools
- **Version Control**: Manage metric definitions in Git like regular code

## Supported Systems

### Data Warehouses

datus-metricflow currently supports:

- **DuckDB** - Embedded analytical database (great for demos and local development)
- **SQLite** - Lightweight embedded database
- **StarRocks** - High-performance analytical database

## Enhancements in datus-metricflow

Building on MetricFlow 0.140.0, we have made the following improvements:

- **Python 3.12 Support**: Upgraded to support the latest Python version
- **Additional Database Support**: Added SQLite and StarRocks adapters
- **Datus Integration**: Seamlessly integrated with the Datus project for unified configuration
- **Standalone Package**: Can be installed as a dependency and used independently
- **MCP Server**: Enables LLM applications to query metrics through the Model Context Protocol

## Installing datus-metricflow

### Prerequisites
Make sure you have Python 3.12 installed on your system.

### Installation Steps

1. **Install via pip**
   ```bash
   pip install datus-metricflow
   ```

2. **Setup with Your Database**

   If you have a Datus configuration file with database connections:
   ```bash
   datus-mf setup --namespace <your_namespace>
   ```

   This will automatically generate an env_settings.yml file for Metricflow MCP server from the Datus configuration.

3. **Load Environment Variables**

   If you need run native metricflow command like `mf list-metrics` or `mf query`, you need to load the environment variables for current shell:
   ```
   source ~/.datus/metricflow/datus_env.sh
   ```

## What Happens After Setup?

The setup process creates:

- A semantic models directory at `~/.metricflow/semantic_models/` where you define your metrics
- Configuration files that store your database connection settings
- Environment variables needed to connect to your data warehouse

## Next Steps

After installation:

1. **Define Your Metrics**: Create YAML files in the semantic models directory or use subagents to define your business metrics
2. **Query Your Metrics**: Use the `mf query` command or integrate with Datus Agent to query your metrics
3. **Integrate with Tools**: Use the MCP server to connect MetricFlow with LLM applications
