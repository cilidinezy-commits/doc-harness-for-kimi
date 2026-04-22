# Doc Harness — Check (Kimi CLI Version)

Audit the project's documentation health and reflect on working principles.

**Trigger**: User says something like "check the project docs", "audit documentation health", "run a health check", "reflect on working principles", "check documentation".

**Purpose**: Two purposes — (1) catch file-level problems, (2) remind yourself of the rules you should be following.

---

## Part 1: File Health Audit

### 1.1 Core files exist?

Look for these 4 core files:
- `CLAUDE.md`, `CURRENT_STATUS.md`, `FILE_INDEX.md`, `WORKLOG.md`

Missing → `❌ MISSING: [file]`
None found → `❌ No Doc Harness found. Suggest init.` → stop.

Also check `DOC_HARNESS_SPEC.md`:
- Present → `✅ Spec present`
- Missing → `⚠️ DOC_HARNESS_SPEC.md not found — recommended but not required`

### 1.2 CURRENT_STATUS freshness

Read "Last updated" date (first 5 lines only).
- Today → `✅ Fresh`
- 1-3 days → `⚠️ [N] days old`
- >3 days → `❌ Stale — update or execute pause protocol`

### 1.3 CLAUDE.md one-line status

Read "One-line status (as of DATE)" (first 5 lines only).
- Date matches CURRENT_STATUS → `✅ Current`
- Older → `⚠️ Stale`

### 1.4 FILE_INDEX completeness

Compare FILE_INDEX entries against actual files on disk. Use recursive check with sub-index prune (same algorithm as spec §1.4).

- All files registered → `✅ Complete`
- Unregistered files → `❌ UNREGISTERED: [list]`
- Ghost entries → `❌ GHOST: [list]`

### 1.5 WORKLOG TOC consistency

Compare TOC table rows with actual `##` headings.
- Match → `✅ Consistent`
- Mismatch → `⚠️ TOC has [N] entries, [M] sections found`

### 1.6 WORKLOG length

Count total lines.
- <1000 → `✅ [N] lines`
- 1000–1500 → `⚠️ [N] lines — consider archival`
- >1500 → `❌ [N] lines — archival overdue`

### 1.7 Inbox status (if adopted)

If `inbox/` exists, do four sub-checks:

**(a) Unread count** (direct children only, exclude `_archive/` and `_malformed/`):
- All read/actioned → `✅ Inbox clean`
- Any unread → `⚠️ Inbox has N unread message(s)`

**(b) Archival trigger**: actioned messages older than 30 days
- ≥5 → `⚠️ Archival due`

**(c) Malformed messages**: count in `inbox/_malformed/`
- ≥1 → `⚠️ Malformed messages quarantined`

**(d) Recent outbox**: mtime within 3 days, verify CURRENT_STATUS mentions them

### 1.8 Car body length

Count lines between car-body heading and next `##`.
- <200 → `✅ [N] lines`
- 200–250 → `⚠️ Approaching limit`
- >250 → `❌ Over limit — consider phase transition`

### 1.9 Mid-transition coherence

Read three values:
- **A**: CLAUDE.md "current phase"
- **B**: WORKLOG TOC newest entry
- **C**: CURRENT_STATUS tire tracks newest phase summary

Compare per spec §6.3.1 coherence table.
- Coherent → `✅ Coherent`
- Inconsistent → `❌ Mid-transition detected — apply repair`

### 1.10 Embedded operational-rules version

Grep for `<!-- doc-harness-ops-version: N.N -->` in CLAUDE.md.
- Matches installed skill version → `✅ Up to date`
- Older → `⚠️ Stale — re-embed operational_rules.md`
- Missing → `⚠️ No version marker found`

---

## Part 2: Principle Reflection

### 2.1 Iron rules

Read CLAUDE.md Iron Rules section. For each rule:
```
🔒 "[rule text]"
   → Have I followed this? Any violations?
```

### 2.2 Driving manual

Read CURRENT_STATUS Driving Manual section. For each principle:
```
📋 "[principle text]"
   → Am I keeping this in mind?
```

### 2.3 "Write It Down" check

```
📝 Write It Down check:
   → Any important information ONLY in context right now?
   → Unsaved analysis results, decisions, insights?
   → If yes → save NOW. If context compression imminent → run flush.
```

### 2.4 Phase coherence

List all `####` step titles from car body. Ask:
```
🔄 Phase coherence:
   Steps in car body:
   1. [step title 1]
   2. [step title 2]
   ...
   → Do all these steps serve the SAME phase goal?
   → Or do some represent a different objective that should be a new phase?
```

### 2.5 Recovery Chain health

```
🗺️ Recovery Chain:
   → Two-layer (must-read + task-conditional)?
   → Must-read ≤ 3 entries?
   → All entries point to project-internal files?
   → Any entry consistently read every session? → promote.
   → Any must-read rarely needed? → demote.
```

### 2.6 Session-end checklist

```
- [ ] Car body reflects all session work?
- [ ] Headlights accurate?
- [ ] CURRENT_STATUS date refreshed?
- [ ] CLAUDE.md one-line status refreshed?
- [ ] New files registered in FILE_INDEX?
- [ ] Anything unsaved? → Write it down!
```

---

## Output Format

```
═══════════════════════════════════════
  Doc Harness — Health Check
  Project: [name]
  Date: [today]
═══════════════════════════════════════

── Part 1: File Health ──
[1.1] Core files:      ✅/❌
[1.2] CURRENT_STATUS:  ✅/⚠️/❌
[1.3] CLAUDE.md:       ✅/⚠️
[1.4] FILE_INDEX:      ✅/❌
[1.5] WORKLOG TOC:     ✅/⚠️
[1.6] WORKLOG length:  ✅/⚠️/❌
[1.7] Inbox:           ✅/⚠️
[1.8] Car body:        ✅/⚠️/❌
[1.9] Mid-transition:  ✅/❌
[1.10] Ops version:    ✅/⚠️

── Part 2: Principle Reflection ──
🔒 Iron Rules: [reflections]
📋 Driving Manual: [reflections]
📝 Write It Down: [status]
🔄 Phase coherence: [assessment]
🗺️ Recovery Chain: [health check]

── Actions Needed ──
[specific fixes for ❌/⚠️ items]
═══════════════════════════════════════
```

## When to Use

- Every 1-2 hours during long sessions
- Before session end
- After context compact recovery
- When feeling drift from principles
- When user requests
- Before sync or flush (to understand the full picture before repairing)
