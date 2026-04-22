# Doc Harness — Specification Reference (Kimi CLI Version)

Normative spec for the Doc Harness system. Read when implementing edge cases, phase transitions, inter-project messaging, or resolving protocol ambiguities.

**Trigger**: User asks about a specific spec question, or the agent encounters an edge case not covered by init.md/check.md/sync.md/flush.md.

---

## 1. Core Documents

Every Doc Harness project has **five core documents** plus the operational rules.

| File | Role | Target reader | Update frequency |
|------|------|---------------|-----------------|
| `CLAUDE.md` | Entry point, Recovery Chain, iron rules | Every agent / human | Every phase transition |
| `CURRENT_STATUS.md` | Live state, tire tracks, car body, headlights, driving manual | Working agent | Every session |
| `FILE_INDEX.md` | File registry, project map | Agents looking for files | When files added/removed |
| `WORKLOG.md` | Historical phases with links | Agents investigating history | Every phase transition |
| `DOC_HARNESS_SPEC.md` | Full normative spec | Edge-case resolution | Rarely (spec updates) |

`DOC_HARNESS_SPEC.md` is always `append`. `CURRENT_STATUS.md` is `modify`. `FILE_INDEX.md` is `modify` (append entries, update dates). `WORKLOG.md` is `append`. `CLAUDE.md` is `modify`.

## 2. Recovery Chain

Two-layer navigation system for any agent arriving with zero context.

- **Must-read**: Maximum 3 entries. Usually CLAUDE.md + CURRENT_STATUS.md.
- **Task-conditional**: Per-category entry points. Always reviewed at phase transitions.

All entries must be self-contained (no external dependencies, no agent memory).

## 3. Tire Tracks, Car Body, Headlights

Inside CURRENT_STATUS.md:

| Section | Content | Update |
|---------|---------|--------|
| Tire Tracks | `##` heading per completed phase, 10-20 lines summary | At phase transition |
| Car Body | `####` per working step, plus `#### Phase Goal` | After each step |
| Headlights | `-` bullet under `### Immediate Actions` and `### Future Plans` | When plans change |
| Driving Manual | `-` bullet per working principle | When principles change |

**Car body line limit**: ~200 lines (≈50% of 4096-token comfort zone). At threshold, trigger phase transition. If user declines, record deferral in car body.

## 4. Phase Transitions

The 5-step protocol:

1. **Migrate**: Copy car body steps into WORKLOG as a completed phase section
2. **Update WORKLOG TOC**: Append new row; renumber if needed
3. **Compress**: Add 10-20 line tire-track summary to CURRENT_STATUS
4. **Clear car body**: Insert new phase goal and heading
5. **Refresh CLAUDE.md**: Update date, one-line status, current phase name

## 5. WORKLOG Archival

Trigger: WORKLOG ≥1000 lines. Keep most recent 3 phases in active WORKLOG. Move earlier phases to `WORKLOG_ARCHIVE_YYYY-MM-DD.md`. Register in FILE_INDEX. Record in car body. Prefer atomic git commit.

## 6. Context-Aware Corollary

**Rule**: When remaining context <20% (≈20k tokens), trigger `flush` before next tool call.

Kimi does not expose token counts directly. Use these proxies:
- Context compression is imminent (system warns of compaction)
- Session has been running for many turns with heavy tool outputs
- User says "save everything before we continue"

When in doubt, flush early.

## 7. Inter-project Inbox/Outbox (Optional)

Adopted by creating `inbox/` and `outbox/` folders.

**Inbox format** (files named `YYYY-MM-DD-[type]-[source]-[subject].md`):
```markdown
# [type]: [subject]

**Date**: YYYY-MM-DD
**From**: [source project or agent]
**Status**: unread / read / actioned

## Body

[content]

## Action required

- [ ] [specific task with deadline if any]
```

**Outbox format**: Same structure. Written by this project for consumption by other projects.

**Processing rule**: Inbox messages must be processed within 3 working days. Change status to `read` when opened, `actioned` when completed. Do not delete — archive to `inbox/_archive/`.

## 8. Mid-transition Coherence

Three values must agree during normal operation:
- CLAUDE.md "current phase"
- WORKLOG TOC newest entry
- CURRENT_STATUS car body "Phase Goal"

Mid-transition (inconsistency) is normal during Phase Transitions Step 1–5. If detected outside a transition, run repair per spec §6.3.

## 9. Sync vs Flush

| Aspect | Sync | Flush |
|--------|------|-------|
| Drift scan | Yes | Yes (Phase A) |
| Date refresh | Yes | Yes |
| File registration | Yes | Yes |
| Phase transition trigger | Yes | Yes |
| WORKLOG archival trigger | Yes | Yes |
| Context inventory | No | Yes (Phase B) |
| New document creation | No | Yes (Phase C) |
| Flush marker | No | Yes (Phase E) |

## 10. Compatibility Notes

- **No slash commands**: Kimi CLI skills use natural-language triggers. The `description` field in SKILL.md frontmatter must contain all trigger phrases.
- **No `--auto` / `--interactive` flags**: Use conversation context to decide whether to ask the user. Default: ask. Auto: only when user explicitly requests.
- **References directory**: Kimi CLI supports `references/` folder for on-demand loaded docs. Keep reference docs focused; they are loaded only when triggered.
- **Bilingual**: The primary skill is English (`kimi-skill/`). A Chinese mirror (`kimi-skill-zh/` or similar) follows the same Iron Rule 1 convention.

## 11. Versioning

The Doc Harness spec version is tracked in `CLAUDE.md` via the embedded operational rules version marker:
```html
<!-- doc-harness-ops-version: X.Y -->
```

During sync, compare this against the skill's installed version. If stale, re-embed.
