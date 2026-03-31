# Agent Directory Setup

Read this file and follow the instructions to ensure the agents directory is ready.

---

## Target Path

| Platform | Path |
|---|---|
| Windows | `C:\Users\<username>\.claude\agents\` |
| macOS/Linux | `~/.claude/agents/` |

---

## Check and Create

Run:

```bash
# Check if it exists
ls ~/.claude/agents/ 2>/dev/null && echo "EXISTS" || echo "MISSING"

# Create if missing
mkdir -p ~/.claude/agents/
```

On Windows (PowerShell):
```powershell
if (!(Test-Path "$env:USERPROFILE\.claude\agents")) {
    New-Item -ItemType Directory -Path "$env:USERPROFILE\.claude\agents" -Force
    Write-Host "Created agents directory"
} else {
    Write-Host "Agents directory already exists"
}
```

---

## What Goes Here

Only `.md` agent files. Each file is one agent. Claude Code scans this directory at startup and loads every `.md` it finds.

**Allowed:**
- `AgentName.md` — agent definition files
- `AgentName_Helper.md` — variants and helpers
- Subdirectories for organization (e.g., `specialists/`, `irc/`)

**Do NOT put here:**
- Code files (`.py`, `.js`, etc.)
- Log files
- `.env` files or secrets
- Large data files

---

## Naming Convention

StubsArmy uses **nickname-based filenames**:

```
~/.claude/agents/
├── Sparky.md            ← main orchestrator
├── Sparky_Helper.md     ← lightweight aide
├── Ledger_Liz.md        ← specialist
└── Byte_Watcher.md      ← specialist
```

The `name:` field inside each file is the routing slug (lowercase-hyphenated). The filename is the nickname.

---

## Verify After Creating Agents

After writing agent files, confirm Claude Code can see them:

```bash
ls ~/.claude/agents/
```

Claude Code will pick them up automatically on the next session start — no registration or reload needed.
