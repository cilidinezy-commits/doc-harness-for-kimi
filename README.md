# Doc Harness for Kimi CLI

[→ Full project documentation (Claude Code + Kimi versions)](https://github.com/cilidinezy-commits/doc-harness)

**Document-based project control for AI-human collaboration.**

Doc Harness is a [Kimi Code CLI](https://github.com/MoonshotAI/kimi-cli) skill that gives you and AI agents a structured way to manage any project. It keeps your project organized as it grows, ensures AI agents follow the principles you establish, captures every important result and decision in persistent documents, and makes it possible for any new agent — or you, after a break — to pick up exactly where things left off.

Works for any project: writing a thesis, building a SaaS feature, analyzing data, developing a software library, or anything else that spans multiple sessions and produces files.

**Zero dependencies** (plain Markdown, no database / external service) · **MIT**.

---

## 30-Second Overview

Once installed, the agent maintains 5 status documents for your project:
- **CLAUDE.md** — Project entry point (overview, iron rules, recovery chain)
- **CURRENT_STATUS.md** — Active state (current work, recent completed, next steps, working principles)
- **FILE_INDEX.md** — File catalog organized by category
- **WORKLOG.md** — Permanent work history (append-only)
- **DOC_HARNESS_SPEC.md** — Complete specification (reference)

Context reset, new agent, coming back after a weekend — reading these 5 files is enough to pick up seamlessly.

---

## Install

Kimi CLI discovers skills in `~/.kimi/skills/` (or `~/.config/agents/skills/`). Installing means copying this repo's contents into that directory.

```bash
# Create the skills directory if it doesn't exist
mkdir -p ~/.kimi/skills

# Clone this repo into the skills directory
cd ~/.kimi/skills
git clone https://github.com/cilidinezy-commits/doc-harness-for-kimi.git doc-harness
```

**Verify**: Restart Kimi CLI, then say something like:
> *"Help me set up project documentation"*

The agent should recognize the skill and offer to initialize Doc Harness.

---

## Usage

No slash commands needed — Kimi uses natural-language triggers. Just talk to the agent:

| You say | The Moment | Agent does |
|---------|-----------|------------|
| *"Set up doc-harness for this project"* | Starting a project (new or mid-flight) | Creates the 5 core documents tailored to your project |
| *"Check the project docs"* | Regular maintenance / things feel messy | Audits file health + reflects on whether rules are being followed |
| *"Sync the project state"* | Docs have fallen behind reality | Repairs drift: registers missing files, refreshes stale dates, triggers phase transition or archival if thresholds hit |
| *"Save everything before compact"* | Context about to compress / session ending | Emergency save: extracts all important context into documents + runs sync |
| *"Recall why we chose PostgreSQL"* | Can't find something / need decision history | Searches registered documents hierarchically and returns cited answers |
| *"Find all docs about caching"* | Need to locate files by topic | File lookup across the document hierarchy |

You can also manually load the skill with `/skill:doc-harness` if you want to force it.

---

## Optional Documents (Create When Needed)

Doc Harness has two optional documents that you create only when your project actually accumulates the kind of content they hold.

### PARKING_LOT.md — Deferred Items (Not Visions)

**When to create**: When you have work that needs to happen *soon*, but is *blocked* by a precondition.

**What goes in it**: The item, what's blocking it, the unblock condition, and when to review.

**Example**:
```
- Deploy to production
  Blocked by: AWS account not yet provisioned
  Unblock when: Ops team confirms account ready
  Review: 2026-04-25
```

**Not a vision board**: PARKING_LOT is for *near-term* work that is *temporarily* blocked. Long-term dreams ("rewrite in Rust") go in headlights' "Future Plans" instead.

**How to use**: Review during sync/check. When the unblock condition is met, move the item back to CURRENT_STATUS headlights and start working.

### PHILOSOPHY.md — Principles From Practice

**When to create**: When you notice recurring patterns, lessons from mistakes, or principles that generalize beyond the current phase.

**What goes in it**: Principles forged by this project's specific practice — "We tried X three times and Y always works better."

**Example**:
```
- "Never trust CSV encoding headers" — discovered 2026-03-15
  Context: Three data imports failed because we assumed UTF-8.
  Rule: Always sniff encoding with chardet before parsing.
```

**Not iron rules**: Iron rules are mandatory constraints ("Never commit API keys"). Philosophy is empirical wisdom ("Every time we skip tests, a bug returns in 3 days"). Iron rules live in CLAUDE.md; philosophy lives here until it proves universal enough to promote.

**How to use**: Review at phase transitions. Principles that survive 3+ phases can be promoted to CLAUDE.md iron rules.

---

## What Problem Does This Solve?

When you work with AI agents on a project over days or weeks:

- **Your AI loses its memory.** Context windows compress or reset. Yesterday's analysis is forgotten today.
- **Files pile up without a map.** After a few sessions you have 30 files and nobody remembers what's where.
- **Principles drift.** You set rules ("validate on out-of-sample data"), but the AI gradually forgets them.
- **Handoffs fail.** A different agent arrives with zero context. You spend 20 minutes re-explaining.

Doc Harness solves all four by making the **documents** the source of truth — not the AI's context.

---

## The Five Documents

| Document | Role | Answers |
|----------|------|---------|
| **CLAUDE.md** | Entry point | "What is this project? What are the rules? Where do I start?" |
| **CURRENT_STATUS.md** | Active state | "What just happened? What's happening now? What's next?" |
| **FILE_INDEX.md** | File catalog | "What files exist, what are they, and where are they?" |
| **WORKLOG.md** | Permanent history | "What exactly happened in phase 2, three weeks ago?" |
| **DOC_HARNESS_SPEC.md** | Complete reference | Full specification for unusual situations |

---

## Upgrade

```bash
cd ~/.kimi/skills/doc-harness
git pull
```

Then restart Kimi CLI to pick up the changes.

---

## Uninstall

```bash
rm -rf ~/.kimi/skills/doc-harness
```

---

## Full Documentation

This repo contains only the Kimi CLI skill. For the complete project — including Claude Code version, Chinese version, specification, design rationale, and contribution guidelines — see the main repo:

**[cilidinezy-commits/doc-harness](https://github.com/cilidinezy-commits/doc-harness)**
