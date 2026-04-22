# Doc Harness — Flush (Kimi CLI Version)

Emergency save: systematically extract all important context information into documents and register them before context compression or session end.

**Trigger**: User says something like "save everything", "flush context", "prepare for compact", "ensure nothing is lost", "compress context but save important info first".

**Core guarantee**: After a successful flush, a brand-new agent reading the project's Recovery Chain should recover state **as if context had never been compressed** — knowing *what information exists* and *where to find it*.

---

## Relationship to Sync

`flush` **includes everything `sync` does** as its first phase. The difference is the mandatory **Context Inventory** (Phase B) and **Extraction** (Phase C).

If a project has no drift (all files registered, dates fresh, car body under limit), `sync` is a no-op. `flush` still performs the context inventory and extraction.

---

## How Kimi Decides: Ask or Auto?

Same as sync — default is to ask user before significant extractions. Auto only if user explicitly says "auto flush", "do it automatically", "no need to ask".

**Heuristic for "important" information** (any one suffices):
- User explicitly emphasized it ("this is important", "remember this")
- It represents a decision, rationale, or trade-off (not a transient thought)
- It required significant effort to produce (>5 minutes of analysis, multi-step reasoning, external lookup)
- Losing it would cause confusion or duplicated work for the next agent

---

## Procedure

### Phase A: Run Sync

Execute the complete `sync` procedure (see sync.md) in the same mode (ask-user or auto-approved). This ensures the document foundation is sound before adding new extractions.

If sync triggers a phase transition or WORKLOG archival, complete it fully before proceeding.

### Phase B: Context Inventory

Agent performs a self-scan of its current context. Classify every non-transient item.

**Skip rule**: If an item is already referenced in CURRENT_STATUS car body OR already registered in FILE_INDEX, mark it `DURABLE` and skip.

| Context Information Type | Target Document | Example |
|-------------------------|-----------------|---------|
| Analysis results | `notes/` or `findings/` | "Regression shows R² = 0.87" |
| Data discoveries | `notes/findings/` + registration | "Dataset has 12% missing values" |
| Design decisions | `design/` or car body | "Chose PostgreSQL over SQLite" |
| Architecture choices | `design/` or car body | "Microservice boundary at payment" |
| Error lessons | `PHILOSOPHY.md` or `lessons/` | "Never trust CSV encoding headers" |
| New rules / constraints | CLAUDE.md iron rules or driving manual | "All API responses must include request_id" |
| User verbal requirements | CURRENT_STATUS car body | "User wants dark mode by next release" |
| Intermediate data | `data/` or `tmp/` | "Cleaned dataset before aggregation" |
| Debugging discoveries | `notes/debug/` or car body | "Race condition fixed by adding barrier" |
| Cross-project dependencies | `outbox/` message (if adopted) | "WhoAMI needs model accuracy by Friday" |

**Exclusions** (never extract):
- Fleeting thoughts or brainstorming that did not converge
- Information already known to be wrong or superseded
- Transient tool output (e.g., `ls` listing, temporary error messages)
- Information user explicitly asked NOT to save

### Phase C: Write and Register

For each inventory item not marked `DURABLE`:

1. **Determine target path**
   - Ask-user mode: propose to user; accept, reject, or edit path
   - Auto mode: apply heuristic (existing conventions → standard folders: `notes/`, `design/`, `lessons/`, `data/`)

2. **Write content**
   - Target file exists → append with dated header (`## Added on YYYY-MM-DD`)
   - Target file does not exist → create:
     ```markdown
     # [Title]
     
     > Extracted from context on YYYY-MM-DD during flush
     
     [content]
     ```

3. **Register in FILE_INDEX**
   - Add file path under appropriate category
   - If category does not exist, create it

4. **Record in CURRENT_STATUS car body**
   - `- (YYYY-MM-DD) Flushed: created/updated [path] — [one-line description]`

### Phase D: Verification

Agent simulates a fresh arrival:

```
"I am a new agent with zero context. I read CLAUDE.md → CURRENT_STATUS.
   From car body, do I see references to all files I just created?
   From FILE_INDEX, can I locate each new file by category?
   From Recovery Chain, would I know to read these files if needed?"
```

- Any gap → supplement: add missing references to car body, ensure FILE_INDEX categories are clear
- All covered → pass

### Phase E: Final Flush Marker

Append to CURRENT_STATUS car body:

```markdown
#### Context flushed (YYYY-MM-DD HH:MM) — N items extracted to files
- Sync actions: [phase transition? yes/no] [archival? yes/no]
- New files created: [list]
- Existing files appended: [list]
```

---

## Output Format

```
═══════════════════════════════════════
  Doc Harness — Context Flush
  Project: [name]
  Date: YYYY-MM-DD HH:MM
  Mode: ask-user / auto-approved
═══════════════════════════════════════

── Phase A: Sync ──
[Same output as sync.md]

── Phase B: Context Inventory ──
Scanned context. Found N items:
  [1] [type] — [summary] → [target path]
  [2] [type] — [summary] → [target path]
  ...
  [M] items already durable (skipped)

── Phase C: Write & Register ──
- Created: [list]
- Appended: [list]
- Registered in FILE_INDEX: [category list]
- Recorded in car body: yes

── Phase D: Verification ──
Simulated fresh arrival: [pass / gaps found and fixed]

── Phase E: Flush Marker ──
Recorded in CURRENT_STATUS car body.
═══════════════════════════════════════
```
