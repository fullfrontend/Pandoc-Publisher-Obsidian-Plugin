# AGENTS.md

## Purpose

This repository contains the standalone Obsidian plugin described in
`docs/specification.md`. Treat that specification as the source of truth until
an explicitly approved decision supersedes it.

## Non-negotiable boundaries

- Keep the repository independent from every Obsidian Vault.
- Never commit `data.json`, generated `main.js`, local paths, private documents,
  client assets, or client branding packs.
- Never commit or bundle Full Front-End logos, colors, copy, headers, defaults,
  filters, fixtures, or other identifiable branding resources.
- The only embedded branding is the neutral, reserved `generic` branding.
- Do not edit or migrate files in a Vault unless the task explicitly includes
  that external change.
- Do not silently fall back when a note explicitly requests a missing branding.
- Do not let note frontmatter override publishing-engine or branding invariants.

## Architecture

Keep responsibilities separated along the planned module boundaries:

- `src/publishing`: publication orchestration and process execution;
- `src/brandings`: discovery, schema validation, confinement, and resolution;
- `src/markdown`: source-preserving Obsidian Markdown normalization;
- `src/settings`: persisted settings and defaults;
- `src/ui`: commands, modals, notices, and settings views;
- `resources/common`: invariant Pandoc, Unicode, and callout resources;
- `resources/modes`: branded and internal presentation layers;
- `resources/brandings/generic`: the public reference branding.

Use dependency injection at operating-system and Obsidian boundaries so core
logic remains testable without launching Obsidian or Pandoc.

## Security and portability rules

- Start Pandoc with `child_process.spawn`, `shell: false`, and an argument array.
- Never construct a shell command by concatenating user-controlled paths.
- Resolve and validate every branding resource against its canonical pack root;
  reject absolute paths, traversal, and symlink escapes.
- Use Node path utilities and `path.delimiter`; do not split paths on spaces.
- Keep temporary files unique per publication and clean them up unless failure
  retention is explicitly enabled.
- Redact private document content and local paths from routine logs where
  practical. Never include document contents in telemetry.
- Treat branding-provided Lua and LaTeX as trusted local code and make that trust
  boundary visible to users.
- Preserve the source note byte-for-byte during publication.

## Change workflow

1. Read the relevant specification and implementation-plan sections.
2. Keep the change within one milestone and one coherent responsibility.
3. Add or update tests for behavior and failure cases.
4. Update user-facing documentation with the behavior it describes.
5. Run all available checks and inspect generated plugin artifacts when relevant.
6. Review the staged diff for private assets, local paths, and generated files.
7. Commit with `<type>(<scope>): <imperative description>` in English.

Do not invent successful commands while the repository is still being
scaffolded. Once `package.json` exists, use only its documented scripts and keep
this file and `README.md` synchronized with them.

## Testing expectations

At minimum, cover:

- paths with spaces, accents, apostrophes, and Unicode;
- branding manifest validation and containment failures;
- `generic`, default, explicit, missing, disabled, and duplicate branding cases;
- Markdown transforms outside fenced code while preserving YAML and code;
- all standard callout types, aliases, nesting, fold markers, and unknown types;
- Pandoc argument construction without a shell;
- cancellation, timeouts, concurrent publication prevention, and cleanup;
- successful exit plus expected-PDF existence as the success condition.

Use only neutral or fictional public fixtures.

## Completion criteria

A change is complete when its acceptance criteria pass, its documentation is
accurate, no private resource is present, and the staged diff is suitable for a
public GitHub repository. A first release additionally requires an installable
archive containing `main.js`, `manifest.json`, optional `styles.css`, and all
required generic publishing resources.

