# 06 — Troubleshooting

The most common Hermes issues come from configuration conflicts, not bugs. Here's every failure mode we've seen and the exact fix.

---

## Configuration Issues

### Profile Override Conflicts

**The #1 most common problem.** Every config key in a profile overrides the global config, silently.

**Symptoms:**
- Mnemosyne shows as "built-in" or "'' " after install
- MCP servers don't appear despite being in global config
- SOUL.md changes in `~/.hermes/SOUL.md` have no effect
- Model changes in global config don't take effect

**Diagnose:**
```bash
# Check for conflicting profile overrides
diff <(grep -v "^#" ~/.hermes/config.yaml | grep -v "^$") \
     <(grep -v "^#" ~/.hermes/profiles/*/config.yaml | grep -v "^$")

# Nuclear option: reset a profile to inherit global
rm ~/.hermes/profiles/<name>/config.yaml
hermes gateway restart
```

**Prevention:** After ANY config change to `~/.hermes/config.yaml`, also update each active profile:
```bash
for profile in ~/.hermes/profiles/*/config.yaml; do
    grep -E "(provider:|mcp_servers:|model:)" "$profile"
done
```

### YAML Duplicate Keys

YAML takes the LAST duplicate key value. This silently breaks configs:

```yaml
# BROKEN — provider is '' (last wins)
memory:
  provider: mnemosyne
  provider: ''

# FIXED — only one key
memory:
  provider: mnemosyne
```

**Check:**
```bash
python3 -c "
import yaml, sys
with open(sys.argv[1]) as f:
    lines = f.readlines()
keys = [l.split(':')[0].strip() for l in lines if ':' in l and not l.strip().startswith('#')]
dups = [k for k in set(keys) if keys.count(k) > 1]
print(f'Duplicate keys: {dups}' if dups else 'No duplicates found')
" ~/.hermes/config.yaml
```

## Gateway Issues

### Gateway Won't Start

```bash
ss -tlnp | grep 8080     # Check if port is already in use
pkill -f "hermes.*gateway"  # Kill stale processes
hermes gateway restart --verbose  # Restart with verbose logging
hermes gateway logs      # Check logs
hermes gateway reset     # Nuclear: reset gateway state
```

### Gateway Crashes on Boot

**Common causes:**
1. Malformed YAML in config files
2. Missing Python dependencies (especially after OS updates)
3. Port conflict with another service

```bash
# Validate YAML
python3 -c "import yaml; yaml.safe_load(open('$HOME/.hermes/config.yaml'))"

# Check dependencies
hermes doctor

# Try safe mode (minimal config)
mv ~/.hermes/config.yaml ~/.hermes/config.yaml.bak
hermes gateway start  # Uses defaults
```

## Memory Issues

### Mnemosyne Not Working

See [03-Memory-Setup/README.md](../03-Memory-Setup/README.md#common-issues) for the full troubleshooting table.

Quick diagnostic:
```bash
hermes memory status              # What provider is active?
hermes mnemosyne diagnose         # Dependency check
hermes doctor                     # Full system health
```

### Memory Not Persisting Across Sessions

```bash
# Check auto_sleep is enabled
grep -A5 "mnemosyne:" ~/.hermes/config.yaml

# Manually consolidate
hermes mnemosyne sleep

# Verify consolidation worked
hermes mnemosyne stats            # Working memory count should drop after sleep
```

## Model / Provider Issues

### Model Not Responding

```bash
hermes config get model          # Check current model
hermes config get provider       # Check provider
hermes chat "ping"               # Test connectivity
hermes gateway logs | grep -i "429\|rate\|limit"  # Check for rate limits
```

### Fallback Provider Not Working

```yaml
# CORRECT format (list of dicts)
fallback_providers:
  - provider: openrouter
    model: deepseek/deepseek-v4-flash
  - provider: openrouter
    model: google/gemini-2.5-flash-preview

# WRONG format (single dict — legacy)
fallback_model:
  provider: openrouter
  model: deepseek/deepseek-v4-flash
```

**Triggers:** 429 (rate limit), 529 (overloaded), 503 (unavailable), connection failures.

## Skill Issues

### Skills Not Loading

```bash
hermes skills list               # List loaded skills
/reload-skills                    # Force re-scan
ls -la ~/.hermes/skills/*/       # Check skill file locations
head -20 ~/.hermes/skills/<category>/<name>/SKILL.md  # Verify syntax
```

### Skill Content Not Appearing in Context

Skills load based on relevance matching. If a skill isn't loading:
1. Check the `metadata.hermes.tags` field covers your use case
2. Ensure the skill name isn't too generic (conflicts with built-in skills)
3. Try loading explicitly: `/skill <name>`

## Telegram / Chat Issues

### Bot Not Responding

```bash
hermes gateway status            # Check gateway
hermes config get telegram.bot_token  # Check token
curl "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getMe"  # Test bot
hermes gateway restart           # Restart
```

### Messages Out of Order / Duplicated

Multiple gateway instances are running:
```bash
pkill -f "hermes.*gateway"
hermes gateway start
```

## WSL-Specific Issues

### File Performance

```bash
# WSL2 filesystem operations are slow on /mnt/c/
# Keep Hermes project files in Linux filesystem (~/.hermes/) not Windows mounts
ls -la ~/.hermes/              # Good — Linux filesystem
```

### Network Connectivity

```bash
# WSL2 DNS issues (common on Corporate VPNs)
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# Test connectivity
curl -s https://api.openrouter.ai/v1/models | head -c 100
```

## Quick Fix Cheat Sheet

| Symptom | One-Line Fix |
|---------|-------------|
| Mnemosyne "not found" | `hermes config set memory.provider mnemosyne && hermes gateway restart` |
| Skills not loading | `/reload-skills` |
| Gateway won't start | `hermes gateway reset && hermes gateway start` |
| Config changes not taking | `diff ~/.hermes/config.yaml ~/.hermes/profiles/*/config.yaml` |
| Slow responses | Enable fallback provider (see [02-Configuration](../02-Configuration/README.md)) |
| Memory not persisting | `hermes mnemosyne sleep && hermes mnemosyne stats` |

---

| [← MCP Servers](../05-MCP-Servers/README.md) | 🏠 [Home](../README.md) | [Next: Agent Comparison →](../07-Agent-Comparison/README.md) |
|---|---|---|