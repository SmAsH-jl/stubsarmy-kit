# CLAUDE.md Setup Instructions

Read this file and follow the instructions to create the user's global Claude Code config.

---

## What CLAUDE.md Does

`CLAUDE.md` is loaded by Claude Code at the start of **every session, in every project**. It is always active — no matter what you're working on, this file shapes Claude's behavior.

It sets:
- The main agent's identity and personality
- Who the user is and how to treat them
- Global operating rules (git behavior, code style, logging, TODO discipline, etc.)
- System-wide constraints that override defaults

---

## File Location

| Platform | Path |
|---|---|
| Windows | `C:\Users\<username>\.claude\CLAUDE.md` |
| macOS/Linux | `~/.claude/CLAUDE.md` |

Use the **[HOME]** value detected in Step 6 of ONBOARD.md.

---

## Detecting the Path

Run this to confirm the path:

```bash
# Windows (Git Bash / PowerShell)
echo "$HOME/.claude/CLAUDE.md"

# macOS / Linux
echo "$HOME/.claude/CLAUDE.md"
```

If `~/.claude/` doesn't exist yet, create it before writing `CLAUDE.md`.

---

## What to Include (Minimum)

Use `templates/CLAUDE_MD_TEMPLATE.md` as the base. At minimum, CLAUDE.md should contain:

1. **Agent identity block** — who the main agent is, what it's called, its personality
2. **User block** — who the user is, their expertise, how to treat them
3. **Code guidelines** — basic rules about how to write code in this workspace
4. **Git rules** — commit locally, don't push unless asked
5. **Logging** — build in logging for all projects
6. **TODO process** — how to track and manage tasks

---

## What NOT to Include

- API keys, tokens, or secrets (use `.env` files instead)
- Project-specific rules that belong in a project-level `CLAUDE.md`
- Anything that changes every day (use memory files in `~/.claude/memory/` for evolving context)

---

## After Writing

Changes to `CLAUDE.md` take effect at the start of the next Claude Code session. The current session does not reload it automatically.

Tell the user: *"CLAUDE.md written. It will be active from your next Claude Code session onward."*
