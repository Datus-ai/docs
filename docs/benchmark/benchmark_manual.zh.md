# 基准测试（Benchmark）

使用行业标准数据集评估 Datus Agent 的表现与能力。针对 BIRD 与 Spider 2.0-Snow 等数据集运行全面测试，衡量正确率、执行成功率与查询生成质量。

## 概览

基准测试模式可帮助你：

- **衡量准确性**：评估从自然语言到 SQL 的正确率
- **跟踪成功率**：观察在不同数据库上的执行成功情况
- **对比结果**：将生成的结果与期望输出进行比对
- **发现改进点**：定位可优化与需要迭代的环节

## 快速开始

使用内置基准数据集，快速上手。

### 第一步：安装 Datus Agent

!!! tip
    详细安装与配置说明请参阅 [快速开始指南](../getting_started/quickstart.md)。

```bash title="Terminal"
pip install datus-agent
datus-agent init
```

### 第二步：配置环境变量

根据所使用的基准数据集，设置相应的环境变量：

=== "BIRD"
    ```bash title="Terminal"
    export DEEPSEEK_API_KEY=<your_api_key>
    ```

=== "Spider 2.0-Snow"
    ```bash title="Terminal"
    export DEEPSEEK_API_KEY=<your_api_key>
    export SNOWFLAKE_ACCOUNT=<your_snowflake_account>
    export SNOWFLAKE_USERNAME=<your_snowflake_username>
    export SNOWFLAKE_PASSWORD=<your_snowflake_password>
    ```

### 第三步：下载并准备 BIRD 数据集

