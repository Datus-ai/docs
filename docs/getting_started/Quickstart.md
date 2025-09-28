# Quickstart

Begin using Datus Agent in minutes

This guide introduces three usage modes so you can adopt Datus based on your needs.

## Datus CLI (Chat Mode)

Use Datus like a chatbot: type natural language questions, get SQL or summaries back. Ideal for ad hoc queries and fast metric exploration.

### Step 1: Launch the CLI

!!! tip "Configuration"
    You can connect to your database by adding a new namespace in `agent.yml`. See our [Configuration guide](../configuration/introduction.md) for more details.

```bash title="Terminal"
datus-cli --namespace duckdb-demo
Initializing AI capabilities in background...

Datus - AI-powered SQL command-line interface
Type '.help' for a list of commands or '.exit' to quit.

Namespace duckdb selected
Connected to duckdb using database duckdb
Context: Current: database: duckdb
Type SQL statements or use ! @ . commands to interact.
Datus>
```
### Step 2: Start Chat with Datus

!!! tip
    You can execute SQL in Datus just like in a SQL editor.

List all tables:

```bash title="Terminal"
Datus> .tables
Tables in Database duckdb-demo
+---------------------+
| Table Name          |
+=====================+
| bank_failures       |
| boxplot             |
| calendar            |
| candle              |
| christmas_cost      |
| companies           |
| country_stats_scatter|
| gold_vs_bitcoin     |
| japan_births_deaths |
| japan_population    |
| metrics             |
| niger_population    |
| quotes              |
| radar               |
| sankey              |
| search_trends       |
| tree                |
+---------------------+
```

Ask any question you like. Here is a table named `gold_vs_bitcoin`. Let's view its schema:

```bash title="Terminal"
Datus> desc gold_vs_bitcoin
+------------------+------------------+------------------+------------------+------------------+------------------+
| column_name      | column_type      | null             | key              | default          | extra            |
+==================+==================+==================+==================+==================+==================+
| time             | TIMESTAMP        | YES              | None             | None             | None             |
| gold             | DOUBLE           | YES              | None             | None             | None             |
| bitcoin          | DOUBLE           | YES              | None             | None             | None             |
+------------------+------------------+------------------+------------------+------------------+------------------+
Returned 3 rows in 0.04 seconds
```

!!! tip
    To chat, start a line with `/` and continue in natural language.

Suppose we want to understand the correlation between gold and Bitcoin. We can ask the Datus agent directly:

```bash title="Terminal"
Datus> /Detailed analysis of gold–Bitcoin correlation.
```

Datus will display its thinking process to show how it approaches the task, then provide a detailed report.

