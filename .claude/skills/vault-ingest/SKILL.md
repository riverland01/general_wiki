---
name: vault-ingest
description: >-
  Ingest raw material — clipped articles, papers, PDFs, transcripts, notes —
  from wiki/raw/ into this project's structured knowledge wiki, creating or
  updating entity, concept, comparison, and synthesis pages plus source notes
  with frontmatter, [!summary] callouts, status/confidence, and bidirectional
  backlinks. Use this whenever the user asks to "ingest", "process", "compile",
  "file", "add this source", "fold this into the wiki/vault", or drops a new
  document and wants it absorbed into their knowledge base. Prefer this skill
  over hand-rolling the steps whenever the work is adding sources to the wiki.
---

# vault-ingest

This skill performs the **Ingest** operation defined in
[`design/operations.md`](../../../design/operations.md). The design files are the
authority — this skill executes *within* them and never rewrites them.

## Before you start — read the design (it's the source of truth)
1. Read [`design/operations.md`](../../../design/operations.md) (§ Ingest, Guardrails,
   Concept-promotion rule) and [`design/structure.md`](../../../design/structure.md)
   (Page conventions, Source conventions, Index conventions).
2. If anything here disagrees with those files, **they win** — they may have been
   updated since this skill was written.

## Hard guardrail — never touch `design/`
Ingest is **read-only** with respect to `design/`. You may read the rules there, but
must never edit `design/**`. If the design doesn't fit the source (no page type works,
a rule blocks correct filing), **stop and flag it** — record the friction in the
update-log and propose a Redesign to the human. Do not patch the design yourself.

## Layout you are writing into
Write-targets only; the canonical file-by-file layout is
[`design/structure.md`](../../../design/structure.md) → Folder layout.
```
wiki/raw/                      # flat, immutable originals (own filenames) — never edit
wiki/sources/src-XXXX.md       # one self-contained source note per source
wiki/{entities,concepts,comparisons,synthesis}/   # knowledge pages (+ _index.md catalogs)
wiki/index.md                  # tiny root map
wiki/update-log.md             # append a one-line entry per ingest
```

## Procedure

### 1. Place the original & open a source note
- Assign the next `src-XXXX` id (scan existing `wiki/sources/` for the highest).
- Put the original file in `wiki/raw/` **under its own name — do not rename it**.
- Create `wiki/sources/src-XXXX.md` from [`wiki/sources/_template.md`](../../../wiki/sources/_template.md):
  fill frontmatter (`id, title, author, url, date_published, date_added, type,
  license`), set `raw:` to point at the original file, and leave `related:` to fill in.

### 2. Read it
Read the source fully. For sources with images, read the **text first**, then view the
referenced images separately (they don't reliably read inline in one pass).

### 3. Confirm emphasis (default cadence: one at a time)
Briefly tell the human the key takeaways and confirm what to emphasize before filing.
For batch/low-supervision ingest, skip the back-and-forth — follow the cadence in
`operations.md`.

### 4. Write the source digest
In `wiki/sources/src-XXXX.md`, lead with a `> [!summary]` callout written **for the
next agent** (litmus: if the reader stopped here, would they know whether to open the
raw file?), then the key takeaways and notable details.

### 5. Create / update knowledge pages (one pass)
Touch the pages this source affects (typically a handful, up to ~10–15). For each:
- Use the matching template in `wiki/<category>/_template.md`.
- Give it a `> [!summary]` callout and current `status` (stub|draft|stable) and
  `confidence` (low|medium|high).
- Cite every claim inline with `[src-XXXX]`. If a claim can't be sourced, write
  `(no source in vault)` — **never fabricate a citation.**
- Keep **backlinks bidirectional**: add `src-XXXX` to the page's `sources:` and add the
  page slug to the source note's `related:`; keep page↔page `related:` symmetric.

**Concept-promotion rule:** a concept seen in only **one** source is a *claim*, not yet
a concept — record it on the relevant page but don't give it its own `concepts/` page.
Promote it to its own page on its **second independent appearance**. This is what keeps
the concept layer durable instead of a graveyard of orphan singletons.

### 6. Update the catalogs (tiered index)
- Add/refresh the source line in [`wiki/sources/_index.md`](../../../wiki/sources/_index.md).
- Add/refresh each touched page's line in its `wiki/<category>/_index.md` — each line is
  derived from the page's frontmatter (`title, status, confidence`) + the first line of
  its `[!summary]` callout.
- Touch the root [`wiki/index.md`](../../../wiki/index.md) **only** if a hub page, the
  scope sentence, or an open thread changed. Keep the root tiny.

### 7. Log it
Append one line to [`wiki/update-log.md`](../../../wiki/update-log.md), newest-first,
using the `- [YYYY-MM-DD] INGEST | …` prefix shown in that file's header. Follow its
rolling rule if the live section is full.

### 8. Quick consistency check (optional, light)
Before finishing, glance for obvious breakage you just introduced — asymmetric
backlinks, a dangling `[[link]]`, an orphan page. Fix what's trivial; for anything
larger, note it for a full **Lint** pass (a separate operation) rather than expanding
scope here.

## Reporting back
Summarize concisely: the `src-XXXX` id, which pages you created vs. updated, any
concepts promoted, and anything you flagged. Don't paste full note bodies back — the
state lives on disk; pointing at it is enough.
