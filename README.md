# Pandoc Publisher for Obsidian

Pandoc Publisher is a desktop-only Obsidian plugin for publishing the active
Markdown note as a PDF through Pandoc and XeLaTeX. It is designed around a
portable publishing core, a compact internal-document mode, and isolated local
branding packs.

> [!IMPORTANT]
> The project is in its design and repository-bootstrap phase. There is no
> installable plugin or release yet.

## Planned capabilities

- Publish an active note as a branded or compact internal PDF.
- Use XeLaTeX consistently, including Unicode support.
- Convert Obsidian wikilinks, image embeds, checkboxes, and callouts safely.
- Ship a neutral `generic` branding that works without private assets.
- Discover validated branding packs stored locally in each Vault.
- Keep client and Full Front-End brandings outside this public repository.
- Handle paths containing spaces, accents, apostrophes, and other Unicode
  characters.

## Requirements

The first release will require:

- Obsidian desktop;
- Pandoc;
- XeLaTeX, typically provided by a TeX Live distribution;
- the Eisvogel Pandoc template;
- the fonts and LaTeX packages used by the selected branding.

The plugin will validate these dependencies, but it will not install them.
Exact supported versions and platform instructions will be documented before
the first release.

## Project status

Implementation has not started. The accepted product and architecture decisions
are recorded in the [specification](docs/specification.md). The ordered delivery
work is tracked in the [implementation plan](docs/implementation-plan.md).

The intended repository layout is described in the specification. Source code,
tests, build configuration, embedded generic resources, and release workflows
will be added milestone by milestone.

## Development

Clone the repository over SSH:

```bash
git clone git@github.com:fullfrontend/Pandoc-Publisher-Obsidian-Plugin.git
cd Pandoc-Publisher-Obsidian-Plugin
```

There are no package scripts yet. Once the initial plugin scaffold lands, this
section will contain the verified install, test, build, watch, and Vault-linking
commands.

Development builds will be loaded by linking this repository into:

```text
<Vault>/.obsidian/plugins/pandoc-publisher
```

Local plugin settings (`data.json`), generated bundles, and private branding
packs are intentionally excluded from version control.

## Documentation

- [Product and technical specification](docs/specification.md)
- [Implementation plan](docs/implementation-plan.md)
- [Contributing guide](CONTRIBUTING.md)
- [Security policy](SECURITY.md)

The user guide and branding-pack authoring guide will be completed and tested
before the first distributed release.

## Privacy and branding isolation

The repository and its releases may contain only the neutral embedded `generic`
branding. Business and client branding packs remain in their owners' Vaults and
must never be committed, bundled into fixtures, or attached to public releases.

Branding packs can contain LaTeX headers and Lua filters. Treat them as local
code and install them only from trusted sources.

## Contributing

Read [CONTRIBUTING.md](CONTRIBUTING.md) and [AGENTS.md](AGENTS.md) before making
changes. Commits use Conventional Commits with English descriptions.

## License

Copyright (c) 2026 Full Front-End. All rights reserved. See [LICENSE](LICENSE).