??? example "Detailed Analysis Output"
    ```text
    Creating new chat session...
    Processing chat request...
    +---------------------------------------------------------------------------------------------------------------------------------------+
    |                                                    Action Stream                                                                      |
    +=======================================================================================================================================+
    | 🔧 Tool call - search_files('{"path": "/Users/yt/.datus/workspace", "pattern": ...') - ✓                                            |
    |     No matches found (0.5s)                                                                                                          |
    | 💬 Thinking: Let me check the directory structure to see what's available:                                                           |
    | 🔧 Tool call - list_directory('{"path": "/Users/yt/.datus/workspace"}') - ✓ (0.5s)                                                   |
    | 💬 Thinking: Now let me analyze the gold-Bitcoin correlation by examining the data and performing a comprehensive correlation        |
    | analysis:                                                                                                                            |
    | 🔧 Tool call - read_query('{"sql": "SELECT \n    COUNT(*) as total_records,\n...') - ✓                                               |
    |     ✓ 1 rows (0.5s)                                                                                                                  |
    | 💬 Thinking: Now let me calculate the correlation coefficient and perform a detailed analysis:                                       |
    | 🔧 Tool call - read_query('{"sql": "SELECT \n    -- Correlation coefficient\n...') - ✓                                               |
    |     ✓ 1 rows (0.5s)                                                                                                                  |
    | 💬 Thinking: Let me get more detailed statistics and time-based analysis:                                                            |
    | 🔧 Tool call - read_query('{"sql": "WITH daily_aggregates AS (\n    SELECT \n...') - ✓                                               |
    |     ✓ 1 rows (0.5s)                                                                                                                  |
    | 💬 Thinking: Let me check the distribution of the data and look for any patterns:                                                    |
    | 🔧 Tool call - read_query('{"sql": "SELECT \n    -- Price movement analysis\n...') - ✓                                               |
    |     ✗ Failed:(error_code=500006, e...) (0.5s)                                                                                        |
    | 💬 Thinking: Let me fix the query to properly analyze price movement patterns:                                                       |
    | 🔧 Tool call - read_query('{"sql": "WITH price_changes AS (\n    SELECT \n   ...') - ✓                                               |
    |     ✓ 5 rows (0.5s)                                                                                                                  |
    | 💬 Thinking: Now let me analyze the strength of the correlation across different time periods and price levels:                      |
    | 🔧 Tool call - read_query('{"sql": "WITH price_quartiles AS (\n    SELECT \n ...') - ✓                                               |
    |     ✓ 16 rows (0.5s)                                                                                                                 |
    | 💬 Thinking: Let me also analyze the volatility and risk characteristics:                                                            |
    | 🔧 Tool call - read_query('{"sql": "SELECT \n    -- Risk and return metrics\n...') - ✓                                               |
    |     ✗ Failed:(error_code=500006, e...) (0.5s)                                                                                        |
    | 💬 Thinking: Let me correct the query with the proper function names:                                                                |
    | 🔧 Tool call - read_query('{"sql": "SELECT \n    -- Risk and return metrics\n...') - ✓                                               |
    |     ✓ 1 rows (0.5s)                                                                                                                  |
    | 💬 Thinking: Finally, let me examine the time series data to understand the temporal patterns:                                       |
    | 🔧 Tool call - read_query('{"sql": "SELECT \n    DATE(time) as trading_date,\...') - ✓                                               |
    |     ✓ 1 rows (0.5s)                                                                                                                  |
    | 💬 Chat interaction completed successfully                                                                                           |
    +---------------------------------------------------------------------------------------------------------------------------------------+
    ```

**Generated SQL:**

```sql title="Generated Query"
-- Generated SQL (copied)
SELECT
    -- Overall correlation and basic stats
    CORR(gold, bitcoin) as correlation_coefficient,
    COUNT(*) as total_observations,
    AVG(gold) as avg_gold_price,
    AVG(bitcoin) as avg_bitcoin_price,
    STDDEV(gold) as gold_volatility,
    STDDEV(bitcoin) as bitcoin_volatility,
    COVAR_POP(gold, bitcoin) as covariance,

    -- Price movement patterns
    (SELECT COUNT(*) FROM (
        SELECT
            CASE
                WHEN gold_change > 0 AND bitcoin_change > 0 THEN 'Both Up'
                WHEN gold_change < 0 AND bitcoin_change < 0 THEN 'Both Down'
                WHEN gold_change > 0 AND bitcoin_change < 0 THEN 'Gold Up, Bitcoin Down'
                WHEN gold_change < 0 AND bitcoin_change > 0 THEN 'Gold Down, Bitcoin Up'
            END as pattern
        FROM (
            SELECT
                gold - LAG(gold) OVER (ORDER BY time) as gold_change,
                bitcoin - LAG(bitcoin) OVER (ORDER BY time) as bitcoin_change
            FROM gold_vs_bitcoin
        ) WHERE gold_change IS NOT NULL
    ) WHERE pattern = 'Both Up') as both_up_count,

    (SELECT COUNT(*) FROM (
        SELECT
            CASE
                WHEN gold_change > 0 AND bitcoin_change > 0 THEN 'Both Up'
                WHEN gold_change < 0 AND bitcoin_change < 0 THEN 'Both Down'
                WHEN gold_change > 0 AND bitcoin_change < 0 THEN 'Gold Up, Bitcoin Down'
                WHEN gold_change < 0 AND bitcoin_change > 0 THEN 'Gold Down, Bitcoin Up'
            END as pattern
        FROM (
            SELECT
                gold - LAG(gold) OVER (ORDER BY time) as gold_change,
                bitcoin - LAG(bitcoin) OVER (ORDER BY time) as bitcoin_change
            FROM gold_vs_bitcoin
        ) WHERE gold_change IS NOT NULL
    ) WHERE pattern = 'Both Down') as both_down_count
FROM gold_vs_bitcoin
```

