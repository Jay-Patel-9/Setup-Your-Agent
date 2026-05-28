# 08 - Where Agents Fail (And What To Do About It)

The 60/40 rule: agents reliably handle 60% of security work. The other 40% requires human intuition, creative exploitation, and novel pattern recognition. Here's the breakdown of failure modes and mitigations.

---

## The 60/40 Rule Explained

| 60% - Agents Excel | 40% - Humans Required |
|---|---|
| Systematic enumeration | Creative exploitation chains |
| Pattern matching against known vuln classes | Business logic intuition |
| Documentation and report generation | Recognizing "this feels wrong" |
| Repetitive checklist execution | Novel vulnerability classes |
| Data correlation across findings | Discriminating false positives from decoys |
| Reconnaissance data collection | Exploit development |

## Agent Failure Modes

### 1. Confident Hallucination

**What happens:** Agent presents fabricated CVE IDs, non-existent endpoints, or plausible-sounding exploit chains that don't actually work.

**Mitigation:**
- Always verify agent claims independently
- Use `mnemosyne_remember` to track verified findings separately from agent claims
- Cross-reference CVE IDs against NVD before reporting
- Demand evidence: "Show me the curl command and response"

### 2. Context Window Exhaustion

**What happens:** Long sessions cause context overflow. The agent forgets earlier findings, repeats work, or contradicts itself.

**Mitigation:**
- Use Mnemosyne `sleep` to consolidate working memory regularly
- Keep sessions focused on one phase at a time
- Use `mnemosyne_remember` with high importance for critical findings
- Break complex assessments into multiple sessions

### 3. Tool Call Loops (The Ralph Loop)

**What happens:** Agent enters an infinite loop of tool calls - trying the same failed approach repeatedly without making progress.

**Mitigation:**
- Set explicit iteration budgets: "Try each auth bypass technique exactly once"
- Use `mnemosyne_scratchpad_write` to track what's been tried
- Define failure conditions: "If 3 attempts return 403, move to next vector"
- The `agent-harness-engineering` skill encodes these patterns

### 4. Over-Reliance on Known Patterns

**What happens:** Agent tests only for vulnerabilities in training data. Misses novel attack vectors entirely.

**Example:** Testing for XSS, SQLi, CSRF, IDOR - standard OWASP Top 10 - but missing a business logic flaw where negative quantities cause money to be added.

**Mitigation:**
- Supplement agent work with human-guided business logic testing
- Use custom skills to encode novel vulnerability patterns
- Maintain a "weird findings" list in Mnemosyne

### 5. Rate Limit Blindness

**What happens:** Agent hits API rate limits but doesn't understand it's being throttled. Retries the same request, getting the same rate-limit response.

**Mitigation:**
- Configure fallback providers to handle 429 responses
- Teach: "If you receive HTTP 429, wait 30 seconds before retrying"
- Note rate limit thresholds in Mnemosyne

### 6. False Positive Confirmation

**What happens:** Agent "confirms" a vulnerability that doesn't actually exist. Often due to:
- Misinterpreting error messages
- Treating reflected input as XSS without proper context
- Confusing debug output with sensitive data exposure

**Mitigation:**
- Require proof-of-concept evidence for every finding
- Verify findings with a second tool or approach
- Use `mnemosyne_invalidate` to mark false positives

### 7. Scope Creep and Decoy Chasing

**What happens:** Agent follows decoys down rabbit holes, wasting assessment budget on non-target assets.

**Mitigation:**
- Define scope explicitly in SOUL.md: "Only test *.target.com"
- Track scope in Mnemosyne: `mnemosyne_remember content="Assessment scope: *.target.com" importance=1.0`
- Set explicit time budgets per phase

### 8. Loss of Session State

**What happens:** Between sessions, the agent loses all context.

**Mitigation:**
- This is exactly what Mnemosyne solves - see [03-Memory-Setup](../03-Memory-Setup/README.md)
- Store critical findings with `importance=0.9-1.0`
- Use `mnemosyne_sleep` at end of each session
- Start new sessions with `mnemosyne_recall` to recover context

### 9. Model-Specific Blind Spots

**What happens:** Different models have different weaknesses. One hallucinates CVEs; another misses obvious auth bypass patterns.

**Mitigation:**
- Use `fallback_providers` so a single model's weakness doesn't block you
- Choose models by task: fast for recon, capable for exploitation
- Test your model's security capabilities before relying on it

### 10. The "I Don't Know What I Don't Know" Problem

**What happens:** Agent can't recognize when it's out of its depth.

**This is the fundamental limit of current AI agents.**

**Mitigation:**
- Always pair agent work with human review
- Use skills to encode "when to stop and ask a human" decision points
- Maintain healthy skepticism - the agent is a force multiplier, not a replacement

## Practical Mitigation Checklist

For every finding an agent produces:

- [ ] **Is it real?** - Verify with a second method
- [ ] **Is it in scope?** - Check against defined scope
- [ ] **Is it novel?** - Or is it a known pattern the agent found by the book?
- [ ] **Is it exploitable?** - Theoretical vs. practical impact
- [ ] **Can I reproduce it?** - Evidence or it didn't happen

For every assessment phase:

- [ ] **Set iteration budgets** - Max N attempts per technique
- [ ] **Define success/failure criteria** - What "done" looks like
- [ ] **Use Mnemosyne** - Persist context across sessions
- [ ] **Verify agent claims independently** - Trust but verify
- [ ] **Break work into focused sessions** - Avoid context overflow

## The Honest Assessment

**Agents are excellent for:**
- Systematic, thorough testing of known vulnerability classes
- Reconnaissance and data collection
- Documentation and report generation
- Repeating the same methodology across multiple targets

**Agents are terrible at:**
- Creative, multi-step exploitation chains
- Recognizing business logic flaws
- Discriminating real vulnerabilities from decoys
- Knowing when they're out of their depth

**The sweet spot:** Use agents for the 60% grunt work so you can focus your human expertise on the 40% that requires creativity, intuition, and experience.

---

| [← Agent Comparison](../07-Agent-Comparison/README.md) | 🏠 [Home](../README.md) |
|---|---|