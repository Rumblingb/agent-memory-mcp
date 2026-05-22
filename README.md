# 🧠 Agent Memory MCP

[![MCP Server](https://img.shields.io/badge/MCP-Server-blue)](https://modelcontextprotocol.io)
[![Python](https://img.shields.io/badge/Python-3.10%2B-green)](https://python.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Smithery](https://img.shields.io/badge/Smithery-Listed-purple)](https://smithery.ai)
[![Pro $19/mo](https://img.shields.io/badge/Pro-%2419%2Fmo-635bff)](https://buy.stripe.com/fZu14p3D94RC9PWa791oI0v)

**Give your AI agents persistent memory across sessions.** A simple key-value store with TTL, namespaces, and fuzzy search — like Redis, but designed for AI agents.

## Why Agent Memory?

AI agents have one fatal flaw: **they forget everything between sessions.** 

Every conversation with an agent starts from zero. Past decisions, user preferences, research findings — all gone. Agent builders hack around this with file I/O, database calls, or cramming everything into the system prompt (which costs tokens).

Agent Memory MCP solves this with a single, elegant abstraction: `remember()` and `recall()`. Agents store what matters and retrieve it later — across sessions, across restarts, across days.

```python
# An agent remembers a user preference
await memory_remember(
    key="user:timezone",
    value="America/Chicago",
    namespace="preferences"
)

# Days later, another session recalls it instantly
tz = await memory_recall(key="user:timezone", namespace="preferences")
# → "America/Chicago"
```

## Features

- **7 structured tools** for complete memory management
- **Namespace isolation** — separate memory spaces for different agents/contexts
- **TTL support** — auto-expiring entries with lazy cleanup
- **Fuzzy search** — find memories by substring across keys and values
- **Access tracking** — know how often each memory is used
- **Thread-safe** — file locking via `fcntl` for concurrent access
- **Zero external dependencies** — only `mcp` package required
- **JSON-file storage** — data stays on your machine in `~/.agent-memory/`
- **Response formatting** — markdown (human) or JSON (programmatic)

## Installation

```bash
# Clone the repository
git clone https://github.com/Rumblingb/agent-memory-mcp.git
cd agent-memory-mcp

# Install dependencies (only mcp is required)
pip install mcp

# Or with requirements.txt
pip install -r requirements.txt
```

### MCP Client Configuration

Add to your MCP client's `config.yaml`:

```yaml
mcpServers:
  agent-memory:
    command: python3
    args:
      - /path/to/agent-memory-mcp/server.py
    description: Persistent key-value memory for AI agents
```

**Claude Desktop:**
```json
{
  "mcpServers": {
    "agent-memory": {
      "command": "python3",
      "args": ["/path/to/agent-memory-mcp/server.py"]
    }
  }
}
```

**VS Code / Cursor:**
```json
{
  "mcpServers": {
    "agent-memory": {
      "command": "python3",
      "args": ["server.py"],
      "cwd": "/path/to/agent-memory-mcp"
    }
  }
}
```

## Tools Reference

### 1. `memory_remember`

Store a value under a key with optional TTL.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | string | ✅ | Key identifier |
| `value` | string | ✅ | Value to store |
| `namespace` | string | ❌ | Namespace (default: `"default"`) |
| `ttl_seconds` | integer | ❌ | Auto-expire after N seconds |

**Example:**
```python
# Store user preference with 30-day TTL
await memory_remember(
    key="user:theme",
    value="dark",
    namespace="preferences",
    ttl_seconds=2592000  # 30 days
)

# Store research findings permanently
await memory_remember(
    key="competitor:pricing:stripe",
    value="Stripe charges 2.9% + $0.30 per transaction for US cards",
    namespace="research"
)
```

### 2. `memory_recall`

Retrieve a value by key with full metadata.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | string | ✅ | Key to retrieve |
| `namespace` | string | ❌ | Namespace (default: `"default"`) |

**Response includes:**
- `value` — the stored value
- `created_at` — when it was stored
- `accessed_at` — last access time
- `expires_at` — TTL expiry (if set)
- `access_count` — how many times it's been recalled

### 3. `memory_forget`

Delete a specific key. Returns confirmation.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | string | ✅ | Key to delete |
| `namespace` | string | ❌ | Namespace (default: `"default"`) |

### 4. `memory_search`

Search memories by keyword/substring across both keys and values.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | ✅ | Search query (case-insensitive substring match) |
| `namespace` | string | ❌ | Limit to specific namespace (omit to search all) |
| `limit` | integer | ❌ | Max results (default: 10) |

**Example:**
```python
# Find everything about Stripe
results = await memory_search(query="stripe", limit=20)

# Search only in research namespace
results = await memory_search(
    query="competitor pricing",
    namespace="research"
)
```

### 5. `memory_list_namespaces`

List all namespaces with entry counts.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `format` | string | ❌ | `"markdown"` (default) or `"json"` |

**Response:**
```
## 📁 Memory Namespaces

| Namespace | Entries | Size |
|-----------|---------|------|
| default | 12 | 4.2 KB |
| preferences | 8 | 1.8 KB |
| research | 45 | 28.5 KB |
| agent:code-reviewer | 156 | 89.3 KB |
| agent:researcher | 89 | 52.1 KB |
```

### 6. `memory_clear_namespace`

Delete ALL entries in a namespace. **Destructive.**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `namespace` | string | ✅ | Namespace to clear |

### 7. `memory_stats`

Get global storage statistics.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `format` | string | ❌ | `"markdown"` (default) or `"json"` |

**Response:**
```
## 📊 Memory Stats

| Metric | Value |
|--------|-------|
| Total Entries | 310 |
| Total Size | 175.9 KB |
| Namespaces | 5 |
| Oldest Entry | 2026-04-15 (27 days ago) |
| Newest Entry | just now |
| Expired Entries | 12 |
```

## Pricing

| Tier | Price | Limits |
|------|-------|--------|
| **Free** | $0 | 1,000 entries, 5 namespaces |
| **Pro** | [$19/month](https://buy.stripe.com/fZu14p3D94RC9PWa791oI0v) | Unlimited entries, unlimited namespaces, priority support |

## Architecture

```
┌──────────────────────────────────────────────────────┐
│              AI Agent (Claude/GPT/Codex)              │
│  remember() / recall() / search() / forget()         │
└──────────────────────┬───────────────────────────────┘
                       │ MCP Protocol (stdio JSON-RPC)
┌──────────────────────▼───────────────────────────────┐
│               Agent Memory MCP Server                 │
│                                                       │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐   │
│  │ KV Store │  │ TTL      │  │ Search Engine     │   │
│  │ Engine   │  │ Manager  │  │ (substring match) │   │
│  └────┬─────┘  └────┬─────┘  └────────┬──────────┘   │
│       │              │                │               │
│  ┌────▼──────────────▼────────────────▼──────────┐   │
│  │              ~/.agent-memory/                 │   │
│  │  {namespace}.json  │  _meta.json              │   │
│  └───────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────┘
```

### Data Storage

All data persists as JSON files in `~/.agent-memory/`:

| File | Contents |
|------|----------|
| `{namespace}.json` | Array of memory entries for each namespace |
| `_meta.json` | Global statistics and index |

**Entry structure:**
```json
{
  "key": "user:theme",
  "value": "dark",
  "created_at": "2026-05-12T10:30:00Z",
  "accessed_at": "2026-05-12T14:22:00Z",
  "expires_at": "2026-06-11T10:30:00Z",
  "access_count": 47
}
```

### Concurrency

Uses `fcntl.flock()` for file-level locking. Multiple agent processes can safely read/write concurrently.

### Tool Annotations

| Tool | readOnlyHint | destructiveHint | idempotentHint | openWorldHint |
|------|-------------|-----------------|----------------|---------------|
| `memory_remember` | false | false | false | false |
| `memory_recall` | true | false | true | false |
| `memory_forget` | false | true | true | false |
| `memory_search` | true | false | true | false |
| `memory_list_namespaces` | true | false | true | false |
| `memory_clear_namespace` | false | true | false | false |
| `memory_stats` | true | false | true | false |

## Usage Scenarios

### Cross-Session User Preferences

```python
# Session 1: Agent learns user preference
await memory_remember(
    key="user:timezone",
    value="America/Chicago",
    namespace="preferences"
)
await memory_remember(
    key="user:currency",
    value="USD",
    namespace="preferences"
)

# Session 2 (days later): Agent recalls preferences instantly
tz = await memory_recall("user:timezone", "preferences")
# No need to ask the user again
```

### Research Accumulation Agent

```python
# Agent stores research findings as it discovers them
await memory_remember(
    key="competitor:acme:api_pricing",
    value="Acme API Pro: $49/mo for 10k calls. Enterprise: $199/mo. No free tier.",
    namespace="research",
    ttl_seconds=86400 * 7  # Keep for 7 days
)

# Later: search all research about competitors
findings = await memory_search(
    query="competitor pricing",
    namespace="research"
)
```

### Agent Scratchpad

```python
# Agent uses memory as a working scratchpad
await memory_remember(
    key="scratch:task:refactor-auth",
    value="Step 1: Extract auth middleware. Step 2: Add token refresh. Step 3: Update tests.",
    namespace="agent:code-reviewer"
)

# Retrieve current task state
state = await memory_recall("scratch:task:refactor-auth", "agent:code-reviewer")
```

### Periodic Cleanup

```python
# Agent cleans up old research data
await memory_clear_namespace(namespace="agent:code-reviewer")
# → "Cleared 156 entries from 'agent:code-reviewer'"
```

## Development

```bash
# Clone and install
git clone https://github.com/Rumblingb/agent-memory-mcp.git
cd agent-memory-mcp
pip install -r requirements.txt

# Test with MCP Inspector
npx @modelcontextprotocol/inspector python3 server.py

# Run tests
python3 -m pytest tests/
```

### Requirements

- Python 3.10+
- `mcp>=1.0.0`

No external dependencies beyond `mcp`. Uses Python stdlib (`json`, `fcntl`, `pathlib`, `datetime`).

## Design Principles

1. **Simplicity over complexity** — Redis-like KV, not a vector database. Agents just need to remember things.
2. **Local-first** — Data stays on your machine. No cloud, no API keys, no latency.
3. **Self-describing** — Tool responses are human-readable markdown by default, JSON for programmatic use.
4. **Tolerant** — Lazy TTL expiry, non-strict search, graceful error handling.

## License

MIT — see [LICENSE](LICENSE) for details.

## Related MCP Servers

- [Agent Cost Tracker MCP](https://github.com/Rumblingb/agent-cost-tracker-mcp) — Track AI agent token usage and costs
- [Search Proxy MCP](https://github.com/Rumblingb/search-proxy-mcp) — Web search for AI agents
- [AgentPassport API](https://github.com/Rumblingb/agentpassport-api) — Governed payment middleware for agents
- [MCP Server Directory](https://rumblingb.github.io/mcp-directory/) — Curated list of all MCP servers

---

Built by [AgentPay Labs](https://agentpay.so) — Governed payment middleware for AI agents.
