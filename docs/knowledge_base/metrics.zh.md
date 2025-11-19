# 业务指标智能化

指标组件专注于从历史 SQL 查询中自动提取和生成业务指标，建立企业级指标管理系统。

## 核心价值

解决常见的企业挑战：
- **重复的 SQL 查询**：复用指标而非重写相似查询
- **不一致的定义**：跨团队标准化指标定义
- **手动分类**：使用层级主题树分类体系组织指标

## 工作原理

数据流经两层：**历史 SQLs → 语义模型 → 业务指标**

语义模型定义表结构、维度和度量。指标是基于这些模型构建的可复用计算。

## 使用方法

**前置条件**：此命令依赖 [datus-metricflow](../metricflow/introduction.md)，请先安装。

### 基本命令

```bash
# 从 CSV（历史 SQLs）
datus-agent bootstrap-kb \
    --namespace <your_namespace> \
    --components metrics \
    --success_story path/to/success_story.csv \
    --metric_meta business_meta

# 从 YAML（语义模型）
datus-agent bootstrap-kb \
    --namespace <your_namespace> \
    --components metrics \
    --semantic_yaml path/to/semantic_model.yaml \
    --metric_meta business_meta
```

### 关键参数

| 参数 | 必需 | 描述 | 示例 |
|-----------|----------|-------------|------------|
| `--namespace` | ✅ | 数据库命名空间 | `sales_db` |
| `--components` | ✅ | 要初始化的组件 | `metrics` |
| `--success_story` | ⚠️ | 包含历史 SQLs 和问题的 CSV 文件（如果没有 `--semantic_yaml` 则必需） | `success_story.csv` |
| `--semantic_yaml` | ⚠️ | 语义模型 YAML 文件（如果没有 `--success_story` 则必需） | `semantic_model.yaml` |
| `--metric_meta` | ✅ | `agent.yml` 中的指标元数据配置 | `business_meta` |
| `--kb_update_strategy` | ✅ | 更新策略 | `overwrite`/`incremental` |
| `--subject_tree` | ❌ | 预定义分类（逗号分隔） | `Sales/Reporting/Daily,Finance/Revenue/Monthly` |
| `--pool_size` | ❌ | 并发线程数 | `4` |

### 主题树分类

使用层级分类法组织指标：`domain/layer1/layer2`（例如 `Sales/Reporting/Daily`）

**两种模式：**
- **预定义**：使用 `--subject_tree` 强制指定特定分类
- **学习**：省略 `--subject_tree` 以复用现有分类或创建新分类

```bash
# 预定义模式示例
--subject_tree "Sales/Reporting/Daily,Finance/Revenue/Monthly"

# 学习模式：省略 --subject_tree 参数
```

**生成的标签格式：**

指标生成后，主题树分类存储在 `locked_metadata.tags` 中，格式为 `"subject_tree: {domain}/{layer1}/{layer2}"`：

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

**YAML 导入注意事项：**

使用 `--semantic_yaml` 从 YAML 文件同步指标到 lancedb 时，必须在 YAML 文件中手动添加包含 subject_tree 格式的 `locked_metadata.tags` 才能成功分类。系统不会自动对从 YAML 导入的指标进行分类，需要自行添加标签：

```yaml
metric:
  name: your_metric
  # ... 其他字段
  locked_metadata:
    tags:
      - "YourDomain"
      - "subject_tree: Domain/Layer1/Layer2"
```

## 数据源格式

### CSV 格式

```csv
question,sql
How many customers have been added per day?,"SELECT ds AS date, SUM(1) AS new_customers FROM customers GROUP BY ds ORDER BY ds;"
What is the total transaction amount?,SELECT SUM(transaction_amount_usd) as total_amount FROM transactions;
```

### YAML 格式

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

## 总结

指标组件从历史 SQLs 建立标准化、可复用的指标定义。通过层级主题树分类法和灵活的分类模式，帮助团队维护一致、可发现的指标，用于数据驱动的决策制定。
