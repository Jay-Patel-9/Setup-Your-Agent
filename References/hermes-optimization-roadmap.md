# Hermes Optimization Roadmap

Community-vetted optimizations for WSL + GLM-5.1 + Telegram setups. Prioritized by impact.

---

## Priority 1: Compression Model (High Impact, Low Effort)

**Problem:** GLM-5.1's 131K token context causes frequent compression loops that burn API quota.

**Fix:**
```yaml
# ~/.hermes/config.yaml
compression:
  enabled: true
  summary_model: openrouter/deepseek/deepseek-v4-flash
```

**Why:** Primary model stays focused on tasks. Compression runs on a free/cheap model. Cost savings compound.

---

## Priority 2: Fallback Providers (High Impact, Medium Effort)

**Problem:** Single-model dependency. Rate limits (429), overloads (529), outages (503) stop all work.

**Fix:**
```yaml
fallback_providers:
  - provider: openrouter
    model: deepseek/deepseek-v4-flash
  - provider: openrouter
    model: google/gemini-2.5-flash-preview
  - provider: nous
    model: nous/hermes-3-llama-3.1-405b
```

**Key format:** List of dicts, NOT a single dict. Each entry needs `provider` and `model` keys.

---

## Priority 3: SOUL.md Personalization (Medium Impact, Low Effort)

**Problem:** Stock SOUL.md is generic. No security specialization.

**Fix:** Create a pentest-specific SOUL.md. Template at [02-Configuration/soul-md-template.md](../02-Configuration/soul-md-template.md).

**CRITICAL:** Update BOTH `~/.hermes/SOUL.md` AND all `~/.hermes/profiles/*/SOUL.md`. Profiles override global.

---

## Priority 4: Mnemosyne Tuning (Medium Impact, Low Effort)

**Problem:** Default settings optimized for general chat. Security context decays faster.

**Fix:**
```bash
# ~/.hermes/.env
MNEMOSYNE_EMBEDDING_MODEL=BAAI/bge-base-en-v1.5   # 768d vs 384d
MNEMOSYNE_RECENCY_HALFLIFE=72                        # 72h vs 168h
MNEMOSYNE_HOST_LLM_ENABLED=true                      # Better consolidation
```

```yaml
# ~/.hermes/config.yaml
memory:
  mnemosyne:
    sleep_threshold: 30    # Faster consolidation (default 50)
```

---

## Priority 5: Skill Curation (Medium Impact, Medium Effort)

**Problem:** Too many skills waste tokens. Too few means no methodology.

**Essential pentest skills [example]:**
```bash
hermes skills install ctf-web-audit
hermes skills install web-app-auth-bypass
hermes skills install agent-harness-engineering
hermes skills install web-response-manipulation
hermes skills install offensive-osint
hermes skills install osint-methodology
```

**Anti-patterns:**
- Loading every skill (wastes context tokens)
- Skills that overlap significantly (choose one per methodology)
- Skills without clear trigger conditions

---

## Priority 6: Model Selection Strategy (Medium Impact, Zero Effort)

| Task Phase | Model Type | Example |
|-----------|-----------|---------|
| Recon/enumeration | Fast, cheap | DeepSeek-v4-Flash |
| Exploitation/analysis | Capable | GLM-5.1, Claude |
| Report writing | Balanced | Gemini Flash |
| Compression | Dedicated cheap | DeepSeek-v4-Flash |

No config needed — just be aware when choosing primary model.

---

## Verification Checklist

```bash
hermes config get compression.summary_model   # 1. Compression model
hermes config get fallback_providers          # 2. Fallback providers
diff ~/.hermes/SOUL.md ~/.hermes/profiles/*/SOUL.md  # 3. SOUL.md consistency
hermes mnemosyne diagnose                     # 4. Mnemosyne health
hermes skills list | grep -i "security\|ctf"  # 5. Security skills
hermes doctor                                  # 6. Full system check
```

## References

- [r/hermesagent Optimization Thread](https://www.reddit.com/r/hermesagent/)
- [Hermes Configuration Docs](https://hermes-agent.nousresearch.com/docs/configuration)
- [Mnemosyne Setup Reference](mnemosyne-setup.md)
- [SOUL.md Crafting Guide](soul-md-crafting-guide.md)

| 🏠 [Home](../README.md) |
|---|