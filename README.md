# Setup-Pentest-Agent

> Complete guide for configuring Hermes Agent as a penetration testing assistant - from installation to production.

A self-contained GitHub wiki covering installation, configuration, memory setup, skills, MCP servers, troubleshooting, agent comparison, and failure modes.

---

## Sections

| # | Section | Description |
|---|---------|-------------|
| 01 | [Installation](01-Installation/README.md) | Install Hermes, first run, setup wizard, dependency checks |
| 02 | [Configuration](02-Configuration/README.md) | SOUL.md, profiles, security hardening, credential pooling |
| 03 | [Memory Setup](03-Memory-Setup/README.md) | Mnemosyne - persistent local memory for multi-session assessments |
| 04 | [Skills](04-Skills/README.md) | Skill system, pentest skill catalog, authoring, curation |
| 05 | [MCP Servers](05-MCP-Servers/README.md) | Security data sources - HackerOne, Shodan, NVD, custom servers |
| 06 | [Troubleshooting](06-Troubleshooting/README.md) | Profile overrides, gateway issues, YAML pitfalls, WSL quirks |
| 07 | [Agent Comparison](07-Agent-Comparison/README.md) | Hermes vs Claude Code vs Codex vs OpenCode vs OpenClaw |
| 08 | [Where Agents Fail](08-Where-Agents-Fail/README.md) | The 60/40 rule, failure modes, mitigations, honest assessment |

## References

- [Mnemosyne Setup Reference](References/mnemosyne-setup.md) - Full installation, config, and troubleshooting reference
- [Hermes Optimization Roadmap](References/hermes-optimization-roadmap.md) - Community-vetted optimizations prioritized by impact
- [Security MCP Servers](References/security-mcp-servers.md) - h1-brain, Shodan, NVD, and custom server setup
- [SOUL.md Crafting Guide](References/soul-md-crafting-guide.md) - Methodology for personalized SOUL.md creation

## Quick Start

```bash
# 1. Install Hermes
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash

# 2. Configure provider
hermes config setup model #interactive

# 3. Connect
hermes setup 

# 4. Install security skills [Example Skills]
hermes skills install ctf-web-audit
hermes skills install web-app-auth-bypass
hermes skills install vite-spa-api-extraction

# 5. Personalize SOUL.md
cp 02-Configuration/soul-md-template.md ~/.hermes/SOUL.md

# 6. Verify
hermes doctor
```

## Repository Structure

```
Setup-Pentest-Agent/
├── README.md                           ← You are here
├── .gitignore
├── 01-Installation/
│   └── README.md
├── 02-Configuration/
│   ├── README.md
│   └── soul-md-template.md
├── 03-Memory-Setup/
│   └── README.md
├── 04-Skills/
│   └── README.md
├── 05-MCP-Servers/
│   └── README.md
├── 06-Troubleshooting/
│   └── README.md
├── 07-Agent-Comparison/
│   └── README.md
├── 08-Where-Agents-Fail/
│   └── README.md
└── References/
    ├── mnemosyne-setup.md
    ├── hermes-optimization-roadmap.md
    ├── security-mcp-servers.md
    └── soul-md-crafting-guide.md
```

## External References & Links

### Official Documentation
- [Hermes Agent Docs](https://hermes-agent.nousresearch.com/docs) - Configuration, skills, memory, gateway
- [Hermes Agent GitHub](https://github.com/nousresearch/hermes-agent) - Source code, issues, contributions
- [Hermes Skills Hub](https://hermes-agent.nousresearch.com/skills) - Browse and install community skills
- [Hermes Skill Authoring Guide](https://hermes-agent.nousresearch.com/docs/skills#authoring) - Write custom SKILL.md files

### Memory (Mnemosyne)
- [Mnemosyne GitHub](https://github.com/AxDSan/mnemosyne) - Source, issues, deployment scripts
- [Hermes Memory Docs](https://hermes-agent.nousresearch.com/docs/memory) - Provider setup, BEAM architecture
- [BGE Embedding Models](https://huggingface.co/BAAI/bge-base-en-v1.5) - fastembed-supported models for vector search
- [r/hermesagent Mnemosyne Thread](https://www.reddit.com/r/hermesagent/s/ANCT3yxmk7) - Community comparison of memory providers

### SOUL.md & Personalization
- [SOUL.md Crafting Guide](References/soul-md-crafting-guide.md) - Methodology for pentest-specific SOUL.md
- [SOUL.md Template](02-Configuration/soul-md-template.md) - Ready-to-use copy-paste template
- [Hermes SOUL.md Docs](https://hermes-agent.nousresearch.com/docs/configuration#soul-md) - Official reference

### Security MCP Servers
- [h1-brain](https://github.com/AxDSan/h1-brain) - 3,673+ HackerOne disclosed reports (SQLite DB)
- [Shodan MCP Server](https://www.npmjs.com/package/@anthropic/mcp-server-shodan) - Internet-wide scanning data
- [NVD MCP Server](https://www.npmjs.com/package/mcp-server-nvd) - NIST vulnerability database
- [MCP Protocol Spec](https://modelcontextprotocol.io/) - Model Context Protocol specification

### Provider & Model
- [OpenRouter](https://openrouter.ai/) - 100+ models via single API key
- [Hermes Optimization Roadmap](References/hermes-optimization-roadmap.md) - Community-vetted optimizations

### Community
- [r/hermesagent](https://www.reddit.com/r/hermesagent/) - Community tips, troubleshooting, skill sharing

## Disclaimer

This guide is for authorized security testing and educational purposes only. Always obtain proper authorization before testing any system. The authors assume no liability for misuse.

---

*Based on community insights from [r/hermesagent](https://www.reddit.com/r/hermesagent/), [Hermes documentation](https://hermes-agent.nousresearch.com/docs), and real-world pentest experience.*
