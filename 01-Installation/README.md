# 01 — Installation

From zero to a running Hermes Agent in under 10 minutes.

---

## Prerequisites

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| Python | 3.10+ | 3.11+ |
| RAM | 4GB | 8GB+ |
| Disk | 2GB | 5GB+ (with models) |
| OS | Linux, macOS, WSL2 | Ubuntu 22.04+, macOS 13+ |
| Network | Outbound HTTPS | Outbound HTTPS + WebSocket |

**WSL2 users:** Install Hermes inside WSL2, not on the Windows host. The Linux filesystem (`~/.hermes/`) is significantly faster than Windows mounts (`/mnt/c/`).

## Method 1: Official Installer (Recommended)

```bash
# One-line install
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash

# Verify
hermes --version
hermes doctor
```

The installer handles:
- Python venv creation
- Dependency installation
- Gateway setup
- Default configuration

## Method 2: pip Install

```bash
# Create a dedicated venv
python3 -m venv ~/.hermes/hermes-agent/venv
source ~/.hermes/hermes-agent/venv/bin/activate

# Install
pip install hermes-agent

# Verify
hermes --version
```

## Method 3: Docker

```bash
# Pull and run
docker pull nousresearch/hermes-agent:latest
docker run -d \
  --name hermes \
  -v ~/.hermes:/root/.hermes \
  -p 8080:8080 \
  nousresearch/hermes-agent:latest

# Verify
docker exec hermes hermes doctor
```

**Note:** Docker installs require additional configuration for Telegram/webhook connectivity. See the [Hermes docs](https://hermes-agent.nousresearch.com/docs/docker) for details.

## Method 4: From Source

```bash
# Clone
git clone https://github.com/nousresearch/hermes-agent.git ~/.hermes/hermes-agent
cd ~/.hermes/hermes-agent

# Setup venv
python3 -m venv venv
source venv/bin/activate

# Install
pip install -e .

# Verify
hermes --version
```

## Setup Wizard

After installation, run the setup wizard:

```bash
hermes setup
```

The wizard configures:
1. **Provider** — Which LLM API to use (OpenRouter, OpenAI, Anthropic, local)
2. **Model** — Which model within the provider
3. **API Key** — Authentication credentials
4. **Gateway** — Start the message gateway
5. **Platform** — Connect to Telegram, Discord, or web UI

### Provider Configuration

```bash
# OpenRouter (recommended for flexibility — access to 100+ models)
hermes config set provider openrouter
hermes config set model glm-5.1

# Set API key
hermes config set api_key [REDACTED]
# Or use environment variable
export OPENROUTER_API_KEY=[REDACTED]
```

### Telegram Configuration

```bash
# Interactive setup
hermes setup telegram

# Or manual configuration
hermes config set telegram.bot_token [REDACTED]
hermes config set telegram.allowed_users '["your_user_id"]'
hermes gateway restart
```

### Discord Configuration

```bash
hermes setup discord
# Follow the OAuth2 flow — opens browser for bot creation
```

## Dependency Checks

After installation, verify everything is working:

```bash
# Full system check
hermes doctor

# Expected output:
# ✅ Python 3.11+
# ✅ Hermes v2.x
# ✅ Provider: openrouter
# ✅ Model: glm-5.1
# ✅ Gateway: running
# ✅ Memory: built-in (or mnemosyne)
# ✅ Skills: X loaded
```

### Common Dependency Issues

| Issue | Fix |
|-------|-----|
| `python3: command not found` | `sudo apt install python3 python3-venv` (Ubuntu) or `brew install python` (macOS) |
| `pip: command not found` | `python3 -m ensurepip --upgrade` |
| SSL certificate errors | `pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org hermes-agent` |
| Port 8080 in use | `hermes config set gateway.port 8081` or `pkill -f "hermes.*gateway"` |
| WSL2 DNS failures | `echo "nameserver 8.8.8.8" \| sudo tee /etc/resolv.conf` |

## First Test

```bash
# Start a chat session
hermes chat "Hello, what can you help me with?"

# If connected to Telegram, send a message to your bot
# The bot should respond within a few seconds
```

## Next Steps

- **[02-Configuration](../02-Configuration/README.md)** — Personalize with SOUL.md, security hardening
- **[03-Memory-Setup](../03-Memory-Setup/README.md)** — Install Mnemosyne for persistent memory
- **[04-Skills](../04-Skills/README.md)** — Add pentest-specific skills

---

| 🏠 [Home](../README.md) | [Next: Configuration →](../02-Configuration/README.md) |
|---|---|