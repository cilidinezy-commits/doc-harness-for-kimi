# Doc Harness — Recall (Kimi CLI Version)

Retrieve information from the project's Doc Harness document hierarchy. Search systematically across registered documents and return structured, source-cited results.

**Trigger**: User says something like "recall what we decided about X", "find in docs", "search project docs", "where did we discuss Y", "why did we choose Z", "what is the current plan", "summarize what we know about W".

**Purpose**: As projects grow, information scatters across dozens of registered files. `recall` provides a systematic retrieval protocol that mirrors the Recovery Chain's hierarchy — from high-level summaries to low-level details — so you find relevant information efficiently without reading every file.

**Relationship to other commands**:
- `check` audits documentation **health** ("Are the docs OK?")
- `sync` repairs documentation **drift** ("Fix the docs.")
- `flush` saves **context** to files ("Preserve what we know.")
- `recall` retrieves **information** from files ("What do we know about X?")

`recall` is **read-only**. It never modifies files.

---

## Query Type Classification

Classify the user's query into one of four types. The type determines which layers to search and how deeply.

| Type | Pattern | Primary layers | Example queries |
|------|---------|----------------|-----------------|
| **A. Status / Plan** | "What next?", "Current status of X?", "What's the plan?" | Layer 0–1 | "What's the current plan for auth?" |
| **B. History / Decision** | "Why did we...", "When did...", "Phase N", "Who decided..." | Layer 1–2 | "Why did we choose PostgreSQL?" |
| **C. File / Topic lookup** | "Find files about...", "Where is...", "List docs on..." | Layer 3–4 | "Find all docs about caching" |
| **D. Cross-document synthesis** | "Summarize...", "Everything about...", "All mentions of..." | Layer 1–4 | "All discussions about authentication" |

**Ambiguous query** (does not clearly fit one type): Default to **Type D** (synthesis). Note in output: `Query type ambiguous — running comprehensive search.`

---

## The Layered Search Protocol

Search proceeds layer by layer, **top-down**, with strict **token-efficiency rules**. Stop early when the query is fully answered.

**Core constraint**: `recall` must not burn tokens by reading entire documents. Use targeted `Grep` and partial `ReadFile` (first/last N lines, specific sections) instead of full-file reads.

```
User query
    ↓
[Phase 1] Classify query type
    ↓
[Phase 2] Leverage existing context (zero-token layer)
    ↓
[Phase 3] Layered search (priority order, token-budgeted)
    ├── Layer 0: CLAUDE.md (iron rules, overview) — from context or first 20 lines
    ├── Layer 1: CURRENT_STATUS (tire tracks, car body, headlights) — from context or grep
    ├── Layer 2: WORKLOG (TOC + targeted phase sections) — head + grep, never full read
    ├── Layer 3: FILE_INDEX (grep only, never full read) — grep descriptions, recurse via grep
    └── Layer 4: Individual files (grep excerpts only, never full read) — grep + -C 3 context
    ↓
[Phase 4] Synthesize & output
```

### Phase 2: Leverage Existing Context (Zero-Token Layer)

Before reading any file, check if the information is **already in your current context**:

- Did you read CLAUDE.md and CURRENT_STATUS.md at session start (Recovery Chain)?
- Did you recently work on files related to the query topic?
- Is the answer already known from previous turns in this conversation?

If yes → use that information directly. Do not re-read files you already know.

**This is the most important token-saving step.** Most recall queries can be answered from context plus a quick grep.

### Layer 0: CLAUDE.md

**When to read**: If iron rules or recovery chain are relevant and not already in context.

**How to read**: Read **only the first 20 lines** (one-line status, current phase, iron rules header). Do not read the full file unless the query is specifically about recovery chain structure.

### Layer 1: CURRENT_STATUS.md

**When to read**: If not already in context, or if the car body has grown since you last read it.

**How to read**: Use `Grep` with the query keywords, or read specific sections by heading (e.g., only `## Next Steps` for Type A). Do not read the full file end-to-end.

### Layer 2: WORKLOG.md

**When to read**: For Type B (history/decision) and Type D (synthesis). Skip for pure Type A.

**How to read (strict rules)**:
1. **Read only the TOC** (first 20 lines) to identify phase names and anchors
2. **Grep** the query keywords across the full WORKLOG — this tells you which phases mention the topic
3. **Read only the matching phase sections** (not the entire WORKLOG). Use grep results to locate the right `##` headings, then read 20–50 lines around that heading.

