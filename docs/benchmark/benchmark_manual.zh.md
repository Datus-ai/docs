# 基准测试（Benchmark）

使用行业标准数据集评估 Datus Agent 的表现与能力。针对 BIRD 与 Spider 2.0-Snow 等数据集运行全面测试，衡量正确率、执行成功率与查询生成质量。

## 概览

基准测试模式可帮助你：

- **衡量准确性**：评估从自然语言到 SQL 的正确率
- **跟踪成功率**：观察在不同数据库上的执行成功情况
- **对比结果**：将生成的结果与期望输出进行比对
- **发现改进点**：定位可优化与需要迭代的环节

## Docker 快速开始

使用预配置 Docker 镜像，内置基准数据集，快速上手。

### 第一步：拉取镜像

!!! tip
    开始前请确保已安装并启动 Docker。

```bash title="Terminal"
docker pull datusai/datus-agent:0.2.0-rc1
```

### 第二步：启动容器

!!! tip
    Demo 数据集已预置，便于快速体验。

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

### 第三步：运行基准测试

!!! warning
    单个任务可能需要数分钟；全量运行可能耗时数小时到数天，取决于环境。

**BIRD 数据集**

!!! info
    任务 ID 范围：0-1533

=== "按任务 ID 运行"
```bash title="Terminal"
docker exec -it datus python -m datus.main benchmark  \
  --namespace bird_sqlite \
  --benchmark bird_dev \
  --benchmark_task_ids <task_id>
```

=== "运行全部任务"
```bash title="Terminal"
docker exec -it datus python -m datus.main benchmark  \
  --namespace bird_sqlite \
  --benchmark bird_dev
```

**Spider 2.0-Snow 数据集**

!!! info
    任务 ID（instance ID）可在 [spider2-snow.jsonl](https://github.com/xlang-ai/Spider2/blob/main/spider2-snow/spider2-snow.jsonl) 中查询。

!!! note
    启动容器时需配置 Snowflake 相关环境参数。

=== "按任务 ID 运行"
```bash title="Terminal"
docker exec -it datus python -m datus.main benchmark  \
  --namespace snowflake \
  --benchmark spider2 \
  --benchmark_task_ids <task_id>
```

=== "运行全部任务"
```bash title="Terminal"
docker exec -it datus python -m datus.main benchmark  \
  --namespace snowflake \
  --benchmark spider2
```

### 第四步：查看结果

!!! tip
    每次基准运行都会生成详细的性能概览与任务明细。

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
Final Result: {...}
```
