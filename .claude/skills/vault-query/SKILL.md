---
name: vault-query
description: >-
  Answer questions from this project's knowledge wiki by navigating the tiered
  index (wiki/index.md → category _index.md) and bidirectional backlinks, then
  synthesizing a cited answer — without loading the whole vault into context.
  Use this whenever the user asks "what do I know about X", "what does my
  wiki/vault say about Y", "find my notes on Z", "search the wiki", or asks any
  question that should be answerable from notes already ingested. Prefer this
  skill over reading files ad hoc whenever the question targets the wiki's
  contents.
---

# vault-query

This skill performs the **Query** operation defined in
[`design/operations.md`](../../../design/operations.md). The design files are the
authority — this skill executes *within* them and never rewrites them.

## Before you start — read the design (it's the source of truth)
Read [`design/operations.md`](../../../design/operations.md) (§ Query, Guardrails) and
[`design/structure.md`](../../../design/structure.md) (Index conventions, Page
conventions). If anything here disagrees with those files, **they win.**

## Hard guardrail — never touch `design/`
Query is **read-only** with respect to `design/`. It may also create/extend content
pages (step 3) but must never edit `design/**`.

## Why navigation, not full-read
The vault is built so you never load it whole. The `> [!summary]` callouts and the
tiered index are the navigation layer; the bidirectional `sources:`/`related:`
backlinks are the edges you follow. They stand in for a vector index — trust them.

## Procedure

### 1. Navigate the tiered index (cheap → specific)
- Read the tiny root [`wiki/index.md`](../../../wiki/index.md) first: scope, hub pages,
  category counts, open threads.
- Open **only** the relevant category catalog(s) — `wiki/<category>/_index.md` — for
  the question. Don't open catalogs the question doesn't need.
- From the catalog, pick the most relevant candidate pages by their `title`/`status`/
  `confidence` and one-line summary. Read each candidate's `> [!summary]` callout
  before its body; read the full body only when the summary says it's relevant.
- Follow `related:` and `sources:` backlinks **selectively** — pull in a neighbor only
  when it bears on the question, not reflexively. Stop once you can answer.

### 2. Synthesize a cited answer
- Answer from what the pages and source notes say, **citing every claim** with
  `[src-XXXX]` (or `[[page]]` when pointing at a synthesis/page).
- If a claim can't be sourced from the vault, mark it `(no source in vault)` —
  **never fabricate a citation.** Hallucinated sources are how this architecture fails
  worst; surfacing the gap is the only safe move.
- Note gaps and contradictions you hit — they're candidate follow-up work.

### 3. Pick the output format that fits
Prose for a direct question; a comparison table for "X vs Y"; a Marp slide deck, a
chart, or a canvas when that serves the user better. Not every answer is a page.

### 4. File durable answers back (so explorations compound)
If the answer is durably useful (a comparison, an analysis, a discovered connection),
**file it back** rather than letting it vanish into chat: create or extend a page under
`wiki/synthesis/` (template at
[`wiki/synthesis/_template.md`](../../../wiki/synthesis/_template.md)) with a
`> [!summary]` callout, `status`/`confidence`, citations, and bidirectional backlinks.
Then refresh the affected `_index.md` line(s) and append a
`- [YYYY-MM-DD] QUERY | …` entry to
[`wiki/update-log.md`](../../../wiki/update-log.md). If the answer wasn't worth keeping,
say so and skip — don't clutter the vault.

## Reporting back
Give the user the answer with its citations. If you filed anything back, name the page
you created/extended. Keep it tight — the supporting detail lives on disk.