**Analysis Results:**

??? success "Gold-Bitcoin Correlation Analysis"
    ```text
    +===============================================================================================================+
    |                                        Gold-Bitcoin Correlation Analysis                                     |
    +===============================================================================================================+

                                                          📊 Overall Correlation

    • Correlation Coefficient: -0.225 (Weak negative correlation)
    • Total Observations: 954 data points
    • Time Period: Single day - December 1, 2024 (16:00 to 16:15)


                                                            💰 Price Statistics

    • Average Gold Price: 0.684
    • Average Bitcoin Price: 0.785
    • Gold Volatility: 0.200
    • Bitcoin Volatility: 0.233
    • Covariance: -0.0105


                                                        📈 Price Movement Patterns

    Based on 953 observed price changes:

    • Both Assets Moving Up: 216 times (22.67%)
    • Both Assets Moving Down: 203 times (21.30%)
    • Gold Up, Bitcoin Down: 189 times (19.83%)
    • Gold Down, Bitcoin Up: 190 times (19.94%)


                                                              🎯 Key Insights

                                                       1. Weak Negative Correlation

    The -0.225 correlation suggests these assets tend to move in opposite directions more often than together, though the relationship
    is weak.

                                                       2. Similar Movement Patterns

    Both assets show remarkably similar movement patterns:

    • Approximately 44% of the time they move in the same direction
    • Approximately 40% of the time they move in opposite directions

                                                         3. Volatility Comparison

    Bitcoin shows slightly higher volatility (0.233) compared to Gold (0.200), which aligns with Bitcoin's reputation as a more
    volatile asset.

                                                       4. Risk-Adjusted Performance

    • Gold Sharpe Ratio: 3.42
    • Bitcoin Sharpe Ratio: 3.37 Both assets show strong risk-adjusted returns during this period.


                                                               ⚠️ Limitations

    This analysis is based on a very short time window (15 minutes of data). For more robust correlation analysis, data spanning
    multiple days/weeks would be needed to observe:

    • Longer-term correlation trends
    • Market regime changes
    • Impact of macroeconomic events
    • Seasonal patterns


                                                        💡 Investment Implications

    The weak negative correlation suggests these assets could provide some diversification benefits, though the relationship is not
    strong enough for reliable hedging strategies based on this limited dataset.
    ```

If you want detailed information for every step, press `y` at the end:

```bash title="Terminal"
Would you like to check the details? (y/n): y
```

Datus will then show details in a new screen.

### Step 3: Context Management

!!! note
    If you have initialized the knowledge base for metadata.

If you want to add items such as a file, metric, table, or SQL history to the context, use `@`. Manual retrieval gives the LLM more precise context, narrows the search scope, and improves answer accuracy.

```bash title="Terminal"
Datus> /@Table duckdb-demo.main.bank_failures
```

Datus will automatically analyze the table and add metadata to the context.

