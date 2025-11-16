# 工具命令 `!`

## 1. 概览

以 `!` 为前缀的工具命令，为 Datus-CLI 会话提供一系列 AI 加持的能力与实用操作。你可以在不离开交互式终端的前提下，完成模式发现、SQL 生成、查询优化等智能数据任务。

## 2. 命令分类

### 2.1 AI 工作流命令

#### `!run <query>`
通过 AI 智能体运行自然语言查询，并实时展示工作流状态。

```bash
!run Find all users who made purchases in the last 30 days
!run Calculate total revenue by product category
```

该命令会创建交互式工作流，流程包括：

- 自动执行模式匹配（schema linking）
- 生成优化后的 SQL 查询
- 执行查询并展示结果
- 实时显示状态进度

### 2.2 模式发现命令

#### `!sl` / `!schema_linking`
执行智能模式匹配，为你的问题查找相关的数据表与字段。

```bash
!sl user purchase information
!schema_linking sales data by region
```

特性：

- 语义级别搜索相关数据库表
- 展示表定义（DDL）
- 预览样例数据
- 可选匹配模式：fast、medium、slow、from_llm
- 可设置 top_n 返回数量

交互式提示将引导你完成：

- 选择 catalog / database / schema
- 指定匹配数据表数量
- 选择偏好的匹配方式

### 2.3 搜索发现命令

#### `!sm` / `!search_metrics`
使用自然语言在数据目录中搜索对应的指标。

```bash
!sm monthly active users
!search_metrics revenue growth rate
```

支持以下筛选条件：

- Domain
- Layer1（业务层）
- Layer2（子层）
- Top N 结果

#### `!ss` / `!search_sql`
使用自然语言描述搜索历史 SQL。

```bash
!ss queries about user retention
!search_sql monthly sales reports
```

返回内容包括：

- SQL 文本（语法高亮）
- 查询摘要与备注
- 标签与分类
- 域/层级元数据
- 文件路径与相关度（距离分数）

### 2.4 SQL 生成与优化命令

#### `!gen`
基于自然语言任务描述生成 SQL，可选指定表约束。

```bash
!gen Show me top 10 customers by revenue
!gen Calculate year-over-year growth by product
```

命令行为：

- 创建新的 SQL 任务或复用当前任务
- 利用上下文中的最新表结构与指标
- 生成带解释的优化 SQL
- 将结果保存到 CLI 上下文以便后续操作

#### `!fix <description>`
根据描述修复上一次执行的 SQL。

```bash
!fix The date filter should be for last quarter, not last month
!fix Add grouping by region
```

前提要求：

- 需要已有的 SQL 查询记录在上下文中
- 执行前会进行交互式确认
- 会保留原查询供对比参考

### 2.5 分析命令

#### `!reason`
执行 SQL 推理与说明，支持流式输出。

```bash
!reason
```

提供内容：

- SQL 查询分析
- 查询逻辑讲解
- 性能注意事项
- 潜在优化建议

#### `!compare`
将 SQL 结果与预期进行对比。

```bash
!compare
```

特性：

- 交互式输入预期结果（SQL 或数据格式）
- 输出详细的对比分析
- 高亮差异点

### 2.6 实用命令

#### `!save`
将最近一次查询结果保存到文件。

```bash
!save
```

交互式选项：

- 文件类型：json、csv、sql 或 all
- 输出目录（默认 `~/.datus/output`）
- 自定义文件名

#### `!bash <command>`
执行安全的 Bash 命令（有限制）。

```bash
!bash pwd
!bash ls -la
!bash cat config.yaml
```

**安全策略**：仅允许白名单命令：

- `pwd` —— 查看当前目录
- `ls` —— 列出目录内容
- `cat` —— 展示文件内容
- `head` —— 查看文件开头
- `tail` —— 查看文件末尾
- `echo` —— 输出文本

不在白名单内的命令会被拒绝并提示安全警告。

## 3. 最佳实践

1. **先做模式匹配** —— 使用 `!sl` 寻找相关表，再生成查询
2. **善用搜索** —— 使用 `!sm`、`!sh` 优先复用已有指标与查询
3. **迭代优化** —— 用 `!fix` 在现有查询上迭代，而非重新开始
4. **保存成果** —— 通过 `!save` 保存重要的查询结果
5. **关注安全** —— 使用 `!bash` 时注意白名单限制

## 4. 安全注意事项

- 工具命令与 Datus-CLI 进程共享权限
- Bash 命令被限制在安全白名单内
- `!bash` 默认 10 秒超时，防止卡死
- 所有操作都会记录日志以供审计
- API 凭证与数据库连接均经过安全处理

