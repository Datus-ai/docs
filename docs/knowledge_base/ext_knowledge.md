# External Knowledge Intelligence

## Overview

Bootstrap-KB External Knowledge is a component that processes, stores, and indexes domain-specific business knowledge to create an intelligent searchable knowledge base. It transforms business rules and concepts into a structured repository with semantic search capabilities.

## Core Value

### What Problem Does It Solve?

- **Business Knowledge Silos**: Domain knowledge scattered across teams without centralized access
- **Terminology Ambiguity**: Different interpretations of business terms across the organization
- **Context Gap**: SQL agents lacking understanding of business-specific concepts
- **Knowledge Onboarding**: New team members struggling to understand domain-specific terminology

### What Value Does It Provide?

- **Unified Knowledge Base**: Centralized repository for business terminology and rules
- **Semantic Search**: Find relevant knowledge using natural language queries
- **Agent Context Enhancement**: Enriches SQL generation with domain understanding
- **Knowledge Preservation**: Captures business expertise in a structured, searchable format

## Usage

### Basic Command

```bash
# From CSV (direct import)
datus bootstrap-kb \
    --namespace <your_namespace> \
    --components ext_knowledge \
    --ext_knowledge /path/to/knowledge.csv \
    --kb_update_strategy overwrite

# From success story (AI generation)
datus bootstrap-kb \
    --namespace <your_namespace> \
    --components ext_knowledge \
    --success_story /path/to/success_story.csv \
    --kb_update_strategy overwrite
```

### Key Parameters

| Parameter | Required | Description                                                       | Example |
|-----------|----------|-------------------------------------------------------------------|---------|
| `--namespace` | ✅ | Database namespace                                                | `analytics_db` |
| `--components` | ✅ | Components to initialize                                          | `ext_knowledge` |
| `--ext_knowledge` | ⚠️ | Path to knowledge CSV file (required if no `--success_story`)     | `/data/knowledge.csv` |
| `--success_story` | ⚠️ | Path to success story CSV file (required if no `--ext_knowledge`) | `/data/success_story.csv` |
| `--kb_update_strategy` | ✅ | Update strategy                                                   | `overwrite`/`incremental` |
| `--subject_tree` | ❌ | Predefined subject tree categories                                | `Finance/Revenue,User/Engagement` |
| `--pool_size` | ❌ | Concurrent processing threads, default is 1                       | `8` |

## Data Source Formats

### Direct Import (--ext_knowledge)

Import pre-defined knowledge entries directly from a CSV file.

#### CSV Format

| Column | Required | Description | Example |
|--------|----------|-------------|---------|
| `subject_path` | Yes | Hierarchical category path | `Finance/Revenue/Metrics` |
| `name` | Yes | Knowledge entry name | `GMV Definition` |
| `search_text` | Yes | Searchable business term | `GMV` |
| `explanation` | Yes | Detailed description | `Gross Merchandise Volume...` |

#### Example CSV

```csv
subject_path,name,search_text,explanation
Finance/Revenue/Metrics,GMV Definition,GMV,"Gross Merchandise Volume (GMV) represents the total value of merchandise sold through the platform, including both paid and unpaid orders."
User/Engagement/DAU,DAU Definition,DAU,"Daily Active Users (DAU) counts unique users who performed at least one activity within a calendar day."
User/Engagement/Retention,Retention Rate,retention rate,"The percentage of users who return to the platform after their first visit, typically measured at Day 1, Day 7, and Day 30 intervals."
```

### AI Generation (--success_story)

Generate knowledge automatically from question-SQL pairs using AI agent.

#### CSV Format

| Column | Required | Description | Example |
|--------|----------|-------------|---------|
| `question` | Yes | Business question or query intent | `What is the total GMV for last month?` |
| `sql` | Yes | SQL query that answers the question | `SELECT SUM(amount) FROM orders...` |
| `subject_path` | No | Hierarchical category path (optional) | `Finance/Revenue/Metrics` |

#### Example CSV

```csv
question,sql,subject_path
"What is the total GMV for last month?","SELECT SUM(amount) as gmv FROM orders WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)",Finance/Revenue/Metrics
"How many daily active users do we have?","SELECT COUNT(DISTINCT user_id) as dau FROM user_activity WHERE activity_date = CURDATE()",User/Engagement/DAU
"What is our 7-day retention rate?","SELECT COUNT(DISTINCT d7.user_id) / COUNT(DISTINCT d0.user_id) as retention FROM users d0 LEFT JOIN users d7 ON d0.user_id = d7.user_id",User/Engagement/Retention
```

#### How AI Generation Works

The success story mode uses GenExtKnowledgeAgenticNode to:

1. **Analyze Question-SQL Pairs**: Understands the business intent behind each query
2. **Extract Business Concepts**: Identifies key terminology, rules, and patterns
3. **Generate Knowledge Entries**: Creates structured knowledge with search_text and explanation
4. **Classify Categories**: Assigns appropriate subject paths for organization

## Update Strategies

**1. Overwrite Mode**

Clears existing knowledge and loads fresh data:

```bash
datus bootstrap-kb \
    --namespace analytics_db \
    --components ext_knowledge \
    --ext_knowledge /path/to/knowledge.csv \
    --kb_update_strategy overwrite
```

**2. Incremental Mode**

Adds new knowledge entries while preserving existing ones (duplicates are skipped):

```bash
datus bootstrap-kb \
    --namespace analytics_db \
    --components ext_knowledge \
    --success_story /path/to/success_story.csv \
    --kb_update_strategy incremental
```

### Subject Tree Categorization

Subject tree provides a hierarchical taxonomy for organizing knowledge entries.

**1. Predefined Mode (with --subject_tree)**

```bash
datus bootstrap-kb \
    --namespace analytics_db \
    --components ext_knowledge \
    --success_story /path/to/success_story.csv \
    --kb_update_strategy overwrite \
    --subject_tree "Finance/Revenue/Metrics,User/Engagement/DAU"
```

**2. Learning Mode (without --subject_tree)**

When no subject_tree is provided, the system:
- Reuses existing categories from the Knowledge Base
- Creates new categories as needed based on content
- Builds taxonomy organically over time


## Integration with SQL Agent

External knowledge integrates with the SQL generation agent through context search tools:

1. **Automatic Context Retrieval**: When generating SQL, the agent queries relevant business knowledge
2. **Term Resolution**: Ambiguous business terms are resolved using stored definitions
3. **Rule Application**: Business rules stored in knowledge base guide SQL logic

### Example Workflow

1. User asks: "Calculate the GMV for last month"
2. Agent searches knowledge for "GMV"
3. Finds definition: "GMV = total value of merchandise including paid and unpaid orders"
4. Generates SQL with correct business logic

## Summary

The Bootstrap-KB External Knowledge component transforms scattered business knowledge into an intelligent, searchable knowledge base.

**Key Features:**
- **Dual Import Modes**: Direct CSV import or AI-driven generation from success stories
- **Unified Repository**: Centralized storage for business terminology and rules
- **Semantic Search**: Find knowledge using natural language queries
- **Hierarchical Organization**: Navigate knowledge through subject path taxonomy
- **Flexible Classification**: Support both predefined and learning modes
- **Agent Integration**: Enhances SQL generation with domain context
- **Duplicate Detection**: Automatic deduplication in incremental mode

By implementing external knowledge, teams can ensure consistent understanding of business concepts and enable intelligent SQL generation with domain awareness.