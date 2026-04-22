---
name: doc-harness
description: "Document-based project control that lets any AI agent or human resume work from files alone — no external memory needed. Use this skill whenever the user wants to structure a long-running project, track progress across sessions, recover state after context loss, coordinate multiple agents on the same project, audit project documentation health, or stop forgetting what was done last session. Triggers include: 'help me set up this project', 'I keep losing track', 'my agent forgets between sessions', 'organize my project docs', 'audit this project', 'check the documentation', 'what did we do last time', 'update the status docs', 'sync the project state', 'save everything before context compresses', 'flush context', 'prepare for compact', 'register this new file', 'init doc-harness', 'recall', 'remember', 'find in docs', 'search project docs', 'where did we discuss', 'why did we decide', 'what is the current plan', 'summarize what we know about'; multi-week projects (theses, research, analyses, software modules) that span many sessions; cross-project coordination (inbox/outbox for file-based messages between projects); requests to create CLAUDE.md, CURRENT_STATUS.md, FILE_INDEX.md, WORKLOG.md, or DOC_HARNESS_SPEC.md."
---

# Doc Harness — Document-Based Project Control

Doc Harness creates and maintains five status documents per project that enable any agent or human to understand and resume project work purely from reading files — no external memory needed.

**Core principle**: "Write It Down or Lose It" — all information in context is temporary. If it is not written to a file and registered in FILE_INDEX, it will be lost.

---

## The Five Documents

| Document | Role | Answers |
|----------|------|---------|
| **CLAUDE.md** | Project entry point | "What is this project? What are the rules? Where do I start?" |
| **CURRENT_STATUS.md** | Active state | "What just happened? What's happening now? What's next?" |
| **FILE_INDEX.md** | File catalog | "What files exist, what are they, and where are they?" |
| **WORKLOG.md** | Permanent history | "What exactly happened in phase 2, three weeks ago?" |
| **DOC_HARNESS_SPEC.md** | Complete reference | Full specification for unusual situations |

**Metaphor**: A project in motion is like a **moving car**.
- **Tire Tracks** (Recent Completed) — the road behind
- **Car Body** (Current Work) — where you are right now
- **Headlights** (Next Steps) — where you're going
- **Driving Manual** (Working Principles) — rules for this road

---

## Natural-Language Commands

Kimi has no slash commands. Use the table below to map user requests to the correct procedure.

| User says | Procedure | See |
|-----------|-----------|-----|
| "Set up doc-harness" / "init project docs" / "create status documentation" | **Init** — create the five documents for a new or existing project | [references/init.md](references/init.md) |
| "Check documentation health" / "audit project docs" / "run a health check" | **Check** — audit file health and reflect on principles | [references/check.md](references/check.md) |
| "Sync project state" / "update status docs" / "catch up documentation" | **Sync** — repair drift, refresh dates, register missing files | [references/sync.md](references/sync.md) |
| "Save everything" / "flush context" / "prepare for compact" / "ensure nothing is lost" | **Flush** — emergency save: extract all important context into documents | [references/flush.md](references/flush.md) |
| "Recall" / "remember" / "find in docs" / "search project docs" / "where did we discuss" / "why did we decide" / "what is the current plan" | **Recall** — retrieve information from registered documents along the Doc Harness hierarchy | [references/recall.md](references/recall.md) |

---

## Daily Workflow

### Session Start
1. Read project's **CLAUDE.md** → understand overview, iron rules, recovery chain
2. Read **CURRENT_STATUS.md** → tire tracks (history), car body (current state), headlights (next steps), driving manual (principles)
3. If user is present → confirm whether next steps have changed
4. If context was compressed → resume from car body per headlights

### During Work
- Complete a **meaningful step** → update CURRENT_STATUS car body
- Create a new file → **do two things**: (1) register in FILE_INDEX (2) record in car body
- Important analysis / decision / insight → **write to file immediately** and register
- **Watch remaining context** (if runtime exposes it). When low (~<20%), update CURRENT_STATUS before the next tool call. If car body holds substantial unsaved work (≥3 steps / ≥50 lines / ≥100 lines), consider a phase transition now — compression is involuntary session end.

### Session End
1. Ensure car body reflects all work from this session
2. Update headlights "next steps"
3. Refresh CURRENT_STATUS "last updated" date
4. Refresh CLAUDE.md "one-line status (as of date)"
5. Check: all new files registered in FILE_INDEX?
6. Check: any important information only in context? → Write it down!
7. If phase transition occurred: WORKLOG under ~1000 lines? If over, archive per spec §5.5.

---

## Phase Transition (Five Steps)

**Trigger**: Work goal fundamentally changes; user explicitly declares a new phase; project pauses (>3 days); car body exceeds ~200 lines.

**Five steps (strict order)**:
1. **Protect data**: Copy CURRENT_STATUS car body in full to WORKLOG top (after TOC). Add phase summary. → After this step, even if interrupted, no data is lost.
2. Insert new summary (3-5 lines) at top of CURRENT_STATUS tire tracks.
3. If tire tracks exceed 3 → remove the oldest (already in WORKLOG).
4. Clear car body → fill in new phase goal. Review driving manual principles (promote to iron rule / keep / remove).
5. Update CLAUDE.md "current phase" and "one-line status". Update WORKLOG TOC.

**Interrupt safety**: Step 1 is protective. If compact occurs during Steps 2-5, data is safe. On next session start, run mid-transition detection (three-way coherence check: CLAUDE.md phase / WORKLOG TOC / CURRENT_STATUS tire tracks) and repair deterministically.

---

## Agent Behavior Rules

### Iron Rules vs Driving Manual
| Type | Location | Lifecycle |
|------|----------|-----------|
| **Iron rules** | CLAUDE.md | Permanent, phase-independent |
| **Driving manual** | CURRENT_STATUS | Lives and dies with the phase |

At phase transition, review each driving-manual principle: promote to iron rule (if ≥3 phases)? Keep? Remove?

### Single Source of Truth
| Fact Type | Sole Source |
|-----------|------------|
| Current state / progress | CURRENT_STATUS.md |
| File catalog | FILE_INDEX.md |
| Historical details | WORKLOG.md |
| Project overview / rules | CLAUDE.md |

### Anti-patterns
- File created but not registered → register immediately
- Information only in context → "Write It Down"
- CURRENT_STATUS grows indefinitely → transfer to WORKLOG at phase transitions
- WORKLOG grows past ~1000 lines untouched → archive per spec §5.5
- Phase principles become permanent legacy → review at transition

---

## Optional Extensions

- **PARKING_LOT.md** — Deferred items with preconditions for revival. Create only when needed.
- **PHILOSOPHY.md** — Principles forged by project practice. Create only when insights generalize.
- **inbox/outbox** — File-based cross-project messaging. Adopt only if project has dependencies.

See [references/spec.md](references/spec.md) for full details.

---

## Reference Documents

| File | When to read |
|------|-------------|
| [references/init.md](references/init.md) | User asks to set up / initialize doc-harness |
| [references/check.md](references/check.md) | User asks to audit / check documentation health |
| [references/sync.md](references/sync.md) | User asks to sync / update status docs |
| [references/flush.md](references/flush.md) | User asks to save / flush context before compression |
| [references/recall.md](references/recall.md) | User asks to recall / search / find information in project docs |
| [references/spec.md](references/spec.md) | Unusual situation, edge case, or understanding design rationale |

**Language note**: All documents are written in English with Chinese annotations where helpful. Kimi should respond in the user's language regardless of document language.
