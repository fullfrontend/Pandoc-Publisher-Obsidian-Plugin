---
title: Pandoc Publisher implementation plan
status: ready
specification-version: 0.11
date: 2026-07-21
---

# Pandoc Publisher implementation plan

## 1. Outcome

Deliver a desktop Obsidian plugin that publishes the active Markdown note to PDF
through Pandoc and XeLaTeX in either `branded` or `internal` mode. The release
must work with the embedded neutral `generic` branding, support isolated local
branding packs, preserve the source note, and produce actionable diagnostics.

This plan implements `docs/specification.md`. If the plan and specification
conflict, the specification wins until the decision is explicitly amended.

## 2. Delivery rules

- Complete milestones in dependency order. Work inside a milestone may proceed
  in parallel only when the interfaces it consumes are already agreed.
- Keep every commit buildable or documentation-only and use Conventional Commits.
- Add tests alongside behavior; do not postpone the security and path edge cases
  to the final milestone.
- Use only neutral or fictional fixtures in this repository.
- Never import the Full Front-End pack or any client pack into this repository,
  its build, its test artifacts, or its releases.
- Keep Vault migration work outside this repository and perform it only after the
  generic plugin passes its acceptance gates.

## 3. Milestone overview

| Milestone | Deliverable | Depends on | Exit gate |
| --- | --- | --- | --- |
| M0 | Repository bootstrap | — | Complete in repository history |
| M1 | Buildable Obsidian plugin scaffold | M0 | Bundle loads in a clean test Vault |
| M2 | Domain model and settings | M1 | Settings migrate and validate in unit tests |
| M3 | Secure branding core and `generic` pack | M2 | Valid packs load; unsafe packs are rejected |
| M4 | Source-preserving Markdown preprocessing | M1 | Fixtures pass without mutations in code/YAML |
| M5 | Semantic Obsidian callout pipeline | M3, M4 | All types, aliases, and nesting compile |
| M6 | Pandoc/XeLaTeX runtime and diagnostics | M1 | Unicode smoke PDF passes; failures are classified |
| M7 | End-to-end publication engine and modes | M3–M6 | Generic branded and internal PDFs pass integration tests |
| M8 | Obsidian commands and publication UX | M2, M7 | All commands resolve mode/branding correctly |
| M9 | Branding management UI | M3, M8 | Pack lifecycle actions are safe and refresh state |
| M10 | Acceptance, documentation, and release pipeline | M1–M9 | Installable release candidate passes the matrix |
| M11 | Private Vault migration | M10 | Reference documents pass visual acceptance outside repo |

## 4. Detailed milestones

### M0 — Repository bootstrap

Status: complete.

Deliverables:

- initialize `main` and configure the requested SSH remote;
- commit the implementation specification first;
- add ignore, line-ending, editor, and licensing policies;
- add README, agent guidance, contribution and security policies;
- add GitHub issue and pull-request templates;
- add this implementation plan.

### M1 — Plugin and quality scaffold

Create the smallest loadable plugin before implementing publishing behavior.

Tasks:

1. Add `manifest.json`, `versions.json`, `package.json`, lockfile, TypeScript
   configuration, esbuild configuration, and a minimal `src/main.ts`.
2. Establish scripts for type checking, linting, unit tests, integration tests,
   production build, watch mode, and a combined CI check.
3. Add a test runner with separate fast unit tests and tool-dependent integration
   tests. Record external tool versions in diagnostics rather than hard-coding a
   workstation-specific version.
4. Copy every required runtime resource into the distributable bundle without
   embedding local absolute paths.
5. Add `scripts/link-vault.sh` with explicit target validation and refusal to
   overwrite an existing target by default.
6. Add continuous integration for install, lint, typecheck, unit test, build, and
   artifact-content inspection.

Exit gate:

- a production `main.js` builds reproducibly;
- Obsidian can enable and unload the plugin without errors;
- the archive contains only expected public files;
- repository checks run from documented package scripts.

Suggested commits:

```text
build(plugin): scaffold Obsidian plugin toolchain
test(plugin): add unit test harness
build(vault): add safe development link script
ci(test): validate plugin build and public artifacts
```

### M2 — Domain model, settings, and migrations

Define stable boundaries before connecting the UI or file system.

Tasks:

1. Model document modes, branding manifests, resolved brandings, page slots,
   callout themes, publishing requests, results, warnings, and typed failures.
2. Define plugin settings with safe defaults: `generic` branding, sibling PDF
   output, XeLaTeX-only execution, and no workstation-specific absolute path.
3. Add versioned settings migrations and normalize invalid persisted selections.
4. Implement resolution rules independently from Obsidian views:
   explicit frontmatter, configured default, then `generic` only when allowed.
5. Reject attempts to override engine and branding invariants through frontmatter.

