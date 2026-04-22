# Doc Harness — Sync (Kimi CLI Version)

Synchronize project status documents with reality. Repair drift, refresh stale fields, register missing files, and optionally trigger phase transition or WORKLOG archival.

**Trigger**: User says something like "sync project state", "update status docs", "catch up documentation", "repair documentation drift".

**Difference from check**: `check` is read-only diagnosis; `sync` modifies files to fix drift.

---

## How Kimi Decides: Ask or Auto?

Kimi has no `--interactive` or `--auto` flags. Use conversation context to decide:

- **Default behavior**: Ask user before major changes (phase transition, archival, creating new principle documents).
- **Auto behavior**: Only if user explicitly says "do it automatically", "no need to ask", "auto sync", or the command is clearly a background task request.

**Safety principle**: Phase transitions and archival are significant actions. When in doubt, ask.

---

## Procedure

### Step 1: Drift Scan (read-only)

| Check | How | Action threshold |
|-------|-----|-----------------|
| Unregistered files | Compare filesystem vs FILE_INDEX (recursive with sub-index prune) | Any file on disk but not in index |
| CURRENT_STATUS freshness | Read first 5 lines, "Last updated" date | Not today |
| CLAUDE.md one-line status | Read first 5 lines, "One-line status (as of DATE)" | Date mismatches CURRENT_STATUS |
| Car body length | Count lines between car-body heading and next `##` | ≥200 lines |
| WORKLOG length | `wc -l` | ≥1000 lines |
| Inbox/outbox | Same four sub-checks as check §1.7 (if adopted) | Any non-✅ result |

### Step 2: Execute Fixes

Apply all fixes that are unambiguously safe:

1. **Register unregistered files**
   - Determine category by file extension and parent folder name (heuristic)
   - If no clear category, create or use `## Uncategorized`
   - If category exceeds 20 files after registration, flag for sub-index creation (do NOT auto-create)
   - Record in CURRENT_STATUS car body: `#### Sync-registered N files (YYYY-MM-DD)`

2. **Refresh stale dates**
   - Update CURRENT_STATUS "Last updated" to today
   - Update CLAUDE.md "One-line status (as of [DATE])" to today (preserve text, bump date)
   - Update FILE_INDEX "Last updated" to today

3. **Inbox/outbox housekeeping** (if adopted)
   - Same remedial actions as check §1.7 flags would prompt

### Step 3: Phase-Transition Check

If car body ≥200 lines:

- **Default (ask user)**: "Car body is N lines (threshold: 200). Execute phase transition now?"
  - If user agrees → execute 5-step phase transition (spec §6.2)
  - If user declines → record in car body: `#### Phase transition deferred by user on YYYY-MM-DD (car body at N lines)`
- **Auto (user explicitly approved)**: Execute 5-step phase transition immediately. Step 1 (copy to WORKLOG) executes before any other write.

If car body <200 lines but steps no longer serve same phase goal (phase coherence failure per check §2.4):
- Coherence failure is stronger signal than line count. Apply same ask/auto logic.

### Step 4: WORKLOG Archival Check

If WORKLOG ≥1000 lines:

- **Default (ask user)**: "WORKLOG is N lines (threshold: 1000). Archive older phases now?"
- **Auto (user explicitly approved)**: Execute archival per spec §5.5 (keep most recent 3 phases, move earlier to `WORKLOG_ARCHIVE_YYYY-MM-DD.md`, register, record in car body, atomic git commit)

### Step 5: Context-Principle Extraction (ask user)

Scan the current session's context for principles, lessons, or rules that have emerged but are not yet recorded.

Present to user: "During this session/work phase, I observed: [list]. Should any of these be recorded in PHILOSOPHY.md or promoted to CLAUDE.md iron rules?"

If user confirms any item → create/append document, register in FILE_INDEX, record in car body.

In **auto mode** (only if user explicitly approved), skip this step entirely. Auto mode does NOT attempt to synthesize principles from context.

---

## Output Format

```
═══════════════════════════════════════
  Doc Harness — Status Sync
  Project: [name]
  Date: YYYY-MM-DD
  Mode: ask-user / auto-approved
═══════════════════════════════════════

── Scan Results ──
[FILE_INDEX]    N unregistered files found
[CURRENT_STATUS] Last updated: YYYY-MM-DD (stale/fresh)
[CLAUDE.md]     One-line status date: YYYY-MM-DD (stale/fresh)
[Car body]      N lines (under/at/over limit)
[WORKLOG]       N lines (under/at/over limit)

── Actions Taken ──
- Registered N files: [category list]
- Refreshed dates: CURRENT_STATUS, CLAUDE.md, FILE_INDEX
- Phase transition: [not needed / executed / deferred by user]
- WORKLOG archival: [not needed / executed / deferred by user]
- Principles recorded: [none / list]

── Remaining Items (if any) ──
[User decisions pending, or auto-mode notes]
═══════════════════════════════════════
```
