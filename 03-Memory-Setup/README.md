# 03 - Memory Setup (Mnemosyne)

Persistent local memory for Hermes - the bridge between single-session chatbot and multi-session professional tool.

---

## Why Persistent Memory Matters for Pentesting

**Without memory**: Every session starts fresh. You re-explain your environment, preferences, and past findings every time.

**With Mnemosyne**: 
- Findings persist across days/weeks
- Agent recalls your preferences without asking
- Recon data from session 1 feeds exploitation in session 5
- Multi-day assessments maintain full context carry-forward

## Architecture (BEAM)

Bilevel Episodic-Associative Memory:

- **Working Memory** - Short-term store for current session; auto-consolidated to episodic
- **Episodic Memory** - Long-term store with vector (sqlite-vec) + FTS5 hybrid search
- **Scratchpad** - Temporary notes that don't persist across sessions

Consolidation (`mnemosyne_sleep`) moves working → episodic, optionally using a local LLM for summarization.

```
Session input → Working Memory → (sleep/consolidate) → Episodic Memory
                                        ↓
                                   Scratchpad (temporary)
```

## Installation

```bash
HERMES_PY="$HOME/.hermes/hermes-agent/venv/bin/python"

# Core package
"$HERMES_PY" -m pip install --no-cache-dir mnemosyne-memory

# Semantic search (required for vector recall)
"$HERMES_PY" -m pip install --no-cache-dir fastembed

# Local LLM consolidation (optional but recommended)
"$HERMES_PY" -m pip install --no-cache-dir ctransformers huggingface-hub

# Register plugin with Hermes
"$HERMES_PY" -m mnemosyne.install

# Restart gateway to pick up new provider
hermes gateway restart
```

**One-liner alternative:**
```bash
curl -sSL https://raw.githubusercontent.com/AxDSan/mnemosyne/main/deploy_hermes_provider.sh | bash
```

## Configuration

### `~/.hermes/config.yaml`

```yaml
memory:
  provider: mnemosyne          # Must be set (installer does this)
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 2200
  user_char_limit: 1375
  flush_min_turns: 6
  mnemosyne:                   # Provider-specific block
    auto_sleep: true            # Auto-consolidate at session boundaries
    sleep_threshold: 50         # Min working mem entries before auto-sleep
    vector_type: int8           # float32 | int8 (default) | bit (32x smaller)
```

### ⚠️ CRITICAL: Duplicate Key Pitfall

YAML takes the LAST duplicate key value. If your config.yaml has:
```yaml
memory:
  provider: mnemosyne
  provider: ''              # ← This wins! Mnemosyne disabled!
```

**Always check for duplicate keys after editing:**
```bash
grep "provider:" ~/.hermes/config.yaml ~/.hermes/profiles/*/config.yaml
```

### ⚠️ CRITICAL: Profile Override Pitfall

`python -m mnemosyne.install` only updates `~/.hermes/config.yaml` (global). It does NOT touch profile configs.

If `~/.hermes/profiles/<name>/config.yaml` has `memory: { provider: '' }`, it overrides the global `mnemosyne` entry. **This is the most common Mnemosyne installation failure.**

**Fix:** Manually update each profile's config:
```bash
# For each profile
hermes profile use <name>
hermes config set memory.provider mnemosyne
# Or edit directly:
vim ~/.hermes/profiles/<name>/config.yaml
```

### Pentest-Optimized Tuning

```yaml
mnemosyne:
  auto_sleep: true
  sleep_threshold: 30          # Faster consolidation (default 50, security context decays faster)
```

Environment variables (add to `~/.hermes/.env`):

```bash
# Better recall accuracy
MNEMOSYNE_EMBEDDING_MODEL=BAAI/bge-base-en-v1.5   # 768d vs default 384d

# Security context decays faster than default
MNEMOSYNE_RECENCY_HALFLIFE=72                        # 72h vs default 168h

# Use host model for consolidation quality
MNEMOSYNE_HOST_LLM_ENABLED=true
```

## Verification

```bash
hermes memory status              # Should show "Provider: mnemosyne"
hermes mnemosyne stats            # Memory counts
hermes mnemosyne inspect "test"   # Search memories
hermes doctor                     # Full health check
```

If `hermes memory status` shows "Provider: built-in" or "Provider: ''", see the Profile Override Pitfall above.

## The 17 Mnemosyne Tools

| Tool | Purpose |
|------|---------|
| `mnemosyne_remember` | Store a memory with metadata |
| `mnemosyne_recall` | Search memories (hybrid vector+FTS+keyword) |
| `mnemosyne_sleep` | Consolidate working → episodic memory |
| `mnemosyne_stats` | Current session memory counts |
| `mnemosyne_invalidate` | Mark a memory as outdated |
| `mnemosyne_triple_add` | Add knowledge graph triple |
| `mnemosyne_triple_query` | Query knowledge graph |
| `mnemosyne_scratchpad_write` | Write to scratchpad |
| `mnemosyne_scratchpad_read` | Read scratchpad |
| `mnemosyne_scratchpad_clear` | Clear scratchpad |
| `mnemosyne_export` | Export memories to JSON |
| `mnemosyne_import` | Import memories from JSON |
| `mnemosyne_update` | Update an existing memory |
| `mnemosyne_forget` | Delete a memory |
| `mnemosyne_diagnose` | Diagnostic info |
| `mnemosyne_graph_query` | Query memory graph |
| `mnemosyne_graph_link` | Link memories in graph |

## Common Issues

| Problem | Fix |
|---------|-----|
| `ModuleNotFoundError` after pip install | Installed into system Python, not Hermes venv. Use `$HERMES_PY -m pip install ...` |
| `Provider: built-in` despite config | Profile override. Check `~/.hermes/profiles/*/config.yaml` |
| First recall takes 10s | fastembed downloads models on first use. Subsequent calls are instant |
| Memory not persisted across sessions | `auto_sleep: true` must be set. Run `hermes mnemosyne sleep` manually if needed |
| Duplicate `provider:` keys in config.yaml | YAML takes last value. Remove duplicates |

## Comparison with Other Providers

From r/hermesagent community (73 upvote thread):

| Provider | Verdict |
|----------|---------|
| Cloud providers | ❌ Vendor lock-in, data retention |
| Hindsight | Good quality but heavy, costly, buggy |
| OpenViking | Pain to set up |
| Holographic | Fast but quality not there |
| Honcho | Good profiling, same heaviness as Hindsight |
| **Mnemosyne** | ✅ Best balance - easy, lightweight, local, good quality |

---

| [← Configuration](../02-Configuration/README.md) | 🏠 [Home](../README.md) | [Next: Skills →](../04-Skills/README.md) |
|---|---|---|