# Security policy

## Supported versions

The project has no released version yet. Security fixes will target the latest
supported release once releases begin.

## Reporting a vulnerability

Do not open a public issue for a vulnerability that could expose Vault content,
execute unintended local code, escape a branding-pack directory, or inject shell
arguments.

Use GitHub private vulnerability reporting for this repository when it is
available. If it is unavailable, contact the repository owner privately through
their GitHub profile and include only enough information to establish a secure
follow-up channel.

Please provide the affected version or commit, operating system, reproduction
steps, impact, and any suggested mitigation. Do not attach private Vault files,
client brandings, credentials, or sensitive generated documents.

Non-sensitive defects can use the public bug report template.

## Trust boundaries

Pandoc Publisher launches locally installed Pandoc and XeLaTeX. A branding pack
may also declare LaTeX headers or Lua filters, which can execute with the current
user's permissions. Install branding packs only from trusted sources and inspect
custom executable resources before using them.

