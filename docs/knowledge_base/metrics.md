# Business Metrics Intelligence

The Metrics component specifically focuses on automatically extracting and generating business metrics from historical SQL queries, establishing an enterprise-level metrics management system.

## Core Value

Solves common enterprise challenges:
- **Duplicate SQL queries**: Reuse metrics instead of rewriting similar queries
- **Inconsistent definitions**: Standardize metric definitions across teams
- **Manual classification**: Organize metrics with hierarchical subject tree taxonomy

## How It Works

Data flows through two layers: **Historical SQLs → Semantic Model → Business Metrics**

Semantic models define table structure, dimensions, and measures. Metrics are reusable calculations built on top of these models.

## Usage

**Prerequisites**: This command relies on [datus-metricflow](../metricflow/introduction.md), install it first.

### Basic Command

```bash
# From CSV (historical SQLs)
datus-agent bootstrap-kb \
    --namespace <your_namespace> \
    --components metrics \
    --success_story path/to/success_story.csv \
    --metric_meta business_meta

# From YAML (semantic models)
datus-agent bootstrap-kb \
    --namespace <your_namespace> \
    --components metrics \
    --semantic_yaml path/to/semantic_model.yaml \
    --metric_meta business_meta
```

### Key Parameters

| Parameter | Required | Description | Example |
|-----------|----------|-------------|---------|
| `--namespace` | ✅ | Database namespace | `sales_db` |
| `--components` | ✅ | Components to initialize | `metrics` |
| `--success_story` | ⚠️ | CSV file with historical SQLs and questions (required if no `--semantic_yaml`) | `success_story.csv` |
| `--semantic_yaml` | ⚠️ | Semantic model YAML file (required if no `--success_story`) | `semantic_model.yaml` |
| `--metric_meta` | ✅ | Metric metadata configuration in `agent.yml` | `business_meta` |
| `--kb_update_strategy` | ✅ | Update strategy | `overwrite`/`incremental` |
| `--subject_tree` | ❌ | Predefined categories (comma-separated) | `Sales/Reporting/Daily,Finance/Revenue/Monthly` |
| `--pool_size` | ❌ | Concurrent thread count | `4` |

### Subject Tree Categorization

Organizes metrics using hierarchical taxonomy: `domain/layer1/layer2` (e.g., `Sales/Reporting/Daily`)

**Two modes:**
- **Predefined**: Use `--subject_tree` to enforce specific categories
- **Learning**: Omit `--subject_tree` to reuse existing categories or create new ones

```bash
# Predefined mode example
--subject_tree "Sales/Reporting/Daily,Finance/Revenue/Monthly"

# Learning mode: omit --subject_tree parameter
```

**Generated Tag Format:**

When metrics are generated, the subject_tree classification is stored in `locked_metadata.tags` with the format `"subject_tree: {domain}/{layer1}/{layer2}"`:

```yaml
metric:
  name: daily_revenue
  type: measure_proxy
  type_params:
    measure: revenue
  locked_metadata:
    tags:
      - "Finance"
      - "subject_tree: Sales/Reporting/Daily"
```

**Important for YAML Import:**

When using `--semantic_yaml` to sync metrics from YAML files to lancedb, you must manually add the `locked_metadata.tags` with subject_tree format in your YAML file for successful categorization. The system will not automatically classify metrics imported from YAML - you need to include the tags yourself:

```yaml
metric:
  name: your_metric
  # ... other fields
  locked_metadata:
    tags:
      - "YourDomain"
      - "subject_tree: Domain/Layer1/Layer2"
```

## Data Source Formats

### CSV Format

```csv
question,sql
How many customers have been added per day?,"SELECT ds AS date, SUM(1) AS new_customers FROM customers GROUP BY ds ORDER BY ds;"
What is the total transaction amount?,SELECT SUM(transaction_amount_usd) as total_amount FROM transactions;
```

### YAML Format

```yaml
data_source:
  name: transactions
  description: "Transaction records"
  identifiers:
    - name: transaction_id
      type: primary
  dimensions:
    - name: transaction_date
      type: time
    - name: transaction_type
      type: categorical
  measures:
    - name: amount
      type: double
      agg: sum
---
metric:
  name: total_revenue
  description: "Total revenue from all transactions"
  constraint: "amount > 0"
  locked_metadata:
    tags:
      - "Finance"
      - "subject_tree: Finance/Revenue/Total"
```

## Summary

The Metrics component establishes standardized, reusable metric definitions from historical SQLs. With hierarchical subject tree taxonomy and flexible classification modes, it helps teams maintain consistent, discoverable metrics for data-driven decision making.