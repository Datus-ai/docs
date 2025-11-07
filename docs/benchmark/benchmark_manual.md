# Benchmark

Evaluate Datus Agent's performance and capabilities using industry-standard benchmarks. Run comprehensive tests against datasets like BIRD and Spider 2.0-Snow to assess accuracy, execution success rate, and query generation quality.

## Overview

Datus Agent benchmark mode enables you to:

- **Measure Accuracy**: Evaluate how well the agent generates correct SQL from natural language
- **Track Success Rates**: Monitor query execution success across different database types
- **Compare Results**: Validate generated queries against expected outputs
- **Identify Improvements**: Discover areas for optimization and refinement

## Quick Start with Docker

Get started quickly with pre-configured Docker containers that include benchmark datasets.

### Step 1: Pull the Docker Image

!!! tip
    Ensure Docker is installed and running on your system before proceeding.

```bash title="Terminal"
docker pull datusai/datus-agent:0.2.2
```

### Step 2: Launch the Docker Container

!!! tip
    Demo datasets are preloaded, allowing you to quickly explore Datus capabilities without additional setup.

=== "BIRD"
    ```bash title="Terminal"
    docker run --name datus \
    --env DEEPSEEK_API_KEY=<your_api_key>  \
    -d datusai/datus-agent:0.2.2
    ```

=== "Spider 2.0-Snow"
    ```bash title="Terminal"
    docker run --name datus \
    --env DEEPSEEK_API_KEY=<your_api_key>  \
    --env SNOWFLAKE_ACCOUNT=<your_snowflake_account>  \
    --env SNOWFLAKE_USERNAME=<your_snowflake_username>  \
    --env SNOWFLAKE_PASSWORD=<your_snowflake_password>  \
    -d datusai/datus-agent:0.2.2
    ```

### Step 3: Run Benchmark Tests

!!! warning
    Each task may take several minutes to complete. Running all tasks may require hours or days depending on your system configuration.

**BIRD Dataset**

!!! info
    Task ID range: 0-1533

=== "Run by Task ID"
    ```bash title="Terminal"
    docker exec -it datus python -m datus.main benchmark  \
    --namespace bird_sqlite \
    --benchmark bird_dev \
    --benchmark_task_ids <task_id1> <task_id2>
    ```

=== "Run All Tasks"
    ```bash title="Terminal"
    docker exec -it datus python -m datus.main benchmark  \
    --namespace bird_sqlite \
    --benchmark bird_dev
    ```

**Spider 2.0-Snow Dataset**

