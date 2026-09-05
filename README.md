# OMP Configs

Portable, user-level [Oh My Pi](https://github.com/can1357/oh-my-pi) configuration managed with [chezmoi](https://www.chezmoi.io/).

## Platform support

This repository supports both Linux and macOS. The managed OMP paths are the same on both platforms:

```text
~/.omp/agent/config.yml
~/.omp/agent/AGENTS.md
~/.omp/agent/keybindings.yml
```

The committed configuration contains no OS-specific absolute paths. Model roles, themes, composer settings, and keybinding action IDs are portable. Credentials and machine-specific overrides remain local to each machine.

Run `omp config path` after installation to confirm the active agent directory. A named OMP profile or `PI_CODING_AGENT_DIR` can intentionally relocate it. `omp config init-xdg` may separately initialize XDG data, state, and cache directories on Linux or macOS; it does not require a different chezmoi source layout.

This repository is intentionally limited to OMP state that is not installed through plugins:

- `config.yml`, including model/provider selection, `modelRoles`, and portable appearance preferences
- global `AGENTS.md`
- `keybindings.yml`, when custom bindings are added

Installable capabilities belong in [`heyskylark/omp-plugins`](https://github.com/heyskylark/omp-plugins). That repository is reserved for skills, custom agents, tools, extensions, commands, rules, prompts, and hooks, following a collection-oriented structure similar to Cursor's plugin repository.

## Managed files

| Chezmoi source | Local OMP target | Purpose |
| --- | --- | --- |
| `private_dot_omp/private_agent/private_config.yml` | `~/.omp/agent/config.yml` | Models, role aliases, and portable OMP settings |
| `private_dot_omp/private_agent/AGENTS.md` | `~/.omp/agent/AGENTS.md` | Global OMP instructions |
| `private_dot_omp/private_agent/private_keybindings.yml` | `~/.omp/agent/keybindings.yml` | Optional custom keybindings; added when needed |

The `private_` prefixes are chezmoi attributes that preserve private filesystem permissions. They are not part of the target names.

The repository-level `AGENTS.md` and `README.md` are listed in `.chezmoiignore`, so chezmoi does not install them into the home directory.

## Division of responsibility

### Keep here

- Portable `config.yml` values
- Model and provider selection
- `modelRoles` consumed by plugin-provided custom agents
- Appearance settings such as theme and composer shape
- Global, installation-wide instructions in `AGENTS.md`
- User keybinding overrides in `keybindings.yml`

### Keep in omp-plugins

- Skills and their supporting files
- Custom task agents
- Custom tools
- Runtime extensions
- Slash commands
- Capability-specific rules and prompts
- Hooks
- Portable MCP or LSP definitions that contain no credentials

### Keep local or project-specific

- Credentials, OAuth state, API keys, and cookies
- Machine-specific model endpoints and secret-manager commands
- Work-specific settings and MCP servers
- Sessions, logs, caches, databases, generated state, and downloaded binaries

## Prerequisites

Install OMP, Git, and chezmoi.

On macOS:

```sh
brew install chezmoi
```

On Linux, use the official installer:

```sh
sh -c "$(curl -fsLS https://get.chezmoi.io)" -- -b ~/.local/bin
```

Ensure `~/.local/bin` is on `PATH`. Distribution packages are also supported:

```sh
# Debian or Ubuntu
sudo apt-get install chezmoi

# Fedora
sudo dnf install chezmoi

# Arch Linux
sudo pacman -S chezmoi
```

Confirm the installation:

```sh
chezmoi --version
omp config path
```

## Set up a new machine

### Standard setup

Let chezmoi clone the repository into its default source directory and apply it:

```sh
chezmoi init --apply https://github.com/heyskylark/omp-configs.git
```

For SSH authentication:

```sh
chezmoi init --apply git@github.com:heyskylark/omp-configs.git
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
git clone https://github.com/heyskylark/omp-configs.git ~/git/omp-configs
```

Configure chezmoi to use that checkout as its source directory by creating `~/.config/chezmoi/chezmoi.toml`:

```toml
sourceDir = "/absolute/path/to/home/git/omp-configs"

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

## Synchronize configuration

### Make local OMP match this repository

Pull and apply in one command:

```sh
chezmoi update
```

To review before applying:

```sh
chezmoi git -- pull --ff-only
chezmoi diff
chezmoi apply
```

If a local target has diverged, inspect the difference rather than forcing the update. Use `chezmoi merge <target>` when both source and target contain changes that must be reconciled.

### Capture local OMP changes

Inspect and capture only the intended managed target:

```sh
chezmoi status
chezmoi diff ~/.omp/agent/config.yml
chezmoi add ~/.omp/agent/config.yml
```

For global instructions:

```sh
chezmoi diff ~/.omp/agent/AGENTS.md
chezmoi add ~/.omp/agent/AGENTS.md
```

For custom keybindings, once `~/.omp/agent/keybindings.yml` exists:

```sh
chezmoi diff ~/.omp/agent/keybindings.yml
chezmoi add ~/.omp/agent/keybindings.yml
```

Review repository changes before committing:

```sh
chezmoi git -- status --short
chezmoi git -- diff
```

Do not enable automatic commits or pushes.

### Edit source first

For deliberate changes, edit and apply the managed target directly:

```sh
chezmoi edit --apply ~/.omp/agent/config.yml
chezmoi edit --apply ~/.omp/agent/AGENTS.md
```

Then inspect both source-to-target and Git differences:

```sh
chezmoi diff
chezmoi git -- diff
```

## Model roles and plugin agents

Concrete model selectors belong under `modelRoles` in `config.yml`. Custom agents distributed through `omp-plugins` should refer to aliases such as `@review` rather than hard-coding a provider and model.

This separation allows plugin behavior to remain portable while each OMP installation controls its own model routing.

Credentials remain in OMP auth storage or environment variables, never in either repository.

## Plugin installation and updates

[`heyskylark/omp-plugins`](https://github.com/heyskylark/omp-plugins) is an OMP-native marketplace of independently installable stacks. Add it and install HStack:

```sh
omp plugin marketplace add heyskylark/omp-plugins
omp plugin install hstack@omp-plugins
```

For local plugin development:

```sh
omp plugin link ~/git/omp-plugins/hstack
```

Refresh the catalog and upgrade HStack with:

```sh
omp plugin marketplace update omp-plugins
omp plugin upgrade hstack@omp-plugins
```

`marketplace update` refreshes marketplace metadata; it does not reinstall plugins. `plugin upgrade` installs the newer declared version. Marketplace startup behavior can also be configured with `marketplace.autoUpdate` as `off`, `notify`, or `auto`.

After changing installed capabilities, run:

```text
/reload-plugins
```

This refreshes skills, slash commands, and MCP servers in the active TUI session. Restart OMP for changed agents, tools, hooks, or extension modules.

## Machine-specific settings

Do not capture hostnames, absolute local paths, local model endpoints, hardware-specific options, or secret-manager commands in the managed `config.yml`.

Keep machine-only overrides in an unmanaged file such as:

```text
~/.config/omp/local.yml
```

Load it from that machine's shell environment:

```sh
export PI_CONFIG_FILES="$HOME/.config/omp/local.yml"
```

`PI_CONFIG_FILES` is a high-precedence overlay. Reserve it for genuine machine constraints rather than ordinary preferences.

## MCP policy

`~/.omp/agent/mcp.json` is intentionally unmanaged. Do not import it wholesale because it may contain literal credentials or work-specific services.

Portable MCP definitions may eventually live in `omp-plugins` when they use hosted OAuth, environment-variable indirection, or reviewed portable commands. Work-specific MCP servers belong in the relevant project's `.omp/mcp.json`.

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

Check source-to-target synchronization:

```sh
chezmoi status
chezmoi diff
```

Check unpublished repository changes:

```sh
chezmoi git -- status --short --branch
chezmoi git -- diff
```

Check OMP configuration loading:

```sh
omp config path
omp config get modelRoles --json
omp config get theme.dark --json
```

Use `/hotkeys` in OMP to verify custom keybindings and inspect the interactive surface after changing appearance settings.

Before pushing, inspect the complete diff and verify that it contains no credentials or machine-specific information.
