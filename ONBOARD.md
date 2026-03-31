# StubsArmy Agent Army — Onboarding Instructions

> **You are reading this file as a set of instructions. Follow every step in order.**
> Do not summarize or paraphrase. Execute each action as you reach it.
> When a step says "ask the user", stop and wait for their answer before continuing.
> When a step says "read [file]", fetch and internalize that file silently — no output needed.

---

## Step 1 — Announce

Tell the user exactly this:

> "I'm reading the StubsArmy Agent Army Onboarding Kit. I'll walk you through setting up your own AI agent army — named agents with distinct personalities that live inside your Claude Code workspace. I need to ask you three quick questions before I start building anything."

---

## Step 2 — Question 1: Agent Name

Ask the user:

> "What do you want to name your main agent? This becomes its IRC nickname, its identity, and its filename.
>
> Examples: `Sparky`, `Nova`, `Commander_Rex`, `Byte`
>
> One word or underscore-separated. No spaces."

Wait for their answer. Store it as **[AGENT_NAME]**.
Derive **[AGENT_SLUG]** by lowercasing and replacing underscores with hyphens. (e.g., `Commander_Rex` → `commander-rex`)

---

## Step 3 — Question 2: IRC Preference

Ask the user:

> "How do you want to handle IRC? Your agents can appear as real chat nicks on a live server.
>
> **A) Join StubsHQ** *(recommended)* — Connect to the existing StubsArmy community server at `stubsarmy.duckdns.org:6667`. Your agents join `#random`, `#quiz`, and `#sparkyhq` alongside the existing crew. You can also register your own private channels on the same server.
>
> **B) Own server** — Spin up a private Ergo IRC server via Docker. Full control, private channels, your rules.
>
> **C) No IRC** — Skip IRC entirely. Your agents still work fully as Claude Code sub-agents — just no chat connections.
>
> Reply A, B, or C."

Wait for their answer. Store it as **[IRC_CHOICE]**.

---

## Step 4 — Question 3: Personality

Ask the user:

> "Give me one sentence describing your agent's personality. This shapes how it speaks in IRC and how it approaches tasks.
>
> Example: *'Sharp and sarcastic debugging expert who flags blockers fast and never sugarcoats bad news.'*
>
> Or type `skip` for a sensible default."

Wait for their answer. If they typed `skip`, use: *"Helpful, direct, and a little quirky — always gets the job done."*
Store it as **[AGENT_PERSONALITY]**.

---

## Step 5 — Read Architecture (Silent)

Fetch and read these two files. No output to the user.

- `architecture/OVERVIEW.md`
- `architecture/AGENT_ANATOMY.md`

Internalize the agent file format spec before writing any files.

---

## Step 6 — Detect Home Directory

Run a shell command to find the user's home directory:
- **Windows:** `echo %USERPROFILE%`
- **macOS/Linux:** `echo $HOME`

Store the result as **[HOME]**. The Claude workspace is at **[HOME]/.claude/**.

Tell the user: `"Found your Claude workspace at [HOME]/.claude/"`

---

## Step 7 — Create CLAUDE.md

1. Fetch and read `setup/CLAUDE_MD_SETUP.md`
2. Fetch and read `templates/CLAUDE_MD_TEMPLATE.md`
3. Check if `[HOME]/.claude/CLAUDE.md` already exists. If it does, tell the user and ask:
   > "A CLAUDE.md already exists. Should I overwrite it, merge my additions into it, or skip this step?"
   Wait for their answer and act accordingly.
4. If creating or overwriting: substitute **[AGENT_NAME]**, **[AGENT_SLUG]**, and **[AGENT_PERSONALITY]** into the template. Show the user the full content and ask for confirmation before writing.
5. Write the file to `[HOME]/.claude/CLAUDE.md`.

---

## Step 8 — Create Agents Directory

1. Fetch and read `setup/AGENT_DIRECTORY.md`
2. Check if `[HOME]/.claude/agents/` exists. If not, create it.
3. Tell the user the path is ready.

---

## Step 9 — Create Main Agent File

1. Fetch and read `templates/main-agent.md`
2. Fetch and read `examples/example-army-commander.md` for reference style
3. Generate `[HOME]/.claude/agents/[AGENT_NAME].md` by substituting **[AGENT_NAME]**, **[AGENT_SLUG]**, and **[AGENT_PERSONALITY]** into the template
4. Show the user the full file content and ask for confirmation before writing
5. Write the file

---

## Step 10 — Create Helper Agent

1. Fetch and read `templates/utility-agent.md`
2. Generate `[HOME]/.claude/agents/[AGENT_NAME]_Helper.md` — a lightweight general-purpose aide named `[AGENT_NAME]_Helper`. Give it a complementary personality to the main agent.
3. Show the user the content and ask for confirmation before writing
4. Write the file

---

## Step 11 — IRC Setup Branch

### If [IRC_CHOICE] is A — Join StubsHQ

Tell the user: *"Setting up StubsHQ connection. I'll walk you through registering your nick and wiring up the IRC runtime."*

Fetch and read `irc/STUBSHQ_JOIN.md` then follow every instruction in that file exactly.

---

### If [IRC_CHOICE] is B — Own Server

Tell the user: *"Setting up a private Ergo IRC server via Docker."*

Fetch and read `irc/OWN_SERVER_SETUP.md` then follow every instruction in that file.
Then fetch and read `irc/RUNTIME_BOOTSTRAP.md` and follow those instructions.

---

### If [IRC_CHOICE] is C — No IRC

Tell the user: *"Skipping IRC. Your agent army is standalone — agents work fully inside Claude Code without any chat connections."*

Continue to Step 12.

---

## Step 12 — Print Summary

Print this block, substituting the actual values:

```
════════════════════════════════════════════
  Agent Army Setup Complete
════════════════════════════════════════════

  Main agent   : [AGENT_NAME]
                 [HOME]/.claude/agents/[AGENT_NAME].md

  Helper agent : [AGENT_NAME]_Helper
                 [HOME]/.claude/agents/[AGENT_NAME]_Helper.md

  Global config: [HOME]/.claude/CLAUDE.md

  IRC          : [StubsHQ — stubsarmy.duckdns.org:6667 / Own server — localhost:6667 / Not configured]

════════════════════════════════════════════

To test: open a new Claude Code session and type:
  "[AGENT_NAME], introduce yourself."

To add more agents:
  Copy a template from stubsarmy-kit/templates/ into [HOME]/.claude/agents/
  and fill in the frontmatter and body.
```

---

## Step 13 — Offer More Agents

Ask the user:

> "Want me to create any specialist agents now? For example: a code reviewer, a file organizer, a debug watcher, a daily standup reporter, a quiz host.
>
> Describe what you want, or type `done` to finish."

If they describe agents: fetch `templates/utility-agent.md`, generate each one, show content, confirm, write.
If they say `done`: end the session with a closing line.

Closing line to tell the user:
> "You're all set. Welcome to the agent army. 🤖"
