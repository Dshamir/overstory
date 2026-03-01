# Overstory — User Guide

A step-by-step guide to using Overstory with Claude Code. No jargon. No assumptions. If you can type into a terminal, you can use this.

---

## Part 1: Before You Start

### What You Need Installed

Overstory needs four things on your machine. Here's what they are and why:

| Tool | What it is | Why Overstory needs it |
|------|-----------|----------------------|
| **Bun** | A fast JavaScript runtime (like Node.js, but faster) | Runs Overstory's code |
| **Claude Code** | Anthropic's AI coding assistant for the terminal | The AI brain inside each agent |
| **tmux** | A terminal multiplexer (lets you run many terminals inside one) | Each agent runs in its own tmux session |
| **git** | Version control | Agents work in isolated copies of your code (git worktrees) |

### Installing Overstory

Once you have the tools above, install Overstory globally:

```bash
bun install -g @os-eco/overstory-cli
```

This gives you the `ov` command.

### Verify It Works

```bash
ov --version
```

You should see something like `0.7.7`.

For a deeper health check:

```bash
ov doctor
```

This checks that all your dependencies are installed, your configuration is valid, and everything is wired up correctly. Green checks mean you're good. Red X's tell you exactly what to fix.

---

## Part 2: Setting Up Your Project

### Step 1: Initialize Overstory

Navigate to your project and run:

```bash
cd /path/to/your/project
ov init
```

**What this does:**
- Creates an `.overstory/` directory in your project
- Sets up SQLite databases for messaging, sessions, metrics, and events
- Generates a config file (`.overstory/config.yaml`)
- Optionally bootstraps ecosystem tools (mulch for expertise, seeds for issue tracking, canopy for prompt management)

**What you'll see:**
The command walks you through a few questions (project name, which tools to install). If you want to skip the questions and accept defaults:

```bash
ov init --yes
```

### Step 2: Install Hooks

Hooks connect Overstory to your Claude Code session. Without them, messages from agents won't show up automatically.

```bash
ov hooks install
```

**What this does:**
- Adds event hooks to `.claude/settings.local.json`
- When your session starts, Overstory loads context automatically (`ov prime`)
- When you send a message, Overstory checks for new mail from agents (`ov mail check --inject`)

**To verify hooks are active:**

```bash
ov hooks status
```

### Step 3: Check Ecosystem Health

```bash
ov ecosystem
```

This shows the version and health status of all integrated tools (Overstory itself, mulch, seeds, canopy). Make sure everything is green.

---

## Part 3: Your First Agent

Let's walk through the simplest possible workflow: sending a scout to explore your codebase, then sending a builder to make changes.

### Scenario

You have a task — let's say task ID `task-1` — and you want an agent to investigate the codebase before making changes.

### Step 1: Write a Spec (Optional but Helpful)

A spec tells the agent exactly what to do. You can write one:

```bash
ov spec write task-1 --body "Explore the authentication module. Find all files related to user login, identify the auth flow, and report back what you find."
```

This creates `.overstory/specs/task-1.md`.

### Step 2: Spawn a Scout

A scout is a read-only agent. It looks at code but never changes it. Perfect for investigation.

```bash
ov sling task-1 --capability scout --name auth-scout --spec .overstory/specs/task-1.md
```

**What happens:**
1. Overstory creates a git worktree (an isolated copy of your repo)
2. It starts a tmux session named `ov-auth-scout`
3. Claude Code launches inside that session with instructions from the scout agent definition
4. The scout starts working immediately

### Step 3: Check What's Happening

See all active agents:

```bash
ov status
```

Watch the live dashboard:

```bash
ov dashboard
```

Check if the scout sent you any messages:

```bash
ov mail check
```

Read a specific message:

```bash
ov mail read <message-id>
```

### Step 4: Spawn a Builder

Once you've seen the scout's findings, spawn a builder to make changes:

```bash
ov sling task-1 --capability builder --name auth-builder --files "src/auth/*.ts,src/middleware/auth.ts"
```

The `--files` flag limits which files the builder can touch. This is a safety feature — the agent can only modify the files you specify.

