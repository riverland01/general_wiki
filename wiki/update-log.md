# Wiki Update Log

Records every change to **content** — ingests and queries. Design changes do **not**
go here (see [`design/design-change-log.md`](../design/design-change-log.md)).

## Progressive disclosure — how this file works
- This file shows **one line per event, newest first**, in *Recent* below.
- Full detail for each event lives in a dated shard: `update-log/YYYY-MM.md`.
- **Rolling rule:** when *Recent* exceeds **50 entries**, move the oldest into the
  current month's shard and replace them here with a single pointer in *Archived*.
- To investigate: read the one-liner here, open the shard only if you need detail.
  Never load all shards at once.

Entry formats (one line each). The consistent `- [YYYY-MM-DD] TYPE` prefix keeps the
log parseable with plain unix tools (e.g. `grep "^- \[" update-log.md | head -5`):
`- [YYYY-MM-DD] INGEST | src-XXXX "title" — created/updated N pages — [detail](update-log/YYYY-MM.md#anchor)`
`- [YYYY-MM-DD] QUERY | "question" — answered; promoted [[page]] (or: not promoted) — [detail](update-log/YYYY-MM.md#anchor)`

---

## Recent (newest first)
_(no activity yet)_

## Archived shards
_(none yet)_ — e.g. `update-log/2026-06.md` — 50 entries, 2026-06-01 → 2026-06-30.
