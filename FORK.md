# About this fork

A fork of [`ratatui-markdown`](https://github.com/celestia-island/ratatui-markdown)
by langyo, carrying **one** bug fix on top of the released `v0.3.6`.

## Why it exists

`MarkdownRenderer::parse` emits a fenced code block *before* the paragraph
that precedes it, whenever the fence opens on the line directly after the
paragraph with no blank line between — which CommonMark §4.5 explicitly
allows, and which is how most prose introduces a code sample:

```
Run it with:
```sh
cd local
```
```

The fence arm of `parse_inner` called `flush_table`, which returns early when
no table is buffered and so never emits `paragraph_lines`. The pending
paragraph survived the whole block and was flushed by the tail of
`parse_inner`, landing after the code block. Every other block opener
(blank line, image, heading, list) had its own paragraph flush; the fence
arm was the one that did not.

The fix adds `flush_pending` — table flush plus paragraph flush — and calls
it from the fence arm. Upstream fixed it the same way, in the same shape, in
`36eb5c79` (2026-07-17).

## Why not just use upstream

Upstream's fix landed a month *after* the project relicensed from
`MIT OR Apache-2.0` to `SySL-1.0` (`6d964eb5`, 2026-06-17), so it cannot be
taken without also taking the new terms. This fork is cut at tag `v0.3.6`
(`9f4a2c06`, 2026-05-21), whose history contains no post-relicense commit —
verified with `git merge-base --is-ancestor` — and whose manifest declares
`license = "MIT OR Apache-2.0"`. It is used here under those terms.

Should upstream publish the fix under the original licence, this fork should
be retired in favour of it.

## What changed from v0.3.6

- `src/markdown/parser.rs` — added `flush_pending`; the fence arm calls it
  instead of `flush_table`.
- `src/markdown/tests.rs` — two regression tests: a fence after a paragraph,
  and a fence after a table (so the added flush does not disturb the path it
  wraps).

Nothing else. All 299 upstream tests still pass.
