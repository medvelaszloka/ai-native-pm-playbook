# Agentic Coding Playbook

> Personal playbook for solo product-driven development using Claude Code.
> Inspired by Daniel Roth's Bob & Ray system, extended with planning, tiered models, and decision boundaries.
>
> Author: Bence Damjanovics | Role: Senior PM
> Last updated: 2026-03-18

---

## 1. Philosophy

### The Blue Room Principle

I decide WHAT gets built and HOW it feels. The agents decide HOW to build it.

If I want a blue room, the agents build a blue room. They don't argue for green.
But whether they use Blue Paint from Producer X or Producer Y, I don't care. I care about the outcome.

**My role:** Architect, product manager, picky customer.
**Agents' role:** The crew that builds to my spec, using their best technical judgment.

### Core beliefs

- I am not a project manager shuffling tickets. I am the person who lives in the house.
- Agents should resolve technical decisions autonomously. They escalate product/UX decisions to me.
- One tool, one context, tiered models. Simplicity over theoretical optimization.
- Planning is cheap. Bad architecture is expensive. Always plan before building.

---

## 2. Decision Boundary Framework

Every decision during development falls into one of three tiers:

| Tier | Type | Examples | Who decides |
|------|------|----------|-------------|
| **Business/UX** | What & Why | Feature scope, user flows, visual direction, what to cut, priorities, how something should feel | **Me, always** |
| **Architecture** | Structural How | Data model shape, API design, state management approach, new dependencies, major refactors | **Agents propose, I approve** |
| **Implementation** | Tactical How | Which API to call, variable names, file structure, algorithms, library choices, code patterns | **Agents decide autonomously** |

### The rule baked into every agent:

> If a decision touches WHAT the user experiences or WHY we're doing something, then stop and ask me.
> If it's about HOW to implement an already-approved direction, then just do it.

---

## 3. Agent Roles

Five roles, not five separate tools. All run inside Claude Code.

### Planner (Opus)

