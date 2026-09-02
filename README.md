# OMP Hstack

Portable, user-level [Oh My Pi](https://github.com/can1357/oh-my-pi) configuration managed with [chezmoi](https://www.chezmoi.io/).

The repository is the source of truth for selected files under `~/.omp/agent`. It intentionally excludes credentials, MCP configuration, sessions, logs, caches, databases, downloaded tools, and machine-specific state.

## Managed files

| Chezmoi source | Local OMP target |
| --- | --- |
| `private_dot_omp/private_agent/AGENTS.md` | `~/.omp/agent/AGENTS.md` |
| `private_dot_omp/private_agent/private_config.yml` | `~/.omp/agent/config.yml` |

The `private_` prefixes are chezmoi attributes that preserve private filesystem permissions. They are not part of the target names.

Repository-level `AGENTS.md` and `README.md` are listed in `.chezmoiignore`, so chezmoi does not install them into the home directory.

## Prerequisites

Install OMP, Git, and chezmoi. On macOS:

```sh
brew install chezmoi
```

Confirm the installation:

```sh
chezmoi --version
```

## Set up a new machine

### Standard setup

Let chezmoi clone the repository into its default source directory and apply it:

```sh
chezmoi init --apply https://github.com/heyskylark/omp-hstack.git
```

For SSH authentication:

```sh
chezmoi init --apply git@github.com:heyskylark/omp-hstack.git
```

Inspect the result:

```sh
chezmoi source-path
chezmoi managed --path-style absolute
chezmoi status
```

OMP credentials and service authorizations are deliberately not included. Authenticate OMP separately on each machine.

### Use an existing clone in `~/git`

Clone the repository:

```sh
git clone https://github.com/heyskylark/omp-hstack.git ~/git/omp-hstack
```

Configure chezmoi to use that checkout as its source directory by creating `~/.config/chezmoi/chezmoi.toml`:

```toml
sourceDir = "/absolute/path/to/home/git/omp-hstack"

[git]
autoAdd = false
autoCommit = false
autoPush = false
```

Use the actual absolute home path; chezmoi does not expand `~` in this value. Then apply and inspect:

```sh
chezmoi apply
chezmoi status
```

## Make local OMP match this repository

There are two supported workflows.

### One-command update

Pull the latest repository revision and apply it:

```sh
chezmoi update
```

This is the normal command on secondary machines.

### Review before applying

Pull the source repository explicitly, inspect the target changes, then apply:

```sh
chezmoi git -- pull --ff-only
chezmoi diff
chezmoi apply
```

A clean result is:

```sh
chezmoi status
```

with no output.

To apply only one managed target:

```sh
chezmoi diff ~/.omp/agent/config.yml
chezmoi apply ~/.omp/agent/config.yml
```

If the local target has diverged, chezmoi prompts before overwriting it. Inspect the difference rather than forcing the update. Use `chezmoi merge <target>` when both source and target contain changes that must be reconciled.

## Make this repository match local OMP

Use this workflow after changing OMP through `/settings`, `omp config set`, or direct edits under `~/.omp/agent`.

### Capture an updated managed file

First inspect what changed:

```sh
chezmoi status
chezmoi diff ~/.omp/agent/config.yml
```

Copy the local target back into chezmoi source state:

```sh
chezmoi add ~/.omp/agent/config.yml
```

For the global OMP instructions:

```sh
chezmoi diff ~/.omp/agent/AGENTS.md
chezmoi add ~/.omp/agent/AGENTS.md
```

Review the repository diff before committing:

```sh
chezmoi git -- status --short
chezmoi git -- diff
```

Then commit and push from the source repository:

```sh
chezmoi git -- add .
chezmoi git -- commit -m "Update OMP configuration"
chezmoi git -- push
```

Do not enable automatic commits or pushes. OMP configuration may execute tools, hooks, extensions, and stdio MCP commands; every source change should be reviewed.

### Edit source first

For deliberate changes, editing the chezmoi source and applying it immediately avoids a capture step:

```sh
chezmoi edit --apply ~/.omp/agent/config.yml
```

The same pattern works for other managed targets:

```sh
chezmoi edit --apply ~/.omp/agent/AGENTS.md
```

Afterward:

```sh
chezmoi diff
chezmoi git -- diff
```

`chezmoi diff` compares source state with local targets. `chezmoi git -- diff` shows changes waiting to be committed in the repository.

## Add a new managed OMP file

Create the target file in its normal OMP location, then add it to chezmoi:

```sh
chezmoi add ~/.omp/agent/keybindings.yml
```

Examples for custom agents and skills:

```sh
chezmoi add ~/.omp/agent/agents/reviewer.md
chezmoi add ~/.omp/agent/skills/example/SKILL.md
```

OMP discovers user agents from `~/.omp/agent/agents/*.md`. Skills must use the non-nested layout:

```text
~/.omp/agent/skills/<skill-name>/SKILL.md
```

Supporting scripts, templates, and reference files can live inside the same skill directory and should be added individually or by adding that directory recursively.

Before committing any newly managed path, verify where chezmoi will install it:

```sh
chezmoi source-path ~/.omp/agent/skills/example/SKILL.md
chezmoi target-path "$(chezmoi source-path ~/.omp/agent/skills/example/SKILL.md)"
```

## Model roles and custom agents

Keep agent behavior separate from model selection:

- define concrete model selectors under `modelRoles` in `config.yml`;
- refer to those roles from agent frontmatter with aliases such as `@review`;
- keep credentials in OMP auth storage or environment variables, never in this repository.

This allows a model to be changed once without rewriting every agent definition.

## Machine-specific settings

Do not capture hostnames, absolute local paths, local model endpoints, hardware-specific options, or secret-manager commands in the managed `config.yml`.

When a machine needs an OMP-only override, keep it in an unmanaged file such as:

```text
~/.config/omp/local.yml
```

and load it from that machine's shell environment:

```sh
export PI_CONFIG_FILES="$HOME/.config/omp/local.yml"
```

`PI_CONFIG_FILES` is a high-precedence overlay and can override project settings. Reserve it for true machine constraints rather than ordinary preferences.

## MCP policy

`~/.omp/agent/mcp.json` is intentionally unmanaged. Do not run:

```sh
chezmoi add ~/.omp/agent/mcp.json
```

The existing local file may contain literal credentials or work-specific services.

If portable MCP management is added later, construct a new source file containing only an explicitly reviewed subset. Safe entries should use one of:

- definition-only hosted OAuth endpoints, with credentials retained in local OMP auth storage;
- environment-variable indirection for tokens;
- portable stdio commands with reviewed and preferably pinned packages.

Work-specific MCP servers belong in the relevant project's `.omp/mcp.json`. Slack is currently omitted.

## Never manage

Do not add any of the following:

```text
~/.omp/agent/mcp.json
~/.omp/agent/agent.db*
~/.omp/agent/history.db*
~/.omp/agent/models.db*
~/.omp/agent/sessions/
~/.omp/agent/blobs/
~/.omp/agent/terminal-sessions/
~/.omp/agent/cache/
~/.omp/agent/managed-skills/
~/.omp/agent/*.lock
~/.omp/agent/last-changelog-version
~/.omp/logs/
~/.omp/cache/
~/.omp/run/
~/.omp/wt/
~/.omp/natives/
~/.omp/stats.db*
~/.omp/install-id
```

Also exclude `.env` files, API keys, tokens, cookies, OAuth client secrets, authorization headers, session exports, generated models, and downloaded binaries.

## Routine checks

### Is the local OMP configuration synchronized?

```sh
chezmoi status
chezmoi diff
```

No output means the managed targets match chezmoi source state.

### Does the repository have unpublished changes?

```sh
chezmoi git -- status --short --branch
chezmoi git -- diff
```

### Which paths are managed?

```sh
chezmoi managed --path-style absolute
```

### Does OMP load the expected configuration?

```sh
omp config path
omp config get theme.dark --json
```

### Final pre-push review

```sh
chezmoi status
chezmoi diff
chezmoi git -- diff --check
chezmoi git -- status --short --branch
```

Inspect the complete staged diff and verify that it contains no credentials or machine-specific information before pushing.
