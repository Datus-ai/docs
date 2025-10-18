# Tool Commands `!`

## 1. Overview

Tool commands (prefixed with `!`) provide specialized AI-powered capabilities and utility operations within the Datus-CLI environment. These commands enable schema discovery, SQL generation, query optimization, and other intelligent data operations without leaving the interactive session.

## 2. Command Categories

### 2.1 AI Workflow Commands

#### `!run <query>`
Run a natural language query through the AI agent with live workflow status display.

```bash
!run Find all users who made purchases in the last 30 days
!run Calculate total revenue by product category
```

This command creates an interactive workflow that:

- Performs automatic schema linking
- Generates optimized SQL queries
- Executes queries and displays results
- Provides real-time status updates

### 2.2 Schema Discovery Commands

#### `!sl` / `!schema_linking`
Perform intelligent schema linking to discover relevant tables and columns for your query.

```bash
!sl user purchase information
!schema_linking sales data by region
```

Features:

- Semantic search for relevant database tables
- Table definition (DDL) display
- Sample data preview
- Configurable matching methods: fast, medium, slow, from_llm
- Adjustable top_n results

Interactive prompts guide you through:

- Catalog/database/schema selection
- Number of tables to match
- Matching method preference

### 2.3 Search & Discovery Commands

#### `!sm` / `!search_metrics`
Use natural language to search for corresponding metrics in your data catalog.

```bash
!sm monthly active users
!search_metrics revenue growth rate
```

Allows filtering by:

- Domain
- Layer1 (business layer)
- Layer2 (sub-layer)
- Top N results

#### `!sh` / `!search_history`
Search historical SQL queries using natural language descriptions.

```bash
!sh queries about user retention
!search_history monthly sales reports
```

Returns:

- SQL query text with syntax highlighting
- Query summary and comments
- Tags and categorization
- Domain/layer metadata
- File path and relevance distance

### 2.4 SQL Generation & Optimization Commands

#### `!gen`
Generate SQL queries based on natural language task descriptions with optional table constraints.

```bash
!gen Show me top 10 customers by revenue
!gen Calculate year-over-year growth by product
```

The command:

- Creates a new SQL task or reuses the current one
- Leverages recent table schemas and metrics from context
- Generates optimized SQL with explanation
- Stores result in CLI context for further operations

#### `!fix <description>`
Fix issues in the last executed SQL query based on your description.

```bash
!fix The date filter should be for last quarter, not last month
!fix Add grouping by region
```

Requirements:

- Requires a previous SQL query in context
- Interactive confirmation before execution
- Preserves original query for reference

### 2.5 Advanced AI Commands

#### `!gen_metrics`
Generate metrics definitions from SQL queries and table structures.

```bash
!gen_metrics
```

Features:
- Extracts metric definitions from SQL
- Interactive configuration of:
  - Task description
  - Source SQL query
  - Prompt version

#### `!gen_semantic_model`
Generate semantic models for data modeling and documentation.

```bash
!gen_semantic_model
```

Interactive prompts for:

- Table name selection
- Catalog/database/schema metadata
- Business layer (layer1/layer2)
- Domain classification
- Prompt version

### 2.6 Analysis Commands

#### `!reason`
Perform SQL reasoning and explanation with streaming output.

```bash
!reason
```

Provides:

- SQL query analysis
- Explanation of query logic
- Performance considerations
- Potential optimization suggestions

#### `!compare`
Compare SQL results with expected outcomes.

```bash
!compare
```

Features:

- Interactive expectation input (SQL query or data format)
- Detailed comparison analysis
- Discrepancy identification

### 2.7 Utility Commands

#### `!save`
Save the last query result to a file.

```bash
!save
```

Interactive options:

- File type: json, csv, sql, or all
- Output directory (defaults to ~/.datus/output)
- Custom filename

#### `!bash <command>`
Execute safe bash commands (security restricted).

```bash
!bash pwd
!bash ls -la
!bash cat config.yaml
```

**Security**: Only whitelisted commands are allowed:

- `pwd` - Print working directory
- `ls` - List files
- `cat` - Display file contents
- `head` - Show file beginning
- `tail` - Show file end
- `echo` - Display text

Commands not in the whitelist will be rejected with a security warning.

## 3. Best Practices

1. **Start with Schema Linking** - Use `!sl` to discover relevant tables before generating queries
2. **Leverage Search** - Use `!sm` and `!sh` to find existing metrics and queries before creating new ones
3. **Iterative Refinement** - Use `!fix` to refine queries instead of starting from scratch
4. **Save Results** - Use `!save` to preserve important query results
5. **Security First** - Be aware of bash command restrictions when using `!bash`

## 4. Security Considerations

- Tool commands run with the same privileges as the Datus-CLI process
- Bash commands are restricted to a whitelist of safe operations
- `!bash` commands timeout after 10 seconds to prevent hanging
- All operations are logged for audit purposes
- API credentials and database connections are handled securely