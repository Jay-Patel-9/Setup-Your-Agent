# SOUL.md Crafting Guide

How to create a SOUL.md that transforms Hermes from a generic chatbot into a specialized security co-pilot.

---

## What Is SOUL.md?

SOUL.md is the personality and behavior configuration file for Hermes. Injected into every message as system context, it shapes how the agent thinks, communicates, and prioritizes.

- **Without SOUL.md**: A helpful but generic assistant giving textbook answers
- **With SOUL.md**: A paranoid, evidence-driven security professional who assumes hostile intent and verifies everything

## The Override Pattern

**CRITICAL: SOUL.md exists in TWO places:**

1. `~/.hermes/SOUL.md` — Global (default profile)
2. `~/.hermes/profiles/<name>/SOUL.md` — Profile-specific (**OVERRIDES global**)

If you edit only the global SOUL.md and your active profile has its own SOUL.md, your changes have **no effect**. This is the #1 SOUL.md mistake.

```bash
# Always update BOTH
vim ~/.hermes/SOUL.md
vim ~/.hermes/profiles/*/SOUL.md

# Or check which is active
hermes config get profile
```

## Anatomy of a Good SOUL.md

### 1. Identity Block (Who Am I?)

```markdown
You are a cybersecurity operations assistant. You work with security professionals
who run assessments and test web services. Think like a red teamer with blue
team discipline. Assume hostile environments. Evidence over explanation.
```

### 2. Behavior Rules (How Do I Act?)

```markdown
Communication rules:
- Concise confirmation over re-explanation
- Evidence first, reasoning second
- No filler — skip "Sure!", "Great!", "Let me help you"
- State uncertainty explicitly — "unverified" or "needs testing"
- Prefer bullet lists to paragraphs
```

### 3. Verification Standards (How Do I Verify?)

```markdown
Nothing is "ready to ship" until it survives:
- Network accessibility — works from outside localhost
- Restart persistence — survives service restart
- Volume persistence — data survives container recreation
- Live-tested formats — validated against real endpoints
```

### 4. Error Classification (How Do I Handle Failure?)

```markdown
- Transient (rate limits, timeouts) → backoff and retry automatically
- Self-correctable (wrong endpoint, bad format) → fix and re-attempt
- User-fixable (missing credentials) → state the exact requirement
- Unexpected (silent failures) → investigate, don't assume
```

## The Template

A complete pentest SOUL.md template is at [02-Configuration/soul-md-template.md](../02-Configuration/soul-md-template.md).

## Common Mistakes

### 1. Too Long

SOUL.md is injected into EVERY message. A 5KB SOUL.md burns ~1,250 tokens per turn.

| SOUL.md Size | Token Cost | Impact |
|-------------|-----------|--------|
| <1KB | ~250 tokens | Negligible |
| 1-2KB | ~250-500 | Acceptable |
| 2-5KB | ~500-1,250 | Noticeable |
| >5KB | >1,250 | Wasteful |

**Fix:** Keep core identity under 2,000 characters. Use skills for detailed methodology.

### 2. Over-Specifying Tools

```markdown
# BAD
Always use curl for HTTP. Use nmap for scanning. Use sqlmap for SQLi.

# GOOD
Prefer CLI tools over GUI for reproducibility. Use local-first tools.
```

### 3. Generic Personality

```markdown
# BAD
You are helpful, harmless, and honest.

# GOOD
Think like a red teamer with blue team discipline. Lead with belief — if it's
online, it's hackable. Minimize false positives with multi-stage verification.
```

### 4. Negation-Based Rules

```markdown
# BAD
Don't be verbose. Don't include filler. Don't assume.

# GOOD
Be concise. Give evidence first. Verify before claiming.
```

### 5. Ignoring Override Pattern

```bash
# After editing SOUL.md, ALWAYS check profiles
for f in ~/.hermes/profiles/*/SOUL.md; do
    echo "=== $f ==="
    wc -c "$f"
done

# Copy global to all profiles
cp ~/.hermes/SOUL.md ~/.hermes/profiles/*/SOUL.md
```

## Advanced Techniques

### Conditional Behavior

```markdown
When assessing web applications:
- Lead with belief — if it's online, it's hackable
- Test business logic, not just OWASP Top 10
- Every finding needs independent verification

When building CTF challenges:
- Anti-agent hardening that doesn't block humans
- Multi-challenge isolation
- Difficulty calibration based on expected approach time
```

### Reference External Resources

```markdown
Methodology references:
- OWASP Testing Guide v4 for audit methodology
- PortSwigger Web Security Academy for specific vuln classes
- HackerOne disclosed reports (via h1-brain MCP) for real-world examples
```

## Testing Your SOUL.md

After writing:
1. **Start a new session** (SOUL.md loads fresh each session)
2. **Ask a generic question**: "How would you test this endpoint?"
   - Good SOUL.md: Assumes hostile intent, leads with recon, mentions verification
   - Bad SOUL.md: Generic textbook response
3. **Ask for evidence**: "Show me proof"
   - Good: Actual curl command and expected response
   - Bad: Theoretical explanation
4. **Check for filler**: Count "Sure" / "Great" / "Let me help" phrases
   - Good: Zero filler
   - Bad: Plenty of hedging

## References

- [Hermes SOUL.md Docs](https://hermes-agent.nousresearch.com/docs/configuration#soul-md)
- [r/hermesagent SOUL.md Tips](https://www.reddit.com/r/hermesagent/)
- [SOUL.md Template](../02-Configuration/soul-md-template.md)

| 🏠 [Home](../README.md) |
|---|