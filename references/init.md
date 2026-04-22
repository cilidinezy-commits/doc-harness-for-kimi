# Doc Harness — Init (Kimi CLI Version)

Create the five Doc Harness files for a new or existing project.

**Trigger**: User says something like "set up doc-harness", "init project docs", "create status documentation", "organize this project so we don't lose track".

---

## Step 1: Gather Information

Extract from conversation context. If anything is unclear, ask the user — do NOT guess.

**Required** (must have before creating):
- Project name (short, for document titles)
- Project description (1-3 sentences)
- First phase name (what is the initial work called)
- First concrete task (what to do first — for "headlights")

**Optional** (ask if unclear, OK to leave blank):
- Initial iron rules
- Key technical details (tools, languages, paths)
- Known sub-projects
- **Inter-project inbox/outbox**: Ask "Does this project coordinate with other projects (has dependencies in either direction)?" Default yes if user mentioned dependencies; no for standalone projects.

---

## Step 2: Check Existing Files and Project State

Check the target directory for three situations:

**(a) Clean directory** (no files, or only trivial starter files) → proceed with blank templates.

**(b) Doc Harness already present** (CLAUDE.md + CURRENT_STATUS.md + ... exist) → do NOT overwrite. Warn user and suggest check instead.

**(c) Non-empty directory without Doc Harness** (has code, notes, accumulated files) → **mid-project adoption**. Do NOT treat as clean init:
- Scan existing files to reconstruct project history
- Propose (not impose) a draft WORKLOG reflecting past phases
- Ask user to confirm/correct before writing
- Register existing files in FILE_INDEX in bulk if many (>~20 files)
- Mark any superseded documents

**(d) Non-empty directory, user explicitly requests clean start** → confirm explicitly: "Existing files will NOT be reconstructed into WORKLOG. Do you want to proceed?" Record override decision in WORKLOG's first phase summary.

---

## Step 3: Create 5 Files

### File 1: CLAUDE.md

```markdown
# [PROJECT_NAME] — Entry Document

**Last updated**: [TODAY]
**Current phase**: [PHASE_NAME] (⏳)
**One-line status (as of [TODAY])**: [DESCRIPTION] — project just initialized

---

## Recovery Chain

Recovery Chain is the entry ritual for any agent or human resuming work on this project.
It has two layers.

### Must-read (baseline)

1. This file (CLAUDE.md)
2. `CURRENT_STATUS.md`

### Task-conditional

- If looking up a specific file: read `FILE_INDEX.md`
- If investigating historical phase details: read `WORKLOG.md`
- [Add project-specific entries as work categories emerge]

### Meta-rules

- Recovery Chain is **self-contained**: every entry points to a file inside the project.
  No dependency on agent-side features (memory, chat history, external services).
- Recovery Chain is **living**: review and update at phase transitions.

## Project Overview

[DESCRIPTION expanded to 3-5 lines]

## Project-Level Iron Rules

[USER_PROVIDED_RULES or defaults:]
- "Write it down or lose it" — important info must be saved to files and registered
- [project-specific rules]

## Key Technical Information

[From context, or: "To be filled as project progresses"]

## Sub-projects

[If any, or remove section]

---

## Doc Harness — Operational Rules

[INSERT FULL CONTENT OF operational_rules.md HERE]
```

**Important**: Read the operational_rules.md from the doc-harness skill directory and embed its FULL content (from `<!-- doc-harness-ops-start -->` through `<!-- doc-harness-ops-end -->`, inclusive) into CLAUDE.md. Preserve both sentinels and the version tag.

### File 2: CURRENT_STATUS.md

```markdown
# CURRENT_STATUS — [PROJECT_NAME]

**Last updated**: [TODAY]
**Current phase**: [PHASE_NAME]

---

## Recent Completed (Tire Tracks)

(Project just started, no completed phases yet)

---

## Current Work (Car Body)

### Phase Goal
[What this initial phase aims to accomplish]

### Completed Steps
(No steps completed yet)

---

## Next Steps (Headlights)

### Immediate Actions
1. [First concrete task from gathered info]

### Future Plans
- [Broader goals from context]

---

## Current Working Principles (Driving Manual)

- Follow the Doc Harness operational rules (see CLAUDE.md)
- [Phase-specific principles from context]
```

### File 3: FILE_INDEX.md

```markdown
# FILE_INDEX — [PROJECT_NAME]

**Last updated**: [TODAY]

---

## Core Documents
- `CLAUDE.md` — Project entry point
- `CURRENT_STATUS.md` — Active status
- `FILE_INDEX.md` — This file
- `WORKLOG.md` — Work history
- `DOC_HARNESS_SPEC.md` — Doc Harness specification (reference)
```

### File 4: WORKLOG.md

```markdown
# WORKLOG — [PROJECT_NAME]

## TOC
| Phase | Time | Anchor |
|-------|------|--------|
| (No phases completed yet) | | |
```

### File 5: DOC_HARNESS_SPEC.md

Read the spec.md from the doc-harness skill directory and write its content to the project directory as `DOC_HARNESS_SPEC.md`.

If spec.md is not accessible, note in FILE_INDEX: "DOC_HARNESS_SPEC.md — not yet deployed (operational rules in CLAUDE.md are sufficient for daily use)".

---

## Step 3.6: (Optional) Enable inter-project inbox/outbox

If the user opted in, do the following:

1. **Create folders**: `inbox/` and `outbox/` at project root
2. **Add iron rule block** to CLAUDE.md inside Iron Rules section
3. **Add to Recovery Chain** task-conditional layer in CLAUDE.md
4. **Register in FILE_INDEX**

See spec.md Chapter 14 for the exact templates.

---

## Step 4: Verify

- All 5 files exist
- FILE_INDEX lists all 5 (plus inbox/outbox if enabled)
- CURRENT_STATUS has all 4 sections
- CLAUDE.md contains embedded operational rules (and inter-project iron rule block if enabled)
- WORKLOG has empty TOC
- If inter-project comms enabled: inbox/ and outbox/ exist; Recovery Chain includes task-conditional entry

Report: "Doc Harness initialized for [PROJECT_NAME]. 5 files created[, with inter-project inbox/outbox enabled if applicable]."
