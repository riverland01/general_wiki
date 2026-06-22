# Design-Change Log

Records every change to the **design** of the wiki — [`structure.md`](structure.md),
[`operations.md`](operations.md), conventions, layout. Content changes do **not** go
here (see [`wiki/update-log.md`](../wiki/update-log.md)).

## Progressive disclosure — how this file works
- This file shows **one line per change, newest first**, in *Recent* below.
- Full rationale for each change lives in a dated shard: `design-change-log/YYYY-MM.md`.
- **Rolling rule:** when *Recent* exceeds **30 entries**, move the oldest into the
  current month's shard and replace them here with a single pointer in *Archived*.
- To investigate a change: read its one-liner here, then open its shard only if you
  need the full reasoning. Never load all shards at once.

Entry format (one line). The consistent `- [YYYY-MM-DD]` prefix keeps the log
parseable with plain unix tools (e.g. `grep "^- \[" design-change-log.md | head -5`):
`- [YYYY-MM-DD] <what changed> — why (1 phrase) — [detail](design-change-log/YYYY-MM.md#anchor)`

---

## Recent (newest first)
- [2026-06-22] De-duplicated cross-file prose and added a standing anti-duplication rule: collapsed the verbatim "core idea" copy in `operations.md` to a pointer at `AGENTS.md`; made `structure.md`'s folder tree canonical and trimmed `AGENTS.md`'s tree + the skill's write-target list to defer to it; added a **Single source of truth** section to `structure.md` (canonical-ownership map + author/revamp rules, skills as the only sanctioned restatement) and a **Divergent duplication** check to Lint (§3). — one canonical home per fact so revamps don't reintroduce stale copies.
- [2026-06-22] Documented the per-category `_template.md` skeletons and the `README.md` explainers (`raw/`, `update-log/`, `design-change-log/`) in the `structure.md` folder tree, and noted templates/catalogs are scaffolding that index scans skip; also fixed the log grep helper `tail -5`→`head -5` in `design-change-log.md` to match the newest-first ordering (same fix applied to `wiki/update-log.md`). — kill scaffold/doc drift surfaced by a lint pass.
- [2026-06-22] Removed the vestigial `.agent/skills/` folder — `AGENTS.md`'s trigger table routes straight to canonical `.claude/skills/`, so the pointer breadcrumbs did no work. Scrubbed the `.agent` references in `AGENTS.md`, `structure.md`, and `operations.md`. — fewer moving parts; one skills location.
- [2026-06-22] Flipped the canonical router to `AGENTS.md` (the portable cross-agent standard, auto-loaded by Codex); `CLAUDE.md` is now a thin pointer to it. Added a hardened **skill-trigger table** to `AGENTS.md` so Codex (no auto-activation) reliably fires `vault-ingest`/`vault-query`. Claude Code follows the pointer and still auto-activates by description. Skills stay canonical in `.claude/skills/`; `.agent/skills/*` are optional pointer breadcrumbs. — harden Codex skill-firing while keeping one source of truth.
- [2026-06-22] Made the repo Codex-compatible with a single source of truth: `CLAUDE.md` and `.claude/skills/` are canonical; `AGENTS.md` is a thin pointer to `CLAUDE.md` and `.agent/skills/*` are pointers to the canonical skills (no duplicated content). Skills-first rule and the structure-layout tree name both skill dirs. — let Codex and Claude Code drive the same wiki without mirror drift. (Superseded by the AGENTS.md-canonical flip above.)
- [2026-06-22] Built the two operation skills in `.claude/skills/` — `vault-ingest` (Ingest) and `vault-query` (Query) — matching the design; both read `structure.md`/`operations.md` as source of truth, enforce the never-edit-`design/` guardrail, and follow summary-callout / bidirectional-backlink / no-fabricated-citation / concept-promotion rules. Pointed the skills-first rule at them. — make "skills first" concrete.
- [2026-06-22] Added an execution-model preference to operations: **skills first, scripts as fallback** — operations invoke a fitting agentic skill when available and only fall back to Python/shell/manual tooling otherwise; deterministic index rebuild stays a script/hook by design. — lean on agentic skills, keep procedure + cost-discipline in one place.
- [2026-06-22] Incorporated ideas from the LLM-wiki supplement (human-approved): (1) summary-first discipline — every page/source leads with a `> [!summary]` callout written for the next agent, with a litmus test; it's the navigation/index layer. (2) Content rules — concept-promotion heuristic (2nd-appearance before a concept gets its own page), mandatory bidirectional `sources:`↔`related:` backlinks, and "mark `(no source in vault)`, never fabricate a citation". (3) `status`/`confidence` frontmatter on pages, surfaced in catalogs; Lint now flags stale stubs, singleton concepts, asymmetric backlinks. Also documented the auto-index script+hook and the ~100–200-note scale ceiling as deferred scaling paths. — make the architecture actually scale. (Skipped: context-read budgets.)
- [2026-06-22] Added design-protection guardrails: `design/` is authoritative and read-only to Ingest/Query/Lint; it changes only via human-approved, propose-first, logged Redesign. Hardened Redesign §4 and noted the boundary in `structure.md` + `CLAUDE.md`. — stop operations from silently rewriting the design.
- [2026-06-22] Split `schema.md` into [`structure.md`](structure.md) (shape: layout, page/source formats, tiered index, log format) and [`operations.md`](operations.md) (procedures: ingest/query/lint/redesign). — separate "where things live" from "how to act on them".
- [2026-06-22] Simplified sources: `wiki/raw/` is now a flat folder of originals under their own names (no `src-XXXX/` subfolders, no renaming), and each source is a single self-contained `wiki/sources/src-XXXX.md` (frontmatter provenance + `raw:` pointer + digest), replacing the per-source `meta.md`+`summary.md` folder. — fewer moving parts; one file per source.
- [2026-06-22] Tiered the index (tiny root map → per-category `_index.md`, loaded on demand) and split sources into immutable `wiki/raw/` (originals) + LLM-written `wiki/sources/` (meta + summary, read summary-first). — progressive disclosure on the hot query path + keep large raw files out of context.
- [2026-06-22] Folded missing gist aspects into schema — query output formats, broader Lint (missing pages, web-search gap-filling, proactive suggestions), conversational + batch ingest cadence, image-handling, optional search/git, log grep-prefix. — align scaffold with the LLM-Wiki idea.
- [2026-06-22] Split repo into `wiki/` + `design/` domains, each with its own progressive-disclosure log; moved index into wiki domain and slimmed it. — separate content vs. design history; avoid context bloat.

## Archived shards
_(none yet)_ — e.g. `design-change-log/2025-12.md` — 30 entries.