Exit gate:

- settings round-trip and migration tests pass;
- explicit missing branding is a typed blocking error;
- an invalid stored default recovers to `generic` with a visible warning;
- `internal` mode never resolves a client branding.

Suggested commits:

```text
feat(settings): add versioned plugin settings
feat(publishing): model publication requests and failures
test(publishing): cover mode and branding resolution
```

### M3 — Branding discovery, validation, and embedded Generic

Build the trust boundary before exposing pack-management actions.

Tasks:

1. Define and document branding schema version 1, including required fields,
   defaults, supported colors and dimensions, and extension points.
2. Discover only direct child directories containing `branding.yaml`.
3. Parse YAML defensively and return all useful validation errors per pack.
4. Canonicalize pack and resource paths. Reject absolute paths, traversal outside
   the pack, missing resources, duplicate IDs, reserved `generic`, and symlink
   escapes. Decide and test platform case-sensitivity behavior explicitly.
5. Separate valid, disabled, and invalid packs without allowing invalid packs to
   enter publication resolution.
6. Create the complete neutral embedded `generic` pack using public fonts and
   assets only. Validate it through the same schema plus embedded-root rules.
7. Implement safe duplicate-to-local behavior with collision handling and no
   silent overwrite.

Exit gate:

- valid fictional packs remain portable after their Vault moves;
- every containment and duplicate-ID attack fixture is rejected;
- `generic` is always present, immutable, and cannot be shadowed;
- a clean public-artifact scan finds no private branding identifiers or assets.

Suggested commits:

```text
feat(branding): define branding pack schema
feat(branding): discover and validate local packs
feat(branding): add embedded generic branding
test(branding): reject unsafe resource paths
```

### M4 — Obsidian Markdown preprocessing

Implement transformations as a deterministic pipeline over parsed source regions,
not unrestricted global replacements.

Tasks:

1. Protect YAML frontmatter, fenced code blocks, and inline code from transforms.
2. Convert wikilinks to readable labels, including aliases and section targets.
3. Resolve image embeds through an injected Vault resolver and emit valid Markdown
   paths. Warn and preserve readable labels for unsupported embedded files.
4. Preserve standard Markdown images and checkboxes.
5. Normalize only known problematic control or Unicode variants.
6. Return transformed source plus structured warnings without modifying the note.

Exit gate:

- byte-focused fixtures prove YAML and code contents are unchanged;
- ambiguous filenames, missing embeds, and Unicode paths have deterministic
  behavior;
- repeated preprocessing is idempotent where applicable;
- the source file remains unchanged in integration tests.

Suggested commits:

```text
feat(markdown): normalize Obsidian wikilinks
feat(markdown): resolve image embeds through the Vault
test(markdown): preserve frontmatter and code literals
```

### M5 — Callout parsing and LaTeX rendering

Keep callout semantics generic and delegate appearance to modes and brandings.

Tasks:

1. Implement the common Pandoc Lua filter for official Obsidian callout syntax.
2. Normalize types and aliases to semantic roles; fall back to `note` for unknown
   types unless a branding mapping exists.
3. Support implicit and explicit titles, empty bodies, `+`/`-` markers, nested
   callouts, and Markdown content.
4. Emit stable LaTeX environment names without business-specific prefixes.
5. Add common environment contracts and both generic branded and compact internal
   renderings. Ensure identification never relies on color alone.
6. Confirm ordinary and historical-label blockquotes remain ordinary quotations.

Exit gate:

- the complete standard type and alias fixture matrix passes;
- nested and foldable callouts compile to PDF;
- snapshot or AST tests prove semantic output is branding-neutral;
- generic and internal renderings pass visual inspection.

Suggested commits:

```text
feat(callouts): parse Obsidian callout syntax
feat(callouts): render semantic LaTeX environments
test(callouts): cover aliases nesting and fold markers
```

### M6 — External-tool runtime and configuration diagnostics

Make operating-system interaction explicit, injectable, and observable.

Tasks:

1. Resolve configured Pandoc and XeLaTeX executables without invoking a shell.
2. Construct argument arrays and environment variables with `path.delimiter` and
   the Vault root as the working directory.
3. Capture bounded standard output and error output, exit status, spawn failures,
   cancellation, and timeout. Redact routine diagnostics where appropriate.
4. Manage unique temporary workspaces and deterministic cleanup; retain them only
   on configured failure paths.
5. Detect Pandoc, XeLaTeX, Eisvogel, required packages, Unicode resources, and
   selected branding resources.
6. Generate a minimal Unicode-and-callout smoke PDF and require both exit code 0
   and the expected file's existence.
7. Produce typed, localized-ready diagnostic results for the UI.

Exit gate:

