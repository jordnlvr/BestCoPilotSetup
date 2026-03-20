# Architecture — How the System Works

This document explains every component of the Copilot agent framework, how they connect, and why each piece exists.

---

## System Overview

```
┌──────────────────────────────────────────────────────────────┐
│                    VS Code + GitHub Copilot                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  ┌───────────┐  │
│  │ Chat     │  │ Agent    │  │ Skill     │  │ Prompt    │  │
│  │ Modes(7) │  │ Files(9) │  │ Files(7)  │  │ Files(9)  │  │
│  └────┬─────┘  └────┬─────┘  └─────┬─────┘  └─────┬─────┘  │
│       │              │              │              │         │
│       ▼              ▼              ▼              ▼         │
│  ┌──────────────────────────────────────────────────────┐   │
│  │          Copilot Instructions (Project Brain)         │   │
│  │    .github/copilot-instructions.md + AGENTS.md        │   │
│  └───────────────────────┬──────────────────────────────┘   │
│                          │                                   │
│  ┌───────────────────────┼──────────────────────────────┐   │
│  │      Instruction Files (9) — Scoped by File Type      │   │
│  │  typescript.instructions.md  react.instructions.md    │   │
│  │  python.instructions.md      docker.instructions.md   │   │
│  │  testing.instructions.md     workspace-safety          │   │
│  │  model-routing               brain.instructions.md    │   │
│  │  self-improvement.instructions.md                     │   │
│  └───────────────────────┬──────────────────────────────┘   │
│                          │                                   │
│  ┌───────────────────────┼──────────────────────────────┐   │
│  │              MCP Servers (10) — External Intelligence  │   │
│  │  Tavily · Context7 · Playwright · PostgreSQL          │   │
│  │  Sequential Thinking · GitHub · Memory · Filesystem   │   │
│  │  Git · Codebase Memory                                │   │
│  └───────────────────────┬──────────────────────────────┘   │
│                          │                                   │
│  ┌───────────────────────┼──────────────────────────────┐   │
│  │        Brain System — Persistent Memory (/memories/)   │   │
│  │  active-context · lessons-learned · dead-ends          │   │
│  │  tech-decisions · progress                             │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Root Files — Session Continuity              │   │
│  │  CLAUDE.md (compaction anchor) · HANDOFF.md           │   │
│  │  AGENTS.md (always-on rules)                          │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Hooks — Automated Quality Gates              │   │
│  │  afterFileEdit: JSON, TypeScript, Python validation    │   │
│  │  afterSessionStart: Session audit logging              │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Component Deep Dive

### 1. Chat Modes (7 files in `.github/chatmodes/`)

Chat modes define **how Copilot behaves** for different types of work. Each mode specifies:
- **Model**: Which LLM to use (controls cost and capability)
- **Tools**: Which tools are available (read-only modes restrict editing)
- **Agents**: Which specialist agents can be invoked as subagents
- **Handoffs**: Buttons that appear after responses to chain to other modes/agents

| Mode | Model | Cost | Purpose | Restriction |
|------|-------|------|---------|-------------|
| Quick | GPT-5 mini | FREE (0x) | Simple tasks, boilerplate | None |
| Planning | GPT-5 mini | FREE (0x) | Task breakdown, estimation | No implementation |
| Build | Claude Sonnet 4.6 | 1x | Full autonomous development | None |
| Research | Claude Sonnet 4.6 | 1x | Deep investigation | Read-only |
| Review | Claude Sonnet 4.6 | 1x | Code analysis | Read-only |
| Architect | Claude Opus 4.6 | 3x | System design | Design docs only |
| Deep | Claude Opus 4.6 | 3x | Hardest problems | None |

**Why 3 tiers?** Cost optimization. You get ~1500 premium requests/month on Pro+. Using GPT-5 mini for simple stuff (free) and Sonnet for real work (1x) leaves budget for Opus (3x) on hard problems. This strategy uses ~650 of 1500 monthly requests even with heavy use.

**The Struggle Rule**: If the current model can't handle the task (repeated failures, going in circles), STOP and escalate to a more powerful mode. Two strikes = switch models.

### 2. Specialist Agents (9 files in `.github/agents/`)

Agents are **experts with specific domains**. Each agent has:
- **Tools**: Only the tools relevant to their domain
- **Model**: Matched to the complexity of their work
- **Handoffs**: Connections to other agents for workflow chaining

| Agent | Model | Domain | Edits Files? |
|-------|-------|--------|:---:|
| architect | Opus 4.6 | System design, ADRs, API design | No |
| security-auditor | Opus 4.6 | OWASP, threat modeling, vulnerability assessment | No |
| researcher | Sonnet 4.6 | Web research, documentation, tech evaluation | No |
| reviewer | Sonnet 4.6 | Code review, quality + security analysis | No |
| debugger | Sonnet 4.6 | Root cause diagnosis, evidence-based fixing | Yes |
| devops | Sonnet 4.6 | Docker, CI/CD, deployment, infrastructure | Yes |
| db-expert | Sonnet 4.6 | PostgreSQL schema, queries, optimization | Yes |
| performance | Sonnet 4.6 | Profiling, optimization, load testing | Yes |
| scout | GPT-5 mini | Fast file search, codebase exploration | No |

**Key design**: High-stakes agents (architect, security) use Opus for deep reasoning. Read-only agents can't accidentally modify files. The scout uses the cheapest model because it just searches.

### 3. Workflow Pipelines (via Handoffs)

Agents connect to each other through **handoff buttons** that appear after every response. This creates workflow pipelines:

```
Feature Development:
  Planning → Architect → Build → Review → Security → Performance

