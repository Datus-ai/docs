# Semantic Model Generation Guide

## Overview

The semantic model generation feature helps you create MetricFlow semantic models from database tables through an AI-powered assistant. The assistant analyzes your table structure and generates comprehensive YAML configuration files that define metrics, dimensions, and relationships.

## What is a Semantic Model?

A semantic model is a YAML configuration that defines:
- **Measures**: Metrics and aggregations (SUM, COUNT, AVERAGE, etc.)
- **Dimensions**: Categorical and time-based attributes
- **Identifiers**: Primary and foreign keys for relationships
- **Data Source**: Connection to your database table

## How It Works

Start Datus CLI with `datus --namespace <namespace>`, and begin with a subagent command:

```
  /gen_semantic_model generate a semantic model for table <table_name>
```


### Interactive Generation

When you request a semantic model, the AI assistant:
1. Retrieves your table's DDL (structure)
2. Checks if a semantic model already exists
3. Generates a comprehensive YAML file
4. Validates the configuration using MetricFlow
5. Prompts you to save it to the Knowledge Base

### Generation Workflow

```
User Request → DDL Analysis → YAML Generation → Validation → User Confirmation → Storage
```

### User Confirmation

After generating the semantic model, you'll see:

```
=============================================================
Generated YAML: table_name.yml
Path: /path/to/file.yml
=============================================================
[YAML content with syntax highlighting]

SYNC TO KNOWLEDGE BASE?

1. Yes - Save to Knowledge Base
2. No - Keep file only

Please enter your choice: [1/2]
```

**Options:**
- **Option 1**: Saves the semantic model to your Knowledge Base (RAG storage) for AI-powered queries
- **Option 2**: Keeps the YAML file only without syncing to the Knowledge Base

## Configuration

### Agent Configuration

In `agent.yml`, configure the semantic model generation node:

```yaml
agentic_nodes:
  gen_semantic_model:
    model: claude                    # LLM model to use
    system_prompt: gen_semantic_model
    prompt_version: "1.0"
    tools: db_tools.*, generation_tools.*, filesystem_tools.*
    hooks: generation_hooks          # Enables user confirmation workflow
    mcp: metricflow_mcp             # MetricFlow validation server
    workspace_root: /path/to/semantic_models
    agent_description: "Semantic model generation assistant"
    rules:
      - Use get_table_ddl tool to get complete table DDL
      - Generate comprehensive semantic models
      - Validate using metricflow_mcp
```

### Key Configuration Options

| Parameter | Description | Example |
|-----------|-------------|---------|
| `model` | LLM model for generation | `claude`, `deepseek`, claude is recommended |
| `workspace_root` | Directory to save YAML files | `/Users/you/.datus/data/semantic_models` |
| `tools` | Available tools for the assistant | `db_tools.*`, `filesystem_tools.*` |
| `hooks` | Enable user confirmation | `generation_hooks` |
| `mcp` | MetricFlow validation server | `metricflow_mcp` |

## Semantic Model Structure

### Basic Template

```yaml
data_source:
  name: table_name                    # Required: lowercase with underscores
  description: "Table description"

  sql_table: schema.table_name        # For databases with schemas
  # OR
  sql_query: |                        # For custom queries
    SELECT * FROM table_name

  measures:
    - name: total_amount              # Required
      agg: SUM                        # Required: SUM|COUNT|AVERAGE|etc.
      expr: amount_column             # Column or SQL expression
      create_metric: true             # Auto-create queryable metric
      description: "Total transaction amount"

  dimensions:
    - name: created_date
      type: TIME                      # Required: TIME|CATEGORICAL
      type_params:
        is_primary: true              # One primary time dimension required
        time_granularity: DAY         # Required for TIME: DAY|WEEK|MONTH|etc.

    - name: status
      type: CATEGORICAL
      description: "Order status"

  identifiers:
    - name: order_id
      type: PRIMARY                   # PRIMARY|FOREIGN|UNIQUE|NATURAL
      expr: order_id

    - name: customer
      type: FOREIGN
      expr: customer_id
```

## Summary

The semantic model generation feature provides:
- ✓ Automated YAML generation from table DDL
- ✓ Interactive validation and error fixing
- ✓ User confirmation before storage
- ✓ Knowledge Base integration
- ✓ Duplicate prevention
- ✓ MetricFlow compatibility
