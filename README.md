# yaml-lite

A hand-rolled parser for exactly the YAML subset GitHub Actions workflow
files use: block mappings, block and flow sequences, plain and quoted
scalars, and literal/folded block scalars. Nothing else — no anchors,
aliases, tags, multi-document streams, implicit multi-line plain scalars, or
non-empty flow mappings. Hitting one of those is treated as a bug in this
parser's scope, not a thing to guess at, so it throws rather than silently
returning something wrong.

Zero dependencies, zero build step: `yaml-lite.js` is a single
`module.exports = { parseWorkflowYaml }` file with no imports beyond Node's
own runtime.

## Why this exists instead of a real YAML library

This parser exists so a repository's own tests can assert real structure
against its GitHub Actions workflows — "no `${{ }}` expression is spliced
into any `run:` script" being the motivating case — instead of
regex/string-matching over serialized YAML text, which turned out fragile in
practice (a block-boundary regex repeatedly false-positived on its own fixes
across several review rounds before this parser replaced it).

**If your repository already has a `package.json` and a dependency graph**,
use a real, spec-compliant library instead — [`js-yaml`](https://www.npmjs.com/package/js-yaml)
is the obvious choice, and strictly better than this file for that case: it
handles the full YAML spec (this file's deliberate exclusions included),
is widely audited, and removes the maintenance burden of a hand-rolled
parser entirely. This repository exists for the opposite situation: a
handful of repositories (`mikelward/codex-review`, `mikelward/npm-update`,
`mikelward/lanes`, `mikelward/ci-commit-artifact`) are deliberately
dependency-free — no `package.json`, no lockfile, no build step, because
the code they ship is read and run unpinned by every consumer at `@main`,
and an unpinned third-party dependency there would be exactly the kind of
supply-chain surface that architecture exists to avoid. For those, a small,
fully-understood, line-by-line-verified-against-real-YAML parser is the
right tool; a real spec implementation isn't an option without either a
build step (bundling many files into one) or a runtime dependency.

## Using this in a dependency-free repository

There's no package registry involved, and no vendored copies either — copies
of this exact file drifting apart is the problem that motivated extracting
this repository in the first place. Consume it the way the fleet consumes
all its shared machinery (`mikelward/lanes`, `mikelward/codex-review`, the
reusable workflows themselves): track `@main`.

- **In CI**: add a checkout of this repository into your test job before the
  suite runs —

  ```yaml
  - uses: actions/checkout@v5
    with:
      repository: mikelward/yaml-lite
      path: .yaml-lite
      persist-credentials: false
  ```

- **In the test suite**: resolve the parser from `.yaml-lite/yaml-lite.js`,
  falling back to a sibling clone (`../yaml-lite/yaml-lite.js`) for local
  runs, and **fail loudly — never skip — when neither exists**, with the
  clone command in the error message. A skip would let CI go green with the
  structural checks silently not running; a missing checkout must be red.
  See `resolve-yaml-lite.js` in `mikelward/ci-commit-artifact` for the
  reference shape.

- **Locally**: `git clone https://github.com/mikelward/yaml-lite ../yaml-lite`
  once, next to your checkout.

The trade this makes, deliberately: a merge here reaches every consumer's
next CI run with nothing to sync — and can therefore redden a consumer's CI
directly. Consumers' suites are part of a change's blast radius; this
repository's own suite is necessary, not sufficient.

## Testing

```
node --test *.test.js
```

No install step — there's nothing to install. `yaml-lite.test.js` covers
the parser's documented behavior test-by-test, each added after (or
alongside) a specific bug; `yaml-lite.fuzz.test.js` is seeded, randomized
adversarial testing — it can't check "is this the *correct* structural
result" without a reference implementation this repository deliberately
doesn't depend on, but it does check that the parser never does anything
worse than throw its own clearly-labeled error: no hang, no stack overflow,
no uncaught non-`Error` throw, no error outside the `yaml-lite: ...`
convention.
