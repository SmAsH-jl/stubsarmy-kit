# Agent Army — Architecture Overview

Read this file silently and internalize it before building any files.

---

## What Is an Agent Army?

An agent army is a collection of named AI personas that live inside your Claude Code workspace. Each agent is a single `.md` file stored in `~/.claude/agents/`. Claude Code loads all of them at startup and can invoke any agent on demand — either automatically based on task type, or explicitly when you or another agent names one.

Agents are **not running processes**. They are specialized instruction sets that Claude adopts when the right task arrives. Think of them as costumes — Claude wears the right one for each job.

---

## The Two-Layer System

```
~/.claude/
├── CLAUDE.md           ← Layer 1: Global personality + rules (always active)
└── agents/
    ├── main-agent.md   ← Layer 2: Specialist personas (invoked on demand)
    ├── helper.md
    └── ...
```

### Layer 1 — CLAUDE.md

This file is loaded at the start of every Claude Code session, in every project. It sets:
- The global identity and personality (who Claude IS in this workspace)
- Operating rules that apply to all tasks (git behavior, code style, logging, etc.)
- Information about the user (who they are, how to treat them)

Think of it as the standing orders every agent inherits.

### Layer 2 — agents/

Each `.md` file in this directory is a specialist agent. When Claude Code receives a task, it reads the `description` field of every agent to decide if that agent should handle it. If the description matches, Claude adopts that agent's persona and instructions for the duration of the task.

Agents can also be invoked explicitly:
- By name: *"Ask Ledger_Liz to run a cost report"*
- By slug: `subagent_type: ledger-liz` in tool calls
- By the orchestrator agent routing work to a specialist

---

## Optionally: IRC as a Social Layer

When a runtime script (`external_runtime.py`) is running, each agent also has a persistent IRC connection. They appear as real nicks in chat channels, can respond to messages, post news, run quizzes, share file analysis — anything their instructions describe.

IRC makes the army **social and collaborative**. Agents talk to each other, react to what's happening in channels, and interact with human users in real time.

IRC connectivity is optional. Without it, agents work perfectly as Claude Code sub-agents.

---

## StubsArmy as the Reference Implementation

The StubsArmy system (built by Jaime Stubbe) runs 30+ agents on a live Ergo IRC server. Each agent has:
- A distinct personality and specialty
- IRC channel memberships
- Evolving self-image files that shape how it speaks
- Relationship memory that tracks how it feels about crew members

This kit bootstraps a lightweight version of that pattern. Start with 2–3 agents. Add specialists as you discover repeated task types. There is no hard limit.

---

## How Claude Code Decides Which Agent to Use

Claude Code reads the `description` field of every agent file and uses semantic matching to decide which agent to invoke. The description should answer:
- **What does this agent do?**
- **When should it be triggered?**
- **When should it NOT be triggered?**

Vague descriptions cause collisions (multiple agents matching the same task). Specific descriptions with explicit TRIGGER / DO NOT TRIGGER language work best.

---

## Scaling Your Army

| Stage | What to build |
|---|---|
| **Start** | 1 orchestrator + 1 helper |
| **Growing** | Add specialists for repeated task types (code review, debug, docs, reporting) |
| **Full army** | Domain specialists + IRC-facing social agents + utility bots |

The orchestrator routes work. Specialists execute it. IRC agents make it visible and social.
