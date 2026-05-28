# Security MCP Servers Reference

Complete reference for setting up and using MCP servers that extend Hermes with security-focused data sources.

---

## What Is MCP?

The Model Context Protocol (MCP) connects external tools and data sources to AI agents. Each MCP server exposes tools that Hermes discovers automatically. Hermes has a **native MCP client** — no extra packages or daemons needed.

---

## h1-brain (HackerOne Knowledge Base)

3,673+ public disclosed HackerOne reports in a searchable SQLite database (17MB, via Git LFS).

### Installation

```bash
git clone https://github.com/AxDSan/h1-brain.git /opt/h1-brain
cd /opt/h1-brain
python3 -m venv venv && source venv/bin/activate
pip install mcp httpx
git lfs install && git lfs pull

# Verify
python -c "import sqlite3; conn = sqlite3.connect('h1_brain.db'); \
print(f'{len(conn.execute(\"SELECT * FROM reports\").fetchall())} reports loaded')"
```

### Hermes Configuration

```yaml
mcp_servers:
  h1-brain:
    command: python
    args: ["/opt/h1-brain/server.py"]
    env:
      H1_USERNAME: "[REDACTED]"
      H1_API_TOKEN: "[REDACTED]"
```

Without credentials: public reports only. With credentials: full personal sync.

### Tools (12 total, prefixed `mcp_h1_brain_*`)

- `search_reports` — Full-text search
- `get_report` — Get report by ID
- `list_reports` — Paginated listing
- `get_report_comments` — Comments on a report
- `get_report_activities` — Activity timeline
- `get_report_summaries` — Condensed summaries
- `search_by_severity` — Filter by severity level
- `search_by_category` — Filter by vulnerability category
- `get_stats` — Database statistics
- `get_random_report` — Random report for learning
- `get_trending` — Trending reports
- `sync_personal` — Sync personal reports (requires auth)

---

## Shodan MCP Server

```yaml
mcp_servers:
  shodan:
    command: npx
    args: ["-y", "@anthropic/mcp-server-shodan"]
    env:
      SHODAN_API_KEY: "[REDACTED]"
```

**Requires:** Shodan API key (free tier at https://shodan.io)

**Tools:** search, host, exploits, org, query_tags

---

## NVD (National Vulnerability Database)

```yaml
mcp_servers:
  nvd:
    command: npx
    args: ["-y", "mcp-server-nvd"]
```

**No API key required** (public NVD API, rate-limited without key).

**Tools:** search_cve, get_cve, get_cwe

---

## Other Security MCP Servers

| Server | Purpose | Install |
|--------|---------|---------|
| `mcp-server-virustotal` | File/URL/IP analysis | GitHub: AgentMakerLab |
| `mcp-server-whois` | Domain WHOIS lookups | `npx -y mcp-server-whois` |

---

## Custom MCP Server Template

```python
#!/usr/bin/env python3
"""Minimal security MCP server template"""
import json
import sys

def handle_request(request):
    method = request.get("method", "")
    if method == "tools/list":
        return {"tools": [{"name": "my_tool", "description": "Does something useful",
                           "inputSchema": {"type": "object",
                                           "properties": {"target": {"type": "string"}},
                                           "required": ["target"]}}]}
    elif method == "tools/call":
        params = request.get("params", {})
        if params.get("name") == "my_tool":
            result = do_something(params["arguments"]["target"])
            return {"content": [{"type": "text", "text": str(result)}]}
    return {"error": f"Unknown method: {method}"}

for line in sys.stdin:
    try:
        print(json.dumps(handle_request(json.loads(line))), flush=True)
    except json.JSONDecodeError:
        print(json.dumps({"error": "Invalid JSON"}), flush=True)
```

**MCP Server Requirements:**
1. stdio transport — Read JSON from stdin, write JSON to stdout
2. `tools/list` method — Return available tools and schemas
3. `tools/call` method — Execute a tool and return results
4. No stderr pollution — Errors go in the response, not stderr

---

## Verification

```bash
hermes gateway restart          # Load new MCP servers
hermes tools | grep mcp_       # List registered tools
hermes doctor                   # System health check
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| MCP tools not appearing | `hermes gateway restart` |
| `command not found` for npx | Install Node.js |
| `command not found` for python | Use absolute path |
| Server crashes on start | `hermes gateway logs` |
| Profile missing servers | Update profile config too |
| Rate limiting | Add API key for higher limits |
| Git LFS not pulling | `git lfs install && git lfs pull` |

## References

- [MCP Protocol Spec](https://modelcontextprotocol.io/)
- [h1-brain GitHub](https://github.com/AxDSan/h1-brain)
- [Shodan MCP Server](https://www.npmjs.com/package/@anthropic/mcp-server-shodan)
- [NVD MCP Server](https://www.npmjs.com/package/mcp-server-nvd)
- [Hermes MCP Docs](https://hermes-agent.nousresearch.com/docs/mcp)

| 🏠 [Home](../README.md) |
|---|