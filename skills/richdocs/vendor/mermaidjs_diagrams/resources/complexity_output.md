# Complexity analyser — reference output

A verbatim run of `scripts/mermaid_complexity.ts` against the fixture that
ships with this skill. Reproduce it from the skill root (after the one-off
`bun install`, see SKILL.md § Complexity Analysis):

```console
$ bun run scripts/mermaid_complexity.ts resources/examples/test_complexity.md
resources/examples/test_complexity.md:153-206: NodeCountExceedsCognitiveLimit 51 nodes > 50 (Huang 2020 cognitive limit)
resources/examples/test_complexity.md:214-336: NodeCountExceedsHardLimit 120 nodes > 100 hard limit
resources/examples/test_complexity.md:214-336: VisualComplexityExceedsCritical VCS 120.0 > 100 critical threshold
resources/examples/test_complexity.md:394-423: NodeCountExceedsCognitiveLimit 60 nodes > 50 (Huang 2020 cognitive limit)
resources/examples/test_complexity.md:469-498: NodeCountExceedsCognitiveLimit 52 nodes > 50 (Huang 2020 cognitive limit)
resources/examples/test_complexity.md:108-145: NodeCountExceedsAcceptable 36 nodes > 35 acceptable threshold
resources/examples/test_complexity.md:394-423: VisualComplexityExceedsAcceptable VCS 100.0 > 60 acceptable threshold
resources/examples/test_complexity.md:430-439: SubgraphNestingTooDeep subgraph nesting depth 3 (≥3) hinders readability
resources/examples/test_complexity.md:469-498: VisualComplexityExceedsAcceptable VCS 98.5 > 60 acceptable threshold
$ echo $?
1
```

## Reading the output

- **One finding per line**, `path:line_start-line_end: Code detail`. The line
  range is the mermaid fence's span in the source file.
- **Errors first, then warnings**; within a band, findings sort by a plain
  string compare on the `path:start-end` location (`mermaid_complexity.ts`
  §main). That is *not* source order — line numbers are compared as text, so
  `70-90` lands after `671-707`. It only looks like file order above because
  these fences all start at three-digit lines. Read the line range, don't
  infer position from the ordering.
- **Detail text is prose, not key-value pairs.** The finding states the
  measured value against the threshold it breached
  (`36 nodes > 35 acceptable threshold`), so a line stands on its own without
  the reader needing to know which preset was active.
- **Exit code 1** because findings were emitted. A clean file prints nothing
  and exits `0`; a usage error exits `2`.

The fixture breaches every node-count, VCS, and nesting threshold the analyser
enforces (`ParserFailure` has its own fixture in
`resources/examples/test_complexity_recommend.md`), so this transcript doubles
as a regression baseline — if a change to the scoring formula moves a number
here, that is the diff to justify.