Bug Fix:
  Scout → Debugger → Build → Review

Research to Implementation:
  Research → Architect → Build → Review

Quality Hardening:
  Review → Security → Performance → Build (fix findings)
```

Handoffs pass context from one agent to the next via the `prompt` field. The user clicks a button and the conversation transfers with full context.

### 4. Skills (7 directories in `.github/skills/`)

Skills are **reusable multi-step procedures** that agents and modes can invoke. They define HOW to do complex tasks.

| Skill | Steps | Purpose |
|-------|-------|---------|
| web-research | 6 tiers | Exhaustive multi-source research protocol |
| code-audit | 5 phases | Full security + quality + performance audit |
| self-test | 5 checks | Pre-completion verification (compile, lint, test) |
| session-handoff | 3 phases | Session continuity (start → during → end) |
| feature-planning | 5 phases | Requirements → Architecture → Tasks → Risks → Tests |
| full-pipeline | 7 stages | Research → Architect → Build → Review → Security → Performance → Harden |
| quick-fix | 3 stages | Diagnose → Fix → Verify |

### 5. Prompt Templates (9 files in `.github/prompts/`)

One-click prompt files users can invoke from the Copilot prompt menu. Each triggers a specific protocol.

| Prompt | Triggers |
|--------|----------|
| beast-mode | 6-tier exhaustive research on any topic |
| debug-error | Structured error diagnosis protocol |
| optimize | Performance analysis and optimization |
| plan-feature | Architecture + task breakdown for a feature |
| review-code | Comprehensive code review (security, quality, performance) |
| security-scan | Full OWASP security audit |
| self-grade | 6-criterion self-evaluation (Accuracy, Completeness, Autonomy, Memory, Honesty, Resilience) |
| session-start | Brain boot protocol — load memory, restore context, think before acting |
| session-end | Brain persistence — save state, write handoff, run self-grade |

### 6. Scoped Instruction Files (9 files in `.github/instructions/`)

Instructions that activate **only when specific file types are being edited**. The `applyTo` frontmatter controls targeting.

| Instruction | Applies To | Key Rules |
|-------------|-----------|-----------|
| typescript | `*.ts, *.tsx, *.js, *.jsx` | Strict types, no `any`, async/await, error handling |
| react | `*.tsx, *.jsx` | Functional components, hooks rules, accessibility |
| python | `*.py` | Type hints, PEP 8, security (no eval/exec with user input) |
| testing | `*.test.*, *.spec.*` | AAA pattern, behavior testing, mock rules |
| docker | `Dockerfile, docker-compose*` | Multi-stage builds, health checks, port rules |
| workspace-safety | `**/*` (always) | Port allocation, file restrictions, git safety |
| model-routing | `**/*` (always) | 3-tier cost optimization, struggle rule |
| brain | `**/*` (always) | Memory architecture, compaction survival, brain rules |
| self-improvement | `**/*` (always) | Proactive learning, 3-strike enforcement |

### 7. MCP Servers (10 configured)

Model Context Protocol servers give Copilot access to **external tools and data sources**.

| Server | Type | What It Does |
|--------|------|-------------|
| **Tavily** | HTTP | AI-powered web search, content extraction, site crawling, domain mapping |
| **Context7** | npx | Up-to-date library documentation for any npm/Python package |
| **Playwright** | npx | Browser automation — navigate, click, screenshot, test web UIs |
| **PostgreSQL** | npx | Direct database queries from Copilot chat |
| **Sequential Thinking** | npx | Extended reasoning chains for complex problems |
| **GitHub** | npx | Repository management, issue/PR operations, code search |
| **Memory** | npx | Persistent knowledge graph across sessions |
| **Filesystem** | npx | Direct filesystem access for the project directory |
| **Git** | npx | Git operations (commits, branches, diffs) |
| **Codebase Memory** | binary | Semantic code indexing and retrieval |

### 8. The Brain System (`/memories/` + root files)

The brain is a **persistent memory system** that survives across sessions, compaction events, and context window limits.

```
/memories/
├── active-context.md    ← What's happening RIGHT NOW
├── lessons-learned.md   ← Patterns that work/don't
├── dead-ends.md         ← Failed approaches (NEVER repeat)
├── tech-decisions.md    ← Architecture decisions + rationale
└── progress.md          ← Running accomplishment log

