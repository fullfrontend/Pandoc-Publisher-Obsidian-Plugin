# Contributing

Pandoc Publisher is currently in pre-implementation development. Changes should
follow the accepted specification and the current implementation milestone.

## Before starting

1. Read `docs/specification.md`, `docs/implementation-plan.md`, and `AGENTS.md`.
2. Check existing issues and pull requests to avoid duplicate work.
3. Discuss changes that alter scope, branding isolation, schema compatibility,
   or publishing invariants before implementing them.

## Development workflow

Create a focused branch from `main`:

```bash
git switch -c feat/short-description
```

Package scripts will be documented here after the TypeScript scaffold is added.
Until then, do not assume that a build or test command exists.

Keep changes small, include tests, and update documentation whenever behavior or
configuration changes. Never use real client data or Full Front-End assets in
tests, screenshots, examples, or releases.

## Commit convention

Use Conventional Commits with English, imperative descriptions:

```text
<type>(<scope>): <description>
```

Examples:

```text
feat(branding): validate local branding manifests
fix(markdown): preserve wikilinks inside fenced code
test(callouts): cover nested foldable callouts
docs(user-guide): explain explicit branding selection
```

Common types are `feat`, `fix`, `docs`, `refactor`, `test`, `build`, `ci`,
`chore`, `perf`, and `style`. Use `!` and a `BREAKING CHANGE:` footer for an
incompatible change.

## Pull requests

A pull request should:

- solve one coherent problem;
- link its issue or implementation milestone;
- explain behavior and trade-offs;
- include tests and their results;
- include screenshots for visible UI changes;
- disclose remaining limitations;
- contain no generated bundle unless the release process explicitly requires it;
- pass a final privacy review for local paths and private branding assets.

By contributing, you confirm that you have the right to submit the change. The
project license does not grant reuse rights beyond those explicitly stated.