!!! note
    此步骤仅用于 BIRD 基准测试。如需运行 Spider 2.0-Snow，请跳至 [第四步](#第四步运行基准测试)。

将 BIRD dev 数据集下载并解压到 Datus 主目录（默认为 `~/.datus`）：

```bash title="Terminal"
cd ~/.datus
wget https://bird-bench.oss-cn-beijing.aliyuncs.com/dev.zip
unzip dev.zip
mkdir -p benchmark/bird
mv dev_20240627 benchmark/bird
cd benchmark/bird/dev_20240627
unzip dev_databases
cd ~
```

解压后目录结构如下：

```text
~/.datus/
└── benchmark/
    └── bird/
        └── dev_20240627/
            ├── dev_databases/
            │   ├── california_schools/
            │   │   └── california_schools.sqlite
            │   ├── ...
            │   └── <other_databases>/
            └── ...
```

然后为 BIRD 数据集构建知识库：

```bash title="Terminal"
datus-agent bootstrap-kb --namespace bird_sqlite --benchmark bird_dev
```

### 第四步：运行基准测试

!!! warning
    单个任务可能需要数分钟；全量运行可能耗时数小时到数天，取决于环境。

**BIRD 数据集**

!!! info
    任务 ID 范围：0-1533

=== "按任务 ID 运行"
    ```bash title="Terminal"
    datus-agent benchmark \
    --namespace bird_sqlite \
    --benchmark bird_dev \
    --benchmark_task_ids <task_id1> <task_id2>
    ```

=== "运行全部任务"
    ```bash title="Terminal"
    datus-agent benchmark \
    --namespace bird_sqlite \
    --benchmark bird_dev
    ```

**Spider 2.0-Snow 数据集**

!!! info
    任务 ID（instance ID）可在 [spider2-snow.jsonl](https://github.com/xlang-ai/Spider2/blob/main/spider2-snow/spider2-snow.jsonl) 中查询。

!!! note
    运行 Spider 2.0-Snow 基准测试前，请确保已配置 Snowflake 相关环境变量。

=== "按任务 ID 运行"
    ```bash title="Terminal"
    datus-agent benchmark \
    --namespace snowflake \
    --benchmark spider2 \
    --benchmark_task_ids <task_id1> <task_id2>
    ```

=== "运行全部任务"
    ```bash title="Terminal"
    datus-agent benchmark \
    --namespace snowflake \
    --benchmark spider2
    ```

### 第五步：结果评估

#### 运行评估
=== "按任务 ID 进行评估"
```bash title="Run Evaluation"
datus-agent eval \
  --namespace snowflake \
  --benchmark spider2 \
  --output_file evaluation.json \
  --task_ids <task_id1> <task_id2>
```
=== "评估全部"
```bash title="Run Evaluation"
datus-agent eval \
  --namespace snowflake \
  --output_file evaluation.json \
  --benchmark spider2
```

#### 评估结果

=== "结果摘要"
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
=== "结果详情"
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
          "actual_path": "<USER_HOME>/.datus/save/bird_school/0.csv",
          "gold_path": "<USER_HOME>/.datus/benchmark/california_schools/california_schools.csv",
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

`details` 关键字段说明：
- **comparison_results**: 比较结果
  - `actual_sql`: 由 Datus Agent 生成的 SQL
  - `actual_path`: 由 Datus Agent 生成的 SQL 执行后的结果
  - `gold_sql`: 标准答案 SQL
  - `gold_path`: 标准答案的结果
  - `comparison`: 对比结果：
    - `match_rate`: 匹配度
    - `matched_columns`: 匹配成功的列
    - `missing_columns`: 未匹配到的列
    - `extra_columns`: 标准答案不存在的列
    - `actual_tables`: Datus Agent 生成结果用到的表
    - `expected_tables`: 标准答案中用到的表
    - `matched_tables`: 匹配的表
- **tool_calls**: 记录各工具的调用次数

## 内置数据集快速体验

使用预置的 California Schools 数据集快速体验基准测试与评估流程，无需额外下载。

### 第一步：初始化数据集
```bash
datus-agent tutorial
```

![tutorial](../assets/tutorial.png)

该步骤做了以下工作：

1. 增加 California Schools 的数据库配置和基准测试配置到 agent.yml
2. 初始化 California Schools 的表的元数据信息
3. 使用 `benchmark/california_schools/success_story.csv` 构建指标信息
4. 使用 `benchmark/california_schools/reference_sql` 的 SQL 文件构建参考 SQL

### 进行基准测试和评估
```bash
datus-agent benchmark --namespace california_schools --benchmark california_schools --benchmark_task_ids 0 1 2 --workflow <your workflow>
```
👉 详见 [第四步：运行基准测试](#第四步运行基准测试)

```bash
datus-agent eval --namespace california_schools --benchmark california_schools --task_ids 0 1 2
```
👉 详见 [第五步：结果评估](#第五步结果评估)

## 自定义基准测试
> 若希望基于自定义数据集进行测试，请参考以下配置示例。

### 添加 Benchmark 配置
```yaml
agent:
  namespace:
    california_schools:
      type: sqlite
      name: california_schools
      uri: sqlite:///benchmark/bird/dev_20240627/dev_databases/california_schools/california_schools.sqlite # 数据库文件路径，sqlite:///为相对路径； sqlite:////为绝对路径
  benchmark:
    california_schools:              # benchmark 名称
      question_file: california_schools.csv       # 存放测试问题的文件
      question_id_key: task_id                    # 问题ID的字段名
      question_key: question                      # 问题内容的字段名
      ext_knowledge_key: expected_knowledge       # 扩展知识/问题说明字段名

      # 以下是评估阶段使用的配置
      gold_sql_path: california_schools.csv       # 标准 SQL 的文件路径
      gold_sql_key: gold_sql                      # 对应标准 SQL 的键名
      gold_result_path: california_schools.csv    # 标准答案结果文件路径
      gold_result_key: ""                         # 当结果文件为单文件时，对应的字段名
```
---

📖 Benchmark 配置字段说明

| 字段                    | 说明                                                                                                                                     |
|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| **question_file**     | 题目文件路径，支持 `.csv`、`.json`、`.jsonl` 格式。路径相对于 `{agent.home}/benchmark/{benchmark_name}`。                                                  |
| **question_id_key**   | 每个任务问题的唯一标识字段名。                                                                                                                        |
| **question_key**      | 自然语言问题的字段名。                                                                                                                            |
| **ext_knowledge_key** | 附加知识或问题说明字段名。                                                                                                                          |
| **gold_sql_path**     | 标准 SQL 文件路径，支持两种情况：<br/>1. 例如BIRD_DEV的单一文件；支持 `.csv`、`.json`、`.jsonl` 格式。<br/>2. 每个任务独立一个SQL文件（例如spider2）。                            |
| **gold_sql_key**      | 当 `gold_sql_path` 为单文件时，标准 SQL 所在字段名。                                                                                                  |
| **gold_result_path**  | 标准结果文件路径，每个结果应是CSV格式的字符串。支持三种情况：<br/>1. 单一`.csv`、`.json`、`.jsonl` 格式的文件；<br/>2. 每个任务独立一个CSV文件（例如spider2）。<br/>3. 不配置，系统会执行标准 SQL 获取结果。 |
| **gold_result_key**   | 标准结果字段，当`gold_result_path`为单文件时，该字段有意义且必须配置。                                                                                           |

---

### 构建基础知识库

根据你的情况，构建metadata、metrics和reference_sql。具体参考 [knowledge_base/introduction](../knowledge_base/introduction.md)

### 运行基准测试
👉 详见 [第四步：运行基准测试](#第四步运行基准测试)

### 结果评估
👉 详见 [第五步：结果评估](#第五步结果评估)


## 多轮基准测试与评估

重复执行基准测试 + 评估流程（即 [第四步](#第四步运行基准测试) 和 [第五步](#第五步结果评估)），并对比各轮次的结果。适用于衡量 Agent 性能的稳定性与一致性。

### 用法

=== "datus-agent 子命令"
    ```bash title="Terminal"
    datus-agent multi-round-benchmark \
      --namespace bird_sqlite \
      --benchmark bird_dev \
      --workflow chat_agentic \
      --round 4 \
      --workers 2
    ```

=== "独立 CLI 入口"
    ```bash title="Terminal"
    datus-multi-benchmark \
      --namespace bird_sqlite \
      --benchmark bird_dev \
      --workflow chat_agentic \
      --round 4 \
      --workers 2
    ```

=== "Python 模块"
    ```bash title="Terminal"
    python -m datus.multi_round_benchmark \
      --namespace bird_sqlite \
      --benchmark bird_dev \
      --workflow chat_agentic \
      --round 4 \
      --workers 2
    ```

### 参数说明

| 参数                      | 必填  | 默认值                                            | 说明                              |
|-------------------------|-----|------------------------------------------------|---------------------------------|
| `--namespace`           | 是   | —                                              | 基准测试命名空间，如 `bird_sqlite`        |
| `--benchmark`           | 是   | —                                              | 基准测试名称，如 `bird_dev`             |
| `--workflow`            | 否   | `reflection`                                   | 要执行的工作流                         |
| `--round`               | 否   | `4`                                            | 基准测试迭代轮数                        |
| `--max_steps`           | 否   | `30`                                           | 每次工作流执行的最大步数                    |
| `--workers`             | 否   | `1`                                            | 并行工作线程数                         |
| `--task_ids`            | 否   | 全部任务                                           | 指定要测试的任务 ID（空格或逗号分隔）            |
| `--group_name`          | 否   | workflow 名称                                     | 集成测试组名称，用于输出目录命名                |
| `--delete_history`      | 否   | `false`                                        | 每轮开始前删除已有的轮次输出目录                |
| `--summary_report_file` | 否   | —                                              | 汇总报告文件路径，每轮结果会追加写入              |
| `--debug`               | 否   | `false`                                        | 启用 debug 级别日志                   |
| `--config`              | 否   | `~/.datus/conf/agent.yml` 或 `conf/agent.yml`   | Agent 配置文件路径                    |

> **注意：** `--round`、`--max_steps`、`--workers`、`--task_ids`、`--group_name`、`--delete_history` 和
> `--summary_report_file` 同时支持 kebab-case 写法（如 `--max-steps`）及旧版参数名
>（如 `--max_round`、`--max_workers`），以保持向后兼容。

### 输出结构

`{timestamp}` 为多轮基准测试开始时间，格式为 `YYYYMMDD_HHmm`（如 `20260129_1530`）。`{ts}` 为 Unix 时间戳（秒），如 `1769611836`。

每轮测试在 `{agent.home}/integration/` 下创建独立的输出目录。全部轮次完成后，还会导出一份汇总 Excel 文件：

```text
{agent.home}/integration/
├── {group_name}_0/
│   ├── save/{namespace}/{timestamp}/
│   │   ├── 0.json                                     # 任务元数据
│   │   ├── 0.sql                                      # 生成的 SQL
│   │   ├── 0.csv                                      # 查询执行结果
│   │   ├── 1.json
│   │   ├── 1.sql
│   │   ├── 1.csv
│   │   └── ...
│   ├── trajectory/{namespace}/{timestamp}/
│   │   ├── 0_{ts}.yaml                                # 工作流轨迹（如 0_1769611836.yaml）
│   │   ├── 1_{ts}.yaml
│   │   └── ...
│   └── evaluation_round_{timestamp}_0.json            # 第 0 轮评估报告
├── {group_name}_1/
│   ├── save/{namespace}/{timestamp}/
│   │   ├── 0.json
│   │   ├── 0.sql
│   │   ├── 0.csv
│   │   ├── 1.json
│   │   ├── 1.sql
│   │   ├── 1.csv
│   │   └── ...
│   ├── trajectory/{namespace}/{timestamp}/
│   │   ├── 0_{ts}.yaml
│   │   ├── 1_{ts}.yaml
│   │   └── ...
│   └── evaluation_round_{timestamp}_1.json
├── ...
└── {group_name}_summary_{timestamp}.xlsx              # 跨轮次汇总报告
```

汇总 Excel 文件包含跨轮次对比表：

| task_id                  | round_0         | round_1         | ... | Matching Rate |
|--------------------------|-----------------|-----------------|-----|---------------|
| 0                        | Matched         | Result Mismatch | ... | 50.00%        |
| 1                        | Column Mismatch | Matched         | ... | 50.00%        |
| Summary of Matching Rate | 50.00%          | 50.00%          | ... | 50.00%        |
| Round Duration           | 5m 30s          | 5m 12s          | ... |               |

### 任务状态定义

每轮中的每个任务会被归入以下状态之一，按评估优先级从高到低排列（匹配到的第一个状态即为最终状态）：

| 状态              | 说明                                    |
|-------------------|---------------------------------------|
| `Not Executed`    | 基准测试或评估出现异常，任务未被处理                    |
| `Matched`         | 生成的 SQL 结果与标准答案匹配                     |
| `Gen SQL Failed`  | Agent 生成的 SQL 在数据库中执行失败                |
| `Gold SQL Failed` | 标准 SQL 执行失败——请检查配置或标准 SQL 的正确性        |
| `Match Failed`    | 未获取到评估结果，可能由未捕获的异常导致                  |
| `Table Mismatch`  | Agent 定位的表不准确（错误或缺失）                   |
| `Result Mismatch` | 表匹配正确，但行或列的值与期望输出不一致                  |
| `Column Mismatch` | 表和行匹配正确，但部分列不同或缺失                     |