Root Files:
├── CLAUDE.md            ← Compaction survival anchor (read FIRST after context loss)
├── HANDOFF.md           ← Session-to-session state transfer
└── AGENTS.md            ← Always-on behavioral instructions
```

**How it works:**
1. **Session start**: AI reads all brain files → knows who the user is, what happened before, what to avoid
2. **During session**: AI updates brain files IN THE MOMENT when discoveries or decisions happen
3. **Session end**: AI updates all brain files, writes HANDOFF.md, runs self-grade

**Compaction survival**: When the context window fills up and conversation history is compressed, the AI reads `CLAUDE.md` first to restore critical constraints, then reads brain files to restore full context. This prevents the "amnesia" problem.

### 9. Hooks (`.github/hooks/hooks.json`)

Automated quality gates that run after specific events:

| Hook | Trigger | Action |
|------|---------|--------|
| JSON validation | After editing a `.json` file | Parse and validate syntax |
| TypeScript check | After editing a `.ts/.tsx` file | Run `tsc --noEmit` |
| Python check | After editing a `.py` file | Parse with `ast.parse()` |
| Session log | After session start | Log timestamp for continuity tracking |

### 10. Self-Grading Protocol

After every significant task, the AI grades itself on 6 criteria:

1. **Accuracy** — Was the output correct? Any errors or hallucinations?
2. **Completeness** — Was the full task addressed? Edge cases considered?
3. **Autonomy** — Did it execute without unnecessary questions?
4. **Memory** — Did it read brain files? Update them with discoveries?
5. **Honesty** — Did it admit gaps? Only declare done when verified?
6. **Resilience** — When blocked, did it find alternatives? Escalate?

Grades: A+ through F. **If any criterion falls below A, the AI fixes the gap before reporting completion.**

---

## Why This Design?

### Problem: AI is stateless
**Solution**: Brain system with persistent files + session start/end protocols

### Problem: AI is generalist
**Solution**: 9 specialist agents with domain-specific tools + knowledge

### Problem: AI forgets failures
**Solution**: dead-ends.md — checked before every new approach

### Problem: AI costs money
**Solution**: 3-tier model routing — free/standard/heavy matched to task complexity

### Problem: AI has no external knowledge
**Solution**: 10 MCP servers providing web search, docs, databases, browser, GitHub, filesystem

### Problem: Quality varies wildly
**Solution**: Self-grading protocol + hooks + reviewer agent + code review checklist

### Problem: Work doesn't chain well
**Solution**: Handoff buttons connecting agents into workflow pipelines
