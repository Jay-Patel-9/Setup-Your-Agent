# 07 - Agent Comparison

An honest assessment of current AI coding agents for security work. No fanboyism - just capabilities, limits, and when each tool fits.

---

## The Landscape (2025-2026)

| Agent | Core Model | Transport | Open Source | Security Focus |
|-------|-----------|-----------|-------------|----------------|
| **Hermes** | Any (pluggable) | Telegram/Discord/web | ✅ Core engine OSS | Community-driven |
| **Claude Code** | Claude Sonnet/Opus | CLI/Terminal | ❌ Proprietary | General dev |
| **Codex CLI** | OpenAI models | CLI/Terminal | ✅ Apache 2.0 | General dev |
| **OpenCode** | Multiple (pluggable) | TUI/CLI | ✅ BSD-3 | General dev |
| **OpenClaw** | Multiple | CLI/Terminal | ✅ MIT | General dev |

## Agent-by-Agent Breakdown

### Hermes

**Strengths:**
- 🔌 Pluggable model - use any LLM provider (OpenRouter, local, OpenAI, Anthropic, custom)
- 💾 Persistent memory (Mnemosyne) - sessions carry forward
- 🛠 Skill system - encode pentest methodology, not just prompt engineering
- 🔗 MCP servers - extend with external data sources (HackerOne, Shodan, etc.)
- 📱 Multi-platform - Telegram, Discord, web UI, CLI
- 🏠 Fully local-first - data never leaves your machine unless you send it
- 🔄 Fallback providers - no single-point-of-failure on model access
- 📝 SOUL.md - deep personality/specialization injection

**Weaknesses:**
- Setup complexity - more moving parts than simpler agents
- Community skills vary in quality
- No built-in IDE integration (no VSCode extension)
- Gateway process management requires basic sysadmin skills
- Token costs can add up with verbose skills loaded

**Best for:** Security professionals who want a persistent, customizable assistant that learns their workflow.

### Claude Code

**Strengths:**
- 🔒 Excellent code understanding - best-in-class for code review
- 🏗 Strong refactoring - can handle large-scale code changes
- 📋 Good context management - handles large codebases well
- 🎯 Precise edits - minimal hallucination in code generation

**Weaknesses:**
- 🔒 Locked to Anthropic models only
- 💰 Expensive - especially Opus for complex tasks
- 🚫 No persistent memory across sessions
- 🚫 No skill system (prompts only)
- 🚫 No Telegram/mobile interface
- 🚫 No MCP server integration

**Best for:** One-off code tasks where you want the best single-shot code quality and don't need persistence.

### Codex CLI

**Strengths:**
- 🆓 Open source (Apache 2.0)
- 🏠 Can use local models
- 🔄 Sandboxed execution - safe for running untrusted code
- 📊 Good for batch/automated tasks

**Weaknesses:**
- 🤖 Limited to OpenAI models (official builds)
- 🚫 No persistent memory
- 🚫 No skill system
- 🚫 No mobile/chat interface
- 🚫 No MCP servers
- 📉 Weaker code understanding than Claude Code

**Best for:** Automated code generation tasks in CI/CD pipelines.

### OpenCode

**Strengths:**
- 🆓 Open source (BSD-3)
- 🔌 Multi-provider support
- 🖥 Nice TUI interface
- 🏠 Can run fully local

**Weaknesses:**
- 🚫 No persistent memory
- 🚫 No skill system
- 🚫 No MCP server support
- 🚫 No mobile/chat interface
- 📦 Newer project - less battle-tested

**Best for:** Developers who want a local, open-source coding assistant with a nice terminal UI.

## Comparison Matrix

| Feature | Hermes | Claude Code | Codex | OpenCode |
|---------|--------|-------------|-------|----------|
| Persistent memory | ✅ Mnemosyne | ❌ | ❌ | ❌ |
| Skill system | ✅ | ❌ | ❌ | ❌ |
| MCP servers | ✅ | ❌ | ❌ | ❌ |
| Model choice | ✅ Any | ❌ Anthropic | ❌ OpenAI | ✅ Multi |
| Mobile interface | ✅ Telegram | ❌ | ❌ | ❌ |
| Open source | ✅ | ❌ | ✅ | ✅ |
| Local-first | ✅ | ❌ | ✅ | ✅ |
| Fallback models | ✅ | ❌ | ❌ | ✅ |
| Skill sharing hub | ✅ | ❌ | ❌ | ❌ |
| SOUL.md/personality | ✅ | ❌ | ❌ | ❌ |

## Why Hermes for Security Work

1. **Memory persistence** - Multi-day assessments carry context forward. Findings from Day 1 feed exploitation on Day 3.
2. **Skill-encoded methodology** - Don't rely on the model remembering how to test auth bypass. Encode it once, apply it every time.
3. **MCP data sources** - Query HackerOne reports, Shodan data, CVE databases directly in context.
4. **SOUL.md specialization** - "Red teamer with blue team discipline. Assume hostile environments."
5. **Local-first** - No sensitive target data sent to third-party services unless explicitly configured.
6. **Fallback resilience** - Rate limits don't stop your workflow.

## When NOT to Use Hermes

- You need the absolute best single-shot code generation (Claude Code with Opus)
- You're doing one-off tasks where persistence adds no value
- You don't want to manage a gateway process
- Your workflow is entirely IDE-based with deep IDE integration needs
- Large-scale live knowledge requirements, instead use RAG pipeline with vector DB

---

| [← Troubleshooting](../06-Troubleshooting/README.md) | 🏠 [Home](../README.md) | [Next: Where Agents Fail →](../08-Where-Agents-Fail/README.md) |
|---|---|---|