??? example "Context Analysis Output"
    ```text
    Processing chat request...
    +----------------------------------------------------------------------------------------------------------------------------------+
    |                                                    Action Stream                                                                 |
    +==================================================================================================================================+
    | 🟡 User: @Table duckdb-demo.main.bank_failures                                                                                  |
    | 💬 Generating response with tools...                                                                                            |
    | 💬 Thinking: I'll provide a comprehensive analysis of the bank_failures table. Let me start by examining the data structure     |
    | and then perform various analyses.                                                                                              |
    | 🔧 Tool call - describe_table('{"table_name": "bank_failures"}') - ✓                                                            |
    |     ✓ 7 columns (0.5s)                                                                                                          |
    | 💬 Thinking: Now let me perform a comprehensive analysis of the bank failures data:                                             |
    | 🔧 Tool call - read_query('{"sql": "-- Comprehensive Bank Failures Analysis\n...') - ✓                                          |
    |     ✗ Failed:(error_code=500006, e...) (0.5s)                                                                                   |
    | 💬 Thinking: Let me break this down into separate queries for a comprehensive analysis:                                         |
    | 🔧 Tool call - read_query('{"sql": "-- 1. Basic overview statistics\nSELECT \...') - ✓                                          |
    |     ✓ 1 rows (0.5s)                                                                                                             |
    | 🔧 Tool call - read_query('{"sql": "-- 2. Yearly analysis of bank failures\nS...') - ✓                                          |
    |     ✓ 14 rows (0.5s)                                                                                                            |
    | 🔧 Tool call - read_query('{"sql": "-- 3. State-wise analysis (top 15 states)...') - ✓                                          |
    |     ✓ 15 rows (0.5s)                                                                                                            |
    | 🔧 Tool call - read_query('{"sql": "-- 4. Largest bank failures (top 20 by as...') - ✓                                          |
    |     ✓ 20 rows (0.5s)                                                                                                            |
    | 🔧 Tool call - read_query('{"sql": "-- 5. Acquisition analysis (top acquirers...') - ✓                                          |
    |     ✓ 15 rows (0.5s)                                                                                                            |
    | 🔧 Tool call - read_query('{"sql": "-- 6. Asset size distribution analysis\nW...') - ✓                                          |
    |     ✓ 6 rows (0.5s)                                                                                                             |
    | 🔧 Tool call - read_query('{"sql": "-- 7. Recent failures (2023-2024)\nSELECT...') - ✓                                          |
    |     ✓ 8 rows (0.5s)                                                                                                             |
    | 💬 Chat interaction completed successfully                                                                                      |
    +----------------------------------------------------------------------------------------------------------------------------------+

    +==================================================================================================================================+
    |                                       🏦 Comprehensive Bank Failures Analysis (2008-2024)                                       |
    +==================================================================================================================================+

                                                           📊 Executive Summary

    • Total Failures: 545 banks failed between 2008-2024
    • Total Assets: $1.696 trillion in failed bank assets
    • Time Span: 16 years (January 2008 - October 2024)
    • Geographic Reach: 42 states, 419 cities affected
    • Average Bank Size: $3.11 billion


                                                        📈 Key Historical Patterns

                                                           Major Crisis Periods

    1 2008-2009 Financial Crisis: 166 failures, $938.1B assets
    • Peak: 2009 (140 failures, $169.5B assets)
    • Includes Bear Stearns ($395B) and Washington Mutual ($307B)
    2 2010-2014 Post-Crisis Cleanup: 270 failures, $150.0B assets
    • Peak: 2010 (157 failures - highest count)
    • Smaller institutions, regulatory cleanup
    3 2023 Banking Stress: 6 failures, $572.7B assets
    • Major failures: First Republic ($229B), Silicon Valley Bank ($209B), Signature Bank ($118B)


                                                        🗺️ Geographic Distribution

                                                      Top 5 States by Failure Count

    1 Georgia: 91 failures (16.7%) - $34.4B assets
    2 Florida: 73 failures (13.4%) - $39.2B assets
    3 Illinois: 66 failures (12.1%) - $37.7B assets
    4 California: 43 failures (7.9%) - $559.8B assets
    5 Minnesota: 23 failures (4.2%) - $3.2B assets

                                                       Top 5 States by Assets Lost

    1 California: $559.8B (33.0% of total)
    2 New York: $513.4B (30.3% of total)*
    3 Washington: $318.9B (18.8% of total)
    4 Illinois: $37.7B (2.2% of total)
    5 Florida: $39.2B (2.3% of total)

    *Includes Bear Stearns and Signature Bank


                                                          💰 Asset Size Analysis

                                                            Size Distribution


    Asset Range   Failures   % of Total   Total Assets   % of Assets   Avg Size
    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    Under $100M   136        24.9%        $7.7B          0.5%          $56M
    $100M-$500M   275        50.5%        $66.5B         3.9%          $242M
    $500M-$1B     57         10.5%        $40.3B         2.4%          $707M
    $1B-$5B       55         10.1%        $119.6B        7.1%          $2.2B
    $5B-$10B      7          1.3%         $43.8B         2.6%          $6.3B
    Over $10B     15         2.8%         $1,418.2B      83.6%         $94.5B


    Key Insight: While 75% of failures were under $1B in assets, the 15 largest failures (>$10B) represent 83.6% of all failed assets.


                                                         🏆 Largest Bank Failures

    1 Bear Stearns (2008) - $395.0B (J.P. Morgan Chase)
    2 Washington Mutual (2008) - $307.0B (JPMorgan Chase)
    3 First Republic Bank (2023) - $229.0B (JPMorgan Chase)
    4 Silicon Valley Bank (2023) - $209.0B (First Citizens)
    5 Signature Bank (2023) - $118.4B (Flagstar Bank)


                                                         🏛️ Acquisition Landscape

                                                        Top Acquiring Institutions

    1 U.S. Bank, N.A.: 13 acquisitions ($38.7B)
    2 State Bank and Trust: 10 acquisitions ($3.5B)
    3 Ameris Bank: 9 acquisitions ($1.9B)
    4 Stearns Bank, N.A.: 9 acquisitions ($1.7B)
    5 First-Citizens Bank: 7 acquisitions ($7.8B)

                                                             Notable Patterns

    • JPMorgan Chase: Acquired 3 institutions but captured $930B+ in assets
    • 20 banks: Had no acquirer (insured depositors paid directly)
    • Consolidation: Larger banks acquired most failed institutions


                                                       📅 Recent Trends (2023-2024)

                                                           2023 Major Failures

    • First Republic Bank: $229B (May 2023)
    • Silicon Valley Bank: $209B (March 2023)
    • Signature Bank: $118B (March 2023)
    • Silvergate Bank: $16B (March 2023)

                                                              2024 Failures

    • Republic First Bancorp: $6B (April 2024)
    • First National Bank of Lindsay: $108M (October 2024)


                                                             🎯 Key Insights

                                                      1. Systemic Risk Concentration

    • 15 largest failures (2.8% of total) represent 83.6% of failed assets
    • "Too big to fail" remains a significant concern

                                                       2. Geographic Vulnerability

    • Southeast (GA, FL) had highest failure counts
    • West Coast (CA, WA) had largest asset concentrations

                                                          3. Temporal Clustering

    • Clear crisis periods: 2008-2009, 2010-2014, 2023
    • Regulatory changes followed each crisis period

                                                             4. Size Matters

    • Most failures were small banks
    • But systemic risk comes from very large institutions

                                                         5. Acquisition Patterns

    • Large banks benefit from crisis acquisitions
    • Significant industry consolidation


                                                      ⚠️ Risk Management Implications

    1 Regulatory Focus: Largest institutions require heightened supervision
    2 Regional Monitoring: Southeast banking markets need closer oversight
    3 Crisis Preparedness: Friday intervention patterns suggest need for rapid response capabilities
    4 Systemic Monitoring: Asset concentration remains critical risk indicator

    This analysis provides a comprehensive view of US bank failures over 16 years, highlighting patterns, risks, and regulatory
    implications for the banking sector.
    Would you like to check the details? (y/n): n
    ```

!!! tip
    For more command references and options, see [CLI](../cli/introduction.md) or simply type `.help`.

## Benchmark (Docker Mode)

Now that you have Datus running, explore the Benchmark mode to evaluate performance features and capabilities.

### Step 1: Pull the Docker Image

!!! tip
    Ensure Docker is installed and running on your system before proceeding.

```bash title="Terminal"
docker pull datusai/datus-agent:0.1.13
```

### Step 2: Launch the Docker Container

!!! tip
    Demo datasets are preloaded, allowing you to quickly explore Datus capabilities without additional setup.

=== "BIRD"
    ```bash title="Terminal"
    docker run --name datus \
    --env DEEPSEEK_API_KEY=<your_api_key>  \
    -d datusai/datus-agent:0.1.13
    ```

=== "Spider 2.0-Snow"
    ```bash title="Terminal"
    docker run --name datus \
    --env DEEPSEEK_API_KEY=<your_api_key>  \
    --env SNOWFLAKE_ACCOUNT=<your_snowflake_acount>  \
    --env SNOWFLAKE_USERNAME=<your_snowflake_username>  \
    --env SNOWFLAKE_PASSWORD=<your_snowflake_password>  \
    -d datusai/datus-agent:0.1.13
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