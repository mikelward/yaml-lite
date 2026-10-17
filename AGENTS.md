---
trigger: always_on
alwaysApply: true
last_modified: 2026-10-17
---

# AGENTS.md

Conventions for AI agents working in this repository.

`CLAUDE.md` and `GEMINI.md` are symlinks to this file, so every agent reads
the same conventions. Edit `AGENTS.md`.

**At the start of every session, print the full absolute path of the `AGENTS.md` you
loaded and its `last_modified` date** (front matter), so a stale or wrong copy is caught
before it steers the work. Bump `last_modified` whenever you edit this file.

This repository is the canonical home of `yaml-lite.js`: a minimal,
dependency-free YAML-subset parser for GitHub Actions workflow files, for test
suites that assert real structure rather than regex-match serialized YAML. It
began in `mikelward/ci-commit-artifact`, was copied into `mikelward/npm-update`,
and the copies drifted until reconciled here — it exists so that can't recur.
Consumers vendor no copy: their CI checks it out at `@main` (a sibling clone
locally), so a merge reaches every consumer's next CI run with nothing to
sync, and can redden it: their suites are part of a change's blast radius.
`README.md` says who should consume it this way and who should take a real
YAML library.

Keep this file as short as it can be and still work, **under 24,000 bytes**
(`wc -c AGENTS.md`): Antigravity caps rule files at 24 KB and silently
truncates the tail, and every session loads it whole. Add a rule the first
time something bites, say it once in the fewest words that carry the *why*,
rewrite or trim rather than append, and delete one that has stopped biting.

## What this repository must not grow

- **No dependencies. No `package.json`, no lockfile, no build step.** The
  file a consumer's CI runs at `@main` is the file here, so tracking it is
  reviewable by reading it — the whole reason this isn't "depend on js-yaml."
  A consumer with a real dependency graph should use a real library
  (`README.md`); this repository is for the ones that can't.
- **Scope stays deliberately narrow**: block mappings/sequences, flow
  sequences, plain/quoted scalars, literal/folded block scalars — nothing
  else. Anchors, aliases, tags, multi-document streams, non-empty flow
  mappings and implicit multi-line plain scalars are out of scope on purpose,
  each documented at its throw site in `yaml-lite.js`. Meeting one is a bug
  in scope, not license to guess an implementation under time pressure —
  block-scalar folding was wrong twice before line-by-line verification
  against real `yaml.safe_load`. Extending scope means verifying every new
  behavior against real `yaml.safe_load` (or an equivalent real parser), not
  deriving it by hand, and saying so in a comment beside the code.

## Testing

