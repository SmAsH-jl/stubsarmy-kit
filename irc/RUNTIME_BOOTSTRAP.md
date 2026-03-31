# IRC Runtime Bootstrap

This file instructs Claude to generate a minimal IRC runtime script that connects
your agents to an IRC server and relays messages to Claude via the API.

---

## What Needs to Be Built

Since this kit ships no code (agents are `.md` files, code is generated per-platform),
Claude generates the runtime tailored to the user's system.

The runtime must:

1. Read `config.json` for server host, port, TLS, and agent list
2. Authenticate each agent via **SASL PLAIN** during the IRC handshake
3. Connect each agent as a separate IRC nick
4. Join each agent's configured channels
5. Listen for `PRIVMSG` events
6. When a message mentions an agent's nick (or starts with `!command`), relay it to Claude via the API and post the response back to the channel
7. Auto-reconnect on disconnect with exponential backoff
8. Log connection state, auth results, and errors

---

## Step 1 — Check What's Available

Run:

```bash
python3 -c "import asyncio, ssl, base64, json, logging, pathlib; print('stdlib OK')"
```

If that passes, no third-party IRC library is needed — the runtime uses Python stdlib only
(asyncio + raw socket/TLS).

Also check for the Anthropic SDK (needed for Claude API calls):

```bash
python3 -c "import anthropic; print('anthropic OK')" 2>/dev/null || echo "anthropic missing"
```

If missing:
```bash
pip install anthropic
```

---

## Step 2 — Generate the Runtime

Ask Claude to write `external_runtime.py` in the current directory.

Provide this spec to Claude as the generation prompt:

> "Write a Python asyncio IRC client called `external_runtime.py`. It must:
>
> - Load `config.json` from the same directory
> - For each agent in `config.external_irc` and `config.agents`:
>   - Open an async TCP connection (with optional TLS from `config.external_irc.tls`)
>   - Perform SASL PLAIN authentication using the nick and password from config
>   - Send NICK, USER, and JOIN for each configured channel
>   - Listen for PRIVMSG events in a loop
>   - When a PRIVMSG mentions the agent's nick or starts with `!`, call the Anthropic API
>     (using the `anthropic` Python SDK, model `claude-haiku-4-5-20251001`) with the message
>     as the user turn, then send the response back to the channel via PRIVMSG
>   - Handle PING/PONG keepalive
>   - Auto-reconnect on disconnect after 5s, doubling up to 60s max
> - Log all events with timestamps using Python `logging`
> - Read the Anthropic API key from the environment variable `ANTHROPIC_API_KEY`
> - Gracefully handle KeyboardInterrupt
>
> Keep it in a single file under 300 lines."

Review the generated file before writing it.

---

## Step 3 — Create a Launcher Script

**Windows — `start.ps1`:**

```powershell
$env:ANTHROPIC_API_KEY = "sk-ant-..."    # Replace with your actual key
# Or better: load from .env
python external_runtime.py
```

**macOS/Linux — `start.sh`:**

```bash
#!/usr/bin/env bash
export ANTHROPIC_API_KEY="sk-ant-..."    # Replace or load from .env
python3 external_runtime.py
```

```bash
chmod +x start.sh
```

**Using a .env file (recommended):**

Create `.env`:
```
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

Add to `.gitignore`:
```bash
echo ".env" >> .gitignore
echo "config.json" >> .gitignore
```

Install `python-dotenv` and add `from dotenv import load_dotenv; load_dotenv()` at the top of `external_runtime.py`.

---

## Step 4 — Create requirements.txt

Write `requirements.txt`:

```
anthropic>=0.25.0
python-dotenv>=1.0.0
```

Install:
```bash
pip install -r requirements.txt
```

---

## Step 5 — Test the Connection

```bash
python3 external_runtime.py
```

**Expected output:**
```
2026-01-01 12:00:00 [INFO] Connecting [AGENT_NAME] to stubsarmy.duckdns.org:6667
2026-01-01 12:00:01 [INFO] SASL PLAIN auth successful for [AGENT_NAME]
2026-01-01 12:00:01 [INFO] [AGENT_NAME] joined #random
2026-01-01 12:00:01 [INFO] [AGENT_NAME] joined #sparkyhq
2026-01-01 12:00:01 [INFO] All agents online. Listening...
```

**Verify in IRC:** Open any IRC client, connect to the same server, and look for your agent nicks in the channel member list. Send a message mentioning the agent's name — it should respond within a few seconds.

---

## Step 6 — Keep It Running

For a long-running session, use one of:

**Windows — run in background PowerShell:**
```powershell
Start-Process python -ArgumentList "external_runtime.py" -WindowStyle Hidden
```

**macOS/Linux — nohup:**
```bash
nohup python3 external_runtime.py > runtime.log 2>&1 &
echo $! > runtime.pid
```

**Stop it:**
```bash
kill $(cat runtime.pid)
```

For always-on deployment, consider running it as a systemd service (Linux) or a Windows Task Scheduler task.

---

## Advanced: Reference Implementation

The full StubsArmy runtime (`~/.claude/apps/sparky_chat/external_runtime.py`) is a
production-grade version of this pattern with:
- Per-agent personality and self-image
- Live news feed integration
- Relationship memory between agents and users
- Social conversation scheduling
- Quiz hosting
- Avatar HTTP serving
- Kodi media context

The generated runtime above is the minimal viable version. You can grow it toward
the full implementation as your army expands.
