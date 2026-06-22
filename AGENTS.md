# Knowledge Wiki — Root

This repository is an **LLM-maintained knowledge wiki**, split into two domains. This
is the **canonical** project router, read by Codex and any agent that loads
`AGENTS.md`. (Claude Code enters via [`CLAUDE.md`](CLAUDE.md), a thin pointer to here.)
The real rules live inside each domain.

> **Single source of truth:** `AGENTS.md` is canonical. `CLAUDE.md` is a thin pointer
> to it (Claude Code's entry point) and holds no instructions of its own. The two
> skills are canonical in `.claude/skills/`; the trigger table below routes agents that
> don't auto-activate them. Edit the canonical files — the pointers never need updating.

```
.
├── AGENTS.md        ← you are here (canonical router)
├── CLAUDE.md        ← thin pointer to AGENTS.md (Claude Code entry point)
├── .claude/skills/  ← vault-ingest, vault-query (canonical)
├── wiki/            ← WIKI DOMAIN — the knowledge + its sources
└── design/          ← DESIGN DOMAIN — how the wiki is built (structure, operations, change log)
```
_Top-level sketch only. The canonical file-by-file layout lives in
[`design/structure.md`](design/structure.md) → Folder layout — this defers to it._

## The two domains

- **`wiki/`** — *what the wiki knows.* Knowledge pages, raw sources, the index, and
  the **update log** (every ingest/query that changed content). Start at
  [`wiki/index.md`](wiki/index.md).
- **`design/`** — *how the wiki is built.* The structure & conventions
  ([`design/structure.md`](design/structure.md)), the operating procedures
  ([`design/operations.md`](design/operations.md)), and the **design-change log**
  (every change to the structure or rules).

The split keeps them separate: editing knowledge is logged in the wiki domain;
editing the *design* of the wiki is logged in the design domain. Never cross the
streams — a content edit doesn't touch the design log, and vice versa.

**`design/` is authoritative and protected.** The content operations (Ingest, Query,
Lint) read the design but must **never** write to `design/`. The design changes only
through the deliberate, human-approved **Redesign** operation — see
[`design/operations.md`](design/operations.md) → "Guardrails" and §4.

## The core idea

The wiki is a **persistent, compounding artifact** — not a re-derived search result.
Every source ingested and question asked makes it permanently richer. The human
curates and explores; the LLM synthesizes, cross-references, cites, and logs.

## Operations run as skills — load the skill before acting

The wiki's two operations are implemented as skills. **When a request matches a row
below, open that skill file and follow it before doing the work — do not improvise the
procedure.**

| When the user wants to… | Skill | File (canonical) |
|--------------------------|-------|------------------|
| ingest / process / compile / file a source, or drops a document to absorb into the wiki | **vault-ingest** | [`.claude/skills/vault-ingest/SKILL.md`](.claude/skills/vault-ingest/SKILL.md) |
| "what do I know about X", "what does my wiki say about…", "find my notes on…", or search/query the wiki | **vault-query** | [`.claude/skills/vault-query/SKILL.md`](.claude/skills/vault-query/SKILL.md) |

Claude Code activates these **automatically** by their descriptions. Codex and other
agents have **no auto-activation**, so this table is the trigger: match a row → read
that skill file first → then execute it. Both skills defer to `design/` as the source
of truth, and like every operation are read-only with respect to `design/`.

## Where to go

| You want to… | Read first |
|--------------|------------|
| Understand what the wiki contains | [`wiki/index.md`](wiki/index.md) |
| Add a source / answer a question | the skill table above, then [`design/operations.md`](design/operations.md) |
| Understand the folder layout & formats | [`design/structure.md`](design/structure.md) |
| See recent content changes | [`wiki/update-log.md`](wiki/update-log.md) |
| Change the wiki's rules/structure | [`design/structure.md`](design/structure.md) or [`design/operations.md`](design/operations.md), then log it in [`design/design-change-log.md`](design/design-change-log.md) |
