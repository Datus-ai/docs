# Skills

Skills is a skill discovery and loading system for Datus-agent, following the [agentskills.io](https://agentskills.io) specification. It enables modular, on-demand capability expansion through SKILL.md files.

## Quick Start

This tutorial demonstrates how to use the **report-generator** skill with the California Schools dataset to generate analysis reports.

### Step 1: Create a Skill

Create a skill directory with a `SKILL.md` file:

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

**SKILL.md** content:

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

### Step 2: Configure Skills in agent.yml

```yaml
skills:
  directories:
    - ~/.datus/skills
    - ./skills
  warn_duplicates: true

permissions:
  default: allow
  rules:
    # Require confirmation for skill loading
    - tool: skills
      pattern: "*"
      permission: ask
    # Require confirmation for skill script execution
    - tool: skill_bash
      pattern: "*"
      permission: ask
```

!!! tip
    Using `ask` permission for skills and skill_bash requires manual confirmation before execution, which helps prevent accidental or dangerous operations.

### Step 3: Use the Skill in a Chat Session

Start a chat session and ask your question:

```
> What is the highest eligible free rate for K-12 students in the schools
> in Alameda County? Generate a report using the final result.
```

The agent will:

1. **Load the skill** - When generating a report is needed, the LLM calls `load_skill(skill_name="report-generator")` to get the skill instructions.

2. **Execute SQL query** - Query the California Schools database to find the answer.

3. **Generate report** - Execute the skill's script to create a report:

    ```python
    skill_execute_command(
        skill_name="report-generator",
        command="python scripts/generate_report.py --input results.json --format markdown --title 'Alameda County K-12 Free Rate Analysis'"
    )
    ```

![Chat session showing skill loading and report generation](../assets/skills1.png)
![Chat session showing skill loading and report generation](../assets/skills3.png)

### Step 4: View the Generated Report

The report will be generated in the skill's working directory:

![Generated markdown report showing the analysis results](../assets/skill5.png)

## Permission System

The permission system controls which skills and tools are available to the agent.

### Permission Levels

| Level | Behavior |
|-------|----------|
| `allow` | Skill is available and can be used freely |
| `deny` | Skill is hidden from agent (never appears in prompts) |
| `ask` | User confirmation required before each use |

### Configuration Example

```yaml
permissions:
  default: allow
  rules:
    # Allow all skills by default
    - tool: skills
      pattern: "*"
      permission: allow

    # Require confirmation for database write operations
    - tool: db_tools
      pattern: "execute_sql"
      permission: ask

    # Hide internal/admin skills
    - tool: skills
      pattern: "internal-*"
      permission: deny

    # Require confirmation for potentially dangerous skills
    - tool: skills
      pattern: "dangerous-*"
      permission: ask
```

### Pattern Matching

Patterns use glob-style matching:

- `*` matches anything
- `report-*` matches skills starting with "report-"
- `*-admin` matches skills ending with "-admin"

### Node-Specific Permissions

Override permissions for specific nodes:

```yaml
agentic_nodes:
  chat:
    skills: "report-*, data-*"  # Only expose matching skills
    permissions:
      rules:
        - tool: skills
          pattern: "admin-*"
          permission: deny
```

## Using Skills in Subagents

By default, the **chat subagent loads all discovered skills** automatically. Other subagents (report generation, SQL generation, metrics, etc.) **do not load any skills** unless explicitly configured in `agent.yml`.

| Subagent Type | Skills Loaded by Default |
|---------------|------------------------|
| Chat | All discovered skills |
| All other subagents (report, SQL, metrics, etc.) | None |

### Enabling Skills for Customized Subagents

To enable skills in a customized subagent, add the `skills` field under the subagent's config in `agentic_nodes` section of `agent.yml`:

```yaml
agentic_nodes:
  # This subagent only sees report and data skills
  school_report:
    node_class: gen_report
    skills: "report-*, data-*"
    model: deepseek

  # This subagent only sees SQL-related skills
  school_sql:
    skills: "sql-*"
    model: deepseek

  # This subagent sees all skills (chat loads all by default, but explicit is clearer)
  school_chat:
    skills: "*"
    model: deepseek
```

The `skills` field accepts a comma-separated list of glob patterns. Only skills whose names match at least one pattern will be available to that subagent. The `node_class` field supports two values: `gen_sql` (default) and `gen_report`.

When a subagent has `skills` configured:

1. **Skill discovery** — The system scans `skills.directories` (or defaults: `~/.datus/skills`, `./skills`) to find all `SKILL.md` files.
2. **Pattern filtering** — Only skills matching the subagent's `skills` glob patterns are exposed.
3. **Permission filtering** — The `permissions` rules further filter which skills are allowed, denied, or require confirmation.
4. **System prompt injection** — Available skills are appended as `<available_skills>` XML to the subagent's system prompt, enabling the LLM to call `load_skill()` and `skill_execute_command()`.

**Example: Enable Report Generation Skill in a Subagent**

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

With this configuration, the `attribution_report` subagent will have access to the `report-generator` skill. The LLM can call `load_skill(skill_name="report-generator")` to get instructions, then use `skill_execute_command()` to run scripts.

!!! note
    If no global `skills:` section is present in `agent.yml`, the system automatically creates a default skill manager that scans `~/.datus/skills` and `./skills`.

!!! tip
    The `skill_execute_command` tool defaults to `ask` permission level. This means the user will be prompted for confirmation before any skill script executes, unless explicitly overridden in the `permissions` config.

### Running Skills in Isolated Subagent

Skills can also be configured to run in an isolated subagent context by setting `context: fork` in the SKILL.md frontmatter:

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

Available subagent types for isolated execution:

| Agent Type | Use Case |
|------------|----------|
| `Explore` | Codebase exploration, file searching, understanding structure |
| `Plan` | Implementation planning, architectural decisions |
| `general-purpose` | Multi-step tasks, complex research |

### Invocation Control

| Field | Default | Description |
|-------|---------|-------------|
| `disable_model_invocation` | `false` | If true, only user can invoke via `/skill-name` |
| `user_invocable` | `true` | If false, hidden from CLI menu (only model invokes) |

## SKILL.md Reference

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique skill identifier |
| `description` | Yes | Brief description shown in available skills list |
| `tags` | No | List of tags for categorization |
| `version` | No | Semantic version string |
| `allowed_commands` | No | List of permitted script patterns |
| `context` | No | Set to `"fork"` to run in subagent |
| `agent` | No | Subagent type: `Explore`, `Plan`, `general-purpose` |
| `disable_model_invocation` | No | If true, only user can invoke |
| `user_invocable` | No | If false, hidden from CLI menu |

### Command Pattern Format

```
prefix:glob_pattern
```

Examples:
- `python:*` - Allow any python command
- `python:scripts/*.py` - Allow scripts in scripts/ directory only
- `sh:*.sh` - Allow shell scripts
- `python:-c:*` - Allow python -c inline code

### Security Features

- Commands only execute if they match allowed patterns
- Working directory locked to skill location
- Timeout enforcement (default: 60 seconds)
- Environment variables: `SKILL_NAME`, `SKILL_DIR`

## Troubleshooting

### Skill Not Discovered

1. Check skill directory is in `skills.directories` config
2. Verify SKILL.md has valid YAML frontmatter (between `---` markers)
3. Both `name` and `description` fields are required

### Script Execution Denied

1. Verify command matches an `allowed_commands` pattern
2. Ensure skill was loaded first via `load_skill()`
3. Check pattern format: `prefix:glob_pattern`

### Debug Logging

Enable debug logging:

```bash
export DATUS_LOG_LEVEL=DEBUG
```
