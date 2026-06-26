# Wiki Operations (Design Domain)

This document defines **how to act on the wiki** — the procedures for adding sources,
answering questions, and keeping it healthy. For where files live and how they're
formatted, see [`structure.md`](structure.md). Any change to these procedures must be
recorded in [`design-change-log.md`](design-change-log.md).

## Core idea

The wiki is a **persistent, compounding artifact**, not a re-derived search result —
see [`../AGENTS.md`](../AGENTS.md) → "The core idea" (canonical) for the full statement.

---

## Guardrails — operations respect the design

The files in `design/` are the **authority**. Operations execute *within* the design;
they never rewrite it. These rules bind every operation below:

- **Design files are read-only to Ingest, Query, and Lint.** These operations may
  *read* [`structure.md`](structure.md) and this file to learn the rules, but must
  **never edit anything under `design/`** as a side effect — no reorganizing folders,
  no changing page formats, no editing conventions while filing a source or answering
  a question.
- **Conform, don't reinterpret.** Follow the conventions as written. If one is
  ambiguous or silent on a case, **ask the human** — do not invent a local exception
  or quietly drift from the rule.
- **Only Redesign (§4) may change the design**, and only when **explicitly requested
  or approved by the human**. Redesign is never auto-triggered by another operation.
- **When the design doesn't fit, stop and flag — don't patch.** If a source won't fit
  any page type, or a rule blocks correct filing, record the friction (a Lint note or
  an update-log line) and **propose** a Redesign. Wait for approval before touching
  `design/`.
- **No silent design edits.** Every change to `design/` is logged in
  [`design-change-log.md`](design-change-log.md). If it isn't logged, it shouldn't happen.
- **Approval gate — report before you write.** Before any operation **creates or edits a
  wiki entry** (a `wiki/sources/src-XXXX.md` note, a knowledge page under
  `wiki/{entities,concepts,comparisons,synthesis}/`, or a filed-back synthesis), first
  **report a short plan to the human and wait for explicit approval**: name the target
  file(s), say create-vs-update for each, and give the one-line gist of what goes in.
  Only write after they approve. Reads, navigation, and answering in chat never need the
  gate — it is solely about durable writes into `wiki/`. The gate **holds even in batch
  mode**: approval may be granted once per batch, but it is never silently dropped.
  (Placing the original into `wiki/raw/`, reserving the `src-XXXX` id, and opening the
  source-note *shell* — frontmatter + `raw:` pointer, **no digest body yet** — are setup,
  not content; fold them into the same plan you present. The gated writes are the digest
  body and the knowledge pages.)

---

## Execution — skills first, scripts as fallback

Carry out each operation with the **highest-level capability available**, in this order:

1. **Agentic skills first.** If a skill fits the operation — the purpose-built
   **`vault-ingest`** (Ingest) and **`vault-query`** (Query) skills in `.claude/skills/`
   (Claude Code auto-activates them; Codex and other agents are routed to them by the
   trigger table in [`../AGENTS.md`](../AGENTS.md)), or any other available skill that
   matches the task — invoke it.
   Skills encode the procedure and its cost discipline in one place, so prefer them
   over hand-rolling the steps. (Those two skills defer back to this file and
   [`structure.md`](structure.md) as the source of truth.)
2. **Scripts / tooling as fallback.** Only when no suitable skill is available, fall
   back to direct tooling — a Python script, shell, or manual edits via the file tools.

Exception: purely **deterministic** maintenance (e.g. rebuilding the index) stays a
script/hook by design — that is the one seam the agent doesn't hand-drive (see
[`structure.md`](structure.md) → Optional → Auto-generated index). The skills-first
preference is about *agent-driven* work — Ingest, Query, Lint — not that seam.

---

