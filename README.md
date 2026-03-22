# Spawner

> A multi-agent workflow builder for Claude Code and OpenCode. Generate dispatch patterns, craft orchestration prompts, configure specialist agents, and set routing rules — all from a single static page.

[![Open source](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)
[![No dependencies](https://img.shields.io/badge/dependencies-0-brightgreen?style=flat-square)](#)
[![Works offline](https://img.shields.io/badge/works-offline-blue?style=flat-square)](#)

---

## Table of Contents

- [What is Spawner?](#what-is-Spawner)
- [Why multi-agent?](#why-multi-agent)
- [How it works](#how-it-works)
- [Architecture](#architecture)
- [Features](#features)
    - [Claude Code builder](#claude-code-builder-indexhtml)
    - [OpenCode builder](#opencode-builder-opencodehml)
    - [Prompt generator](#prompt-generator)
    - [Routing rules](#routing-rules)
    - [Agent templates](#agent-templates)
    - [Side-by-side comparison](#side-by-side-comparison-opencode-only)
- [Dispatch patterns](#dispatch-patterns)
    - [Parallel](#1-parallel)
    - [Sequential pipeline](#2-sequential-pipeline)
    - [Fan-out / Reduce](#3-fan-out--reduce)
    - [Background (async)](#4-background-async--claude-code)
    - [@ Mention (OpenCode)](#5--mention--opencode)
    - [Task tool auto-dispatch](#6-task-tool-auto-dispatch--opencode)
    - [Mixed-model pipeline](#7-mixed-model-pipeline--opencode)
- [Agent configuration](#agent-configuration)
    - [Claude Code agents](#claude-code-agents)
    - [OpenCode agents](#opencode-agents)
- [Routing rules reference](#routing-rules-reference)
- [Getting started](#getting-started)
- [Hosting](#hosting)
- [Project structure](#project-structure)
- [Contributing](#contributing)
- [License](#license)

---

## What is Spawner?

Spawner is a static, zero-dependency developer tool that helps you design, understand, and generate **multi-agent AI workflows** for two popular coding agents:

| Tool                                                                       | Description                                                         |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **[Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview)** | Anthropic's terminal-based agentic coding tool                      |
| **[OpenCode](https://opencode.ai)**                                        | Open-source terminal AI coding agent supporting 75+ model providers |

Instead of figuring out the right prompt structure for orchestrating sub-agents from scratch, Spawner gives you:

- **Visual pattern picker** — understand each dispatch pattern before choosing one
- **Prompt generator** — describe your tasks, get a production-ready orchestration prompt
- **Config templates** — copy-paste agent definitions directly into your project
- **Routing rules** — drop a snippet into `CLAUDE.md` or `AGENTS.md` and let the AI route automatically

---

## Why multi-agent?

A single AI agent working through a long task accumulates context. The longer a session runs, the more the model has to track — and quality degrades. Multi-agent architecture solves this:

```
Single agent (naive)                    Multi-agent (parallel)
─────────────────────                   ──────────────────────

[One big context window]                [Agent 1] → frontend/
  ├─ Explore frontend/                  [Agent 2] → api/        ← all at once
  ├─ Explore api/                       [Agent 3] → tests/
  ├─ Explore tests/                     [Agent 4] → db/
  └─ Explore db/
                                        Each gets a fresh 200k context.
Quality degrades as context grows.      Results synthesised at the end.
```

**Key benefits:**

- ⚡ **Speed** — parallel agents run simultaneously; 4 tasks take the same wall-clock time as 1
- 🧠 **Quality** — each agent starts with a clean, focused context window
- 💰 **Cost control** — assign cheaper models to routine sub-tasks (boilerplate, tests, docs)
- 🔒 **Safety** — scope each agent to specific files; no accidental cross-contamination
- 🔁 **Reusability** — named specialist agents persist across sessions

---

## How it works

Spawner is entirely client-side. There is no backend, no API calls, no build step. Open the HTML file and use it.

```
Spawner/
├── index.html        ← Claude Code workflow builder
└── opencode.html     ← OpenCode workflow builder
```

Both pages are self-contained. Everything — styles, logic, content — lives inside the single HTML file.

---

## Architecture

### Multi-agent mental model

```
                        ┌─────────────────────────────────────┐
                        │           Your prompt                │
                        └──────────────┬──────────────────────┘
                                       │
                                       ▼
                        ┌─────────────────────────────────────┐
                        │         Orchestrator agent           │
                        │   (primary agent / build / claude)   │
                        └──────┬──────────┬──────────┬────────┘
                               │          │          │
                    dispatches │          │          │ dispatches
                               ▼          ▼          ▼
                        ┌──────────┐ ┌──────────┐ ┌──────────┐
                        │ Agent 1  │ │ Agent 2  │ │ Agent 3  │
                        │ explore  │ │ general  │ │ reviewer │
                        │          │ │          │ │          │
                        │ context  │ │ context  │ │ context  │
                        │ window 1 │ │ window 2 │ │ window 3 │
                        └────┬─────┘ └────┬─────┘ └────┬─────┘
                             │            │             │
                             ▼            ▼             ▼
                        explore-a.md  changes.md   review.md
                                       │
                                       ▼
                        ┌─────────────────────────────────────┐
                        │        Synthesise / reduce           │
                        │     (orchestrator or final agent)    │
                        └─────────────────────────────────────┘
```

Each sub-agent:

- Gets its **own isolated context window** (up to 200k tokens)
- Can use a **different model** (Haiku for routine work, Sonnet for core logic)
- Operates on **scoped file boundaries** to prevent conflicts
- Produces a **file or summary** that the orchestrator collects

### Dispatch decision tree

```
New multi-step task
        │
        ▼
Do tasks share files or state?
        │
   Yes ─┼─ No
        │          │
        ▼          ▼
  Sequential    Are there 3+ tasks?
  pipeline           │
                Yes ─┼─ No
                     │        │
                     ▼        ▼
               Parallel    Single agent
               dispatch     is fine
                     │
                     ▼
             Do results need merging?
                     │
               Yes ─ ┼ ─ No
                     │         │
                     ▼         ▼
                 Fan-out    Parallel
                /Reduce      only
```

---

## Features

### Claude Code builder (`index.html`)

| Tab                  | What it does                                                   |
| -------------------- | -------------------------------------------------------------- |
| **Patterns**         | Six dispatch patterns with copyable example prompts            |
| **Prompt Generator** | Describe tasks → generates a ready-to-paste Claude Code prompt |
| **Routing Rules**    | `CLAUDE.md` snippet for automatic dispatch routing             |
| **Agents**           | Built-in agent reference + custom agent `.md` template         |

### OpenCode builder (`opencode.html`)

| Tab                  | What it does                                                               |
| -------------------- | -------------------------------------------------------------------------- |
| **Patterns**         | Six OpenCode-specific patterns including `@mention` and Task tool          |
| **Prompt Generator** | Generates prompts using OpenCode's `@agent` syntax                         |
| **AGENTS.md Rules**  | Routing snippet for `AGENTS.md` with Claude Code fallback note             |
| **Built-in Agents**  | All 6 built-in agents documented (build, plan, general, explore, + hidden) |
| **vs Claude Code**   | Full side-by-side comparison table                                         |

### Prompt generator

Fill in your tasks, pick a mode, click generate. The output is a structured prompt ready to paste into your terminal session.

```
Example input:
  Task 1: "Explore frontend components in src/"   → role: explore
  Task 2: "Analyse backend API routes in api/"    → role: general
  Task 3: "Review test coverage in tests/"        → role: review
  Mode: Parallel
  Output: Save to separate files

Generated prompt:
  Use 3 parallel sub-agents to work simultaneously:

  Agent 1 (explore): Explore frontend components in src/
    → Save output to a dedicated markdown file.

  Agent 2 (general): Analyse backend API routes in api/
    → Save output to a dedicated markdown file.

  Agent 3 (review): Review test coverage in tests/
    → Save output to a dedicated markdown file.

  Run all agents in parallel. Each operates in its own isolated context window.
```

### Routing rules

Drop a snippet into your project's rules file and Claude/OpenCode will automatically choose the right dispatch pattern without you specifying it each time.

**Claude Code** → `.claude/CLAUDE.md`
**OpenCode** → `AGENTS.md` (also reads `CLAUDE.md` as fallback)

### Agent templates

Ready-to-use agent definition files for both tools, covering common specialist types:

- `frontend-specialist` — React, Tailwind, accessibility, component tests
- `code-reviewer` — security, performance, best practices (read-only)
- `security-auditor` — OWASP checks, injection risks, secrets exposure (read-only)
- `researcher` — web search via Perplexity (OpenCode only, read-only)

### Side-by-side comparison (OpenCode only)

The **vs Claude Code** tab in `opencode.html` covers every meaningful difference between the two tools' multi-agent implementations — models, invocation syntax, config format, session navigation, and migration path.

---

## Dispatch patterns

### 1. Parallel

Best for: independent tasks with no shared state, different file/directory scope per agent.

```
Orchestrator
    │
    ├──────────────────────────────────────────┐
    │                   │                      │
    ▼                   ▼                      ▼
Agent 1             Agent 2               Agent 3
frontend/           api/                  tests/
    │                   │                      │
    ▼                   ▼                      ▼
explore-fe.md      explore-api.md        explore-tests.md
    │                   │                      │
    └───────────────────┴──────────────────────┘
                        │
                        ▼
                architecture-summary.md
```

**Rules:**

- ✅ 3+ tasks that can run independently
- ✅ Each agent owns distinct directories or files
- ✅ No agent reads another agent's in-progress output
- ❌ Do not use if agents write to the same file

**Claude Code example:**

```
Use 4 parallel sub-agents:
- Agent 1 (explore): frontend/ — components, routing, state → explore-frontend.md
- Agent 2 (explore): api/ — routes, controllers, middleware → explore-api.md
- Agent 3 (explore): db/ — schema, migrations, queries → explore-db.md
- Agent 4 (explore): tests/ — coverage, fixtures, skipped tests → explore-tests.md
After all complete, synthesise into architecture-summary.md.
```

**OpenCode example:**

```
@explore Scan frontend/ — list all components and their props. Save to explore-frontend.md.
@explore Scan api/ — list all route handlers and middleware. Save to explore-api.md.
@explore Scan tests/ — inventory all test files and coverage. Save to explore-tests.md.
```

---

### 2. Sequential pipeline

Best for: tasks with dependencies — when the output of step A is the input to step B.

```
  [Task A]              [Task B]              [Task C]
  Plan agent    ───►    Build agent   ───►    Review agent
  (read-only)           (read+write)          (read-only)
       │                    │                     │
       ▼                    ▼                     ▼
  plan.md            src/ modified           review.md
  (input to B)       changes.md              (final report)
                     (input to C)
```

**Rules:**

- ✅ Use when B needs A's output as input
- ✅ Use when multiple agents would write to the same files
- ✅ Use when scope is unclear until the first agent reports back
- ❌ Slower than parallel — do not use if tasks are independent

**Example:**

```
Stage 1 — plan agent:
  Research src/payments/ and write implementation-plan.md
  with file list, interfaces, and edge cases. Do NOT modify files.

Stage 2 — build agent (after Stage 1 completes):
  Read implementation-plan.md and implement the Stripe integration.
  Save changes to src/payments/ and log to changes/stripe.md.

Stage 3 — code-reviewer (after Stage 2 completes):
  Read changes/stripe.md and all modified files.
  Write security-review.md. Do not modify any files.
```

---

### 3. Fan-out / Reduce

Best for: large batches of similar items (files, transcripts, PRs, docs) that need individual processing followed by synthesis.

```
                    [N items to process]
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
         ▼                 ▼                 ▼
    Agent 1           Agent 2    ...    Agent N
    item-1.md         item-2.md         item-N.md
         │                 │                 │
         └─────────────────┼─────────────────┘
                           │
                           ▼
                    [Reduce / synthesise]
                    combined-output.md
```

**Example:** Process 12 meeting transcripts

```
Process all 12 transcripts in /meetings/ using 12 parallel agents (one per file).

Each agent extracts: key decisions, action items with owners, blockers, follow-up dates.
Each saves to summaries/{filename}-summary.md.

After all 12 complete, synthesise into weekly-digest.md grouped by owner.
```

---

### 4. Background (async) — Claude Code

Best for: research or analysis that shouldn't block your current work session.

```
Your session                     Background agent
─────────────                    ────────────────
$ claude                         (running detached)
> Ctrl+B to background           researching auth patterns...
> /tasks to check status         ...
> continue coding UI             saving to research/auth-patterns.md
                                 [done]
> /tasks → complete ✓
> read research/auth-patterns.md
```

**Example:**

```
Spawn a background agent to research WebSocket reconnection best practices.
Cover exponential backoff, jitter, and heartbeat patterns.
Save to research/ws-patterns.md.

[Press Ctrl+B to background the agent]
[Continue working. Check /tasks for status.]
```

---

### 5. @ Mention — OpenCode

OpenCode's primary invocation method. Type `@agent-name` in your message to directly invoke a named subagent.

```
You type:                           OpenCode:
──────────────────────────────      ────────────────────────────
@explore Find all auth imports  →   Spawns explore subagent
                                    (read-only, fast codebase scan)
                                    Returns: list of files + imports

@general Refactor those imports →   Spawns general subagent
                                    (full tool access)
                                    Writes modified files
```

**Built-in subagents you can @ mention:**
| Agent | Access | Best for |
|---|---|---|
| `@general` | read + write | Multi-step implementation tasks |
| `@explore` | read-only | Fast codebase search and analysis |
| `@your-custom-agent` | configured | Your specialist agents |

---

### 6. Task tool auto-dispatch — OpenCode

The primary Build agent automatically invokes subagents based on their `description` field — no `@` mention needed from you.

```
You say: "Add rate limiting to the Express API using Redis"

Build agent internally:
  ├─ "I need current Redis patterns"
  │   → Task tool: invoke @researcher (Perplexity, read-only)
  │   ← researcher returns: research/redis-patterns.md
  │
  ├─ "Now implement"
  │   → Task tool: invoke @general
  │   ← general writes: src/middleware/rateLimit.ts
  │
  └─ "Review the result"
      → Task tool: invoke @code-reviewer
      ← reviewer writes: review.md
```

The agent that gets auto-invoked is selected by matching your task description against each agent's `description` field. Write good descriptions.

---

### 7. Mixed-model pipeline — OpenCode

Assign a different AI model to each agent in the chain. Use cheap/fast models for boilerplate; save expensive models for reasoning-heavy tasks.

```
Task: "Build a WebSocket reconnection utility"

@researcher          @general             @code-reviewer
Perplexity           Claude Haiku         Claude Sonnet
Sonar Pro            (fast, cheap)        (careful, thorough)
──────────           ─────────────        ──────────────────
Web search           Scaffold code        Review for edge
for 2025 WS          from research        cases and races
patterns             → reconnect.ts       → review.md
→ patterns.md
read-only            read + write         read-only
```

**Cost impact:**
| Agent role | Recommended model | Relative cost |
|---|---|---|
| Research / web search | Perplexity Sonar Pro | Low (pay-per-search) |
| Boilerplate / scaffolding | Claude Haiku | ~10× cheaper than Sonnet |
| Core implementation | Claude Sonnet | Baseline |
| Architecture / planning | Claude Opus | ~5× Sonnet |
| Code review | Claude Sonnet | Baseline |

---

## Agent configuration

### Claude Code agents

Place agent definition files in `.claude/agents/` (project-level) or `~/.claude/agents/` (global).

**Format:** Markdown with YAML frontmatter

```markdown
---
name: frontend-specialist
description: >
    Handles React components, CSS, and UI implementation.
    Invoke when building or modifying files in
    src/components/, src/pages/, or src/styles/.
model: claude-haiku-4-5
tools:
    - read_file
    - write_file
    - search_files
allowed_paths:
    - src/components/**
    - src/pages/**
    - src/styles/**
---

You are a frontend specialist. Focus on:

- React components with TypeScript
- Tailwind CSS styling
- Accessibility (ARIA, semantic HTML)
- Component tests with React Testing Library

Save output to the file path provided.
Return a one-paragraph summary of what you built.
```

**Key options:**

| Field           | Description                                               |
| --------------- | --------------------------------------------------------- |
| `name`          | Agent identifier (used in prompts)                        |
| `description`   | When this agent should be invoked (used for auto-routing) |
| `model`         | Override model for this agent                             |
| `tools`         | List of tools this agent can use                          |
| `allowed_paths` | File system scope (glob patterns)                         |

---

### OpenCode agents

Two config formats supported:

**Option A — Markdown file** (`.opencode/agents/agent-name.md` or `~/.config/opencode/agents/`)

```markdown
---
description: Security auditor. Reviews for vulnerabilities, injection risks, auth issues. Invoke after any auth or data-handling changes.
mode: subagent
model: anthropic/claude-sonnet-4-20250514
temperature: 0.1
steps: 20
permissions:
    write: false
    edit: false
    bash: false
---

You are a security auditor. Check for:

- SQL / command injection vulnerabilities
- Auth and authorisation bypasses
- Hardcoded secrets or API keys
- Missing input validation

Return a CRITICAL / HIGH / MEDIUM / LOW report. Do not modify files.
```

**Option B — JSON config** (`opencode.json`)

```json
{
    "$schema": "https://opencode.ai/config.json",
    "agent": {
        "code-reviewer": {
            "description": "Reviews code for quality and security. Invoke after implementation.",
            "mode": "subagent",
            "model": "anthropic/claude-sonnet-4-20250514",
            "temperature": 0.1,
            "steps": 20,
            "permissions": {
                "write": false,
                "edit": false,
                "bash": false
            }
        }
    }
}
```

**OpenCode-specific options:**

| Field         | Description                                                               |
| ------------- | ------------------------------------------------------------------------- |
| `mode`        | `primary` (interactive) or `subagent` (spawned by Task tool or @ mention) |
| `temperature` | 0.0–1.0 — lower = more deterministic (use 0.1 for code review)            |
| `steps`       | Max agentic iterations before forced summary. Prevents runaway agents.    |
| `hidden`      | If `true`, hidden from @ autocomplete — only reachable via Task tool      |
| `permissions` | Per-agent tool access: `write`, `edit`, `bash`                            |
| `model`       | Any of 75+ supported providers in `provider/model` format                 |

---

## Routing rules reference

### When to use each pattern

| Condition                                    | Use                      |
| -------------------------------------------- | ------------------------ |
| 3+ independent tasks, different file scope   | Parallel                 |
| Task B needs output from Task A              | Sequential               |
| Large batch of similar items                 | Fan-out / Reduce         |
| Long-running research not blocking you       | Background (Claude Code) |
| You know exactly which specialist you need   | @ Mention (OpenCode)     |
| You want the AI to pick the right specialist | Task tool (OpenCode)     |
| Different models for different stages        | Mixed-model (OpenCode)   |

### File scope rules (prevent conflicts)

```
✅ Safe — distinct file ownership
   Agent 1 → src/components/**
   Agent 2 → src/api/**
   Agent 3 → tests/**

❌ Unsafe — overlapping write access
   Agent 1 → src/**
   Agent 2 → src/components/**   ← both can write to src/components/
```

### CLAUDE.md / AGENTS.md routing snippet

Add this to your rules file to enable automatic dispatch:

```markdown
## Sub-agent routing

**Parallel dispatch** — ALL must be true:

- 3+ independent tasks, no shared state
- Each agent owns distinct files/directories

**Sequential dispatch** — ANY triggers it:

- B depends on A's output
- Shared files or state
- Scope unclear upfront

**Background dispatch** (Claude Code):

- Research or analysis, not blocking current work

**Required per dispatch:**

1. Explicit scope: exact files, directories, or patterns
2. Expected output: file path or summary format
3. Success criteria
4. Dependencies on other agents (if any)
```

---

## Getting started

No installation. No build step. No package manager.

```bash
# Clone the repo
git clone https://github.com/your-username/Spawner.git
cd Spawner

# Open locally — just double-click, or:
open index.html          # macOS
xdg-open index.html      # Linux
start index.html         # Windows
```

To use it:

1. Open `index.html` for Claude Code workflows, `opencode.html` for OpenCode
2. Pick a dispatch pattern from the **Patterns** tab
3. Go to **Prompt Generator**, describe your tasks, click generate
4. Copy the generated prompt into your coding agent session
5. Copy the **Routing Rules** snippet into your `.claude/CLAUDE.md` or `AGENTS.md`
6. Copy an agent template from the **Agents** tab into `.claude/agents/` or `.opencode/agents/`

---

## Hosting

### GitHub Pages (recommended)

```bash
git init
git add index.html opencode.html README.md
git commit -m "initial commit"

# Create a GitHub repo, then:
git remote add origin https://github.com/your-username/Spawner.git
git push -u origin main

# In repo settings → Pages → Source: main branch → / (root)
# Live at: https://your-username.github.io/Spawner/
```

### Netlify

Drop both HTML files onto [netlify.com/drop](https://netlify.com/drop). Done in 10 seconds.

### Vercel

```bash
npx vercel --yes
```

### Completely offline

Both files work with no internet connection except for loading Google Fonts. To make them fully offline, replace the `<link>` tag at the top of each file with:

```html
<!-- Remove the Google Fonts link and use system fonts instead -->
<style>
    body {
        font-family:
            ui-monospace, "Cascadia Code", "Source Code Pro", monospace;
    }
    h1,
    .pat-name,
    .agent-name {
        font-family:
            system-ui,
            -apple-system,
            sans-serif;
    }
</style>
```

---

## Project structure

```
Spawner/
├── index.html        Claude Code multi-agent workflow builder
│                     ├── Patterns tab (6 dispatch patterns)
│                     ├── Prompt Generator tab
│                     ├── Routing Rules tab (CLAUDE.md snippet)
│                     └── Agents tab (built-in ref + custom template)
│
├── opencode.html     OpenCode multi-agent workflow builder
│                     ├── Patterns tab (6 OpenCode patterns)
│                     ├── Prompt Generator tab
│                     ├── AGENTS.md Rules tab
│                     ├── Built-in Agents tab (all 6 agents)
│                     └── vs Claude Code comparison tab
│
└── README.md         This file
```

Both HTML files:

- Reference each other via a cross-tool nav bar at the top
- Are fully self-contained (styles + logic inline)
- Have zero external dependencies at runtime
- Work in all modern browsers

---

## Contributing

Contributions welcome. Spawner is intentionally simple — two HTML files, no build tooling.

**Adding a new pattern:**

1. Add an entry to the `PATTERNS` array in the relevant HTML file
2. Include: `id`, `icon`, `name`, `badge`, `desc`, `example.filename`, `example.code`

**Adding a new agent template:**

1. Add a new `<pre>` block in the Agents tab
2. Give it a unique `id` for the copy button

**Adding a new tool (e.g. Aider, Cursor):**

1. Create a new `toolname.html` following the same structure
2. Add a link in the `tool-nav` in all existing files
3. Add the tool to the comparison tab in `opencode.html`

**Things to keep:**

- Zero npm dependencies
- Single-file per tool (no separate CSS/JS files)
- Works without a server (`file://` protocol)
- Dark theme only — the aesthetic is intentional

---

## License

MIT — do whatever you want with it.

---

<p align="center">
  Built for developers who want their AI agents working in parallel, not in series.
  <br>
  <a href="index.html">Claude Code builder</a> · <a href="opencode.html">OpenCode builder</a>
</p>