### Step 5: Check the Builder's Work

```bash
ov inspect auth-builder
```

This shows you exactly what the agent is doing: recent tool calls, current status, and tmux output.

### Step 6: Merge the Work

When the builder is done:

```bash
ov merge --branch overstory/auth-builder
```

This merges the builder's branch back into your current branch.

### Step 7: Clean Up

```bash
ov worktree clean --completed
```

This removes worktrees for agents that have finished their work.

---

## Part 4: Command Cheat Sheet

Every `ov` command, grouped by what you're trying to do.

### Setup & Configuration

| Command | What it does |
|---------|-------------|
| `ov init` | Set up Overstory in your project |
| `ov hooks install` | Connect Overstory to Claude Code hooks |
| `ov hooks status` | Check if hooks are active |
| `ov hooks uninstall` | Remove hooks |
| `ov doctor` | Run health checks on your setup |
| `ov ecosystem` | Check versions and health of all tools |
| `ov upgrade` | Upgrade Overstory to the latest version |
| `ov upgrade --all` | Upgrade all ecosystem tools |
| `ov upgrade --check` | Check for updates without installing |

### Spawning & Stopping Agents

| Command | What it does |
|---------|-------------|
| `ov sling <task-id> --capability <type> --name <name>` | Spawn an agent |
| `ov sling <task-id> --capability lead --name <name>` | Spawn a team lead (can manage sub-agents) |
| `ov stop <agent-name>` | Stop a running agent |
| `ov stop <agent-name> --clean-worktree` | Stop an agent and remove its worktree |
| `ov coordinator start` | Start the top-level coordinator agent |
| `ov coordinator stop` | Stop the coordinator |
| `ov coordinator status` | Check coordinator state |

### Talking to Agents

| Command | What it does |
|---------|-------------|
| `ov mail send --to <agent> --subject "..." --body "..."` | Send a message to an agent |
| `ov mail check` | Check your inbox for new messages |
| `ov mail check --inject` | Check inbox and inject messages into your session |
| `ov mail list` | List all messages |
| `ov mail list --unread` | List only unread messages |
| `ov mail read <id>` | Read a specific message (marks it as read) |
| `ov mail reply <id> --body "..."` | Reply to a message |
| `ov mail purge --all` | Delete all messages |
| `ov mail purge --days 7` | Delete messages older than 7 days |
| `ov nudge <agent> "message"` | Send a quick text nudge via tmux |

### Watching What's Happening

| Command | What it does |
|---------|-------------|
| `ov status` | Show all active agents and their states |
| `ov status --verbose` | Show extra detail per agent |
| `ov dashboard` | Open the live TUI dashboard |
| `ov inspect <agent>` | Deep-dive into a single agent |
| `ov inspect <agent> --follow` | Continuously refresh agent inspection |
| `ov feed --follow` | Live event stream across all agents |
| `ov trace <agent-or-task>` | Chronological timeline for an agent or task |
| `ov logs` | Query log files across all agents |
| `ov logs --agent <name> --follow` | Tail logs for a specific agent |
| `ov errors` | Show aggregated errors across agents |
| `ov replay` | Interleaved chronological replay across agents |

### Merging Work

| Command | What it does |
|---------|-------------|
| `ov merge --branch <name>` | Merge a specific agent's branch |
| `ov merge --all` | Merge all completed agent branches |
| `ov merge --dry-run` | Check for conflicts without merging |
| `ov merge --into <branch>` | Merge into a specific branch (instead of the default) |

### Organizing Tasks

| Command | What it does |
|---------|-------------|
| `ov spec write <task-id> --body "..."` | Write a task specification |
| `ov group create '<name>' <id1> <id2> ...` | Create a group of related tasks |
| `ov group status` | Check progress on all groups |
| `ov group status <group-id>` | Check progress on a specific group |
| `ov group add <group-id> <id>` | Add a task to a group |
| `ov group remove <group-id> <id>` | Remove a task from a group |
| `ov group list` | List all groups |
| `ov run` | Show current run status |
| `ov run list` | List recent runs |
| `ov run show <id>` | Show details for a specific run |
| `ov run complete` | Mark the current run as completed |