- no execution path uses `shell: true` or concatenated commands;
- paths containing spaces, accents, and apostrophes pass integration tests;
- a `pdflatex` configuration is rejected;
- missing executable, missing template, TeX failure, timeout, and absent output
  are distinguishable failures.

Suggested commits:

```text
feat(pandoc): add safe process execution
feat(publishing): manage temporary workspaces
feat(pandoc): add configuration diagnostics
test(pandoc): cover Unicode paths and process failures
```

### M7 — Publication orchestration and document modes

Compose the tested components into one publication transaction.

Tasks:

1. Resolve note, mode, branding, metadata, resources, output path, and overwrite
   policy into an immutable publication request.
2. Compose common Pandoc defaults, mode defaults, branding resources, sanitized
   editorial metadata, and computed runtime arguments in the specified order.
3. Build the branded mode with Eisvogel, title page, dedicated table of contents,
   page numbering, and generic branding.
4. Build the compact internal mode without title page, table of contents, logo,
   or client branding.
5. Prevent simultaneous publication of the same source/output pair while allowing
   independent publications.
6. Verify the expected PDF before replacing a destination. Define a recoverable
   overwrite strategy so a failed run cannot destroy the previous PDF.
7. Return output actions and a reproducible, safely quoted diagnostic command for
   display only; never execute the display string.

Exit gate:

- neutral branded and internal fixtures compile end to end;
- branded structure and internal compactness meet their invariants;
- a failed replacement leaves the previous PDF intact;
- simultaneous duplicate requests are rejected clearly;
- no publication test changes its Markdown fixture.

Suggested commits:

```text
feat(publishing): compose Pandoc publication requests
feat(publishing): add branded document mode
feat(internal): add compact document mode
fix(publishing): replace existing PDFs atomically
test(publishing): cover end-to-end PDF generation
```

### M8 — Obsidian commands and publication experience

Expose the engine through accessible, cancellable user workflows.

Tasks:

1. Register the main publish command and direct branded, internal, and
   choose-branding commands.
2. Parse supported frontmatter properties without forwarding legacy visual
   properties into the branding layer.
3. Add mode and branding pickers that show only allowed active options.
4. Show progress, prevent duplicate actions, and offer cancel where the process
   can be terminated safely.
5. Add success actions to open or reveal the PDF and copy diagnostic details.
6. Add an error modal with stage, clear summary, bounded tool output, missing
   resources, and retained temporary path when enabled.
7. Register the configuration-test command using M6 diagnostics.

Exit gate:

- command behavior matches frontmatter/default precedence exactly;
- no active Markdown note and unsupported platform cases fail clearly;
- an explicitly missing branding blocks before Pandoc starts;
- notices and modals remain usable with long Unicode filenames.

Suggested commits:

```text
feat(plugin): register PDF publication commands
feat(ui): add publication progress and results
feat(ui): add configuration diagnostics view
test(plugin): cover command resolution
```

### M9 — Settings and branding management UI

Add safe management workflows on top of the validated M3 service.

Tasks:

1. Add executable paths, relative branding directory, default branding, output,
   overwrite, post-publish action, temporary retention, and log-level settings.
2. Provide create, open, and rescan directory actions with Vault-relative path
   enforcement.
3. Show embedded, valid local, disabled, and invalid packs separately with
   actionable validation details.
4. Add create, duplicate, rename display name, enable/disable, set-default, reveal,
   and test actions. Preserve stable IDs unless the user explicitly creates a new
   pack identity.
5. Put destructive deletion behind confirmation and prefer Obsidian trash APIs or
   another recoverable operation.
6. Invalidate stale selections after rescan without making an explicitly missing
   note branding silently publishable.

Exit gate:

- every lifecycle action revalidates the resulting pack state;
- `generic` cannot be edited, disabled, deleted, or shadowed;
- no path outside the configured Vault directory can be managed;
- deletion is recoverable where the platform provides a trash facility.

Suggested commits:

```text
feat(settings): add publishing settings tab
feat(branding): add branding pack management
feat(branding): duplicate embedded generic pack
test(branding): cover lifecycle safety rules
```

### M10 — Acceptance, documentation, and releases

Turn the feature-complete build into a supportable release candidate.

Tasks:

1. Build the acceptance matrix from specification section 16 and map every item
   to an automated test or a documented manual verification.
2. Test on macOS first, then record actual Windows and Linux support based on CI
   and hands-on results rather than assumptions.
3. Add representative neutral documents, paths, images, callouts, malformed packs,
   and process-failure fixtures.
4. Render and visually compare branded, internal, Unicode, callout, long-table,
   image, and multi-page outputs.
5. Complete `docs/user-guide.md` and `docs/branding-pack-guide.md`, including a
   field-by-field Generic example and tested platform setup instructions.
