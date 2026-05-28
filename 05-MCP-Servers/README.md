# 05 - MCP Servers (Model Context Protocol)

MCP servers extend Hermes with external tool integrations - security databases, OSINT APIs, and custom data sources.

---

## What Is MCP?

The Model Context Protocol (MCP) is a standard for connecting external data sources and tools to AI agents. Each MCP server exposes tools that Hermes can discover and use automatically.

Hermes has a **native MCP client** - no extra Python packages or daemons needed. Just add a server config and restart the gateway.

## Configuration

### `~/.hermes/config.yaml`

```yaml
mcp_servers:
  h1-brain:
    command: python
    args: ["/opt/h1-brain/server.py"]
    env:
      H1_USERNAME: "[REDACTED]"         # Your HackerOne username
      H1_API_TOKEN: "[REDACTED]"         # Your HackerOne API token
  
  shodan:
    command: npx
    args: ["-y", "@anthropic/mcp-server-shodan"]
    env:
      SHODAN_API_KEY: "[REDACTED]"
  
  custom-tool:
    command: python
    args: ["/path/to/your/server.py"]
```

### ⚠️ Profile Override Pitfall

Same pattern as memory providers: **profile configs override global configs silently**. After adding MCP servers to `~/.hermes/config.yaml`, check each active profile:

```bash
# Check for existing MCP configs in all profiles
grep -r "mcp_servers" ~/.hermes/profiles/*/config.yaml

# If a profile has an empty mcp_servers key, it overrides your global config
# Either add the same servers to each profile, or remove the key from profiles
```

### Environment Variables

For sensitive tokens, use environment variables instead of hardcoding:

```yaml
mcp_servers:
  h1-brain:
    command: python
    args: ["/opt/h1-brain/server.py"]
    env:
      H1_USERNAME: "${H1_USERNAME}"    # Read from environment
      H1_API_TOKEN: "${H1_API_TOKEN}"
```

Set these in `~/.hermes/.env`:
```bash
H1_USERNAME=your_username
H1_API_TOKEN=your_token
```

## Security-Focused MCP Servers

### h1-brain (HackerOne Knowledge Base)

3,673+ public disclosed HackerOne reports in a searchable SQLite database (17MB, via Git LFS).

```bash
# Setup
git clone https://github.com/AxDSan/h1-brain.git /opt/h1-brain
cd /opt/h1-brain
python3 -m venv venv && source venv/bin/activate
pip install mcp httpx
git lfs install && git lfs pull

# Add to Hermes
hermes config set mcp_servers.h1-brain.command "python"
hermes config set mcp_servers.h1-brain.args '["/opt/h1-brain/server.py"]'
```

**Without API credentials:** Limited to public disclosed reports only.
**With H1 credentials:** Full personal report sync.

**12 tools** (prefixed `mcp_h1_brain_*`): search_reports, get_report, list_reports, search_by_severity, search_by_category, get_stats, get_random_report, get_trending, and more.

### Shodan MCP Server

```yaml
mcp_servers:
  shodan:
    command: npx
    args: ["-y", "@anthropic/mcp-server-shodan"]
    env:
      SHODAN_API_KEY: "[REDACTED]"
```

**Requires:** Shodan API key (free tier available)  
**Tools:** search, host, exploits, org, query_tags

### Other Security MCP Servers

| Server | Purpose | Source |
|--------|---------|--------|
| `mcp-server-nvd` | NIST vulnerability database | `npx -y mcp-server-nvd` |
| `mcp-server-virustotal` | VirusTotal file/URL/IP analysis | GitHub: AgentMakerLab |
| `mcp-server-whois` | Domain WHOIS lookups | `npx -y mcp-server-whois` |

## Building Custom MCP Servers

Any tool that reads stdin and writes JSON can be an MCP server:

```python
#!/usr/bin/env python3
"""Minimal MCP server template"""
import json
import sys

def handle_request(request):
    method = request.get("method", "")
    
    if method == "tools/list":
        return {"tools": [{"name": "my_tool", "description": "Does something useful",
                           "inputSchema": {"type": "object", "properties": {"target": {"type": "string"}},
                                           "required": ["target"]}}]}
    elif method == "tools/call":
        params = request.get("params", {})
        if params.get("name") == "my_tool":
            result = do_something(params["arguments"]["target"])
            return {"content": [{"type": "text", "text": str(result)}]}
    return {"error": f"Unknown method: {method}"}

for line in sys.stdin:
    print(json.dumps(handle_request(json.loads(line))), flush=True)
```

### MCP Server Requirements

1. **stdio transport** - Read JSON from stdin, write JSON to stdout
2. **tools/list method** - Return available tools and their schemas
3. **tools/call method** - Execute a tool and return results
4. **No stderr pollution** - Errors go in the response, not stderr

## Verification

```bash
hermes gateway restart          # Load new MCP servers
hermes tools | grep mcp_       # List registered MCP tools
hermes doctor                   # System health check
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| MCP tools not appearing | Restart gateway: `hermes gateway restart` |
| `command not found` error | Check Python/path is absolute, not relative |
| Server crashes silently | Check stderr: `hermes gateway logs` |
| Profile missing servers | Update profile config too (see Profile Override Pitfall) |
| `npx` not found | Install Node.js: `sudo apt install -y nodejs npm` |

---

| [← Skills](../04-Skills/README.md) | 🏠 [Home](../README.md) | [Next: Troubleshooting →](../06-Troubleshooting/README.md) |
|---|---|---|