### Tracking Costs

| Command | What it does |
|---------|-------------|
| `ov costs` | Show overall cost breakdown |
| `ov costs --live` | Show real-time token usage for active agents |
| `ov costs --self` | Show cost for your current session |
| `ov costs --agent <name>` | Show cost for a specific agent |
| `ov costs --by-capability` | Group costs by agent type |
| `ov costs --run <id>` | Show costs for a specific run |
| `ov metrics` | Show session metrics |

### Cleaning Up

| Command | What it does |
|---------|-------------|
| `ov clean --all` | Wipe everything (nuclear option) |
| `ov clean --mail` | Delete the mail database |
| `ov clean --sessions` | Wipe session data |
| `ov clean --worktrees` | Remove all worktrees and kill tmux sessions |
| `ov clean --branches` | Delete all `overstory/*` branches |
| `ov clean --logs` | Remove all agent logs |
| `ov worktree list` | List all worktrees and their status |
| `ov worktree clean` | Remove worktrees for finished agents |
| `ov worktree clean --all` | Force remove all worktrees |

---

## Part 5: Agent Types Explained

Overstory has seven types of agents. Each has a specific role and specific permissions.

### Scout — "The Researcher"

**What it does:** Reads code, explores the codebase, reports findings.
**What it can't do:** Change any files. It's read-only.
**When to use it:** Before making changes, when you need to understand how something works.

```bash
ov sling task-1 --capability scout --name my-scout
```

### Builder — "The Coder"

**What it does:** Writes code, creates files, runs tests.
**What it can't do:** Spawn other agents. It works alone.
**When to use it:** When you have a clear task and want code written.

```bash
ov sling task-1 --capability builder --name my-builder --files "src/api/*.ts"
```

### Reviewer — "The Critic"

**What it does:** Reviews code for bugs, style issues, and correctness. Reports findings.
**What it can't do:** Change any files. Like a scout, it's read-only.
**When to use it:** After a builder finishes, to validate the work before merging.

```bash
ov sling task-1 --capability reviewer --name my-reviewer
```

### Merger — "The Integrator"

**What it does:** Merges branches, resolves conflicts, ensures the combined code works.
**What it can't do:** Write new features. It only combines existing work.
**When to use it:** When multiple builders have finished and their branches need to be combined.

```bash
ov sling task-1 --capability merger --name my-merger
```

### Lead — "The Team Captain"

**What it does:** Reads a task, breaks it into subtasks, spawns scouts and builders, reviews their work, merges results.
**What it can't do:** Act as the top-level orchestrator.
**When to use it:** When you have a big task that needs decomposition. Just sling a lead and it handles the rest.

```bash
ov sling task-1 --capability lead --name feature-lead
```

### Coordinator — "The CEO"

**What it does:** Manages the entire fleet. Spawns leads, monitors progress, handles escalations.
**What it can't do:** Write code directly. It delegates everything.
**When to use it:** When you want full automation — just start the coordinator and walk away.

```bash
ov coordinator start
```

### Monitor — "The Security Guard"

**What it does:** Watches all agents continuously. Detects stuck agents, failed tasks, and unhealthy states. Nudges slackers.
**What it can't do:** Write code or manage tasks. It only observes and alerts.
**When to use it:** During long-running sessions when you want automatic health monitoring.

```bash
ov monitor start
```

---

## Part 6: Common Workflows

### "I have one simple task"

Skip the management layers. Sling a builder directly.

```bash
ov spec write task-1 --body "Add input validation to the signup form"
ov sling task-1 --capability builder --name signup-builder --files "src/forms/signup.ts"
ov status           # check progress
ov merge --branch overstory/signup-builder
```

### "I have a big feature with multiple parts"

Sling a lead. It will decompose the work, spawn its own scouts and builders, and manage everything.

```bash
ov spec write task-2 --body "Implement the new dashboard page with charts, filters, and export functionality"
ov sling task-2 --capability lead --name dashboard-lead
ov dashboard        # watch the team work
ov merge --all      # merge everything when done
```

### "I want full autopilot"