6. Add release automation that validates version/tag consistency and packages
   `main.js`, `manifest.json`, optional `styles.css`, and required resources.
7. Inspect the archive for source maps, settings, local paths, private identifiers,
   and missing runtime files. Test installation from the archive in a clean Vault.
8. Finalize changelog, compatibility statement, support policy, and release notes.

Exit gate:

- every acceptance criterion is passed or explicitly deferred with approval;
- a clean-machine installation can publish both Generic branded and internal PDFs;
- documentation commands and examples are exercised where automatable;
- the packaged archive contains no private branding or local data;
- the release candidate is installable without the development Vault.

Suggested commits:

```text
test(acceptance): cover publishing acceptance matrix
docs(user-guide): add installation and publishing guide
docs(branding): add branding pack authoring guide
ci(release): package installable plugin artifacts
chore(release): prepare first release candidate
```

### M11 — Private Full Front-End Vault migration

This milestone is operational work outside the public repository. Do not begin it
until M10 passes. Never stage its assets or fixtures here.

Tasks:

1. Build the private local Full Front-End pack by separating generic publishing
   behavior from existing business-specific resources.
2. Link the development plugin into the Vault using the safe linking script and
   configure the Vault-relative branding directory.
3. Validate a quotation or report and a compact internal document before changing
   the reference whitepaper.
4. Test every callout type against the private palette and compare the blue
   `info` and yellow `tip` rendering with approved reference PDFs.
5. Convert all historical whitepaper boxes to official Obsidian callouts in one
   separately reviewed Vault commit without rewriting their bodies.
6. Confirm that no historical labelled blockquote remains, then run the full
   whitepaper visual acceptance test.
7. Keep the historical script available until the plugin outputs are accepted;
   retire or adapt it only through a separately authorized migration.

Exit gate:

- the private pack remains only in the Vault;
- the quotation/report, internal document, and migrated whitepaper pass content
  and visual comparison;
- a fresh public plugin archive still contains no Full Front-End resources;
- rollback instructions and reference PDFs are retained until acceptance.

## 5. Cross-cutting verification matrix

Every relevant milestone must extend these dimensions rather than relying on one
final test pass.

| Dimension | Required cases |
| --- | --- |
| Paths | spaces, accents, apostrophes, Unicode, moved Vault, platform delimiters |
| Branding | Generic, valid local, invalid, disabled, duplicate, absent explicit ID |
| Markdown | YAML, fenced and inline code, links, embeds, images, checkboxes |
| Callouts | types, aliases, custom mapping, titles, fold markers, nesting, unknown |
| Processes | spawn failure, non-zero exit, timeout, cancellation, absent output |
| Files | collision, overwrite disabled, recoverable replacement, cleanup, concurrency |
| Privacy | local settings, absolute paths, private IDs/assets, source maps, logs |
| Packaging | clean install, resource lookup, version/tag match, uninstall/reinstall |

## 6. Prototype decisions to close

Resolve these during M5–M8 with a fixture or UI prototype and record the final
choice in the specification before release:

1. Page-number display immediately after the branded cover.
2. Whether internal-mode numbering is always on or conditional on page count.
3. Default post-publish behavior: action-only or automatic PDF opening.
4. Text-only callout differentiation versus a bundled, license-compatible icon
   set. Prefer text and border semantics unless icons materially improve access.

These decisions must remain settings- and migration-compatible where the
specification already promises a behavior.

## 7. Principal risks and mitigations

| Risk | Mitigation |
| --- | --- |
| Branding path escape or code execution surprise | Canonical containment checks, trusted-pack warning, hostile fixtures |
| Differences between Obsidian Markdown and Pandoc parsing | Region-aware preprocessing and AST/fixture tests |
| TeX installation variance | Configuration diagnostics, documented supported matrix, neutral fonts |
| Failed overwrite destroys an existing PDF | Publish to a unique temporary output and replace only after verification |
| Development Vault leaks into public artifacts | Artifact scans, ignore rules, fictional fixtures, mandatory staged-diff review |
| Private rendering logic contaminates generic core | Semantic callout roles and private pack acceptance only after generic release candidate |
| Release misses runtime resources | Manifest-driven resource copy and clean-Vault archive test |

## 8. Definition of first-release readiness

The first release is ready only when:

- M1 through M10 exit gates pass;
- every applicable specification acceptance criterion has evidence;
- both Generic branded and internal PDFs work from an installed archive;
- source notes remain unchanged and failed runs preserve existing PDFs;
- pack containment, explicit-missing-branding, and shell-free execution tests pass;
- the user and branding-pack guides describe the shipped behavior;
- the release archive is free of local settings, local paths, client data, and
  Full Front-End branding;
- remaining platform limitations and prototype decisions are explicit in the
  README, changelog, and release notes.