**Never read the full WORKLOG** — it may be 500+ lines.

### Layer 3: FILE_INDEX + Sub-indexes

**When to read**: For Type C and Type D. For Type A/B, skip unless Layers 1–2 were insufficient.

**How to read (strict rules)**:
1. **Grep only** — use `Grep` on FILE_INDEX.md with query keywords. Do **not** read the full FILE_INDEX.
2. **Record matching entries** (path + description)
3. **Sub-index recursion**: If a matching entry points to a sub-FILE_INDEX.md, **grep that sub-index too** — do not read it in full.

**Why no full reads**: FILE_INDEX can be 100+ lines. Grep gives you exactly the matches in a single tool call.

### Layer 4: Individual Files

**When to read**: When Layers 1–3 identify relevant files and descriptions are insufficient.

**How to read (strict rules)**:
1. **Grep first** — run `Grep` on each candidate file with query keywords. Use `-C 3` (3 lines of context) to capture the surrounding paragraph.
2. **Only if grep finds nothing but the file is strongly suspected** → read the first 30 lines (overview/TOC) to see if the topic is discussed under a different term.
3. **Never read a file in full** unless it is <50 lines and you have strong evidence it contains the answer.

**Scope limit**: If >10 files match in Layer 3:
- Present the top 5 most relevant matches (by description relevance)
- Tell the user: "5+ files match. Showing top 5. Specify a category or rephrase to narrow."
- Do not read beyond 5 files in Layer 4 without user confirmation.

---

## Search Rules (Normative)

1. **Only search registered files** (in FILE_INDEX or sub-indexes). Unregistered files are invisible to `recall`.
2. **Context first, files second**: Always check if the answer is already in your conversation context before reading anything.
3. **Grep before read**: For FILE_INDEX and individual files, use `Grep` as the first tool call. Full-file `Read` is a last resort.
4. **Never full-read large files**:
   - WORKLOG >200 lines → grep + read only matching sections
   - FILE_INDEX >50 lines → grep only
   - Any individual file >100 lines → grep + -C 3 context only
5. **Sub-indexes via grep**: Recurse into sub-FILE_INDEX.md using grep, not full reads.
6. **Cite every claim**: Every fact must carry a citation like `(CURRENT_STATUS.md §Car Body)` or `(notes/design.md#L42)`.
7. **"Not found" is acceptable**: Say so clearly. Do not hallucinate.
8. **No file modifications**: `recall` is read-only. Note drift but do not fix.
9. **Stop early**: Do not proceed to Layer 4 if Layers 1–2 already answer the query.

---

## Output Format

```
═══════════════════════════════════════
  Doc Harness — Recall
  Query: [user's exact query]
  Type: [A | B | C | D | ambiguous→D]
  Layers searched: [0–1 | 0–2 | 0–3 | 0–4]
═══════════════════════════════════════

── Layer 0: Project Overview ──
[relevant excerpts from CLAUDE.md, if any]

── Layer 1: Current Status ──
[relevant excerpts from CURRENT_STATUS, with citations]

── Layer 2: Worklog History ──
[relevant phase summaries from WORKLOG, with citations]
(omit if skipped)

── Layer 3: File Index Matches ──
[matching FILE_INDEX entries with descriptions]
(omit if no matches or skipped)

── Layer 4: Document Details ──
[if read: key excerpts from individual files, with citations]
(omit if skipped)

── Synthesis ──
[concise answer to the user's question; every claim cited]

── Not Found ──
[if applicable: topics not found in registered documents]

── Drift Note ──
[if applicable: unregistered files that may be relevant]
═══════════════════════════════════════
```

**Omission rule**: If a layer was skipped, omit its section entirely.

---

## Edge Cases

### Query matches nothing

```
── Synthesis ──
No registered documents mention "[query topic]".

Possible reasons:
- The topic has not been documented yet.
- Relevant files exist but are not registered in FILE_INDEX.
- The topic is described using different terminology.

Suggestion: Run sync to ensure all files are registered, then retry.
```

### Query matches unregistered files

Note them in the `Drift Note` section but do not read their contents:

```
── Drift Note ──
The following unregistered files also mention "[topic]" but were not read:
- `tmp/draft.md`

Consider registering them in FILE_INDEX if they contain durable information.
```

### Very large project (>100 registered files)

1. Read FILE_INDEX categories first
2. Only search categories whose descriptions match the query
3. For matching categories with >20 files, present category list and ask user which to deep-read
4. Never read more than 20 individual files in a single recall
