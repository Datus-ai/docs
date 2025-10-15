# SQL Summary Generation Guide

## Overview

The SQL Summary feature helps you analyze, classify, and catalog SQL queries for knowledge reuse. It automatically generates structured YAML summaries that are stored in a searchable Knowledge Base, making it easy to find and reuse similar queries in the future.

## What is a SQL Summary?

A **SQL summary** is a structured YAML document that captures:
- **Query Text**: The complete SQL query
- **Business Context**: Domain, categories, and tags
- **Semantic Summary**: Detailed explanation for vector search
- **Metadata**: Name, comment, file path

**Use Cases**:
- Build a searchable SQL query library
- Share and reuse proven queries across teams
- Document complex query patterns
- Enable semantic search: "Find queries related to revenue analysis"

## Quick Start

Launch the SQL summary generation subagent:

```bash
/gen_sql_summary Analyze this SQL: SELECT SUM(revenue) FROM sales GROUP BY region.(You can also add some description on this SQL)
```

## How It Works

### Generation Workflow

```
User provides SQL + description → Agent analyzes query → Retrieves context (taxonomy + similar queries) →
Generates unique ID → Creates YAML → Saves file → User confirms → Syncs to Knowledge Base
```

### Step-by-Step Process

1. **Understand SQL**: The AI analyzes your query structure and business logic
2. **Get Context**: Calls `prepare_sql_summary_context()` to retrieve:
   - Existing taxonomy (domains, categories, tags)
   - Similar SQL summaries for classification reference
3. **Generate Unique ID**: Uses `generate_sql_summary_id()` based on SQL + comment
4. **Create Unique Name**: Generates a descriptive name (max 20 characters)
5. **Classify Query**: Assigns domain, layer1, layer2, and tags following existing patterns
6. **Generate YAML**: Creates structured summary document
7. **Save File**: Writes YAML to workspace using `write_file()`
8. **User Confirmation**: Shows the generated YAML and prompts for approval
9. **Sync to Knowledge Base**: Stores in LanceDB for semantic search

### Interactive Confirmation

After generation, you'll see:

```
==========================================================
Generated SQL History YAML
File: /path/to/sql_summary.yml
==========================================================
[YAML content with syntax highlighting]

  SYNC TO KNOWLEDGE BASE?

  1. Yes - Save to Knowledge Base
  2. No - Keep file only

Please enter your choice: [1/2]
```

## Configuration

### Agent Configuration

In `agent.yml`:

```yaml
agentic_nodes:
  gen_sql_summary:
    model: deepseek                        # LLM model for analysis
    system_prompt: gen_sql_summary         # Prompt template
    prompt_version: "1.0"
    tools: generation_tools.prepare_sql_summary_context, generation_tools.generate_sql_summary_id, filesystem_tools.write_file
    hooks: generation_hooks                # Enable confirmation workflow
    workspace_root: /path/to/sql_history   # Directory to save YAML files
    agent_description: "SQL history analysis assistant"
```

### Key Configuration Options

| Parameter | Description | Example |
|-----------|-------------|---------|
| `model` | LLM model for SQL analysis | `deepseek`, `claude`, `openai`, `kimi`|
| `workspace_root` | Directory for SQL summary YAML files | `/Users/you/.datus/data/sql_history` |
| `tools` | Required tools for the workflow | See tools section below |
| `hooks` | Enable interactive confirmation | `generation_hooks` |

## YAML Structure

The generated SQL summary follows this structure:

```yaml
id: "abc123def456..."                      # Auto-generated MD5 hash
name: "Revenue by Region"                  # Descriptive name (max 20 chars)
sql: |                                     # Complete SQL query
  SELECT
    region,
    SUM(revenue) as total_revenue
  FROM sales
  GROUP BY region
comment: "Calculate total revenue grouped by region"
summary: "This query aggregates total revenue from the sales table, grouping results by geographic region. It uses SUM aggregation to calculate revenue totals for each region."
filepath: "/Users/you/.datus/data/sql_history/revenue_by_region.yml"
domain: "Sales"                            # Business domain
layer1: "Reporting"                        # Primary category
layer2: "Revenue Analysis"                 # Secondary category
tags: "revenue, region, aggregation"       # Comma-separated tags
```

### Field Descriptions

| Field | Required | Description | Example |
|-------|----------|-------------|---------|
| `id` | Yes | Unique hash (auto-generated) | `abc123def456...` |
| `name` | Yes | Short descriptive name (max 20 chars) | `Revenue by Region` |
| `sql` | Yes | Complete SQL query | `SELECT ...` |
| `comment` | Yes | Brief one-line description | User's message or generated summary |
| `summary` | Yes | Detailed explanation (for search) | Comprehensive query description |
| `filepath` | Yes | Actual file path | `/path/to/file.yml` |
| `domain` | Yes | Business domain | `Sales`, `Marketing`, `Finance` |
| `layer1` | Yes | Primary category | `Reporting`, `Analytics`, `ETL` |
| `layer2` | Yes | Secondary category | `Revenue Analysis`, `Customer Insights` |
| `tags` | Optional | Comma-separated keywords | `revenue, region, aggregation` |


## Summary

The SQL Summary feature provides:

✅ **Automated Analysis**: AI understands query structure and purpose
✅ **Smart Classification**: Taxonomy-based categorization with consistency
✅ **Deduplication**: Automatic hash-based duplicate detection
✅ **Semantic Search**: Vector embeddings enable intelligent query discovery
✅ **Language Preservation**: Maintains Chinese/English consistency
✅ **Interactive Workflow**: Review and approve before syncing
✅ **Knowledge Reuse**: Build searchable SQL query library
