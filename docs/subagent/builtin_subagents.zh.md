# 内置 Subagent

## 概览

**内置 Subagent**  是集成在 Datus Agent 系统中的专用 AI 助手。每个subagent专注于数据工程自动化的特定方面——分析 SQL、生成语义模型、将查询转换为可复用指标——共同构成从原始 SQL 到具备知识感知的数据产品的闭环工作流。

本文档涵盖三个核心subagent：

1. **[gen_sql_summary](#gen_sql_summary)** — 总结和分类 SQL 查询
2. **[gen_semantic_model](#gen_semantic_model)** — 生成 MetricFlow 语义模型
3. **[gen_metrics](#gen_metrics)** — 生成 MetricFlow 指标定义

## 配置

内置subagent开箱即用，默认设置即可工作。你可以在 `agent.yml` 文件中自定义：

```yaml
agent:
  agentic_nodes:
    gen_semantic_model:
      model: anthropic  # 可选：指定使用的 AI 模型
      max_turns: 10     # 可选：最大对话轮数（默认：30）

    gen_metrics:
      model: anthropic
      max_turns: 8

    gen_sql_summary:
      model: deepseek_v3
      max_turns: 5
```

**可选配置参数**：

- `model`：此subagent使用的 AI 模型（例如 `anthropic`、`deepseek_v3`）
- `max_turns`：subagent完成前的最大对话轮数

---

## gen_sql_summary

### 概览

SQL 摘要功能帮助你分析、分类和编目 SQL 查询，用于知识复用。它自动生成结构化的 YAML 摘要，存储在可搜索的知识库中，便于将来查找和复用相似的查询。

### 什么是 SQL 摘要？

**SQL 摘要** 是一个结构化 YAML 文档，包含：

- **查询文本**：完整的 SQL 查询
- **业务上下文**：域、类别和标签
- **语义摘要**：用于向量搜索的详细说明
- **元数据**：名称、注释、文件路径

### 快速开始

启动 SQL 摘要生成 subagent：

```bash
/gen_sql_summary Analyze this SQL: SELECT SUM(revenue) FROM sales GROUP BY region. (You can also add some description on this SQL)
```

### 生成工作流

```mermaid
graph LR
    A[用户提供 SQL + 描述] --> B[智能体分析查询]
    B --> C[检索上下文]
    C --> D[生成唯一 ID]
    D --> E[创建 YAML]
    E --> F[保存文件]
    F --> G[用户确认]
    G --> H[同步到知识库]
```

**详细步骤**：

1. **理解 SQL**：AI 分析你的查询结构和业务逻辑
2. **获取上下文**：调用 `prepare_sql_summary_context()` 检索：
   - 现有分类体系（域、类别、标签）
   - 类似的 SQL 摘要用于分类参考
3. **生成唯一 ID**：使用 `generate_sql_summary_id()` 基于 SQL + 注释生成
4. **创建唯一名称**：生成描述性名称（最多 20 个字符）
5. **分类查询**：按照现有模式分配域、layer1、layer2 和标签
6. **生成 YAML**：创建结构化摘要文档
7. **保存文件**：使用 `write_file()` 将 YAML 写入工作空间
8. **用户确认**：显示生成的 YAML 并提示批准
9. **同步到知识库**：存储到 LanceDB 用于语义搜索

### 交互式确认

生成后，你会看到：

```
==========================================================
Generated Reference SQL YAML
File: /path/to/sql_summary.yml
==========================================================
[带语法高亮的 YAML 内容]

  SYNC TO KNOWLEDGE BASE?

  1. Yes - Save to Knowledge Base
  2. No - Keep file only

Please enter your choice: [1/2]
```

### 配置

#### 智能体配置

在 `agent.yml` 中：

```yaml
agentic_nodes:
  gen_sql_summary:
    model: deepseek                        # 用于分析的 LLM 模型
    system_prompt: gen_sql_summary         # 提示模板
    prompt_version: "1.0"
    tools: generation_tools.prepare_sql_summary_context, generation_tools.generate_sql_summary_id, filesystem_tools.write_file
    hooks: generation_hooks                # 启用确认工作流
    workspace_root: /path/to/reference_sql   # 保存 YAML 文件的目录
    agent_description: "reference SQL analysis assistant"
```

#### 关键配置选项

| 参数 | 描述 | 示例 |
|-----------|-------------|---------|
| `model` | 用于 SQL 分析的 LLM 模型 | `deepseek`、`claude`、`openai`、`kimi` |
| `workspace_root` | SQL 摘要 YAML 文件的目录 | `/Users/you/.datus/data/reference_sql` |
| `tools` | 工作流所需的工具 | 见下方工具部分 |
| `hooks` | 启用交互式确认 | `generation_hooks` |

### YAML 结构

生成的 SQL 摘要遵循以下结构：

```yaml
id: "abc123def456..."                      # 自动生成的 MD5 哈希
name: "Revenue by Region"                  # 描述性名称（最多 20 个字符）
sql: |                                     # 完整 SQL 查询
  SELECT
    region,
    SUM(revenue) as total_revenue
  FROM sales
  GROUP BY region
comment: "Calculate total revenue grouped by region"
summary: "This query aggregates total revenue from the sales table, grouping results by geographic region. It uses SUM aggregation to calculate revenue totals for each region."
filepath: "/Users/you/.datus/data/reference_sql/revenue_by_region.yml"
domain: "Sales"                            # 业务域
layer1: "Reporting"                        # 主要类别
layer2: "Revenue Analysis"                 # 次要类别
tags: "revenue, region, aggregation"       # 逗号分隔的标签
```

#### 字段说明

| 字段 | 必需 | 描述 | 示例 |
|-------|----------|-------------|---------|
| `id` | 是 | 唯一哈希（自动生成） | `abc123def456...` |
| `name` | 是 | 简短描述性名称（最多 20 个字符） | `Revenue by Region` |
| `sql` | 是 | 完整 SQL 查询 | `SELECT ...` |
| `comment` | 是 | 简短的单行描述 | 用户消息或生成的摘要 |
| `summary` | 是 | 详细说明（用于搜索） | 全面的查询描述 |
| `filepath` | 是 | 实际文件路径 | `/path/to/file.yml` |
| `domain` | 是 | 业务域 | `Sales`、`Marketing`、`Finance` |
| `layer1` | 是 | 主要类别 | `Reporting`、`Analytics`、`ETL` |
| `layer2` | 是 | 次要类别 | `Revenue Analysis`、`Customer Insights` |
| `tags` | 可选 | 逗号分隔的关键字 | `revenue, region, aggregation` |

---

## gen_semantic_model

### 概览

语义模型生成功能帮助你通过 AI 助手从数据库表创建 MetricFlow 语义模型。助手分析你的表结构并生成全面的 YAML 配置文件，定义指标、维度和关系。

### 什么是语义模型？

语义模型是定义以下内容的 YAML 配置：

- **度量（Measures）**：指标和聚合（SUM、COUNT、AVERAGE 等）
- **维度（Dimensions）**：分类和时间属性
- **标识符（Identifiers）**：用于关系的主键和外键
- **数据源（Data Source）**：与数据库表的连接

### 快速开始

使用 `datus --namespace <namespace>` 启动 Datus CLI，然后使用subagent命令：

```bash
/gen_semantic_model generate a semantic model for table <table_name>
```

### 工作原理

#### 交互式生成

当你请求语义模型时，AI 助手会：

1. 检索你的表的 DDL（结构）
2. 检查是否已存在语义模型
3. 生成全面的 YAML 文件
4. 使用 MetricFlow 验证配置
5. 提示你保存到知识库

#### 生成工作流

```mermaid
graph LR
    A[用户请求] --> B[DDL 分析]
    B --> C[YAML 生成]
    C --> D[验证]
    D --> E[用户确认]
    E --> F[存储]
```

### 交互式确认

生成语义模型后，你会看到：

```
=============================================================
Generated YAML: table_name.yml
Path: /path/to/file.yml
=============================================================
[带语法高亮的 YAML 内容]

SYNC TO KNOWLEDGE BASE?

1. Yes - Save to Knowledge Base
2. No - Keep file only

Please enter your choice: [1/2]
```

**选项**：

- **选项 1**：将语义模型保存到你的知识库（RAG 存储）用于 AI 驱动的查询
- **选项 2**：仅保留 YAML 文件，不同步到知识库

### 配置

#### 智能体配置

在 `agent.yml` 中，配置语义模型生成节点：

```yaml
agentic_nodes:
  gen_semantic_model:
    model: claude                    # 使用的 LLM 模型
    system_prompt: gen_semantic_model
    prompt_version: "1.0"
    tools: db_tools.*, generation_tools.*, filesystem_tools.*
    hooks: generation_hooks          # 启用用户确认工作流
    mcp: metricflow_mcp             # MetricFlow 验证服务器
    workspace_root: /path/to/semantic_models
    agent_description: "Semantic model generation assistant"
    rules:
      - Use get_table_ddl tool to get complete table DDL
      - Generate comprehensive semantic models
      - Validate using metricflow_mcp
```

#### 关键配置选项

| 参数 | 描述 | 示例 |
|-----------|-------------|---------|
| `model` | 用于生成的 LLM 模型 | `claude`、`deepseek`（推荐使用 claude） |
| `workspace_root` | 保存 YAML 文件的目录 | `/Users/you/.datus/data/semantic_models` |
| `tools` | 助手可用的工具 | `db_tools.*`、`filesystem_tools.*` |
| `hooks` | 启用用户确认 | `generation_hooks` |
| `mcp` | MetricFlow 验证服务器 | `metricflow_mcp` |

### 语义模型结构

#### 基本模板

```yaml
data_source:
  name: table_name                    # 必需：小写加下划线
  description: "Table description"

  sql_table: schema.table_name        # 对于有 schema 的数据库
  # OR
  sql_query: |                        # 对于自定义查询
    SELECT * FROM table_name

  measures:
    - name: total_amount              # 必需
      agg: SUM                        # 必需：SUM|COUNT|AVERAGE|etc.
      expr: amount_column             # 列或 SQL 表达式
      create_metric: true             # 自动创建可查询指标
      description: "Total transaction amount"

  dimensions:
    - name: created_date
      type: TIME                      # 必需：TIME|CATEGORICAL
      type_params:
        is_primary: true              # 需要一个主时间维度
        time_granularity: DAY         # TIME 必需：DAY|WEEK|MONTH|etc.

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

### 总结

语义模型生成功能提供：

- ✅ 从表 DDL 自动生成 YAML
- ✅ 交互式验证和错误修复
- ✅ 存储前用户确认
- ✅ 知识库集成
- ✅ 防止重复
- ✅ MetricFlow 兼容性

---

## gen_metrics

### 概览

指标生成功能帮助你将 SQL 查询转换为可复用的 MetricFlow 指标定义。使用 AI 助手，你可以分析 SQL 业务逻辑并自动生成标准化的 YAML 指标配置，组织内可一致查询。

### 什么是指标？

**指标**是基于语义模型构建的可复用业务计算。指标提供：

- **一致的业务逻辑**：一次定义，到处使用
- **类型安全**：已验证的结构和度量引用
- **元数据**：显示名称、格式、业务上下文
- **可组合性**：从简单指标构建复杂指标

**示例**：与其重复编写 `SELECT SUM(revenue) / COUNT(DISTINCT customer_id)`，不如定义一次 `avg_customer_revenue` 指标。

### 快速开始

使用 `datus --namespace <namespace>` 启动 Datus CLI，然后使用指标生成subagent：

```bash
/gen_metrics Generate a metric from this SQL: SELECT SUM(amount) FROM transactions, the corresponding question is total amount of all transactions
```

### 工作原理

#### 生成工作流

```mermaid
graph LR
    A[用户提供 SQL 和问题] --> B[智能体分析逻辑]
    B --> C[查找语义模型]
    C --> D[读取度量]
    D --> E[检查重复]
    E --> F[生成指标 YAML]
    F --> G[追加到文件]
    G --> H[验证]
    H --> I[用户确认]
    I --> J[同步到知识库]
```

#### 重要限制

> **⚠️ 仅支持单表查询**
>
> 当前版本**仅支持从单表 SQL 查询生成指标**。不支持多表 JOIN。

**支持**：
```sql
SELECT SUM(revenue) FROM transactions WHERE status = 'completed'
SELECT COUNT(DISTINCT customer_id) / COUNT(*) FROM orders
```

**不支持**：
```sql
SELECT SUM(o.amount)
FROM orders o
JOIN customers c ON o.customer_id = c.id  -- ❌ 不支持 JOIN
```

### 交互式确认

生成后，你会看到：

```
==========================================================
Generated YAML: transactions.yml
Path: /Users/you/.datus/data/semantic_models/transactions.yml
==========================================================
[带语法高亮的 YAML 内容，显示新指标]

  SYNC TO KNOWLEDGE BASE?

  1. Yes - Save to Knowledge Base
  2. No - Keep file only

Please enter your choice: [1/2]
```

**选项**：
- **选项 1**：将指标同步到你的知识库，用于 AI 驱动的语义搜索
- **选项 2**：仅保留 YAML 文件，不同步到知识库

### 配置

#### 智能体配置

在 `agent.yml` 中，配置指标生成节点：

```yaml
agentic_nodes:
  gen_metrics:
    model: claude                          # 用于指标生成的 LLM 模型
    system_prompt: gen_metrics             # 提示模板名称
    prompt_version: "1.0"                  # 模板版本
    tools: generation_tools.*, filesystem_tools.*
    hooks: generation_hooks                # 启用用户确认工作流
    mcp: metricflow_mcp                    # MetricFlow 验证服务器
    max_turns: 40                          # 最大对话轮数
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

#### 关键配置选项

| 参数 | 描述 | 示例 |
|-----------|-------------|---------|
| `model` | 用于生成的 LLM 模型 | `claude`、`openai`、`deepseek`（推荐使用 claude） |
| `workspace_root` | 包含语义模型 YAML 文件的目录 | `/Users/you/.datus/data/semantic_models` |
| `tools` | 可用工具（生成 + 文件系统） | `generation_tools.*, filesystem_tools.*` |
| `hooks` | 启用交互式确认工作流 | `generation_hooks` |
| `mcp` | MetricFlow 验证服务器 | `metricflow_mcp` |
| `max_turns` | 复杂查询的最大对话轮数 | `40` |

### 使用示例

#### 示例 1：简单聚合

**用户输入**：
```bash
/gen_metrics Generate a metric for total order count
```

**智能体操作**：
1. 查找 `orders.yml` 语义模型
2. 读取文件以发现 `order_count` 度量
3. 生成 MEASURE_PROXY 指标：

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

#### 示例 2：转化率

**用户输入**：
```bash
/gen_metrics Create a metric from this SQL:
SELECT
  COUNT(DISTINCT CASE WHEN status = 'completed' THEN order_id END) /
  COUNT(DISTINCT order_id) AS completion_rate
FROM orders
```

**智能体操作**：
1. 分析 SQL 逻辑（带条件计数的比率）
2. 查找 `orders.yml` 并读取度量
3. 生成带约束的 RATIO 指标：

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

#### 示例 3：复杂计算

**用户输入**：
```bash
/gen_metrics Generate average basket size metric:
SELECT SUM(total_amount) / COUNT(DISTINCT order_id)
FROM order_items
```

**智能体操作**：
1. 定位 `order_items.yml`
2. 识别为 RATIO（平均值）
3. 生成指标：

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

### 指标存储方式

#### 文件组织

使用 YAML 文档分隔符 `---` 将指标追加到现有语义模型文件：

```yaml
# 现有语义模型
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
# 第一个指标（追加）
metric:
  name: total_revenue
  type: measure_proxy
  type_params:
    measure: revenue

---
# 第二个指标（追加）
metric:
  name: avg_transaction_value
  type: ratio
  type_params:
    numerator: revenue
    denominator: transaction_count
```

**为什么追加而不是单独文件？**
- 保持相关指标靠近其语义模型
- 更易于维护和验证
- MetricFlow 可以一起验证所有定义

#### 知识库存储

当你选择 "1. Yes - Save to Knowledge Base" 时，指标会存储到向量数据库中，包含：

1. **元数据**：名称、描述、类型、域/层级分类
2. **LLM 文本**：用于语义搜索的自然语言表示
3. **引用**：关联的语义模型名称
4. **时间戳**：创建日期

### 总结

指标生成功能提供：

- ✅ **SQL 到指标转换**：分析 SQL 查询并生成 MetricFlow 指标
- ✅ **智能类型检测**：自动选择正确的指标类型
- ✅ **防止重复**：生成前检查现有指标
- ✅ **验证**：MetricFlow 验证确保正确性
- ✅ **交互式工作流**：同步前审阅和批准
- ✅ **知识库集成**：语义搜索以发现指标
- ✅ **文件管理**：安全地追加到现有语义模型文件

---

## 总结

| subagent | 用途 | 输出 | 存储位置 | 亮点 |
|----------|---------|--------|-----------|------------|
| `gen_sql_summary` | 总结和分类 SQL 查询 | YAML（SQL 摘要） | `/data/reference_sql` | 基于分类体系的分类 |
| `gen_semantic_model` | 从表生成语义模型 | YAML（语义模型） | `/data/semantic_models` | DDL → MetricFlow 兼容模型 |
| `gen_metrics` | 从 SQL 生成指标 | YAML（指标） | `/data/semantic_models` | SQL → MetricFlow 指标 |

这些subagent共同自动化了 **数据工程知识管道** ——从 **查询理解 → 模型定义 → 指标生成 → 可搜索的知识库** 。