Start the coordinator. It reads your available tasks, spawns leads, and manages the entire fleet.

```bash
ov coordinator start --watchdog
```

The `--watchdog` flag also starts the health monitoring daemon, which automatically detects and recovers stuck agents.

To also start the Tier 2 monitor (an AI agent that patrols the fleet):

```bash
ov coordinator start --watchdog --monitor
```

### "Something's stuck"

An agent has been running too long or seems unresponsive. Here's what to do:

```bash
# 1. Check what it's doing
ov inspect stuck-agent

# 2. Try nudging it
ov nudge stuck-agent "Are you still working? Please report your status."

# 3. Check its mail — maybe it sent you an error
ov mail check

# 4. If all else fails, stop it
ov stop stuck-agent
```

### "Everything's done, time to clean up"

```bash
# 1. Merge all completed work
ov merge --all

# 2. Verify everything looks good
ov status

# 3. Clean up worktrees, branches, and databases
ov worktree clean --completed
ov clean --branches

# 4. Or just nuke everything
ov clean --all
```

---

## Part 7: Troubleshooting

### "Agent won't start"

**Check tmux is installed:**
```bash
tmux -V
```

**Run the doctor:**
```bash
ov doctor
```

**Common cause — nested sessions:** If you're running Claude Code and it spawns an agent that also tries to run Claude Code, the nested session may be blocked. Overstory handles this automatically by unsetting the `CLAUDECODE` environment variable, but if something goes wrong, try stopping all agents and starting fresh:
```bash
ov clean --worktrees
```

### "No messages showing up"

**Are hooks installed?**
```bash
ov hooks status
```

If hooks aren't installed, messages won't be automatically injected into your session. Install them:
```bash
ov hooks install
```

**Manually check for messages:**
```bash
ov mail check --inject
```

### "Merge conflicts"

**Always dry-run first:**
```bash
ov merge --branch overstory/my-builder --dry-run
```

This shows you if there are conflicts without actually merging.

**Understanding merge tiers:**
Overstory uses a 4-tier merge system:
1. **Tier 0 — Fast-forward:** No conflicts, clean merge.
2. **Tier 1 — Auto-merge:** Git can resolve the conflicts automatically.
3. **Tier 2 — AI-assisted:** An AI agent resolves the conflicts.
4. **Tier 3 — Manual:** You need to resolve the conflicts yourself.

### "Too many agents running"

Set a concurrency limit in `.overstory/config.yaml`:

```yaml
maxConcurrent: 3
```

Or stop individual agents:
```bash
ov stop agent-name
```

---

## Part 8: Quick Reference Card

### Top 10 Commands You'll Use Daily

```bash
ov status                           # What's happening right now?
ov sling <id> --capability builder --name <name>  # Start a coding agent
ov sling <id> --capability lead --name <name>     # Start a team lead
ov mail check                       # Any new messages?
ov inspect <agent>                  # What's this agent doing?
ov dashboard                        # Live overview of everything
ov merge --all                      # Merge all finished work
ov costs --live                     # How much is this costing?
ov stop <agent>                     # Kill a misbehaving agent
ov doctor                           # Is everything set up correctly?
```

### Config File Locations

| File | Purpose |
|------|---------|
| `.overstory/config.yaml` | Project configuration |
| `.overstory/config.local.yaml` | Machine-specific overrides (gitignored) |
| `.claude/settings.local.json` | Claude Code hooks (installed by `ov hooks install`) |

### Where Data Lives

| Location | What's there |
|----------|-------------|
| `.overstory/mail.db` | Agent messages (SQLite) |
| `.overstory/sessions.db` | Session and run data (SQLite) |
| `.overstory/events.db` | Tool events and timelines (SQLite) |
| `.overstory/metrics.db` | Token usage and metrics (SQLite) |
| `.overstory/merge-queue.db` | Merge queue (SQLite) |
| `.overstory/specs/` | Task specification files |
| `.overstory/agents/` | Agent identity and checkpoint files |
| `.overstory/worktrees/` | Git worktrees (gitignored) |
| `.overstory/logs/` | Agent logs (gitignored) |
