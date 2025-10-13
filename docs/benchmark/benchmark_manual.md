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
docker pull datusai/datus-agent:0.2.0-rc1
```

### Step 2: Launch the Docker Container

!!! tip
    Demo datasets are preloaded, allowing you to quickly explore Datus capabilities without additional setup.

=== "BIRD"
    ```bash title="Terminal"
    docker run --name datus \
    --env DEEPSEEK_API_KEY=<your_api_key>  \
    -d datusai/datus-agent:0.2.0
    ```

=== "Spider 2.0-Snow"
    ```bash title="Terminal"
    docker run --name datus \
    --env DEEPSEEK_API_KEY=<your_api_key>  \
    --env SNOWFLAKE_ACCOUNT=<your_snowflake_acount>  \
    --env SNOWFLAKE_USERNAME=<your_snowflake_username>  \
    --env SNOWFLAKE_PASSWORD=<your_snowflake_password>  \
    -d datusai/datus-agent:0.2.0
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
    --benchmark_task_ids <task_id>
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
    --benchmark_task_ids <task_id>
    ```

=== "Run All Tasks"
    ```bash title="Terminal"
    docker exec -it datus python -m datus.main benchmark  \
    --namespace snowflake \
    --benchmark spider2
    ```

### Step 4: Review Benchmark Results

!!! tip
    Each benchmark generates a comprehensive performance summary with detailed metrics and task breakdowns.

```text title="Benchmark Result" hl_lines="8-10"
================================================================================
BENCHMARK ACCURACY EVALUATION REPORT
================================================================================
Generated Time: 2025-09-18 12:07:30

EXECUTIVE SUMMARY
----------------------------------------
Total tasks analyzed: 1
Execution success rate: 100.0%
Result comparison match rate: 100.0%

DETAILED STATISTICS
----------------------------------------
Total comparisons performed: 1
Successful matches: 1
Mismatches: 0
Comparison errors: 0
Empty result errors: 0
Mismatch rate: 0.0%
Error rate: 0.0%

TASK BREAKDOWN BY CATEGORY
----------------------------------------
Matched tasks (1):
14

Mismatched tasks (0):
None

Failed tasks (0):
None

ADDITIONAL STATISTICS
----------------------------------------
Overall success rate: 100.0%
Successful tasks: 1
Failed tasks: 0
Mismatched tasks: 0

================================================================================ [datus.utils.benchmark_utils]
2025-09-18 12:07:30 [info     ]
Final Result: {'status': 'success', 'generated_time': '2025-09-18T12:07:30.497861', 'summary': {'total_files': 1, 'total_output_nodes': 1, 'total_output_success': 1, 'total_output_failure': 0, 'success_rate': 100.0, 'comparison_summary': {'total_comparisons': 1, 'successful_matches': 1, 'mismatches': 0, 'comparison_errors': 0, 'empty_result_errors': 0, 'match_rate': 100.0}}, 'task_ids': {'failed_task_ids': '', 'matched_task_ids': '14', 'mismatched_task_ids': '', 'empty_result_task_ids': ''}, 'details': {'14': {'total_nodes': 6, 'output_nodes': 1, 'output_success': 1, 'output_failure': 0, 'errors': [], 'node_types': {'start': 1, 'schema_linking': 1, 'generate_sql': 1, 'execute_sql': 1, 'reflect': 1, 'output': 1}, 'completion_time': 1758197249.7893646, 'status': 'completed', 'comparison_results': [{'task_id': '14', 'actual_file_exists': True, 'gold_file_exists': True, 'actual_path': 'output/bird_sqlite/14.csv', 'gold_path': '/app/benchmark/dev_20240627/gold/exec_result/14.csv', 'comparison': {'match': True, 'actual_file_exists': True, 'expected_file_exists': True, 'actual_shape': (5, 1), 'expected_shape': (5, 1), 'actual_preview': 'NCESSchool\n       ----------\n       11707\n       4653\n       8283\n       ...', 'expected_preview': 'NCESSchool\n       ----------\n       11707\n       4653\n       8283\n       ...', 'error': None}}]}}} [__main__]
```