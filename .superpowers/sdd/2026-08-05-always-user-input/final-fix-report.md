# Final Fix Report — Always User Input

## RED evidence

Focused tests were written before production changes and failed as expected:

- `npx --yes bun@latest test index.test.ts --test-name-pattern "multi-select returns a custom response"` — expected `{ kind: "freeform" }`, received a structured selection with the text as its selection/comment.
- `npx --yes bun@latest test index.test.ts --test-name-pattern "describes the comment toggle"` — registered schema description still contained `allowComment`.
- `npx --yes bun@latest test index.test.ts --test-name-pattern "renders call summaries"` — legacy `allowComment: true` produced `[optional comment]`, unlike `false`.

## Files changed

- `index.ts`
- `index.test.ts`
- `.superpowers/sdd/2026-08-05-always-user-input/final-fix-report.md`

## Implementation

- The multi-select dialog fallback now uses one input: comma-separated tokens are a structured selection only when every token exactly matches an authored option title; any other nonblank input is a freeform response. Empty and cancelled inputs retain prior cancellation behavior; valid multi-selections still collect optional context.
- The registered `commentToggleKey` description no longer advertises a removed policy switch.
- `renderCall` no longer reads legacy `allowComment`.
- Regression tests prove legacy `allowFreeform: false` and `allowComment: false` still accept custom responses and selection context. The freeform-help test now reaches the freeform row after both fixed controls.

## Verification

- Focused green tests: five targeted `npx --yes bun@latest test index.test.ts --test-name-pattern ...` commands, each `1 pass`, `0 fail`.
- `npx --yes bun@latest test` — `102 pass`, `0 fail`, `402 expect() calls`.
- `npm run check` — `npm pack --dry-run` completed successfully; package contains 7 files.
- `git diff --check` — exit 0 with no output.

## Self-review

Reviewed the final diff for scope and response compatibility. The fallback returns freeform only for nonblank input that is not an exact title set; selected title ordering, optional comments, empty input, and cancellation behavior remain unchanged. No unrelated files were modified.

## Commit

Commit hash: final `HEAD` (reported in the task response; a commit cannot contain its own computed hash without changing that hash).
