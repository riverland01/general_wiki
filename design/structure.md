# Wiki Structure & Design Choices (Design Domain)

This document defines the **shape** of the wiki — folder layout, page and source
formats, the tiered index, and the log format. It is the "where things live and why"
half of the design. The "how to act on it" half is in [`operations.md`](operations.md).

This file is **authoritative**: Ingest, Query, and Lint read it but must never edit it.
It may be changed **only** via the human-approved Redesign operation
([`operations.md`](operations.md) §4), and every change must be recorded in
[`design-change-log.md`](design-change-log.md).

---

## Single source of truth — no duplicated prose

Each piece of content has **exactly one canonical home**; every other mention is a
short pointer to it, never a copy. This is what lets you revamp a design file without
hunting for stale duplicates — there are none to hunt.

| Content | Canonical home |
|---------|----------------|
| Core idea / philosophy | [`../AGENTS.md`](../AGENTS.md) → "The core idea" |
| Folder layout (file-by-file tree) | this file → "Folder layout" |
| Page / source / index / log **formats & conventions** | this file |
| Procedures (ingest / query / lint / redesign) + guardrails | [`operations.md`](operations.md) |
| Skill-trigger routing for non-auto-activating agents | [`../AGENTS.md`](../AGENTS.md) |

**When authoring or revamping a design file:**
- Before adding a paragraph, ask *where does this belong?* Write it **once** in its
  canonical home; everywhere else, link to it instead of restating it.
- **Pointers, not copies.** `CLAUDE.md` points to `AGENTS.md`; `AGENTS.md`'s tree is a
  sketch that defers to this file; sibling files cross-link rather than re-explain.
- **Skills are the one sanctioned restatement.** `vault-ingest` / `vault-query` may
  paraphrase design rules for operational self-containment, but must keep the
  "if this disagrees with the design files, **they win**" clause and are **never**
  canonical.
- **Derived views are fine** (catalogs from frontmatter, the AGENTS sketch) as long as
  they're labelled as deferring and never become a second source of truth.
- Lint enforces this — see [`operations.md`](operations.md) §3 → *Divergent duplication*.

---

## Layers

| Layer | Location | Who writes it | Mutability |
|-------|----------|---------------|------------|
| **Raw artifacts** | `wiki/raw/` | Humans (drop files in) | Immutable — read, never edit |
| **Source summaries** | `wiki/sources/` | The LLM | One self-contained `src-XXXX.md` per source |
| **Knowledge** | `wiki/{entities,concepts,comparisons,synthesis}/` | The LLM | Continuously maintained |
| **Index** | `wiki/index.md` + per-category `_index.md` | The LLM | Tiered: tiny root map + on-demand catalogs |
| **Design (this domain)** | `design/` | Humans + LLM | Defines the rules (structure + operations) |

---

## Folder layout

```
.
├── AGENTS.md                   # canonical root router (Codex + any AGENTS.md agent)
├── CLAUDE.md                   # pointer to AGENTS.md (Claude Code entry point)
├── .claude/skills/             # vault-ingest, vault-query (canonical; Claude auto-activates)
├── wiki/                       # WIKI DOMAIN — content
│   ├── index.md                #   tiny root map (always loaded)
│   ├── update-log.md           #   content-change log (progressive disclosure)
│   ├── update-log/             #   dated archive shards (+ README.md explainer)
│   ├── raw/                    #   flat, immutable originals (own filenames; + README.md)
│   ├── sources/                #   one src-XXXX.md per source (digest + raw: pointer)
│   │   ├── _index.md           #     sources catalog
│   │   └── _template.md        #     blank source skeleton (copied on ingest)
│   ├── entities/               #   _index.md + _template.md + entity pages
│   ├── concepts/               #   _index.md + _template.md + concept pages
│   ├── comparisons/            #   _index.md + _template.md + comparison pages
│   └── synthesis/              #   _index.md + _template.md + synthesis pages
└── design/                     # DESIGN DOMAIN — how the wiki is built
    ├── structure.md            #   this file — shape & design choices
    ├── operations.md           #   procedures (ingest / query / lint / redesign)
    ├── design-change-log.md    #   design-change log (progressive disclosure)
    └── design-change-log/      #   dated archive shards (+ README.md explainer)
```

_This tree is the **canonical** file-by-file layout. The sketch in
[`../AGENTS.md`](../AGENTS.md) and the write-target list in the skills orient only —
they defer here (see → Single source of truth)._

Each category (and `sources/`) carries a `_template.md` skeleton — the blank page
copied when a new page/source is created. Templates and `_index.md` catalogs are
**scaffolding, not content**: any frontmatter scan or auto-generated index (see
*Optional → Auto-generated index*) skips `_index.md` and `_template.md`.

---

## Page conventions

Every knowledge page is markdown with YAML frontmatter:

```markdown
---
title: Human-readable title
type: entity | concept | comparison | synthesis
status: stub | draft | stable          # maturity of the page
confidence: low | medium | high        # how settled the claims are
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [src-0001, src-0007]          # backlinks → sources this page draws on
related: [other-page-slug, ...]        # outbound links to sibling pages
---

# Title

> [!summary]
> One- or two-paragraph definition, written for the **next agent** navigating the
> wiki. Litmus: if the reader closed the tab here, would they understand what this is
> and when it applies? This callout — not the body — is the navigation layer (see
> Index conventions). Keep it tight and current.

## Section
Body. Cite every claim inline: [src-0001].
Link every wiki reference: [[backpropagation]].
```

