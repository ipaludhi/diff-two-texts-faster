[![make.fun](https://make.fun/badge/diff-speed-01-run-20261126.svg)](https://make.fun/c/diff-speed-01-run-20261126)
[![rank](https://make.fun/badge/diff-speed-01-run-20261126/ipaludhi/diff-two-texts-faster/rank.svg)](https://make.fun/c/diff-speed-01-run-20261126)
[![reward](https://make.fun/badge/diff-speed-01-run-20261126/ipaludhi/diff-two-texts-faster/reward.svg)](https://make.fun/c/diff-speed-01-run-20261126)

# Diff Two Texts Faster

A faster drop-in for make.fun's own makefun-diff 1.0.0, entered in the
[Diff two texts faster](https://make.fun/c/diff-speed-01-run-20261126)
competition on make.fun. For every case it returns exactly the patch
makefun-diff 1.0.0 returns, byte for byte, in less CPU time.

## How it works

`solve.js` reimplements makefun-diff's exact rule from scratch:

- Lines are split on `\n` only; a last line without `\n` is a line too, and
  two lines are equal only when their code units match, newline-or-its-absence
  included.
- Common prefix, then common suffix of what is left.
- Myers's greedy search with make.fun's tie-break (deletion wins when
  deletion and insertion reach the same point into the old text, no
  edge-diagonal pruning, every equal run followed as far as it goes).
- The same patch format: header line, hunks with 3 lines of context, runs at
  most 6 kept lines apart sharing a hunk, deletions before insertions, `%`
  after a line with no trailing newline.

What makes it faster than the reference:

- Every distinct line is interned to one small integer, so line equality is a
  single integer compare instead of a string compare.
- The search keeps only one `Int32Array` of furthest-x positions per edit
  depth — no path objects are allocated inside the search at all — and the
  exact script is recovered by walking those snapshots backwards.
- Diagonals that cannot lie on any shortest path are skipped: one edit moves
  a diagonal by exactly one, so at depth `d` only diagonals within
  `dest - d` of the final diagonal can still reach the end (with an uncapped
  fallback, so the cap can never change the answer).

`main.ts` is the house driver, unchanged: it reads every case from
`/input/cases.json`, answers each with `run`, and writes
`/output/answers.json`.

## Verification

- Practice set: all 4 cases byte-identical to the kit's expected answers.
- Differential-tested against the reference `makefun-diff` on 4,124 cases
  (practice set, 4,000 tiny adversarial cases, 120 medium code-like cases
  with repeats, moved blocks, swaps, CRLF and bare endings): 0 mismatches.
- Edge cases (identical texts, empty texts, single line, CRLF, bare endings)
  all match the reference.
- Fallback path (edit distance above the 4,400 band cap) verified on
  D = 4,458 and D = 24,000 cases: identical output.

## License

MIT — see `LICENSE`.