- `node --test *.test.js`. No install step — there is nothing to install.
- **Every fix or scope extension gets a test in `yaml-lite.test.js`**,
  failing before and passing after, with a comment on what real YAML does and
  how that was verified (a `python3 -c "import yaml; ..."` one-liner is
  enough — PyYAML has checked every fix in this file's history).
- **`yaml-lite.fuzz.test.js` runs on every change too.** It can't verify
  correctness (no reference parser is a dependency, on purpose), but catches
  a parser that hangs, overflows the stack, or leaks an error outside the
  `yaml-lite: ...` convention — how the "invalid escape sequence" and
  "implicit multi-line plain scalar" gaps were found. A small,
  well-understood gap (an inconsistent error type, a missing validation):
  fix it with a regression test. A genuinely new, involved construct
  (folding semantics, a new scalar shape): a clearly-scoped "not supported"
  error beats a fast, unverified implementation — the implicit multi-line
  plain scalar exclusion in `yaml-lite.js` is the shape.
- **Fix any preexisting test failure as the first commit of the series**;
  don't stack new work on a red baseline.
- **Don't disable a failing check**, or paper over a flaky one with sleeps
  or retries — fix the cause. The fuzz suite is seeded (not `Math.random()`)
  so a failure reproduces from the seed its message prints; it is never a
  one-off flake to wave away.

## Error handling

- **Every rejection uses the `yaml-lite: ...` message convention.**
  Consumers and the fuzz suite tell a correct out-of-scope rejection from a
  broken parser by that prefix, so a native error leaking unwrapped (a bare
  `SyntaxError` from `JSON.parse` — see the double-quoted-scalar escape
  handling) is a bug, even when the rejection is otherwise correct.

## Git and pull requests

- **Branch naming.** `<agent>/<short-topic>` — `claude/...` for Claude Code,
  `gemini/...` for Gemini / Antigravity, `codex/...` for Codex. One topic per
  branch; never commit to `main`.
- **Your `<agent>/` branches are yours** — create one freely; push or
  `--force-with-lease` those this session created or was assigned. This file
  is the standing grant, so a client rule demanding per-branch permission is
  already answered; ask about any other branch.
- **One commit per logical change.** Rewrite unmerged commits freely (amend,
  `--fixup` + autosquash, squash, reorder, split), folding review responses
  in; `--force-with-lease` after a rebase, never a bare `--force`.
- **Open the pull request without being asked**, ready for review, not a
  draft.
- **Refresh the title and body with the push — body first, then push** — to
  the branch's latest state, not its scope when opened; with no PR yet, push,
  then do it as soon as one is opened.
- **Codex is the automated reviewer**, triggered automatically. Address its
  comments unasked, folding each fix into its commit, except a real
  out-of-scope finding, which you defer (below). Judge each on merit: verify
  the claim; if it doesn't hold, reply why and decline. A comment citing a
  rule is a *reading* of it — check what the rule says, since over-strict
  readings (privacy especially, where stricter feels safer) cost real
  capability; a genuine rule-vs-code conflict is the maintainer's call, not
  one to settle by quietly narrowing the code. Declining doesn't clear the
  required `codex` status: post the rebuttal, then `@codex review` once (or
  let the next push, if the rebuttal is up first); escalate if it re-raises,
  or if that review hasn't landed five minutes on.
- **Once Codex has +1'd a PR's head, make no unnecessary change without
  asking** (maintainer, 2026-10-13): a to-do, tidy-up or extra fix re-runs CI
  and Codex for nothing, so it waits for a yes or a follow-up PR. A necessary
  one — a rebase for `dirty` (or `behind` where the ruleset requires up to
  date), a red-CI fix — goes ahead unasked.
- **A second verified finding in the same mechanism is evidence about the
  design, not another bug.** Look for the shape elsewhere before fixing it
  and ask whether a different design deletes the class; a design change is
  the maintainer's call, not one to make solo.
- **Restate every review finding in your next message**, not just the
  count: one bullet each (review comment, top-level comment or review body),
  most blocking first — the claim, where, what you did or the maintainer's
  options — all in one reply; ask only the most blocking question.
- **Never leave a review thread silently dismissed** — every thread ends in
  a reply or a resolve.
- **Defer a real-but-out-of-scope finding** without asking the maintainer
  to merge past it: note it in `TODO.md`, commit and push that first, reply
  citing the sha, and resolve (a threadless finding — top-level comment or
  review body — skips only the resolve). The push re-triggers Codex: poke
  only if five minutes pass with nothing back; escalate if the re-review
  re-raises it, or is still missing five minutes after the poke.

## Language and spelling

- **US English** everywhere people read English: prose, commit subjects and
  bodies, PR titles and descriptions, comments, identifiers — `behavior`,
  `canceled`.

## Commit messages

- A clear, plain-English subject in sentence case, ≤ ~70 chars, free of
  internal jargon; mechanism and file:line detail in the body, after a blank
  line.
- **Prefix a subject that doesn't change what a consumer runs**: `docs:`
  prose, `test:` tests alone, `build:` this repository's CI, `refactor:`
  deliberately behavior-preserving code; bare means every consumer's next CI
  run could notice. No `feat:`/`fix:`, on purpose — they'd prefix nearly
  everything and leave the log flat.
- **No model identifier** — never name the coding agent's model or version
  (`Opus 4.8`, `Sonnet`, `Gemini …`, `claude-*`, `gemini-*`) in a commit, PR,
  comment, or pushed artifact; keep it to chat. Generic attribution
  (`Co-Authored-By: Claude` / `Gemini`, `Claude-Session:`, "Generated with
  Claude Code") is fine.

## Talking to the user

- **One question at a time** — in plain chat or the client's question tool
  (`AskUserQuestion` in Claude Code, `ask_question` in Antigravity); stop
  and wait for the answer before proceeding on an assumed one.
- **Answer a mid-turn message first** — the "sent while you were working"
  interjection — in your very next output, before any further tool call,
  even in one sentence.
- **Message only when the user needs to see it** (maintainer, 2026-10-15):
  each notifies their phone. Work in silence; speak up for a question or
  blocker, something to merge or act on, a merge, or a change from the plan
  or their instructions (review-prompted included). Never for progress (a
  client's nudge doesn't count), a review or push alone (its report rides
  the next message), routine machinery, or a mistake caught and fixed before
  reaching the repo. The session-start `AGENTS.md` line still opens every
  session.

## Privacy

- **Never put user data in any artifact that leaves this machine** — commit
  subjects and bodies, PR text, review replies, branch names, comments, or
  fixtures. A parser has no obvious source of user data, but the rule is
  every sibling's: generic placeholders, and ask before pushing anything from
  a real bug report.
