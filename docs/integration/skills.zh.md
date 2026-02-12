# Skills

Skills 是 Datus-agent 的技能发现和加载系统，遵循 [agentskills.io](https://agentskills.io) 规范。它通过 SKILL.md 文件实现模块化、按需扩展的能力。

## 快速开始

本教程演示如何使用 **report-generator** 技能与加州学校数据集生成分析报告。

### 步骤 1：创建技能

创建包含 `SKILL.md` 文件的技能目录：

```
~/.datus/skills/
└── report-generator/
    ├── SKILL.md
    └── scripts/
        ├── generate_report.py
        ├── analyze_data.py
        ├── validate.sh
        └── export.sh
```

**SKILL.md** 内容：

```markdown
---
name: report-generator
description: Generate analysis reports from SQL query results with multiple output formats (HTML, Markdown, JSON)
tags: [report, analysis, visualization, export]
version: "1.0.0"
allowed_commands:
  - "python:scripts/*.py"
  - "sh:scripts/*.sh"
---

# Report Generator Skill

This skill generates professional analysis reports from SQL query results.

## Features

- **Multiple Formats**: Export to HTML, Markdown, or JSON
- **Data Analysis**: Automatic statistical analysis and insights

## Usage

### Generate a Report

python scripts/generate_report.py --input results.json --format html --output report.html

Options:
- `--input`: Input data file (JSON or CSV)
- `--format`: Output format (html, markdown, json)
- `--output`: Output file path
- `--title`: Report title (optional)
```

### 步骤 2：在 agent.yml 中配置技能

```yaml
skills:
  directories:
    - ~/.datus/skills
    - ./skills
  warn_duplicates: true

permissions:
  default: allow
  rules:
    # 技能加载需要确认
    - tool: skills
      pattern: "*"
      permission: ask
    # 技能脚本执行需要确认
    - tool: skill_bash
      pattern: "*"
      permission: ask
```

!!! tip
    为 skills 和 skill_bash 使用 `ask` 权限，在执行前需要手动确认，有助于防止意外或危险操作。

### 步骤 3：在聊天会话中使用技能

启动聊天会话并提出问题：

```
> What is the highest eligible free rate for K-12 students in the schools
> in Alameda County? Generate a report using the final result.
```

Agent 将执行以下操作：

1. **加载技能** - 当需要生成报告时，LLM 调用 `load_skill(skill_name="report-generator")` 获取技能指令。

2. **执行 SQL 查询** - 查询加州学校数据库以获取答案。

3. **生成报告** - 执行技能脚本创建报告：

    ```python
    skill_execute_command(
        skill_name="report-generator",
        command="python scripts/generate_report.py --input results.json --format markdown --title 'Alameda County K-12 Free Rate Analysis'"
    )
    ```

![聊天会话展示技能加载和报告生成](../assets/skills1.png)
![聊天会话展示技能加载和报告生成](../assets/skills3.png)

### 步骤 4：查看生成的报告

报告将在技能的工作目录中生成：

![生成的 Markdown 报告展示分析结果](../assets/skill5.png)

## 权限系统

权限系统控制哪些技能和工具可供 Agent 使用。

### 权限级别

| 级别 | 行为 |
|------|------|
| `allow` | 技能可用且可自由使用 |
| `deny` | 技能对 Agent 隐藏（不会出现在提示中） |
| `ask` | 每次使用前需要用户确认 |

### 配置示例

```yaml
permissions:
  default: allow
  rules:
    # 默认允许所有技能
    - tool: skills
      pattern: "*"
      permission: allow

    # 数据库写操作需要确认
    - tool: db_tools
      pattern: "execute_sql"
      permission: ask

    # 隐藏内部/管理技能
    - tool: skills
      pattern: "internal-*"
      permission: deny

    # 潜在危险技能需要确认
    - tool: skills
      pattern: "dangerous-*"
      permission: ask
```

### 模式匹配

模式使用 glob 风格匹配：

- `*` 匹配任意内容
- `report-*` 匹配以 "report-" 开头的技能
- `*-admin` 匹配以 "-admin" 结尾的技能

### 节点特定权限

为特定节点覆盖权限：

```yaml
agentic_nodes:
  chat:
    skills: "report-*, data-*"  # 仅暴露匹配的技能
    permissions:
      rules:
        - tool: skills
          pattern: "admin-*"
          permission: deny
```

## 在 Subagent 中使用技能

默认情况下，**聊天 Subagent 会自动加载所有已发现的技能**。其他 Subagent（报告生成、SQL 生成、指标等）**不会加载任何技能**，除非在 `agent.yml` 中显式配置。

| Subagent 类型 | 默认加载的技能 |
|---------------|---------------|
| Chat | 所有已发现的技能 |
| 其他所有 Subagent（报告、SQL、指标等） | 无 |

### 为自定义 Subagent 启用技能

要在自定义 Subagent 中启用技能，请在 `agent.yml` 的 `agentic_nodes` 部分中为对应 Subagent 添加 `skills` 字段：

```yaml
agentic_nodes:
  # 此 Subagent 仅加载报告和数据类技能
  school_report:
    node_class: gen_report
    skills: "report-*, data-*"
    model: deepseek

  # 此 Subagent 仅加载 SQL 相关技能
  school_sql:
    skills: "sql-*"
    model: deepseek

  # 此 Subagent 加载所有技能（chat 默认加载全部，但显式声明更清晰）
  school_chat:
    skills: "*"
    model: deepseek
```

`skills` 字段接受逗号分隔的 glob 模式列表。只有名称匹配至少一个模式的技能才会对该 Subagent 可用。`node_class` 字段支持两个值：`gen_sql`（默认）和 `gen_report`。

当 Subagent 配置了 `skills` 时：

1. **技能发现** — 系统扫描 `skills.directories`（或默认路径：`~/.datus/skills`、`./skills`）查找所有 `SKILL.md` 文件。
2. **模式过滤** — 仅暴露匹配 Subagent `skills` glob 模式的技能。
3. **权限过滤** — `permissions` 规则进一步过滤哪些技能被允许、拒绝或需要确认。
4. **系统提示注入** — 可用技能以 `<available_skills>` XML 形式附加到 Subagent 的系统提示中，使 LLM 能够调用 `load_skill()` 和 `skill_execute_command()`。

**示例：在 Subagent 中启用报告生成技能**

```yaml
skills:
  directories:
    - ~/.datus/skills

agentic_nodes:
  attribution_report:
    node_class: gen_report
    skills: "report-generator"
    model: deepseek
```

通过此配置，`attribution_report` Subagent 将可以访问 `report-generator` 技能。LLM 可以调用 `load_skill(skill_name="report-generator")` 获取指令，然后使用 `skill_execute_command()` 运行脚本。

!!! note
    如果 `agent.yml` 中没有全局 `skills:` 部分，系统会自动创建默认的技能管理器，扫描 `~/.datus/skills` 和 `./skills`。

!!! tip
    `skill_execute_command` 工具默认使用 `ask` 权限级别。这意味着在技能脚本执行前用户会收到确认提示，除非在 `permissions` 配置中显式覆盖。

### 在隔离 Subagent 中运行技能

技能也可以通过在 SKILL.md frontmatter 中设置 `context: fork` 来在隔离的 Subagent 上下文中运行：

```markdown
---
name: deep-analysis
description: Perform comprehensive data analysis with multiple iterations
tags: [analysis, research]
context: fork
agent: Explore
---

# Deep Analysis Skill

This skill runs in an isolated Explore subagent for thorough investigation.
```

可用的隔离执行 Subagent 类型：

| Agent 类型 | 用途 |
|------------|------|
| `Explore` | 代码库探索、文件搜索、理解结构 |
| `Plan` | 实现规划、架构决策 |
| `general-purpose` | 多步骤任务、复杂研究 |

### 调用控制

| 字段 | 默认值 | 描述 |
|------|--------|------|
| `disable_model_invocation` | `false` | 如为 true，仅用户可通过 `/skill-name` 调用 |
| `user_invocable` | `true` | 如为 false，从 CLI 菜单隐藏（仅模型调用） |

## SKILL.md 参考

### Frontmatter 字段

| 字段 | 必需 | 描述 |
|------|------|------|
| `name` | 是 | 唯一技能标识符 |
| `description` | 是 | 在可用技能列表中显示的简短描述 |
| `tags` | 否 | 用于分类的标签列表 |
| `version` | 否 | 语义版本字符串 |
| `allowed_commands` | 否 | 允许的脚本模式列表 |
| `context` | 否 | 设为 `"fork"` 以在 subagent 中运行 |
| `agent` | 否 | Subagent 类型：`Explore`、`Plan`、`general-purpose` |
| `disable_model_invocation` | 否 | 如为 true，仅用户可调用 |
| `user_invocable` | 否 | 如为 false，从 CLI 菜单隐藏 |

### 命令模式格式

```
prefix:glob_pattern
```

示例：

- `python:*` - 允许任意 python 命令
- `python:scripts/*.py` - 仅允许 scripts/ 目录中的脚本
- `sh:*.sh` - 允许 shell 脚本
- `python:-c:*` - 允许 python -c 内联代码

### 安全特性

- 命令仅在匹配允许的模式时执行
- 工作目录锁定在技能位置
- 超时强制执行（默认：60 秒）
- 环境变量：`SKILL_NAME`、`SKILL_DIR`

## 故障排除

### 技能未被发现

1. 检查技能目录是否在 `skills.directories` 配置中
2. 验证 SKILL.md 具有有效的 YAML frontmatter（在 `---` 标记之间）
3. `name` 和 `description` 字段都是必需的

### 脚本执行被拒绝

1. 验证命令是否匹配 `allowed_commands` 模式
2. 确保先通过 `load_skill()` 加载了技能
3. 检查模式格式：`prefix:glob_pattern`

### 调试日志

启用调试日志：

```bash
export DATUS_LOG_LEVEL=DEBUG
```
