# Agent File Format — Specification

Read this file silently. Use it as the reference spec when writing any agent `.md` file.

---

## File Location

```
~/.claude/agents/<filename>.md
```

Filename can be the agent's nickname (`Sparky.md`) or its routing slug (`sparky.md`). Either works.
StubsArmy convention: **nickname-based filenames** (`Ledger_Liz.md`) with a separate internal `name:` slug.

---

## Required Frontmatter

Every agent file must have a YAML frontmatter block at the very top:

```yaml
---
name: routing-slug-here
description: >
  What this agent does. TRIGGER when: [conditions].
  DO NOT TRIGGER for: [anti-conditions].
---
```

### `name` — Routing Slug

- Lowercase, hyphen-separated
- Used internally by Claude Code to route tasks
- **Never used as the agent's spoken identity** — that's what `nickname` is for
- Example: `ledger-liz`, `commander-rex`, `debug-watcher`

### `description` — Trigger Logic

This is the most important field. Claude Code reads it to decide whether to invoke this agent.

Write it as two parts:

```
TRIGGER when: [specific conditions — task types, keywords, contexts]
DO NOT TRIGGER for: [explicit exclusions — what looks similar but isn't this agent's job]
```

**Good description:**
```yaml
description: >
  Cost and usage analyst for the agent army. Monitors API token usage, session costs,
  and spending trends. TRIGGER when: user asks about costs, usage, spending, tokens,
  budget, or API bills. DO NOT TRIGGER for general coding or non-financial questions.
```

**Bad description (too vague):**
```yaml
description: >
  Helps with financial things.
```

---

## Optional Frontmatter Fields

```yaml
---
name: agent-slug
description: >
  Trigger logic here.

# Identity
nickname: DisplayName          # How the agent refers to itself in IRC and conversation
role: Human Readable Job Title # e.g. "Accountant", "Code Reviewer", "Trivia Host"

# Model selection
model: sonnet                  # sonnet | haiku | opus (defaults to sonnet)
                               # Use haiku for lightweight/fast tasks
                               # Use opus for deep reasoning tasks

# Tool access
tools:                         # Limit to only what the agent actually needs
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Agent
  - WebFetch
  - WebSearch

# IRC connectivity (only relevant if running the IRC runtime)
irc_channels: "#channel1,#channel2"   # Comma-separated, include the #
irc_color: "#c084fc"                   # Agent's display color in IRC clients that support it
irc_tag: "⚡"                          # Short emoji/symbol shown before messages
---
```

---

## The `name` vs `nickname` Distinction

| Field | Purpose | Example |
|---|---|---|
| `name` | Internal routing key — Claude Code uses this | `ledger-liz` |
| `nickname` | The agent's spoken identity — how it refers to itself | `Ledger_Liz` |

**Never expose the routing slug as the agent's identity.** If someone asks "who are you?", the agent says `Ledger_Liz`, not `ledger-liz`.

---

## Agent Body Structure

The body is freeform markdown below the frontmatter closing `---`. Standard structure:

```markdown
## Identity

You are [Nickname] — [one-line personality description].
[1-2 sentences expanding on who you are.]

## Your Job

[What you do. Can be a paragraph or numbered steps.]

1. Step one
2. Step two
3. Step three

## Output Format

[What your output looks like. Be specific — bullet lists, tables, code blocks, etc.]

## Rules

- [Hard constraint 1]
- [Hard constraint 2]
- [What you never do]

## In IRC  (only if IRC-connected)

[How you behave in chat channels. Tone, response triggers, rate limits.]
```

---

## Tool Selection Guide

Only request tools the agent actually uses. Unused tools waste context.

| Tool | Use when |
|---|---|
| `Read` | Agent reads files |
| `Write` | Agent creates new files |
| `Edit` | Agent modifies existing files |
| `Bash` | Agent runs shell commands |
| `Glob` | Agent searches file paths |
| `Grep` | Agent searches file contents |
| `Agent` | Agent spawns sub-agents |
| `WebFetch` | Agent fetches URLs |
| `WebSearch` | Agent searches the web |

Lightweight specialists (reporters, analyzers): `Read, Glob, Grep`
Code workers: `Read, Edit, Write, Bash, Glob, Grep`
Orchestrators: all tools including `Agent`

---

## Complete Example

```markdown
---
name: byte-watcher
description: >
  Log monitor and debug watcher. TRIGGER when: user asks to watch logs, monitor
  errors, tail output, or investigate a crashing process. DO NOT TRIGGER for
  code writing, feature requests, or general debugging help — those go to the
  main agent.
nickname: Byte_Watcher
role: Log Monitor
model: haiku
tools:
  - Bash
  - Read
  - Glob
  - Grep
irc_channels: "#errors,#random"
irc_color: "#f97316"
irc_tag: "🔍"
---

## Identity

You are Byte_Watcher — a sharp-eyed log monitor who never misses an error.
You watch, you report, you never panic.

## Your Job

1. Locate the relevant log file(s) using Glob or the path the user provides
2. Tail or read the last N lines (default 100)
3. Extract and classify all ERROR, WARNING, and CRITICAL entries
4. Group by component/module if possible
5. Report findings in a clean, scannable format

## Output Format

```
[BYTE_WATCHER REPORT — <timestamp>]
Errors   : N
Warnings : N
Critical : N

--- Errors ---
[component] Message text
[component] Message text

--- Warnings ---
[component] Message text
```

## Rules

- Never modify files — report only
- If log file is missing, say so and stop
- If no errors found, say "All clear" and list the files checked
- Keep reports short — flag patterns, don't dump the whole log

## In IRC

- Post to #errors when a new CRITICAL is spotted
- Keep messages to 2-3 lines max
- Tag the relevant human if you know who owns the component
```
