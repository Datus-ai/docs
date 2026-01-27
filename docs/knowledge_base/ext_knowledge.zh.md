# 外部知识智能化

## 概述

Bootstrap-KB 外部知识是一个处理、存储和索引领域特定业务知识的组件，用于创建智能可搜索的知识库。它将业务规则和概念转化为具有语义搜索能力的结构化知识库。

## 核心价值

### 解决什么问题？

- **业务知识孤岛**：领域知识分散在各团队，缺乏集中访问
- **术语歧义**：组织内对业务术语有不同的理解
- **上下文缺失**：SQL Agent 缺乏对业务特定概念的理解
- **知识传承**：新团队成员难以理解领域特定术语

### 提供什么价值？

- **统一知识库**：业务术语和规则的集中存储库
- **语义搜索**：使用自然语言查询查找相关知识
- **Agent 上下文增强**：通过领域理解丰富 SQL 生成
- **知识沉淀**：以结构化、可搜索的格式捕获业务专业知识

## 使用方法

### 基本命令

```bash
# 从 CSV（直接导入）
datus-agent bootstrap-kb \
    --namespace <your_namespace> \
    --components ext_knowledge \
    --ext_knowledge /path/to/knowledge.csv \
    --kb_update_strategy overwrite

# 从 success story（AI 生成）
datus-agent bootstrap-kb \
    --namespace <your_namespace> \
    --components ext_knowledge \
    --success_story /path/to/success_story.csv \
    --kb_update_strategy overwrite
```

### 关键参数

| 参数                   | 必需 | 描述                                                            | 示例                              |
| ---------------------- | ---- | --------------------------------------------------------------- | --------------------------------- |
| `--namespace`          | ✅   | 数据库命名空间                                                  | `analytics_db`                    |
| `--components`         | ✅   | 要初始化的组件                                                  | `ext_knowledge`                   |
| `--ext_knowledge`      | ⚠️   | 知识 CSV 文件路径（如果没有 `--success_story` 则必需）          | `/data/knowledge.csv`             |
| `--success_story`      | ⚠️   | Success story CSV 文件路径（如果没有 `--ext_knowledge` 则必需） | `/data/success_story.csv`         |
| `--kb_update_strategy` | ✅   | 更新策略                                                        | `overwrite`/`incremental`         |
| `--subject_tree`       | ❌   | 预定义主题树分类                                                | `Finance/Revenue,User/Engagement` |
| `--pool_size`          | ❌   | 并发处理线程数，默认为 1                                        | `8`                               |

## 数据源格式

### 直接导入

从 CSV 文件直接导入预定义的知识条目。

#### CSV 格式

| 列             | 必需 | 描述             | 示例                          |
| -------------- | ---- | ---------------- | ----------------------------- |
| `subject_path` | 是   | 层级分类路径     | `Finance/Revenue/Metrics`     |
| `name`         | 是   | 知识条目名称     | `GMV Definition`              |
| `search_text`  | 是   | 可搜索的业务术语 | `GMV`                         |
| `explanation`  | 是   | 详细说明         | `Gross Merchandise Volume...` |

#### 示例 CSV

```csv
subject_path,name,search_text,explanation
Finance/Revenue/Metrics,GMV Definition,GMV,"Gross Merchandise Volume (GMV) represents the total value of merchandise sold through the platform, including both paid and unpaid orders."
User/Engagement/DAU,DAU Definition,DAU,"Daily Active Users (DAU) counts unique users who performed at least one activity within a calendar day."
User/Engagement/Retention,Retention Rate,retention rate,"The percentage of users who return to the platform after their first visit, typically measured at Day 1, Day 7, and Day 30 intervals."
```

### AI 生成

使用 AI Agent 从问题-SQL 对自动生成知识。

#### CSV 格式

| 列             | 必需 | 描述                 | 示例                                    |
| -------------- | ---- | -------------------- | --------------------------------------- |
| `question`     | 是   | 业务问题或查询意图   | `What is the total GMV for last month?` |
| `sql`          | 是   | 回答问题的 SQL 查询  | `SELECT SUM(amount) FROM orders...`     |
| `subject_path` | 否   | 层级分类路径（可选） | `Finance/Revenue/Metrics`               |

#### 示例 CSV

```csv
question,sql,subject_path
"What is the total GMV for last month?","SELECT SUM(amount) as gmv FROM orders WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)",Finance/Revenue/Metrics
"How many daily active users do we have?","SELECT COUNT(DISTINCT user_id) as dau FROM user_activity WHERE activity_date = CURDATE()",User/Engagement/DAU
"What is our 7-day retention rate?","SELECT COUNT(DISTINCT d7.user_id) / COUNT(DISTINCT d0.user_id) as retention FROM users d0 LEFT JOIN users d7 ON d0.user_id = d7.user_id",User/Engagement/Retention
```

#### AI 生成工作原理

success story 模式使用 GenExtKnowledgeAgenticNode 来：

1. **分析问题-SQL 对**：理解每个查询背后的业务意图
2. **提取业务概念**：识别关键术语、规则和模式
3. **生成知识条目**：创建包含 search_text 和 explanation 的结构化知识
4. **分类归类**：分配适当的 subject path 进行组织

## 更新策略

### 1. 覆盖模式

清除现有知识并加载新数据：

```bash
datus-agent bootstrap-kb \
    --namespace analytics_db \
    --components ext_knowledge \
    --ext_knowledge /path/to/knowledge.csv \
    --kb_update_strategy overwrite
```

### 2. 增量模式

添加新知识条目同时保留现有条目（重复项会被跳过）：

```bash
datus-agent bootstrap-kb \
    --namespace analytics_db \
    --components ext_knowledge \
    --success_story /path/to/success_story.csv \
    --kb_update_strategy incremental
```

## 主题树分类

主题树提供层级分类法来组织知识条目。

### 1. 预定义模式

```bash
datus-agent bootstrap-kb \
    --namespace analytics_db \
    --components ext_knowledge \
    --success_story /path/to/success_story.csv \
    --kb_update_strategy overwrite \
    --subject_tree "Finance/Revenue/Metrics,User/Engagement/DAU"
```

### 2. 学习模式

当未提供 subject_tree 时，系统会：

- 复用知识库中的现有分类
- 根据内容按需创建新分类
- 随时间自然构建分类体系

## 与 SQL Agent 集成

外部知识通过上下文搜索工具与 SQL 生成 Agent 集成：

1. **自动上下文检索**：生成 SQL 时，Agent 查询相关业务知识
2. **术语解析**：使用存储的定义解析模糊的业务术语
3. **规则应用**：知识库中存储的业务规则指导 SQL 逻辑

### 示例工作流

1. 用户提问："Calculate the GMV for last month"
2. Agent 搜索 "GMV" 相关知识
3. 找到定义："GMV = total value of merchandise including paid and unpaid orders"
4. 生成包含正确业务逻辑的 SQL

## 总结

Bootstrap-KB 外部知识组件将分散的业务知识转化为智能、可搜索的知识库。

**核心特性：**

- **双重导入模式**：直接 CSV 导入或从 success story AI 驱动生成
- **统一知识库**：业务术语和规则的集中存储
- **语义搜索**：使用自然语言查询查找知识
- **层级组织**：通过 subject path 分类体系导航知识
- **灵活分类**：支持预定义和学习两种模式
- **Agent 集成**：通过领域上下文增强 SQL 生成
- **重复检测**：增量模式下自动去重

通过实施外部知识，团队可以确保对业务概念的一致理解，并实现具有领域感知能力的智能 SQL 生成。
