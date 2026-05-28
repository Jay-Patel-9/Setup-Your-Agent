# Mnemosyne Setup Reference

Complete reference for installing, configuring, and operating Mnemosyne as the persistent memory provider for Hermes Agent.

---

## Quick Start

```bash
HERMES_PY="$HOME/.hermes/hermes-agent/venv/bin/python"
"$HERMES_PY" -m pip install --no-cache-dir mnemosyne-memory fastembed
"$HERMES_PY" -m mnemosyne.install
hermes gateway restart

# Verify
hermes memory status    # Should show "Provider: mnemosyne"
hermes mnemosyne stats  # Should show 0 working, 0 episodic
```

## Full Installation

### Prerequisites

- Hermes Agent installed and running
- Python 3.10+ (included with Hermes venv)
- ~500MB disk space (for embedding models + database)
- For local LLM consolidation: ~1.5GB for TinyLlama GGUF model

### Step-by-Step

```bash
HERMES_PY="$HOME/.hermes/hermes-agent/venv/bin/python"

# 1. Core package
"$HERMES_PY" -m pip install --no-cache-dir mnemosyne-memory

# 2. Semantic search (required for vector recall)
"$HERMES_PY" -m pip install --no-cache-dir fastembed

# 3. Local LLM consolidation (optional but recommended)
"$HERMES_PY" -m pip install --no-cache-dir ctransformers huggingface-hub

# 4. Register plugin with Hermes
"$HERMES_PY" -m mnemosyne.install

# 5. Restart gateway
hermes gateway restart
```

**One-liner alternative:**
```bash
curl -sSL https://raw.githubusercontent.com/AxDSan/mnemosyne/main/deploy_hermes_provider.sh | bash
```

## Configuration Reference

### `~/.hermes/config.yaml`

```yaml
memory:
  provider: mnemosyne
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 2200
  user_char_limit: 1375
  flush_min_turns: 6

  mnemosyne:
    auto_sleep: true              # Auto-consolidate at session boundaries
    sleep_threshold: 50           # Min working entries before auto-sleep
    vector_type: int8             # float32 | int8 | bit
    embedding_model: BAAI/bge-small-en-v1.5
    vec_weight: 0.5
    fts_weight: 0.3
    importance_weight: 0.2
    llm_enabled: true
    llm_repo: TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF
    host_llm_enabled: false
```

### Environment Variables (`~/.hermes/.env`)

```bash
MNEMOSYNE_DATA_DIR=~/.hermes/mnemosyne/data
MNEMOSYNE_EMBEDDING_MODEL=BAAI/bge-base-en-v1.5   # 768d for better recall
MNEMOSYNE_RECENCY_HALFLIFE=72                        # 72h (security context)
MNEMOSYNE_HOST_LLM_ENABLED=true                      # Use host model
```

### ⚠️ Profile Override Pitfall

`python -m mnemosyne.install` only updates global config. Profile configs override silently.

```bash
# Check EVERY profile
for f in ~/.hermes/profiles/*/config.yaml; do
    echo "=== $f ==="
    grep "provider:" "$f" || echo "(no provider key)"
done

# Fix: update each profile
hermes profile use <name>
hermes config set memory.provider mnemosyne
```

## Pentest-Optimized Settings

```yaml
mnemosyne:
  sleep_threshold: 30    # Faster consolidation (default 50)
```

```bash
# ~/.hermes/.env
MNEMOSYNE_EMBEDDING_MODEL=BAAI/bge-base-en-v1.5   # 768d accuracy
MNEMOSYNE_RECENCY_HALFLIFE=72                        # Security context decays faster
MNEMOSYNE_HOST_LLM_ENABLED=true                      # Better consolidation quality
```

### Embedding Model Comparison

| Model | Dimensions | Size | Accuracy | Speed |
|-------|-----------|------|----------|-------|
| `bge-small-en-v1.5` | 384 | 133MB | Good | Fast |
| `bge-base-en-v1.5` | 768 | 430MB | Better | Medium |
| `bge-large-en-v1.5` | 1024 | 1.2GB | Best | Slower |

**Recommendation:** `bge-base-en-v1.5` for security work.

## Tool Reference

### Core Operations

| Tool | Purpose | Key Params |
|------|---------|-----------|
| `mnemosyne_remember` | Store memory | content, importance, veracity, extract_entities |
| `mnemosyne_recall` | Search memories | query, limit, temporal_weight |
| `mnemosyne_sleep` | Consolidate working → episodic | (none) |
| `mnemosyne_stats` | Memory counts | (none) |
| `mnemosyne_invalidate` | Mark outdated | memory_id, replacement_id |
| `mnemosyne_update` | Update existing | memory_id, content, importance |
| `mnemosyne_forget` | Delete memory | memory_id |
| `mnemosyne_export` | Export to JSON | output_path |
| `mnemosyne_import` | Import from JSON | input_path, force |
| `mnemosyne_diagnose` | Diagnostic check | (none) |

### Knowledge Graph

| Tool | Purpose |
|------|---------|
| `mnemosyne_triple_add` | Add subject-predicate-object triple |
| `mnemosyne_triple_query` | Query triples by pattern |
| `mnemosyne_graph_query` | Multi-hop BFS traversal |
| `mnemosyne_graph_link` | Link memories with relationship |

### Scratchpad

| Tool | Purpose |
|------|---------|
| `mnemosyne_scratchpad_write` | Write temporary note |
| `mnemosyne_scratchpad_read` | Read scratchpad |
| `mnemosyne_scratchpad_clear` | Clear scratchpad |

## File Locations

| Path | Purpose |
|------|---------|
| `~/.hermes/plugins/mnemosyne/` | Plugin symlink |
| `~/.hermes/mnemosyne/data/mnemosyne.db` | Main database |
| `~/.hermes/mnemosyne/data/triples.db` | Knowledge graph |
| `~/.hermes/mnemosyne/data/banks/` | Named memory banks |

## Common Issues

| Problem | Fix |
|---------|-----|
| `ModuleNotFoundError` | Install into Hermes venv: `$HERMES_PY -m pip install mnemosyne-memory` |
| `Provider: built-in` | Profile override — update profile configs |
| `Provider: ''` (empty) | Duplicate `provider:` keys — remove duplicates in YAML |
| First recall takes 10s | fastembed downloads models on first use — subsequent calls instant |
| Memory not persisting | Enable `auto_sleep: true`, run `hermes mnemosyne sleep` manually |
| GPU not used | fastembed uses CPU by default — needs `onnxruntime-gpu` |

## Backup & Migration

```bash
hermes mnemosyne export --output /tmp/mnemosyne_backup.json
hermes mnemosyne import --input /tmp/mnemosyne_backup.json
hermes mnemosyne import --input /tmp/mnemosyne_backup.json --force  # Overwrite
```

## Uninstallation

```bash
"$HERMES_PY" -m mnemosyne.install --uninstall
hermes memory setup    # Switch back to built-in
rm -rf ~/.hermes/mnemosyne/  # Full cleanup
"$HERMES_PY" -m pip uninstall mnemosyne-memory
hermes gateway restart
```

## References

- [Mnemosyne GitHub](https://github.com/AxDSan/mnemosyne)
- [r/hermesagent Discussion](https://www.reddit.com/r/hermesagent/s/ANCT3yxmk7)
- [Hermes Memory Docs](https://hermes-agent.nousresearch.com/docs/memory)
- [BGE Embedding Models](https://huggingface.co/BAAI/bge-base-en-v1.5)

| [← Back to 03-Memory-Setup](../03-Memory-Setup/README.md) | 🏠 [Home](../README.md) |
|---|---