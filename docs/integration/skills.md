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
    - tool: skills
      pattern: "*"
      permission: allow
```

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

![Chat session showing skill loading and report generation](assets/skills1.png)
![Chat session showing skill loading and report generation](assets/skills3.png)

### Step 4: View the Generated Report

The report will be generated in the skill's working directory:

![Generated markdown report showing the analysis results](assets/skill5final.png)

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

## Using Skills in Subagent

Skills can be configured to run in an isolated subagent context for complex tasks.

### Configure Subagent Execution

Add `context: fork` and specify the `agent` type in the SKILL.md frontmatter:

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

## When to Use
- Complex multi-step analysis
- Tasks requiring extensive exploration
- Investigations that may take multiple turns
```

### Available Subagent Types

| Agent Type | Use Case |
|------------|----------|
| `Explore` | Codebase exploration, file searching, understanding structure |
| `Plan` | Implementation planning, architectural decisions |
| `general-purpose` | Multi-step tasks, complex research |

### Example: Research Skill with Subagent

```markdown
---
name: codebase-research
description: Research codebase patterns and architecture
context: fork
agent: Explore
user_invocable: true
---

# Codebase Research

When invoked, this skill spawns an Explore subagent to:
1. Search for relevant files and patterns
2. Analyze code structure
3. Report findings back to the main conversation
```

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
