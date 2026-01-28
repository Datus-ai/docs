# Datus MCP Server

Expose Datus's database and context search tools via the **Model Context Protocol (MCP)**, enabling integration with
Claude Desktop, Claude Code, and other MCP-compatible clients.

**Server Modes:**

- **Static Mode**: Single namespace, suitable for Claude Desktop, CLI tools, or single-tenant HTTP/SSE server
- **Dynamic Mode**: Multi-namespace HTTP/SSE server, supports all namespaces via URL path

**Supported Transport Modes:**

- `http`: Streamable HTTP (bidirectional, default)
- `sse`: Server-Sent Events over HTTP (for web clients)
- `stdio`: Standard input/output (for Claude Desktop and CLI tools)

## Quick Start

- Install Datus:

```bash
pip install datus-agent
```

- Run the MCP server:

```bash
# Static Mode: Single namespace
uvx --from datus-agent datus-mcp --namespace bird_sqlite
uvx --from datus-agent datus-mcp --namespace bird_sqlite --transport http --host 127.0.0.1 --port 8888
# Dynamic Mode: Multi-namespace HTTP/SSE server
uvx --from datus-agent datus-mcp --dynamic --transport http --host 127.0.0.1 --port 8888
uvx --from datus-agent datus-mcp --dynamic --transport sse --host 127.0.0.1 --port 8888

# Or run with python directly
python -m datus.mcp_server --namespace bird_sqlite
# Dynamic Mode: Multi-namespace HTTP/SSE server
python -m datus.mcp_server --dynamic --transport http --host 127.0.0.1 --port 8888
python -m datus.mcp_server --dynamic --transport sse --host 127.0.0.1 --port 8888
```

## Configuration

- **Using uvx:**

```json
{
  "mcpServers": {
    "datus": {
      "command": "uvx",
      "args": [
        "--from",
        "datus-agent",
        "datus-mcp",
        "--namespace",
        "bird_sqlite",
        "--transport",
        "stdio"
      ]
    }
  }
}
```

- **Using python directly:**

```json
{
  "mcpServers": {
    "datus": {
      "command": "python",
      "args": [
        "-m",
        "datus.mcp_server",
        "--namespace",
        "bird_sqlite",
        "--transport",
        "stdio"
      ]
    }
  }
}
```

- **Using HTTP/SSE mode:**
```json
{
  "mcpServers": {
    "DatusServer": {
      "url": "http://127.0.0.1:8888/sse/bird_sqlite",
      "transport": "sse"
    }
  }
}
```

```json
{
  "mcpServers": {
    "DatusServer": {
      "url": "http://127.0.0.1:8888/mcp/bird_sqlite",
      "transport": "http"
    }
  }
}
```

## HTTP Server Mode

**Static Mode (Single Namespace):**

```bash
# Streamable HTTP (default, bidirectional)
datus-mcp --namespace bird_sqlite --transport http --host 0.0.0.0 --port 8000

# SSE mode (for web clients)
datus-mcp --namespace bird_sqlite --transport sse --port 8000
```

Connect to:

- Streamable HTTP: `http://localhost:8000/mcp`
- SSE: `http://localhost:8000/sse`

**Dynamic Mode (Multi-Namespace):**

Run a single server that supports all configured namespaces via URL path:

```bash
# Start dynamic server with sse mode
datus-mcp --dynamic --host 0.0.0.0 --port 8000 --transport sse

# Start dynamic server with http-stream mode
datus-mcp --dynamic --host 0.0.0.0 --port 8000 --transport http

```

Connect to specific namespace:

- HTTP: `http://localhost:8000/mcp/{namespace}`
- SSE: `http://localhost:8000/sse/{namespace}`
- With subagent: `http://localhost:8000/mcp/{namespace}?subagent={subagent_name}`

Example:

- `http://localhost:8000/mcp/bird_sqlite`
- `http://localhost:8000/sse/bird_sqlite`
- `http://localhost:8000/mcp/superset?subagent=sales_dashboard`

Info endpoints:

- `http://localhost:8000/` - Server info and available namespaces
- `http://localhost:8000/health` - Health check

### Available Tools

The MCP server exposes the following tools:

| Category           | Tools                                                                                                                                                             |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Database**       | `list_databases`, `list_schemas`, `list_tables`, `search_table`, `describe_table`, `get_table_ddl`, `read_query`                                                  |
| **Context Search** | `list_subject_tree`, `search_metrics`, `get_metrics`, `search_reference_sql`, `get_reference_sql`, `search_semantic_objects`, `search_knowledge`, `get_knowledge` |

### Command Line Options

```bash
datus-mcp --help

Mode Selection (mutually exclusive, one required):
  --dynamic            Run in dynamic mode: support all namespaces via /mcp/{namespace} URL
  --namespace, -n      Run in static mode with specified namespace

Static Mode Options:
  --sub-agent, -s      Sub-agent name for scoped context
  --database, -d       Database name override
  --transport, -t      Transport type: http (default), sse, stdio

Dynamic Mode Options:
  --transport, -t      Transport type: http (default), sse:
                       http: via /mcp/{namespace} URL
                       sse: via /sse/{namespace} URL

Common Options:
  --config, -c         Path to agent configuration file
  --host               Host to bind for HTTP transports (default: 0.0.0.0)
  --port, -p           Port to bind for HTTP transports (default: 8000)
  --debug              Enable debug logging
```