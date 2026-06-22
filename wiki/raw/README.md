# raw/ — immutable original artifacts

A **flat** folder of original files, kept under their **original names** exactly as
received — e.g. `attention-is-all-you-need.pdf`, `the-bitter-lesson.html`,
`q3-call-transcript.txt`, `figure-2.png`. No per-source subfolders, no renaming to
`src-XXXX`.

**Never edited.** This is the source of truth. The `src-XXXX → filename` mapping lives
in each source's [`../sources/`](../sources/)`src-XXXX.md` (the `raw:` frontmatter
field). The LLM reads a raw file only when that `src-XXXX.md` summary is insufficient —
that summary-first habit is what keeps large raw files out of context.
