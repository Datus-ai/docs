# SQL 摘要生成指南

## 概览

SQL 摘要功能帮助你分析、分类和编目 SQL 查询，用于知识复用。它自动生成结构化的 YAML 摘要，存储在可搜索的知识库中，便于将来查找和复用相似的查询。

## 什么是 SQL 摘要？

**SQL 摘要**是一个结构化 YAML 文档，包含：

- **查询文本**：完整的 SQL 查询
- **业务上下文**：域、类别和标签
- **语义摘要**：用于向量搜索的详细说明
- **元数据**：名称、注释、文件路径

**使用场景**：
- 构建可搜索的 SQL 查询库
- 在团队间共享和复用已验证的查询
- 文档化复杂查询模式
- 启用语义搜索："查找与收入分析相关的查询"

## 快速开始

启动 SQL 摘要生成子代理：

```bash
/gen_sql_summary Analyze this SQL: SELECT SUM(revenue) FROM sales GROUP BY region.(You can also add some description on this SQL)
```

## 工作原理

### 生成工作流

```
用户提供 SQL + 描述 → 智能体分析查询 → 检索上下文（分类体系 + 相似查询） →
生成唯一 ID → 创建 YAML → 保存文件 → 用户确认 → 同步到知识库
```

### 步骤详解

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

## 配置

### 智能体配置

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

### 关键配置选项

| 参数 | 描述 | 示例 |
|-----------|-------------|---------|
| `model` | 用于 SQL 分析的 LLM 模型 | `deepseek`、`claude`、`openai`、`kimi`|
| `workspace_root` | SQL 摘要 YAML 文件的目录 | `/Users/you/.datus/data/reference_sql` |
| `tools` | 工作流所需的工具 | 见下方工具部分 |
| `hooks` | 启用交互式确认 | `generation_hooks` |

## YAML 结构

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

### 字段说明

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


## 总结

SQL 摘要功能提供：

✅ **自动化分析**：AI 理解查询结构和目的
✅ **智能分类**：基于分类体系的一致性分类
✅ **去重**：基于哈希的自动重复检测
✅ **语义搜索**：向量嵌入支持智能查询发现
✅ **语言保持**：维护中英文一致性
✅ **交互式工作流**：同步前审阅和批准
✅ **知识复用**：构建可搜索的 SQL 查询库

