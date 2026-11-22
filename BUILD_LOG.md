# BUILD_LOG — site-generator-speed-01-run-20261122 — faster makefun-site

- Contest: site-generator-speed-01-run-20261122, "Build a static site faster, 22 November 2026", $100.
- Deadline: 2026-11-24 20:00 UTC (2026-11-25 03:00 WIB). Results due 2026-12-09 20:00 UTC.
- Kit level: 36 (kit 0.1.0, build 98fe3e67b4f8). Suggested repo: `makefun-site-generator-speed-01-run-20261122`.
- Account: @verren044 / GitHub verren044. Entry: `solve.js` + house driver `main.ts`.

## Design

One self-contained `solve.js` (27,660 B, no imports, no deps) reimplementing
makefun-site 1.0.0's `build(input)` with identical output on every case. `main.ts`
is the house driver, kept as it ships (reads `/input/cases.json`, answers each case
with `run()`, writes `/output/answers.json`, `--no-incremental-marking`).

Speed work (same semantics, less work):
- Markdown lexer + renderer fused into one pass — no token objects.
- Inline markup jumps between specials with `indexOf` instead of a per-char loop;
  plain runs are escaped in whole chunks (`escapeHtml` fast-paths text with no specials).
- `findUnescaped` / `findSingleStar` via `indexOf` + an odd-backslash-run parity rule
  (equivalent to the reference's left-to-right escape pairing; only backslashes at or
  after the scan start count).
- `trimLine` by char-code scan instead of a regex; slugs cached in a Map;
  `reading_minutes` computed once per source instead of twice per post.
- Templates still parsed once per case and interpreted (no `new Function` / codegen,
  deliberately, to stay clearly inside the security audit); rendering writes into one
  array joined once per page instead of string `+=` per node.

## Files

- `solve.js` — the fast generator (`run(input)`). `main.ts` — house driver.
  `package.json` (`{"type":"module"}`).
- `README.md` (3 live badge lines from the requirements readme_template),
  `LICENSE` (MIT, SPDX first line).

## Practice / verification (all local, 2026-11-23 ~02:00 WIB)

- Kit practice set: 3 cases (257 / 348 / 486 files, 27 / 22 / 21 templates).
- Correctness: `answers.json` byte-identical to the house baseline's output on all 3
  cases (7,446,432 B both). Kit scorer: `{"passed":1,"cases_passed":3,"cases_total":3}`.
- Determinism: two runs into separate dirs, `cmp` clean.
- Speed (wall ms, this VM, 1.5 shared CPUs — noisy, so interleaved rounds):
  6 interleaved rounds, net of a do-nothing control run through the same driver:
  baseline net ~760–900 ms, mine net ~350–500 ms — net ratio 1.7x–2.2x (mean ~2.0x),
  no overlap in any round. The final heat uses CPU time net of the control over 9
  timed rounds (trimmed mean); a ~2x consistent gap is the margin.
- Docker is NOT installed on this VM, so `grade-local --driver docker` could not run;
  verification used the kit's own scorer directly (same program the grader runs).
- Bugs fixed during practice: (1) a typo'd extra `)` in `linkOf` (syntax error);
  (2) my `TAG` regex had doubled backslashes (`[^"[\\\\]]`), which refused every
  source with tags — fixed to the baseline's exact `/^[^"[\]]+$/`, then verified all
  regexes against the baseline's sources (all SAME after the fix).

## Not done by this build (parent's step)

- No GitHub repo created, no push, no submit — builder only, per task.
- Kit `check` / bundle not run here (needs the kit CLI + transcripts; parent handles it).
