# Business Metrics Intelligence

The Metrics component specifically focuses on automatically extracting and generating business metrics from historical SQL queries, establishing an enterprise-level metrics management system.

## Core Value

### What Problem Does It Solve?

- **Duplicate SQL queries**: Large numbers of similar business query needs in enterprises
- **Inconsistent metric definitions**: Different developers have different understandings of the same business metrics
- **Inefficient data analysis**: SQL needs to be rewritten for each query

### What Value Does It Provide?

- **Metric standardization**: Unified business metric definitions ensuring data consistency
- **Knowledge reuse**: Converting historical SQL experience into reusable metrics library
- **Intelligent querying**: Directly calling predefined metrics through natural language

## How It Works

### Data Hierarchy Structure
Data Table/Historical SQLs → Semantic Model → Business Metrics

#### 1. Semantic Model
- Defines table structure and business meaning
- Contains dimensions, measures, identifiers, and other metadata
- Provides semantic foundation for SQL queries

#### 2. Business Metrics
- Calculable metrics defined based on semantic models
- Contains metric name, description, calculation logic
- Supports complex business calculations

## Usage

### Basic Command

```bash
# Initialize metrics component from success story CSV
datus-agent bootstrap-kb \
    --namespace <your_namespace> \
    --components metrics \
    --success_story path/to/success_story.csv \
    --metric_meta business_meta
```

```bash
# Initialize metrics component from semantic YAML
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
| `--success_story` | ⚠️ | A CSV file containing historical SQLs and questions (required if not using `--semantic_yaml`) | `benchmark/semantic_layer/success_story.csv` |
| `--semantic_yaml` | ⚠️ | Semantic model YAML file (required if not using `--success_story`) | `models/semantic_model.yaml` |
| `--metric_meta` | ✅ | Metric metadata | `ecommerce` configuration component in `agent.yml` |
| `--kb_update_strategy` | ✅ | Update strategy | `overwrite`/`incremental` |
| `--pool_size` | ❌ | Concurrent thread count | `4` |

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
```

## Summary

The Bootstrap-KB Metrics component is the core functionality of Datus Agent, helping enterprises establish standardized data metrics systems through automated metric generation and management. It not only significantly improves data analysis efficiency but also ensures enterprise data consistency and reusability.

By properly using the Metrics component, enterprises can build a strong foundation for data-driven decision making.