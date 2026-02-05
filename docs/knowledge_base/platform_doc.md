# Platform Documentation

## Introduction

`platform-doc` ingests official platform documentation (for example Snowflake, StarRocks, Polaris) into a dedicated vector store, so the agent can verify platform-specific SQL syntax and features before generating queries.

The end-to-end pipeline is:

```
Fetch → Parse → Clean → Chunk → Embed → Store
```

## Storage Scope

- **Namespace-independent**: documents are stored per platform and shared across namespaces.
- **Default location**: `~/.datus/data/document/<platform>/`.
- **Selection key**: the `platform` argument decides which document store is queried.

## Build the Documentation Store

Platform docs are initialized with the dedicated `platform-doc` command (separate from `bootstrap-kb`).

### Command Syntax

```bash
datus-agent platform-doc \
  --platform <platform_name> \
  --source <source> \
  --source-type <github|website|local> \
  [options]
```

### Parameters

| Parameter | Default | Description |
|---|---|---|
| `--platform` | `default` | Platform name used as the storage key (recommended). |
| `--source` | - | GitHub repo `owner/repo`, website URL, or local path. Required unless configured in `agent.yml`. |
| `--source-type` | `local` | Source type: `github`, `website`, or `local`. |
| `--version` | auto | Document version label. If omitted, Datus tries to detect the latest version. |
| `--github-ref` | default branch | Git ref (branch or tag) for GitHub sources. |
| `--paths` | `docs README.md` | Paths to fetch for GitHub sources only. |
| `--include-patterns` | - | Include patterns (glob for local, regex for website). |
| `--exclude-patterns` | - | Exclude patterns (glob for local, regex for website). |
| `--chunk-size` | `1024` | Target chunk size in characters. |
| `--max-depth` | `2` | Maximum crawl depth for website sources. |
| `--pool-size` | `4` | Worker threads for processing. |
| `--update-strategy` | `check` | `check` (status-only) or `overwrite` (rebuild). |

### Examples

**1) GitHub (default branch)**

```bash
datus-agent platform-doc \
  --platform starrocks \
  --source StarRocks/starrocks \
  --source-type github \
  --paths docs/en
```

**2) GitHub (specific tag or branch)**

```bash
datus-agent platform-doc \
  --platform starrocks \
  --source StarRocks/starrocks \
  --source-type github \
  --version 4.0.5 \
  --github-ref 4.0.5 \
  --paths docs/en
```

**3) Website crawl**

```bash
datus-agent platform-doc \
  --platform snowflake \
  --source https://docs.snowflake.com/en/sql-reference \
  --source-type website \
  --version latest \
  --max-depth 2
```

**4) Local directory**

```bash
datus-agent platform-doc \
  --platform duckdb \
  --source /path/to/duckdb-docs \
  --source-type local \
  --version v1.0.0
```

## Configure in `agent.yml` (Optional)

You can store per-platform fetch configs in `agent.document` and then run the command with only `--platform`. CLI arguments override YAML values.

```yaml
agent:
  document:
    tavily_api_key: ${TAVILY_API_KEY}
    starrocks:
      type: github
      source: StarRocks/starrocks
      paths: ["docs/sql-reference"]
      chunk_size: 1024
      github_token: ${GITHUB_TOKEN}
    snowflake:
      type: website
      source: https://docs.snowflake.com/en/
      max_depth: 3
      include_patterns: ["/en/user-guide", "/en/sql-reference"]
    local_docs:
      type: local
      source: /path/to/docs
      include_patterns: ["*.md"]
      exclude_patterns: ["CHANGELOG.md"]
```

Run with:

```bash
datus-agent platform-doc --platform starrocks --github-ref 4.0.5
```

## Using the Tools in Datus

Once documents are ingested and platform doc tools are enabled, the agent gains four tools:

| Tool | Purpose | Key Args |
|---|---|---|
| `list_document_nav` | Browse the navigation tree and discover document titles. | `platform`, `version` |
| `get_document` | Retrieve full content for one document by hierarchy path. | `platform`, `titles`, `version` |
| `search_document` | Semantic search by keywords. | `platform`, `keywords`, `version`, `top_n` |
| `web_search_document` | Optional web fallback (Tavily). | `keywords`, `include_domains`, `max_results` |

To expose these tools in custom nodes or subagents, include `platform_doc_tools` (or specific `platform_doc_search_tools.*`) in the tool configuration.

**Recommended call order**: `list_document_nav` → `get_document` → `search_document` → `web_search_document` (fallback).

**`get_document` expects one document at a time**. Pass a single hierarchy path like:

```
titles=["DDL", "CREATE TABLE"]
```

To retrieve multiple documents, call it multiple times.

### CLI shortcut (Datus-CLI)

Inside `datus-cli`, use:

```
!sd
!search_document
```

This runs `search_document` interactively and shows matched chunks.

## Notes and Troubleshooting

- **GitHub API limits**: set `GITHUB_TOKEN` (or `github_token` in config) to avoid rate limiting.
- **Website crawling**: `--paths` is ignored for website sources. Use `--include-patterns` / `--exclude-patterns` instead.
- **No results**: verify the `platform` name matches the store you created and pass the right `version` if multiple versions exist.
- **Web fallback**: `web_search_document` requires `TAVILY_API_KEY`.
