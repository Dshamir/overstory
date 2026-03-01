# Overstory — Executive Summary

## What Is Overstory?

Overstory turns a single Claude Code session into an entire development team. It's a multi-agent orchestration system that spawns AI coding agents in parallel, each working in its own isolated copy of your codebase, coordinating through a built-in messaging system, and merging their work back together automatically. One command, multiple agents, real parallel development.

## The Problem It Solves

AI coding assistants are powerful — but they work one task at a time. If you have a feature with five files to change, your AI writes them one by one while you wait. If you need research done before coding starts, that's another round of waiting. And if two changes touch the same code? You're playing traffic cop.

Overstory eliminates the bottleneck. Instead of one agent doing everything sequentially, you deploy a team of specialized agents that work simultaneously — a scout researches the codebase, a builder writes the code, a reviewer checks the work — all at the same time, all coordinated automatically.

## How It Works (30-Second Version)

```
You (in Claude Code)
  │
  │  "ov sling task-42 --capability lead --name frontend-lead"
  │
  ▼
Team Lead Agent (manages the work)
  ├── Scout Agent (reads code, reports findings)
  ├── Builder Agent (writes code in isolated branch)
  ├── Builder Agent (writes code in isolated branch)
  └── Reviewer Agent (validates the work)
          │
          ▼
    Merge back into your branch
```

Your Claude Code session IS the orchestrator. You type commands. Agents spin up in tmux sessions, each in their own git worktree (an isolated copy of the repo). They communicate through a SQLite-based mail system. When they're done, you merge everything back.

## What Can It Do?

**Parallel Development**
Multiple agents write code at the same time, each in their own branch. No stepping on each other's toes.

**Automatic Code Merging**
A 4-tier conflict resolution system handles merging — from clean fast-forwards to AI-assisted conflict resolution.

**Agent Specialization**
Seven agent types, each with a specific role: scouts explore, builders code, reviewers validate, leads manage teams, mergers integrate branches, coordinators run the show, monitors watch for problems.

**Real-Time Monitoring**
A live dashboard, per-agent inspection, unified event feeds, error aggregation, and chronological replay — you always know what every agent is doing.

**Built-In Safety**
Agents are file-scoped (they can only touch what you allow), hierarchy-limited (no runaway spawning), and quality-gated (tests, linting, and type checks before any work is considered done).

**Multi-Runtime Support**
Not locked into one AI. Runtime adapters for Claude Code, Pi, OpenAI Codex, and GitHub Copilot.

**Expertise Learning**
Integrated with Mulch, a structured expertise system. Agents learn conventions and patterns from each session and carry them forward.

**Cost Tracking**
Token usage and cost breakdowns per agent, per task, per run — so you always know what you're spending.

## Key Numbers

| Metric | Value |
|--------|-------|
| CLI commands | 32+ |
| Agent types | 7 (scout, builder, reviewer, merger, lead, coordinator, monitor) |
| Merge tiers | 4 (fast-forward → AI-assisted) |
| SQLite databases | 5 (mail, sessions, events, metrics, merge queue) |
| Messaging latency | <5ms per query |
| Runtime adapters | 4 (Claude Code, Pi, Codex, Copilot) |

## Who Is It For?

**Solo developers who want a team.** You're one person, but Overstory lets you deploy multiple AI agents that work in parallel — like having a squad of developers at your command.

**Teams who want superpowers.** Already have human developers? Layer Overstory on top to let each person orchestrate their own fleet of AI agents, multiplying output without multiplying headcount.

**Anyone tired of waiting.** If you've ever stared at a single AI agent working through a task and thought "this could be faster" — it can be. Dramatically.
