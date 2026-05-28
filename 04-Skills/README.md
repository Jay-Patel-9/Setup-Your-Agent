# 04 - Skills

Skills are the unlock. Without domain-specific skills, an agent is just a chatbot with curl access.

---

## What Are Skills?

Skills are reusable procedure documents that load into every session. They encode proven methodologies:

- **Tools** = what the agent CAN do (run curl, read files, browse)
- **Skills** = what the agent KNOWS how to do (SSRF testing with 7 bypass techniques, systematic auth bypass methodology)

Skills are plain Markdown files with YAML frontmatter, stored in `~/.hermes/skills/`. They're auto-discovered and loaded into context when relevant.

## Essential Security Skills

### Core Methodology Skills

| Skill | Purpose | Category |
|-------|---------|----------|
| `ctf-web-audit` | Systematic web app audit for business logic vulns | cyber-specialist |
| `web-app-auth-bypass` | Auth bypass testing methodology | cyber-specialist |
| `web-response-manipulation` | mitmproxy response manipulation scripts | cyber-specialist |
| `ctf-prompt-injection` | LLM prompt-injection CTF methodology | cyber-specialist |
| `pentesting-with-agents` | Agent evaluation, demo methodology | cyber-specialist |
| `ctf-challenge-creation` | Build deployable CTF challenges | cyber-specialist |
| `offensive-osint` | Recon toolkit for red-team operations | offensive-osint |
| `osint-methodology` | 5-stage recon pipeline | osint-methodology |
| `cve-2026-9082-drupal-sqli` | Drupal SQL injection PoC and remediation | cyber-specialist |

### Supporting Skills

| Skill | Purpose | Category |
|-------|---------|----------|
| `hermes-agent` | Full Hermes configuration reference | autonomous-ai-agents |
| `agent-harness-engineering` | Verification loops, error classification, context hygiene | software-development |
| `godmode` | LLM jailbreak techniques (red teaming) | red-teaming |

## Managing Skills

```bash
# List installed skills
hermes skills list

# Browse community hub (100+ skills)
hermes skills browse

# Search for specific skills
hermes skills search "security"

# Install from hub
hermes skills install <id>

# Install from URL
hermes skills install https://raw.githubusercontent.com/.../SKILL.md

# Install with custom name
hermes skills install https://.../SKILL.md --name my-custom-skill

# Preview without installing
hermes skills inspect <id>

# Check for updates
hermes skills check

# Update outdated skills
hermes skills update

# Remove a hub skill
hermes skills uninstall <name>

# Load a skill into current session
/skill <name>

# Re-scan skills directory
/reload-skills
```

**Reference:** [Hermes Skills Hub](https://hermes-agent.nousresearch.com/skills) - Browse all community skills. [Skill Authoring Guide](https://hermes-agent.nousresearch.com/docs/skills#authoring) - Write custom SKILL.md files.

## Skill Authoring

Skills are Markdown files with YAML frontmatter stored in `~/.hermes/skills/<category>/<name>/SKILL.md`.

### Anatomy of a Skill

```
~/.hermes/skills/cyber-specialist/my-custom-audit/
├── SKILL.md              # Main skill document (required)
├── references/            # Supporting reference docs
│   └── methodology.md
├── templates/              # Templates for common outputs
│   └── report-template.md
└── scripts/                # Executable scripts
    └── exploit-test.sh
```

### SKILL.md Frontmatter

```yaml
---
name: my-custom-audit
description: Systematic methodology for auditing X-type applications
version: 1.0.0
metadata:
  hermes:
    tags: [security, audit, methodology]
---
```

### Key Sections to Include

1. **When to Use This Skill** - Trigger conditions
2. **Methodology** - Step-by-step procedure with numbered steps
3. **Pitfalls** - Common mistakes and how to avoid them
4. **Verification** - How to confirm each step worked
5. **References** - Links to supporting docs, tools, standards

### Skill Loading Behavior

- Skills are loaded fresh every message (no restart needed)
- Only loaded skills matching the current task appear in context
- Skills have a token budget - keep them focused and avoid bloat
- The `trigger` field in frontmatter helps Hermes decide when to load

### Publishing Skills

```bash
# Publish to the community hub
hermes skills publish ~/.hermes/skills/my-category/my-skill/

# Add a GitHub repo as a skill source
hermes skills tap add https://github.com/example/hermes-skills
```

**Reference:** [Skill Authoring Guide](https://hermes-agent.nousresearch.com/docs/skills#authoring) - Full specification for SKILL.md frontmatter, sections, and validation. [Hermes Skills Hub](https://hermes-agent.nousresearch.com/skills) - Publish and discover community skills.

## Curating Your Skill Library

For a pentesting workflow, organize skills by engagement phase:

### Recon Phase
- `offensive-osint` - Asset discovery, breach data, identity mapping
- `osint-methodology` - Systematic 5-stage recon pipeline
- `vite-spa-api-extraction` - Extract all API endpoints from JS

### Testing Phase
- `ctf-web-audit` - Systematic web app vulnerability hunting
- `web-app-auth-bypass` - Authentication and authorization testing
- `cve-2026-9082-drupal-sqli` - Specific exploit methodology

### Verification Phase
- `web-response-manipulation` - Response manipulation for bypass verification
- `agent-harness-engineering` - Verification loops for findings

### Reporting Phase
- Use `mnemosyne_remember` to store findings
- Export with `mnemosyne_export` for deliverables

## The 60/40 Rule in Practice

Skills automate the 60% grunt work:
- **Enumeration**: Skills systematically check every endpoint
- **Documentation**: Agent logs everything in structured format
- **Pattern matching**: Skills recognize known vulnerability patterns
- **Repetitive testing**: Skills apply the same checklist every time

The 40% human work skills CANNOT do:
- **Creative exploitation chains**: IDOR → JWT forgery → privilege escalation
- **Business logic intuition**: Recognizing when something "feels wrong"
- **Novel vulnerability classes**: Attacks not in training data
- **False positive discrimination**: Knowing when "confirmed" is actually a decoy

---

| [← Memory Setup](../03-Memory-Setup/README.md) | 🏠 [Home](../README.md) | [Next: MCP Servers →](../05-MCP-Servers/README.md) |
|---|---|---|