!!! info
    You can find the task ID (instance ID) in the [spider2-snow.jsonl](https://github.com/xlang-ai/Spider2/blob/main/spider2-snow/spider2-snow.jsonl) file.

!!! note
    Ensure you start the Docker container with Snowflake environment parameters configured.

=== "Run by Task ID"
    ```bash title="Terminal"
    docker exec -it datus python -m datus.main benchmark  \
    --namespace snowflake \
    --benchmark spider2 \
    --benchmark_task_ids <task_id1> <task_id2>
    ```

=== "Run All Tasks"
    ```bash title="Terminal"
    docker exec -it datus python -m datus.main benchmark  \
    --namespace snowflake \
    --benchmark spider2
    ```

### Step 4: Evaluate Results

#### Run Evaluation
=== "Evaluate by Task IDs"
```bash title="Run Evaluation"
docker exec -it datus python -m datus.main eval  \
  --namespace snowflake \
  --benchmark spider2 \
  --output_file evaluation.json \
  --task_ids <task_id1> <task_id2>
```
=== "Evaluate All"
```bash title="Run Evaluation"
docker exec -it datus python -m datus.main eval  \
  --namespace snowflake \
  --output_file evaluation.json \
  --benchmark spider2
```

#### Evaluation Results

=== "Summary"
```text title="Evaluation Result" hl_lines="8-10"
────────────────────────────────────────────────────────────────────────────────
 Datus Evaluation Summary (Total: 30 Queries)
────────────────────────────────────────────────────────────────────────────────
 ✅ Passed:                                 19    (63%)
 ⚠️  No SQL / Empty Result:                 0    (0%)
 ❌ Failed:                                 11    (37%)
     • Table Mismatch:                      4    (13%)
     • Table Matched (Result Mismatch):     7    (23%)
         - Row Count Mismatch:              1    (3%)
         - Column Value Mismatch:           6    (20%)
────────────────────────────────────────────────────────────────────────────────

 Passed Queries:                        12, 40, 63, 80, 209, 279, 340, 436, 738, 774, 778, 853, 864, 1013, 1069, 1177, 1329, 1439, 1488
 No SQL / Empty Result Queries:         None
 Failed (Table Mismatch):               263, 388, 484, 584
 Failed (Row Count Mismatch):           455
 Failed (Column Value Mismatch):        76, 131, 617, 642, 1011, 1205

────────────────────────────────────────────────────────────────────────────────
```

=== "Details"
```json
{
  "status": "success",
  "generated_time": "2025-11-07T15:56:40.050678",
  "summary": {
    "total_files": 1,
    "total_output_nodes": 1,
    "total_output_success": 1,
    "total_output_failure": 0,
    "success_rate": 100.0,
    "comparison_summary": {
      "total_comparisons": 1,
      "match_count": 1,
      "mismatch_count": 0,
      "comparison_error_count": 0,
      "empty_result_count": 0,
      "match_rate": 100.0
    }
  },
  "task_ids": {
    "failed_task_ids": "",
    "matched_task_ids": "0",
    "mismatched_task_ids": "",
    "empty_result_task_ids": ""
  },
  "details": {
    "0": {
      "total_node_count": 4,
      "output_node_count": 1,
      "output_success_count": 1,
      "output_failure_count": 0,
      "errors": [],
      "node_types": {
        "start": 1,
        "chat": 1,
        "execute_sql": 1,
        "output": 1
      },
      "tool_calls": {
        "list_tables": 1,
        "describe_table": 2,
        "search_files": 1,
        "read_query": 3
      },
      "completion_time": 1762173198.4857101,
      "status": "completed",
      "comparison_results": [
        {
          "task_id": "0",
          "actual_file_exists": true,
          "gold_file_exists": true,
          "actual_path": "/Users/lyf/.datus/save/bird_school/0.csv",
          "gold_path": "/Users/lyf/.datus/benchmark/california_schools/california_schools.csv",
          "comparison": {
            "match_rate": 1.0,
            "matched_columns": [
              [
                "eligible_free_rate",
                "`Free Meal Count (K-12)` / `Enrollment (K-12)`"
              ]
            ],
            "missing_columns": [],
            "extra_columns": [
              "School Name",
              "District Name",
              "Academic Year"
            ],
            "actual_shape": [1, 4],
            "expected_shape": [1, 1],
            "actual_preview": "School Name | District Name | eligible_free_rate | Academic Year\n       ----------------------------------------------------------------\n       Oakland Community Day Middle | Oakland Unified | 1.0 | 2014-2015",
            "expected_preview": "`Free Meal Count (K-12)` / `Enrollment (K-12)`\n       ----------------------------------------------\n       1.0",
            "actual_tables": ["frpm"],
            "expected_tables": ["frpm"],
            "matched_tables": ["frpm"],
            "actual_sql_error": null,
            "sql_error": null,
            "actual_sql": "SELECT \n    \"School Name\",\n    \"District Name\", \n    \"Percent (%) Eligible Free (K-12)\" as eligible_free_rate,\n    \"Academic Year\"\nFROM frpm \nWHERE \"County Name\" = 'Alameda'\n    AND \"Percent (%) Eligible Free (K-12)\" IS NOT NULL\nORDER BY \"Percent (%) Eligible Free (K-12)\" DESC\nLIMIT 1;",
            "gold_sql": "SELECT `Free Meal Count (K-12)` / `Enrollment (K-12)` FROM frpm WHERE `County Name` = 'Alameda' ORDER BY (CAST(`Free Meal Count (K-12)` AS REAL) / `Enrollment (K-12)`) DESC LIMIT 1",
            "tools_comparison": {
              "expected_file": {"expected": "", "actual": [], "matched_actual": [], "match": true},
              "expected_sql": {"expected": "", "actual": [], "matched_actual": [], "match": true},
              "expected_semantic_model": {"expected": "", "actual": [], "matched_actual": [], "match": true},
              "expected_metrics": {
                "expected": ["Eligible free rate for K-12"],
                "actual": [],
                "matched_expected": [],
                "matched_actual": [],
                "missing_expected": ["Eligible free rate for K-12"],
                "match": false
              }
            },
            "error": null
          }
        }
      ]
    }
  }
}
```

**Key fields under `details`:**
- **comparison_results**: Comparison results for each task
    - `actual_sql`: SQL generated by Datus Agent
    - `actual_path`: Executed result produced by Datus Agent
    - `gold_sql`: Reference (gold) SQL
    - `gold_path`: Reference (gold) result path
    - `comparison`: Side-by-side comparison results
        - `match_rate`: Matching ratio
        - `matched_columns`: Columns that match
        - `missing_columns`: Columns missing compared to reference
        - `extra_columns`: Extra columns not present in the reference
        - `actual_tables`: Tables used by the generated result
        - `expected_tables`: Tables used by the reference answer
        - `matched_tables`: Intersected tables
- **tool_calls**: Count of each tool invocation

## Custom Benchmark

### Add Benchmark Configuration

```yaml
agent:
  namespace:
    california_schools:
      type: sqlite
      name: california_schools
      uri: sqlite:///benchmark/bird/dev_20240627/dev_databases/california_schools/california_schools.sqlite # Database file path. Use sqlite:/// for relative paths; sqlite://// for absolute paths.
  benchmark:
    california_schools:              # Benchmark name
      question_file: california_schools.csv       # File containing benchmark questions
      question_id_key: task_id                    # Field name for question ID
      question_key: question                      # Field name for question text
      ext_knowledge_key: expected_knowledge       # Field name for external knowledge or additional context

      # Configuration for evaluation phase
      gold_sql_path: california_schools.csv       # File path for reference (gold) SQL
      gold_sql_key: gold_sql                      # Field name for reference SQL
      gold_result_path: california_schools.csv    # File path for reference (gold) results
      gold_result_key: ""                         # Field name for results when using a single file
```

---

📖 **Benchmark Configuration Field Description**

| Field | Description |
|-------|--------------|
| **question_file** | Path to the question file, supporting `.csv`, `.json`, and `.jsonl` formats. The path is relative to `{agent.home}/benchmark/{benchmark_name}`. |
| **question_id_key** | Unique identifier field name for each benchmark question. |
| **question_key** | Field name for the natural language question. |
| **ext_knowledge_key** | Field name for additional knowledge or problem description. |
| **gold_sql_path** | Path to the reference SQL file. It can be a single file (e.g., for BIRD) or task-specific files (e.g., for Spider2). |
| **gold_sql_key** | When `gold_sql_path` is a single file, this specifies the field name containing the reference SQL. |
| **gold_result_path** | Path to the reference (gold) result file. If left empty, the system will automatically execute the reference SQL to obtain results. |
| **gold_result_key** | Field name for the results when `gold_result_path` points to a single file. |

---

### Build Knowledge Base

Construct the metadata, metrics, and reference SQL knowledge bases according to your dataset.  
See [knowledge_base/introduction](../knowledge_base/introduction.md) for details.

### Run Benchmark

See [Step 3: Run Benchmark Tests](#step-3-run-benchmark-tests)

### Evaluate Results

See [Step 4: Evaluate Results](#step-4-evaluate-results)