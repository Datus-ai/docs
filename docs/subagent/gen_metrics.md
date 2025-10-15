# Metrics Generation Guide

## Overview

The metrics generation feature helps you convert SQL queries into reusable MetricFlow metric definitions. Using an AI assistant, you can analyze SQL business logic and automatically generate standardized YAML metric configurations that can be queried consistently across your organization.

## What is a Metric?

A **metric** is a reusable business calculation built on top of semantic models. Metrics provide:
- **Consistent Business Logic**: One definition, used everywhere
- **Type Safety**: Validated structure and measure references
- **Metadata**: Display names, formats, business context
- **Composability**: Build complex metrics from simpler ones

**Example**: Instead of writing `SELECT SUM(revenue) / COUNT(DISTINCT customer_id)` repeatedly, define an `avg_customer_revenue` metric once.

## Important Limitations

**⚠️ Single Table Queries Only**

The current version **only supports generating metrics from single-table SQL queries**. Multi-table JOINs are not supported.

**Supported**:
```sql
SELECT SUM(revenue) FROM transactions WHERE status = 'completed'
SELECT COUNT(DISTINCT customer_id) / COUNT(*) FROM orders
```

**Not Supported**:
```sql
SELECT SUM(o.amount)
FROM orders o
JOIN customers c ON o.customer_id = c.id  -- ❌ JOIN not supported
```

## How It Works

Start Datus CLI with `datus --namespace <namespace>`, and use the metrics generation subagent:

```bash
/gen_metrics Generate a metric from this SQL: SELECT SUM(amount) FROM transactions, the coresponding question is total amount of all transactions
```

### Generation Workflow

```
User provides SQL and question → Agent analyzes logic → Finds semantic model → Reads measures →
Checks for duplicates → Generates metric YAML → Appends to file → Validates →
User confirms → Syncs to Knowledge Base
```

### Interactive Confirmation

After generation, you'll see:

```
==========================================================
Generated YAML: transactions.yml
Path: /Users/you/.datus/data/semantic_models/transactions.yml
==========================================================
[YAML content with syntax highlighting showing the new metric]

  SYNC TO KNOWLEDGE BASE?

  1. Yes - Save to Knowledge Base
  2. No - Keep file only

Please enter your choice: [1/2]
```

**Options:**
- **Option 1**: Syncs the metric to your Knowledge Base for AI-powered semantic search
- **Option 2**: Keeps the YAML file only without syncing to the Knowledge Base

## Configuration

### Agent Configuration

In `agent.yml`, configure the metrics generation node:

```yaml
agentic_nodes:
  gen_metrics:
    model: claude                          # LLM model for metric generation
    system_prompt: gen_metrics             # Prompt template name
    prompt_version: "1.0"                  # Template version
    tools: generation_tools.*, filesystem_tools.*
    hooks: generation_hooks                # Enable user confirmation workflow
    mcp: metricflow_mcp                    # MetricFlow validation server
    max_turns: 40                          # Max conversation turns
    workspace_root: /path/to/semantic_models
    agent_description: "Metric definition generation assistant"
    rules:
      - Analyze user-provided SQL queries to generate MetricFlow metrics
      - Use list_allowed_directories to find existing semantic model files
      - Use read_file to read semantic model and understand measures
      - Use check_metric_exists tool to avoid duplicate generation
      - Use edit_file to append metrics (DO NOT use write_file)
      - Use mf validate-configs to validate configurations
      - After validation, call end_generation to trigger confirmation
```

### Key Configuration Options

| Parameter | Description | Example |
|-----------|-------------|---------|
| `model` | LLM model for generation | `claude`, `openai`, `deepseek`, claude is recommended |
| `workspace_root` | Directory containing semantic model YAML files | `/Users/you/.datus/data/semantic_models` |
| `tools` | Available tools (generation + filesystem) | `generation_tools.*, filesystem_tools.*` |
| `hooks` | Enable interactive confirmation workflow | `generation_hooks` |
| `mcp` | MetricFlow validation server | `metricflow_mcp` |
| `max_turns` | Max conversation turns for complex queries | `40` |



## Usage Examples

### Example 1: Simple Aggregation

**User Input**:
```
/gen_metrics Generate a metric for total order count
```

**Agent Actions**:
1. Finds `orders.yml` semantic model
2. Reads file to discover `order_count` measure
3. Generates MEASURE_PROXY metric:

```yaml
---
metric:
  name: total_orders
  description: Total number of orders
  type: measure_proxy
  type_params:
    measure: order_count
  locked_metadata:
    display_name: "Total Orders"
    increase_is_good: true
```

### Example 2: Conversion Rate

**User Input**:
```
/gen_metrics Create a metric from this SQL:
SELECT
  COUNT(DISTINCT CASE WHEN status = 'completed' THEN order_id END) /
  COUNT(DISTINCT order_id) AS completion_rate
FROM orders
```

**Agent Actions**:
1. Analyzes SQL logic (ratio with conditional counting)
2. Finds `orders.yml` and reads measures
3. Generates RATIO metric with constraint:

```yaml
---
metric:
  name: order_completion_rate
  description: Percentage of orders that reached completed status
  type: ratio
  type_params:
    numerator:
      name: order_count
      constraint: status = 'completed'
    denominator: order_count
  locked_metadata:
    display_name: "Order Completion Rate"
    value_format: ".2%"
    increase_is_good: true
```

### Example 3: Complex Calculation

**User Input**:
```
/gen_metrics Generate average basket size metric:
SELECT SUM(total_amount) / COUNT(DISTINCT order_id)
FROM order_items
```

**Agent Actions**:
1. Locates `order_items.yml`
2. Identifies this as a RATIO (average)
3. Generates metric:

```yaml
---
metric:
  name: avg_basket_size
  description: Average order value (basket size)
  type: ratio
  type_params:
    numerator: total_amount
    denominator: order_count
  locked_metadata:
    display_name: "Average Basket Size"
    value_format: "$$,.2f"
    unit: "dollars"
    increase_is_good: true
```

## How Metrics Are Stored

### File Organization

Metrics are appended to existing semantic model files using the YAML document separator `---`:

```yaml
# Existing semantic model
data_source:
  name: transactions
  sql_table: transactions
  measures:
    - name: revenue
      agg: SUM
      expr: amount
  dimensions:
    - name: transaction_date
      type: TIME

---
# First metric (appended)
metric:
  name: total_revenue
  type: measure_proxy
  type_params:
    measure: revenue

---
# Second metric (appended)
metric:
  name: avg_transaction_value
  type: ratio
  type_params:
    numerator: revenue
    denominator: transaction_count
```

**Why append instead of separate files?**
- Keeps related metrics close to their semantic model
- Easier maintenance and validation
- MetricFlow can validate all definitions together

### Knowledge Base Storage

When you choose "1. Yes - Save to Knowledge Base", the metric is stored in a Vector Database with:

1. **Metadata**: Name, description, type, domain/layer classification
2. **LLM Text**: Natural language representation for semantic search
3. **References**: Associated semantic model name
4. **Timestamp**: Creation date

## Summary

The metrics generation feature provides:

✅ **SQL-to-Metric Conversion**: Analyze SQL queries and generate MetricFlow metrics
✅ **Intelligent Type Detection**: Automatically selects the right metric type
✅ **Duplicate Prevention**: Checks for existing metrics before generation
✅ **Validation**: MetricFlow validation ensures correctness
✅ **Interactive Workflow**: Review and approve before syncing
✅ **Knowledge Base Integration**: Semantic search for metric discovery
✅ **File Management**: Appends to existing semantic model files safely