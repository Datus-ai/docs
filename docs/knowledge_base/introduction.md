# Knowledge Base Introduction

The Datus Agent Knowledge Base is a multi-modal intelligence system that transforms scattered data assets into a unified, searchable repository. Think of it as "Google for your data" with deep understanding of SQL, business metrics, and data relationships.

## Core Purpose

- **Data Discovery**: Find relevant tables, columns, and patterns
- **Query Intelligence**: Understand business intent and generate SQL
- **Knowledge Preservation**: Capture and organize SQL expertise
- **Semantic Search**: Find information by meaning, not keywords

## Core Components

### 1. [Schema Metadata](metadata.md)

**Purpose**: Understand database structure and provide intelligent table recommendations.

- **Stores**: Table definitions, column info, sample data, statistics
- **Capabilities**: Find tables by business meaning, get table structures, semantic search
- **Use**: Automatic table selection, data discovery, schema understanding

### 2. [Semantic Models](semantic_model.md)

**Purpose**: Enrich database schemas with semantic information for better SQL generation.

- **Stores**: Table structures, dimensions, measures, entity relationships
- **Capabilities**: Schema linking, column usage patterns, foreign key discovery
- **Use**: Accurate ad-hoc SQL generation, smart filtering, proper JOIN construction

### 3. [Business Metrics](metrics.md)

**Purpose**: Manage and query standardized business KPIs.

- **Stores**: Metric definitions, subject tree categorization
- **Capabilities**: Direct metric queries via MetricFlow, metrics-first strategy
- **Use**: Consistent reporting, eliminate duplicate SQL, standardized definitions

### 4. [Reference SQL](reference_sql.md)

**Purpose**: Capture, analyze, and make searchable SQL expertise.

- **Stores**: Historical queries, LLM summaries, query patterns, best practices
- **Capabilities**: Find queries by intent, get similar queries, learn patterns
- **Use**: Knowledge sharing, optimization through examples, team onboarding


## Key Features

- **Unified Search**: Single interface across all knowledge domains
- **Semantic Search**: Find by meaning using vector embeddings
- **Intelligent Classification**: Automatic categorization and organization
- **Scalable**: Lazy loading, batch processing, incremental updates
