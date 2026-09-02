# OMP Configs Repository Guidance

## Purpose

This repository is the chezmoi source state for portable, user-level Oh My Pi configuration that cannot be installed as a plugin. Chezmoi applies the managed source files to `~/.omp/agent`.

## Source Mapping

Chezmoi encodes target names and permissions in source filenames:

- `private_dot_omp/private_agent/` maps to `~/.omp/agent/`.
- `private_config.yml` maps to `~/.omp/agent/config.yml`.
- `AGENTS.md` under `private_agent/` maps to the global OMP instruction file.
- `private_keybindings.yml`, when present, maps to `~/.omp/agent/keybindings.yml`.
- This repository-level `AGENTS.md` is excluded from chezmoi application by `.chezmoiignore`.

Use `chezmoi target-path <source-path>` when a mapping is unclear. Do not rename encoded source paths by hand merely to make them look like target paths.

## Managed Content

Keep only portable, authored configuration that is not plugin-distributable:

- `config.yml`, including model/provider settings, `modelRoles`, and appearance preferences
- global `AGENTS.md`
- `keybindings.yml`

Keep model selection separate from agent behavior. Plugin-provided custom agents should refer to role aliases such as `@review`; define concrete selectors once under `modelRoles` in `config.yml`.

Skills, agents, tools, extensions, commands, rules, prompts, and hooks belong in the separate [`heyskylark/omp-plugins`](https://github.com/heyskylark/omp-plugins) repository, not this source state.

## Never Add

Do not add plugin-distributable capabilities such as skills, agents, tools, extensions, commands, rules, prompts, or hooks.

Never add credentials, sessions, generated state, caches, logs, databases, lock files, or machine-specific paths. In particular, do not add:

- `mcp.json` unless the user explicitly approves a sanitized server subset
- literal MCP tokens, authorization headers, OAuth client secrets, or cookies
- literal provider keys in `models.yml`
- `.env` files
- `agent.db*`, `history.db*`, or `models.db*`
- `sessions/`, `blobs/`, `terminal-sessions/`, `cache/`, or `managed-skills/`
- `*.lock`, `last-changelog-version`, downloaded binaries, or generated models

The active `~/.omp/agent/mcp.json` is intentionally unmanaged. Never import it wholesale. If MCP management is later requested, construct a new source file containing only approved generic definitions and environment-variable or OAuth indirection.

Work-specific configuration belongs in the relevant repository's `.omp/` directory, not here. Machine-specific settings belong in an unmanaged local overlay.

## Change Workflow

Prefer source-first edits:

```sh
chezmoi edit --apply ~/.omp/agent/config.yml
```

When OMP modifies a managed target, inspect and capture only the intended file:

```sh
chezmoi diff ~/.omp/agent/config.yml
chezmoi add ~/.omp/agent/config.yml
```

Before proposing a commit:

1. Run `chezmoi status` and `chezmoi diff`.
2. Review the Git diff for secrets and machine-specific data.
3. Verify changed YAML through OMP's own configuration loading.
4. Verify keybinding changes with `/hotkeys` and appearance changes in the interactive OMP surface.
5. Do not commit or push unless the user explicitly approves the reviewed diff.

Do not enable chezmoi automatic commits or pushes for this repository.
