# Brain System — Persistent Memory

The brain is a set of markdown files in `/memories/` (VS Code Copilot's memory system) that persist across sessions and survive context compaction. This is the institutional knowledge layer.

## Why This Exists

AI chat sessions lose context when:
- The conversation gets too long (context window limit)
- VS Code compresses the conversation (compaction)
- A new session starts

The brain solves this by storing critical information in files that the AI reads at every session start and updates during work.

## Architecture

```
/memories/
├── active-context.md     ← What's happening RIGHT NOW
├── lessons-learned.md    ← What works, what doesn't
├── dead-ends.md          ← Failed approaches to NEVER repeat
├── tech-decisions.md     ← Architecture decisions with rationale
└── progress.md           ← Running accomplishment log (append-only)
```

## Rules (Non-Negotiable)

1. **Read at session start** — Always read all brain files before acting
2. **Update in the moment** — Don't wait until session end. If you learn something, write it NOW
3. **Never delete** — Only add or refine entries. History matters.
4. **Keep files short** — Under 100 lines each. Prune stale entries, keep actionable content
5. **After compaction** — Read CLAUDE.md first (restore anchor), then brain files

## Session Protocol

### Start
1. Read `CLAUDE.md` (compaction survival anchor)
2. Read all `/memories/` files
3. Read `HANDOFF.md` (previous session state)
4. THINK — what was the trajectory? What should happen next?

### During
- Discovery → update `lessons-learned.md`
- Failure → update `dead-ends.md`
- Decision → update `tech-decisions.md`
- Context switch → update `active-context.md`
- Task complete → append to `progress.md`

### End
1. Self-grade (Accuracy, Completeness, Autonomy, Memory, Honesty, Resilience)
2. Update all brain files with session learnings
3. Write `HANDOFF.md` with state, unfinished work, next steps, insights

## Templates

See `templates/` for starter content for each brain file. Copy these into your `/memories/` system after the framework is built.

## CLAUDE.md — The Restore Anchor

`CLAUDE.md` at the project root is a special file that survives compaction. It contains:
- Critical constraints that must NEVER be violated
- Current system capabilities
- Brain file inventory
- Who the user is and what they value

Keep it under 50 lines. It's the first thing read after context loss.

## HANDOFF.md — Session Continuity

`HANDOFF.md` in the project root captures:
- Current state (what's done, what's in progress)
- Unfinished work with next steps
- Failed approaches (with `do_not_retry` flags)
- Unacted insights — things noticed but not pursued
- Recommendations for the next session
