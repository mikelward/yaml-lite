# AGENTS.md

Conventions for AI agents working in this repository.

`CLAUDE.md` is a symlink to this file, so every agent reads the same
conventions. Edit `AGENTS.md`.

This repository is the canonical home of `yaml-lite.js`: a minimal,
dependency-free YAML-subset parser for GitHub Actions workflow files, used
by test suites that need to assert real structure instead of matching
regex against serialized YAML text. It originated in
`mikelward/ci-commit-artifact`, was copied "verbatim" into
`mikelward/npm-update`, and the two copies then drifted independently for a
while before being reconciled back into one file here — this repository
exists so that doesn't happen again. Consumers do not vendor copies: their
CI checks this repository out at `@main` and their suites resolve the
parser from that checkout (a sibling clone locally), so a merge here
reaches every consumer's next CI run with nothing to sync — which also
means a change here can redden a consumer's CI directly; their suites are
part of a change's blast radius. See `README.md` for who should consume
this file that way versus who should just take a real YAML library as a
dependency instead.

Keep this file as short as it can be and still work. Every session loads it
whole, so each rule costs context on every turn: add one the first time
something bites, say it once in the fewest words that carry the *why*,
rewrite or trim an existing rule rather than appending beside it, and delete
one that has stopped biting.

## What this repository must not grow

- **No dependencies. No `package.json`, no lockfile, no build step.** The
  file a consumer's CI checks out at `@main` and runs is the file here —
  that's what makes tracking it reviewable by reading it, and it's the
  whole reason this file isn't simply "depend on js-yaml." A consumer with
  a real dependency graph should use a real library instead (see
  `README.md`); this repository is for the ones that can't.
- **Scope is deliberately narrow, and staying narrow is the point.** Block
  mappings/sequences, flow sequences, plain/quoted scalars, literal/folded
  block scalars — nothing else. Anchors, aliases, tags, multi-document
  streams, non-empty flow mappings, and implicit multi-line plain scalars
  are all out of scope on purpose, each documented at its own throw site in
  `yaml-lite.js`. Encountering one is a bug in scope, not license to guess
  at a correct implementation under time pressure — the file's own comments
  describe getting block-scalar folding subtly wrong twice already before
  it was verified line-by-line against real `yaml.safe_load` output. When
  extending scope, verify every new behavior against real `yaml.safe_load`
  (or an equivalent real parser) rather than deriving it by hand, and say so
  in the comment next to the new code.

## Testing

- `node --test *.test.js`. No install step — there is nothing to install.
- **Every fix or scope extension gets a test in `yaml-lite.test.js`** that
  fails before the fix and passes after, plus a comment explaining what real
  YAML actually does and how it was verified (a `python3 -c "import yaml;
  ..."` one-liner is enough — PyYAML is what every fix in this file's
  history has been checked against).
- **`yaml-lite.fuzz.test.js` runs on every change too.** It can't verify
  correctness (no reference parser is a dependency here, on purpose) but it
  does catch a parser that hangs, stack-overflows, or leaks an error outside
  the `yaml-lite: ...` convention — which is exactly how the "invalid escape
  sequence" and "implicit multi-line plain scalar" gaps were found. When
  fuzzing turns up a new gap: if it's a small, well-understood fix (an
  inconsistent error type, a missing validation), fix it directly with a
  regression test. If it's a genuinely new, involved construct (folding
  semantics, a new scalar shape), prefer a clearly-scoped "not supported"
  error over a fast, unverified implementation — see the implicit
  multi-line plain scalar exclusion in `yaml-lite.js` for the shape that
  decision takes.
- **Fix any preexisting test failure as the first commit of the series.**
  Don't stack new work on a red baseline.
- **Don't disable a failing check** to make it pass, and don't paper over a
  flaky one with sleeps or retries — fix the underlying issue. The fuzz
  suite is seeded (not `Math.random()`) specifically so a failure is
  reproducible from the seed printed in the failure message, not something
  to wave away as a one-off flake.

## Error handling

- **Every rejection uses the `yaml-lite: ...` message convention.** A
  consumer or the fuzz suite depends on being able to tell "this parser
  correctly rejected out-of-scope input" apart from "this parser broke" by
  checking that prefix — a native error (a bare `SyntaxError` from
  `JSON.parse`, for instance — see the double-quoted-scalar escape handling
  in `yaml-lite.js`) leaking through unwrapped is a bug, even when the
  underlying rejection is otherwise correct.

## Git and pull requests

- **Branch naming.** `<agent>/<short-topic>` — `claude/...` for Claude Code,
  `codex/...` for Codex. One topic per branch; never commit to `main`.
- **One commit per logical change.** Rewrite unmerged commits freely —
  amend, `--fixup` + autosquash, squash, reorder, split — so each commit
  that lands is coherent, with review responses folded into the commit they
  belong to. `--force-with-lease` after a rebase, never a bare `--force`.
- **Open the pull request without being asked**, ready for review, not a
  draft.
- **Refresh the title and body with the push, not after it** — same step,
  so they describe the branch's latest state, not the scope it had when
  opened.
- **Codex is the automated reviewer**, and its reviews are triggered
  automatically. Address its comments without being asked, folding each fix
  into the commit it belongs to. Judge every comment on merit: verify the
  claim before acting, and if it doesn't hold up, reply saying why and
  decline.
- **Never leave a review thread silently dismissed** — every thread ends in
  a reply or a resolve.

## Language and spelling

- Use **US English** everywhere people read English: prose, commit subjects
  and bodies, pull request titles and descriptions, comments, and
  identifiers — `behavior` not `behaviour`, `canceled` not `cancelled`.

## Commit messages

- A clear, plain-English subject in sentence case, short (≤ ~70 chars) and
  free of internal jargon. Mechanism and file:line detail go in the body,
  after a blank line.
- **Prefix a subject that does not change what a consumer runs**: `docs:`
  for prose, `test:` for tests alone, `build:` for this repository's own
  CI, and `refactor:` for deliberately behavior-preserving code. A bare
  subject means every consumer's next CI run could notice the difference.
  There is no `feat:` or `fix:`, on purpose — they would prefix nearly
  everything and leave the log as flat as it started.

## Talking to the user

- **Respond to a mid-turn message immediately.** When the user sends a
  message while you're still working — surfaced as a "sent while you were
  working" interjection — address it in your very next output, before
  starting or continuing any further tool call, even if it's only one
  sentence. Don't let it queue up behind an in-flight chain of tool calls.

## Privacy

- **Never put user data in any artifact that leaves this machine** — commit
  subjects and bodies, pull request text, review replies, branch names,
  comments, or fixtures. This repository has no obvious source of user data
  (it's a parser and its test fixtures), but the rule is the same as every
  sibling repository's: use generic placeholders and ask before pushing
  anything that came from a real bug report.