**Rules**
- **Lead with the `> [!summary]` callout.** It is the indexing unit the index and
  query navigation read *first*, before opening the body. Sloppy summaries are what
  cap the wiki's scale — apply the litmus test above.
- **Cite everything** with `[src-XXXX]`. No source = no claim. If a claim cannot be
  sourced from the vault, mark it `(no source in vault)` — **never fabricate a
  citation.** Surfacing the gap is the only safe move.
- **Link everything** with `[[slug]]`. A dangling link is a valid TODO marker.
- **Backlinks are bidirectional.** A page's `sources:` and each source's `related:`
  are kept in sync, as is page↔page `related:`. These links are the navigation edges
  the LLM follows — they stand in for a vector index, so keep them honest.
- Slugs are **lowercase-kebab-case**, stable, and match the filename.
- **One page = one subject.** Prefer updating over duplicating.

**Page types**
- `entities/` — one concrete thing (person, tool, company, dataset).
- `concepts/` — one idea, topic, method, abstraction.
- `comparisons/` — contrast 2+ entities/concepts.
- `synthesis/` — essays promoted from queries answering recurring questions.

---

## Source conventions

Sources are split across two folders, summary-first:

```
wiki/raw/                     # IMMUTABLE originals — FLAT, kept under their own names
├── attention-is-all-you-need.pdf
└── q3-call-transcript.txt    #   never renamed, never edited, no per-source subfolders

wiki/sources/                 # LLM-WRITTEN summaries — ONE markdown file per source
├── src-0001.md               #   frontmatter (provenance + `raw:` pointer) + the digest
└── src-0002.md
```

Each `wiki/sources/src-XXXX.md` is self-contained: YAML frontmatter carries the
provenance (id, title, author, url, dates, type, license) plus a `raw:` field pointing
at the original in `wiki/raw/` and a `related:` list of the pages this source feeds.
The body **leads with a `> [!summary]` callout** (same discipline and litmus as pages),
then the digest. That `raw:` field is the `src-XXXX → raw filename` mapping, so the
flat `raw/` folder needs no naming scheme; the `related:` list is the source side of
the bidirectional backlink, kept in sync with each page's `sources:`.

**Read the source `.md` first (its summary callout before its body); open the raw file
only when it's insufficient.** This is what keeps large raw files out of context.

---

## Index conventions (tiered — progressive disclosure)

The index is two tiers so context stays small as the wiki grows:

- **Root map** — `wiki/index.md`. Tiny and **always loaded**: scope, ~5–10 hub pages,
  per-category counts, open threads, and pointers down. Never inline a full listing here.
- **Category catalogs** — `wiki/<category>/_index.md` (one per `entities`, `concepts`,
  `comparisons`, `synthesis`, `sources`). The full one-line-per-page listing, **loaded
  on demand** when a query is about that category. Each line is built from the page's
  frontmatter (`title`, `status`, `confidence`) plus the first line of its
  `> [!summary]` callout — so a catalog is a derived view, not a second source of truth.

A reader pays for the root map plus the *one* catalog a question needs — not every
listing in the wiki. The cheapest tier of all is page **frontmatter**: scan
`title/type/status/confidence/sources/updated` across files (Dataview-style) without
opening bodies. (Because catalogs are derived from frontmatter + summaries, they can
be auto-regenerated — see *Optional → Auto-generated index*.)

---

## Log conventions (progressive disclosure)

Both logs are **summary-first**. The top file holds one line per entry (newest
first) plus pointers to dated archive shards. When the live section passes its cap,
fold the oldest into a shard and replace them with a single pointer. This keeps any
single file small enough to load without context bloat. Each entry leads with a
`- [YYYY-MM-DD]` prefix so the log stays parseable with plain unix tools. See each
log's header for its exact rolling rule and entry format.

- **Content** changes → [`../wiki/update-log.md`](../wiki/update-log.md) (cap ~50 entries)
- **Design** changes → [`design-change-log.md`](design-change-log.md) (cap ~30 entries)

---

## Optional — as the wiki grows

Everything here is modular; adopt only what your domain needs. A text-only wiki
needs no image handling; a small one needs no search engine.

- **Auto-generated index (recommended at scale).** The catalogs and root map are
  derived from frontmatter + `> [!summary]` callouts, so a small dependency-free script
  (`scripts/rebuild_index.py`) can regenerate them, wired to a PostToolUse hook that
  fires only on writes under `wiki/{<category>}/` or `wiki/sources/`. This makes the
  index the one **deterministic seam** the agent doesn't hand-maintain — keeping a
  ~2K-token index in sync with a vault that would be ~800K tokens raw. _Deferred for
  now (no hook installed); documented as the intended path when the wiki grows._
- **Scale ceiling.** This markdown-navigation approach scales reliably to roughly
  **100–200 notes / a few hundred K words**. Past that, the index + backlinks start to
  miss relevant material; that's when a vector/embedding search layer earns its cost.
- **Search.** At small scale (`wiki/index.md` + folder listing) no search infra is
  needed. As pages reach the hundreds, add a local markdown search engine the LLM can
  shell out to (e.g. `qmd` — hybrid BM25/vector, on-device, has CLI + MCP). Document
  the command here when you add it.
- **Version control.** The whole repo is just markdown — keep it in **git** for free
  history, branching, and diffable updates.
- **Frontmatter is queryable.** The YAML on every page is Dataview-compatible if you
  browse in Obsidian.