- Runs once per feature, before any code is written
- Receives my feature description (the WHAT and WHY)
- Outputs:
  - Implementation approach
  - Business/UX decisions that need my input (with options)
  - Architectural choices (proposed, with rationale)
  - Implementation details (just informs, doesn't ask)
  - Risks and edge cases
- **I review and approve the plan before anything gets built**

### Builder (Sonnet)

- Does the heavy lifting, writes all the code
- Follows the approved plan strictly
- Makes all implementation-tier decisions autonomously
- Runs tests after building
- If it hits a business/UX or architecture question not covered by the plan, STOPS and asks me
- Works on a feature branch, always

### Reviewer (Sonnet)

- Reviews the git diff (not the whole codebase)
- Checks against:
  - The approved plan (did we build what we said?)
  - Architecture rules (from ARCHITECTURE.md)
  - Security checklist (hardcoded in agent prompt)
  - Performance red flags
  - Error handling completeness
- Outputs one of:
  - Ship it
  - Fix list (goes back to Builder automatically)
  - Concern for me (blocks until I respond). Only for business/UX issues

### Docs-Keeper (Haiku)

- Runs after merge, mechanical work
- Updates:
  - CHANGELOG.md -> what changed
  - ARCHITECTURE.md -> any new patterns introduced
  - DECISIONS.md -> why this approach was chosen
  - KNOWN_ISSUES.md -> anything deferred
- Keeps context files alive so they don't drift

### Retro (Opus)

- Runs manually — every ~5 merged features, monthly, or when something feels off
- Analyzes:
  - Recent git history across active projects
  - Reviewer patterns (recurring issues, false alarms)
  - Decision boundary friction (too tight or too loose)
  - Workflow bottlenecks (planning overhead, skipped steps)
- Outputs proposed changes to agent definitions and playbook
- **Rule:** Only flags patterns (3+ occurrences), never one-offs. Proposes, never auto-applies.

---

## 4. Workflow

For every feature:

```
1. ME          → Describe the feature (WHAT + WHY + any UX opinions)
                 ↓
2. PLANNER     → Proposes implementation plan
                 Surfaces decisions by tier (business / arch / impl)
                 ↓
3. ME          → Review plan (30-60 seconds)
                 Approve, adjust, or reject
                 "Yes but make it blue, not green"
                 "I don't care about the paint brand, go ahead"
                 ↓
4. BUILDER     → Creates feature branch
                 Implements the approved plan
                 Runs tests
                 ↓
5. REVIEWER    → Reviews the diff against plan + rules
                 Outputs: ship it / fix list / escalate to me
                 ↓
6. BUILDER     → Fixes if needed → back to step 5
    (loop)
                 ↓
7. ME          → Final sanity check, merge
                 ↓
8. DOCS-KEEPER → Updates context files
                 ↓
9. RETRO       → (Periodic) Analyzes patterns, proposes agent improvements
```

### When to break the workflow:

- Tiny bug fix or copy change → skip Planner, go straight to Builder
- Exploratory prototype → skip Reviewer, just build and throw away
- Major pivot or new project → spend extra time on CLAUDE.md and ARCHITECTURE.md first

---

## 5. Cost Strategy

### Single tool, tiered models

| Agent | Model | Why |
|-------|-------|-----|
| Planner | Opus | Needs deep reasoning for architectural tradeoffs. Low volume (runs once per feature). |
| Builder | Sonnet | Bulk of the work. Well-defined tasks. ~5x cheaper than Opus. |
| Reviewer | Sonnet | Needs good judgment but scoped to a diff. Moderate cost, high value. |
| Docs-Keeper | Haiku | Mechanical updates. ~60x cheaper than Opus. |
| Retro | Opus | Needs deep reasoning to spot patterns across reviews. Low volume (runs manually, ~monthly). |

### Why not multi-tool (Claude + Cursor + Codex)?

- Context loss between tools (re-explaining the project every time)
- Multiple subscriptions stack up ($200-400+/month)
- Cognitive overhead of managing tools instead of building product
- Agents across tools share nothing; agents within Claude Code share CLAUDE.md

### Subscription approach

- Claude Code Max subscription for predictable monthly cost
- Model tiering preserves Opus/Sonnet allowance for high-value work
- Haiku for mechanical tasks so I don't burn premium quota

---

## 6. Agent Setup: Global + Project Overrides

Agents are defined GLOBALLY and available in every project. Project-specific context comes from CLAUDE.md, not from redefining agents.

### Global agents (already set up)

Source of truth lives in this repo:

```
/projects/ai-native-pm-playbook/
├── agentic-playbook.md               ← this file (your strategy doc)
├── agents/
│   ├── planner.md                    ← Opus  — plans before code is written
│   ├── builder.md                    ← Sonnet — implements the approved plan
│   ├── reviewer.md                   ← Sonnet — reviews diff against plan + rules
│   ├── docs-keeper.md               ← Haiku  — updates context files post-merge
│   └── retro.md                     ← Opus   — analyzes patterns, proposes agent improvements
└── templates/                        ← starter files for new projects
    ├── CLAUDE.md.template
    ├── ARCHITECTURE.md.template
    └── DECISIONS.md.template

~/.claude/agents/ → symlink → /projects/ai-native-pm-playbook/agents/
```

These define the ROLE, BEHAVIOR, and DECISION BOUNDARIES for each agent. They are universal, they don't change between projects.

### Per-project setup (when starting a new project)

```
project-root/
├── CLAUDE.md                     ← Project-specific context + conventions
├── VISION.md                     ← Product vision, target users, current focus
├── ARCHITECTURE.md               ← Patterns, module boundaries, dependencies
├── DECISIONS.md                  ← Why we chose X over Y (living doc)
├── CHANGELOG.md                  ← What changed and when
├── KNOWN_ISSUES.md               ← Deferred problems, tech debt
```

No agent files needed per project. The global agents read the project's CLAUDE.md automatically and adapt.

### Per-project agent OVERRIDES (only when needed)

If a specific project has special requirements, create a project-level agent with the same name. It replaces the global one for that project only.

```
project-root/
├── .claude/
│   └── agents/
│       └── reviewer.md           ← OVERRIDES global reviewer
│                                    e.g., adds HIPAA checks for a health app
│                                    or WCAG compliance for an accessibility-critical project
```

**When to override:** Only when a project has domain-specific review needs (healthcare, fintech, accessibility). For 90% of projects, the global agents are enough.

### CLAUDE.md should contain:

- Project description (one paragraph)
- Tech stack and key dependencies
- Directory structure overview
- Naming conventions and code style
- Testing approach
- The decision boundary framework (reference this playbook)
- Link to VISION.md, ARCHITECTURE.md, and DECISIONS.md

---

## 7. System Feedback Loop

The agents review code. But who reviews the agents? You do — periodically.

### When to run a retrospective

- Every ~5 merged features, OR
- Monthly (whichever comes first), OR
- When something feels off (same issues keep appearing, workflow feels slow)

### The retrospective checklist

Run through these questions. Update agents and playbook based on what you find.

#### What patterns did the Reviewer keep flagging?

If the same issue appears in 3+ reviews, it's not a code problem — it's an instruction gap.

→ **Fix:** Add it to the Builder's Code Standards so it stops happening.

#### What did you override or reject?

If you keep overriding agent decisions, the decision boundaries are wrong.

→ **Fix:** Adjust which tier that type of decision falls into (move from agent-decides to agent-proposes, or vice versa).

#### What slowed you down?

Was the planning phase overkill for small features? Did you skip steps? Did context files drift out of date?

→ **Fix:** Simplify the workflow, adjust "when to break the workflow" rules.

#### What surprised you?

Good surprises (agent made a great call you didn't expect) and bad surprises (agent went in a completely wrong direction).

→ **Fix:** Capture good patterns in CLAUDE.md. Add guardrails for bad ones in agent prompts.

#### Are the agents the right shape?

Do you need the docs-keeper, or is it noise? Should the planner run on Sonnet instead of Opus? Is the reviewer too strict or too lenient?

→ **Fix:** Update agent definitions and model assignments.

### How to update

1. Make changes to agent files in `/projects/ai-native-pm-playbook/agents/`
2. Commit with a message explaining WHAT changed and WHY (the git history is your system's evolution story)
3. Push to GitHub — your public playbook evolves visibly over time

### What NOT to change reactively

- Don't rewrite agents after a single bad experience — wait for patterns
- Don't add rules for edge cases that won't recur — keep agents lean
- Don't remove checks because they were annoying once — they probably caught real issues

---

## 8. Open Questions

Things not yet decided — revisit as I gain experience with this system:

- [ ] **Reviewer security depth:** Is a security checklist in the reviewer prompt enough, or do security-critical projects need a dedicated security pass? (Current decision: bake it into reviewer. Reassess if building fintech/health/auth-heavy apps.)
- [ ] **Automated testing:** How much test coverage should the Builder generate? TDD-first or tests-after? Depends on the project and stack.
- [ ] **Agent prompt specifics:** The actual content of each agent's .md file needs to be written per-project. This playbook defines the roles; the prompts need project context.
- [ ] **Orchestration automation:** Currently the workflow is manual (I trigger each step). Could be automated with slash commands. Build this after the manual workflow feels solid.
- [ ] **When to use Opus for Builder:** Some complex features (new architecture, unfamiliar patterns) might benefit from Opus on the Builder. Need a heuristic for when to upgrade.

---

## 9. Origins & References

- Daniel Roth's "Bob the Builder & Ray the Reviewer" system — dual-agent Claude Code workflow for iOS development
  - [Lenny's Newsletter: From journalist to iOS developer](https://www.lennysnewsletter.com/p/from-journalist-to-ios-developer)
  - [How I AI Podcast (Apple Podcasts)](https://podcasts.apple.com/us/podcast/how-i-ai/id1809663079)
  - [YouTube: HbWu_eYIHKQ](https://youtu.be/HbWu_eYIHKQ)
- Extended with: planning phase, decision boundary framework, model tiering, docs-keeper, security-in-reviewer approach
