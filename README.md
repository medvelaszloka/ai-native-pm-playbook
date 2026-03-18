# AI-Native PM Playbook

A system for shipping production apps using AI agents. Designed for product managers and non-engineers who build with Claude Code.

## What is this?

This is a complete methodology for building software with AI coding agents. It includes:

- **5 specialized agents** (Planner, Builder, Reviewer, Docs-Keeper, Retro) with defined roles, decision boundaries, and model assignments
- **A decision boundary framework** that keeps you in control of product and UX decisions while agents handle implementation autonomously
- **Code quality standards** enforced by the agents — no spaghetti code, proper tests, self-documenting codebase
- **Project templates** to bootstrap new projects in minutes

## The Philosophy

> I decide WHAT gets built and HOW it feels. The agents decide HOW to build it.

If I want a blue room, the agents build a blue room. They don't argue for green.
But whether they use paint from Producer X or Producer Y, I don't care. I care about the outcome.

## How It Works

```
YOU         → "Build feature X" (describes WHAT + WHY)
PLANNER     → Proposes implementation plan, surfaces decisions by tier
YOU         → Approve plan (30-60 seconds)
BUILDER     → Implements on a feature branch, runs tests
REVIEWER    → Reviews the diff against plan + architecture + security
BUILDER     → Fixes if needed
YOU         → Final check, merge
DOCS-KEEPER → Updates project context files
RETRO       → Analyzes patterns across reviews, proposes agent improvements
```

### Decision Boundaries

| Tier | Examples | Who decides |
|------|----------|-------------|
| **Business/UX** | Feature scope, user flows, visual direction | You, always |
| **Architecture** | Data models, API design, new dependencies | Agents propose, you approve |
| **Implementation** | Which APIs to use, variable names, patterns | Agents decide autonomously |

## Agent Setup

| Agent | Model | Role |
|-------|-------|------|
| **Planner** | Opus | Plans features before any code is written |
| **Builder** | Sonnet | Implements the approved plan |
| **Reviewer** | Sonnet | Reviews diffs against plan, architecture, and security |
| **Docs-Keeper** | Haiku | Updates context files after merge |
| **Retro** | Opus | Analyzes patterns across reviews, proposes agent improvements |

## Getting Started

### 1. Install agents globally

Copy the `agents/` folder to your Claude Code config:

```bash
cp -r agents/ ~/.claude/agents/
```

Or symlink for easy updates:

```bash
ln -s /path/to/ai-native-pm-playbook/agents ~/.claude/agents
```

### 2. Start a new project

Copy the templates into your project root:

```bash
cp templates/CLAUDE.md.template your-project/CLAUDE.md
cp templates/ARCHITECTURE.md.template your-project/ARCHITECTURE.md
cp templates/DECISIONS.md.template your-project/DECISIONS.md
```

Fill in the `CLAUDE.md` with your project's tech stack and conventions. The agents read this automatically.

### 3. Build

Open Claude Code in your project and start describing features. The agents handle the rest.

## Repository Structure

```
├── README.md                        ← you are here
├── agentic-playbook.md              ← full strategy doc with rationale
├── agents/
│   ├── planner.md                   ← Opus agent definition
│   ├── builder.md                   ← Sonnet agent definition
│   ├── reviewer.md                  ← Sonnet agent definition
│   ├── docs-keeper.md               ← Haiku agent definition
│   └── retro.md                     ← Opus agent definition
└── templates/
    ├── CLAUDE.md.template           ← project context starter
    ├── ARCHITECTURE.md.template     ← architecture doc starter
    └── DECISIONS.md.template        ← decisions log starter
```

## Background

Inspired by [Daniel Roth's dual-agent system](https://www.lennysnewsletter.com/p/from-journalist-to-ios-developer) (Bob the Builder & Ray the Reviewer), extended with a planning phase, decision boundary framework, model tiering for cost optimization, and automated documentation upkeep.

Built by a Senior PM who ships apps without writing code.

## License

MIT