## 1. Ingest — add a source
1. Assign next `src-XXXX`. Drop the original file into `wiki/raw/` under its own name
   (don't rename it); create `wiki/sources/src-XXXX.md` with a `raw:` pointer to it.
2. Read it fully. For sources with images, read the **text first**, then view the
   referenced images separately for extra context (LLMs can't reliably read inline
   images in one pass).
3. **Approval gate (required).** Report a short plan to the human and **wait for
   explicit approval before writing anything**: the key takeaways/emphasis, plus the
   target files this source will touch — the `src-XXXX` note and each page, marked
   create-vs-update with a one-line gist. Do not fill in any body until they approve.
4. Once approved, fill in the digest body of `wiki/sources/src-XXXX.md` (the reusable summary).
5. In one pass, create/update the ~10–15 pages it touches. Keep **backlinks
   bidirectional**: the page's `sources:` ↔ the source's `related:`, and page↔page
   `related:`. Give every new/updated page a `> [!summary]` callout and current
   `status`/`confidence` (see [`structure.md`](structure.md) → Page conventions).
6. Update the relevant **category catalog(s)** (`wiki/<category>/_index.md`) and
   `wiki/sources/_index.md`. Touch the root [`../wiki/index.md`](../wiki/index.md)
   **only** if a hub page, the scope, or an open thread changed — keep the root tiny.
7. Append to [`../wiki/update-log.md`](../wiki/update-log.md): a routine `INGEST` line,
   **plus** a `REVISE` line for each piece of existing knowledge this source changed or
   retired (formats in that file's header). Additions that overturn nothing stay `INGEST`
   only — that split is what keeps meaningful change easy to track.

> **Concept-promotion rule:** a concept that appears in only **one** source is a
> *claim*, not yet a concept — record it on the relevant page, but don't give it its
> own `concepts/` page. Promote it to its own page on its **second independent
> appearance**. This is what keeps the concept layer durable instead of a graveyard
> of orphan singletons.

> **Supersession rule:** when a source *changes or retires existing knowledge* (not just
> adds new), edit the claim in place — never leave a stale value in a body or a
> `> [!summary]` — add a dated line to the affected page's capped `## History` ledger,
> and log it as a **`REVISE`** event, not a plain `INGEST`, so meaningful change stays
> greppable apart from routine activity. For a wholly-replaced page, set
> `status: superseded` + `superseded_by:` and redirect its summary. Full protocol:
> [`structure.md`](structure.md) → Superseded knowledge & revision history.

> **Cadence:** default to **one source at a time** with the human in the loop
> (read summaries, check updates, steer emphasis). Switch to **batch ingest** with
> lighter supervision when volume demands it. Document whichever cadence you settle on.
> Either way the **Approval gate** (see Guardrails) still applies — in batch mode the
> human may approve the whole batch's write-plan in one go, but writes never proceed
> unapproved.

## 2. Query — answer a question
1. Navigate the **tiered index** (see [`structure.md`](structure.md) → Index
   conventions): read the tiny root [`../wiki/index.md`](../wiki/index.md) first →
   open only the relevant category `_index.md` → drill into the pages it lists.
   Synthesize an answer **with citations** on every claim, reusing what exists. If a
   claim can't be sourced from the vault, mark it `(no source in vault)` — **never
   fabricate a citation;** hallucinated sources are how this architecture fails worst.
2. Pick the **output format that fits the question** — prose, a comparison table, a
   Marp slide deck, a matplotlib chart, a canvas. Not every answer is a page.
3. If durably useful, **file the answer back** into `wiki/synthesis/` (or extend a
   page) so explorations compound instead of vanishing into chat history. This is a
   write, so the **Approval gate** applies: name the page you'd create/extend and its
   gist, and **wait for approval before writing**. Answering in chat needs no gate; only
   the file-back does.
4. Append a summary line to [`../wiki/update-log.md`](../wiki/update-log.md).

## 3. Lint — health check
Report and, where possible, fix:
- **Contradictions** between pages.
- **Stale claims** a newer source has superseded.
- **Supersession hygiene** — `## History` ledgers over their cap that weren't folded to
  shards; `status: superseded` pages still drawing inbound links (redirect unfinished) or
  now orphaned (safe to delete); claims still citing a source a later `REVISE` retired.
- **Stale stubs** — `status: stub` / low-confidence pages left untouched, or that now
  have enough sources to mature.
- **Orphans** — pages with no inbound links.
- **Singleton concepts** — `concepts/` pages backed by only one source (per the
  promotion rule, should they revert to a claim?).
- **Missing pages** — concepts referenced often but lacking their own page (incl. dangling `[[links]]`).
- **Broken/asymmetric backlinks** (a `sources:`/`related:` pair out of sync) and **uncited claims**.
- **Divergent duplication** — the same rule or prose copied into 2+ files where one
  should be canonical and the rest pointers (per [`structure.md`](structure.md) →
  Single source of truth). Flag drifted copies; collapse them back to a pointer.
- **Data gaps** that a web search could fill.
- **Proactive suggestions** — new questions worth investigating, new sources worth seeking.

## 4. Redesign — change the rules (design domain only)
**Human-initiated only.** Never begin a Redesign as a side effect of Ingest, Query, or
Lint, and never to make an in-progress task more convenient. Steps:
1. **Propose** the change and its rationale to the human; wait for explicit approval.
2. Edit the relevant design file — [`structure.md`](structure.md) for shape/layout,
   this `operations.md` for procedures. Make the minimal change approved, nothing more.
3. Append an entry to [`design-change-log.md`](design-change-log.md).

This is the **only** operation permitted to write under `design/`, and the only one
that touches the design log.
