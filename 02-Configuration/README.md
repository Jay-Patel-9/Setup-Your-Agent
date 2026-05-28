# 02 — Configuration

The default Hermes install is generic. This section transforms it into a security-focused assistant.

---

## SOUL.md — The Personality Injection

SOUL.md is injected into every message as system context. It's the single most impactful configuration change.

### Location

```
~/.hermes/SOUL.md              ← Global (default profile)
~/.hermes/profiles/<name>/SOUL.md  ← Profile-specific (OVERRIDES global!)
```

### The Override Pitfall

**This is the #1 configuration mistake.** Profile SOUL.md files silently override the global one. If you edit `~/.hermes/SOUL.md` but your active profile has its own SOUL.md, your changes have **no effect**.

```bash
# After editing SOUL.md, ALWAYS check all profiles
for f in ~/.hermes/profiles/*/SOUL.md; do
    echo "=== $f ==="
    wc -c "$f"
done

# If profile SOUL.md exists but is small/empty, it overrides your global
cp ~/.hermes/SOUL.md ~/.hermes/profiles/*/SOUL.md
```

### Pentest SOUL.md Template

A ready-to-use pentest SOUL.md template is at [soul-md-template.md](soul-md-template.md). For the methodology behind crafting an effective SOUL.md, see [References/soul-md-crafting-guide.md](../References/soul-md-crafting-guide.md). The official Hermes SOUL.md documentation is at [hermes-agent.nousresearch.com/docs/configuration#soul-md](https://hermes-agent.nousresearch.com/docs/configuration#soul-md).

Key sections to customize:
- **Identity**: "Red teamer with blue team discipline"
- **Behavior rules**: Evidence-first, no filler, state uncertainty
- **Verification standards**: Network accessibility, restart persistence, live-tested formats
- **Error classification**: Transient/self-correctable/user-fixable/unexpected
- **Assessment methodology**: Recon → Auth → Business Logic → Verify → Report

### SOUL.md Size Budget

SOUL.md is injected into EVERY message. Keep it under 2,000 characters for the core identity block. Use skills for detailed methodology — they load only when relevant.

| SOUL.md Size | Token Cost per Turn | Impact |
|-------------|-------------------|--------|
| <1KB | ~250 tokens | Negligible |
| 1-2KB | ~250-500 tokens | Acceptable |
| 2-5KB | ~500-1,250 tokens | Noticable |
| >5KB | >1,250 tokens | Wasteful |

## Configuration Structure

```
~/.hermes/
├── config.yaml              ← Global configuration
├── .env                     ← API keys and secrets
├── SOUL.md                  ← Global personality
├── profiles/
│   ├── default/
│   │   └── config.yaml      ← Default profile config
│   ├── security/
│   │   ├── config.yaml      ← Security profile config
│   │   └── SOUL.md          ← Security profile personality
│   └── general/
│       └── config.yaml      ← General profile config
├── skills/                   ← Installed skills
├── plugins/                  ← Plugin symlinks (e.g., mnemosyne)
└── mnemosyne/               ← Mnemosyne data (if installed)
```

## Essential Configuration

### `~/.hermes/config.yaml`

```yaml
# Provider and model
provider: openrouter
model: glm-5.1

# API key (prefer .env file for secrets)
# api_key: [REDACTED]  ← Use .env instead

# Fallback providers (auto-switch on 429/529/503)
fallback_providers:
  - provider: openrouter
    model: deepseek/deepseek-v4-flash
  - provider: openrouter
    model: google/gemini-2.5-flash-preview

# Compression (use cheap model for context compression)
compression:
  enabled: true
  summary_model: openrouter/deepseek/deepseek-v4-flash

# Gateway
gateway:
  port: 8080
  host: 0.0.0.0

# Memory (set by mnemosyne.install, verify it's correct)
memory:
  provider: mnemosyne
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 2200
  user_char_limit: 1375
  flush_min_turns: 6

# MCP servers (see 05-MCP-Servers)
mcp_servers: {}

# Skills auto-discovered from ~/.hermes/skills/
```

### `~/.hermes/.env`

```bash
# API keys (never commit this file)
OPENROUTER_API_KEY=[REDACTED]
TELEGRAM_BOT_TOKEN=[REDACTED]

# Mnemosyne tuning
MNEMOSYNE_EMBEDDING_MODEL=BAAI/bge-base-en-v1.5
MNEMOSYNE_RECENCY_HALFLIFE=72
MNEMOSYNE_HOST_LLM_ENABLED=true
```

### YAML Duplicate Key Pitfall

YAML takes the LAST value for duplicate keys. This silently breaks configurations:

```yaml
# BROKEN — provider is '' (last wins)
memory:
  provider: mnemosyne
  provider: ''

# FIXED — only one provider key
memory:
  provider: mnemosyne
```

**Always check for duplicate keys after editing:**
```bash
grep "provider:" ~/.hermes/config.yaml ~/.hermes/profiles/*/config.yaml
```

## Security Hardening

### 1. Credential Management

```bash
# NEVER put API keys in config.yaml — use .env
# config.yaml:
api_key: ${OPENROUTER_API_KEY}   # Reference env var

# .env:
OPENROUTER_API_KEY=sk-or-v1-...  # Actual key (git-ignored)
```

### 2. Telegram Access Control

```yaml
telegram:
  bot_token: ${TELEGRAM_BOT_TOKEN}
  allowed_users:
    - "123456789"     # Your Telegram user ID
  # DO NOT leave allowed_users empty — that allows EVERYONE
```

### 3. Gateway Binding

```yaml
gateway:
  host: 127.0.0.1    # Localhost only (not 0.0.0.0)
  port: 8080
```

For remote access, use a reverse proxy (nginx/Caddy) with TLS.

### 4. Disk Encryption

Mnemosyne stores findings, credentials, and assessment context in SQLite. Consider encrypting the data directory:

```bash
# Option 1: Encrypt the entire ~/.hermes/ directory
# Option 2: Use ecryptfs for just the mnemosyne data
sudo mount -t ecryptfs ~/.hermes/mnemosyne/data ~/.hermes/mnemosyne/data
```

## Profile System

Profiles let you switch between configurations for different workflows:

```bash
# List profiles
hermes profile list

# Create a security profile
hermes profile create security

# Switch to security profile
hermes profile use security

# Edit profile-specific config
vim ~/.hermes/profiles/security/config.yaml
vim ~/.hermes/profiles/security/SOUL.md
```

### Profile Override Rules

Every config key in a profile overrides the global config:

| Config Key | Global | Profile | Effective |
|-----------|--------|---------|-----------|
| `model` | glm-5.1 | claude-3.5-sonnet | claude-3.5-sonnet |
| `memory.provider` | mnemosyne | '' (empty) | '' ← BROKEN! |
| `mcp_servers` | {h1-brain: ...} | {} (empty) | {} ← BROKEN! |

**Rule:** If a profile has a key set to an empty value, it shadows the global value. Always check both locations.

## Credential Pooling

For teams using shared infrastructure:

```yaml
# Use environment variables for all secrets
# ~/.hermes/.env (not committed to git)
OPENROUTER_API_KEY=sk-or-v1-...
TELEGRAM_BOT_TOKEN=...
H1_USERNAME=...
H1_API_TOKEN=...

# config.yaml references env vars
provider: openrouter
api_key: ${OPENROUTER_API_KEY}

mcp_servers:
  h1-brain:
    command: python
    args: ["/opt/h1-brain/server.py"]
    env:
      H1_USERNAME: ${H1_USERNAME}
      H1_API_TOKEN: ${H1_API_TOKEN}
```

## Next Steps

- **[soul-md-template.md](soul-md-template.md)** — Copy-paste pentest SOUL.md template
- **[03-Memory-Setup](../03-Memory-Setup/README.md)** — Install Mnemosyne for persistent memory
- **[References/soul-md-crafting-guide.md](../References/soul-md-crafting-guide.md)** — Deep dive on SOUL.md methodology

---

| [← Installation](../01-Installation/README.md) | 🏠 [Home](../README.md) | [Next: Memory Setup →](../03-Memory-Setup/README.md) |
|---|---|---|