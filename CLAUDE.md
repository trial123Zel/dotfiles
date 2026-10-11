# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Personal zsh dotfiles managed by [chezmoi](https://chezmoi.io). The targets are **Ubuntu Server 24.04+ and macOS**, and the repo is meant to grow. It has no build, lint or test tooling.

`.chezmoiroot` points chezmoi at `home/`. Everything under `home/` maps onto `~`, while files at the repo root (like this one) are never deployed. chezmoi's naming rules apply: `dot_` becomes `.`, a `.tmpl` suffix marks a Go template, and `.chezmoiscripts/run_onchange_*` scripts re-run whenever their content changes.

- `home/dot_zshrc.tmpl` → `~/.zshrc`
- `home/dot_config/zsh/aliases.zsh.tmpl` → `~/.config/zsh/aliases.zsh`, sourced explicitly by `.zshrc`. Don't put `.zsh` files in `$ZSH_CUSTOM`: Oh My Zsh auto-sources them, so they would load twice.
- `home/dot_config/starship.toml` → `~/.config/starship.toml`. It reproduces a Powerlevel10k look: a two-line `╭─`/`╰─❯` layout with a `$fill` separator. The python module shows only an active virtualenv because its file detection is turned off on purpose. Username and hostname are always shown.
- `home/.chezmoiexternal.toml` clones Oh My Zsh and the third-party plugins as git repos and pulls them weekly. Add new plugins here and to `plugins=(...)` in `.zshrc`.
- `home/.chezmoiscripts/run_onchange_before_10-install-packages.sh.tmpl` installs dependencies (apt, or Homebrew on macOS), starship, nvm and node LTS, and on Ubuntu runs `chsh` to zsh.
- `home/.chezmoi.toml.tmpl` generates `~/.config/chezmoi/chezmoi.toml`. It sets `sourceDir` to the repo root and pins `umask = 0o022`.

## Workflow

Edit the source here, never the deployed files in `~`; chezmoi overwrites those.

```sh
chezmoi diff                       # preview what apply would change
chezmoi apply                      # deploy (add --exclude scripts to skip the sudo install script)
chezmoi execute-template < home/dot_zshrc.tmpl | zsh -n          # render + syntax-check a template
chezmoi execute-template --override-data '{"chezmoi":{"os":"darwin"}}' < FILE   # render as macOS
STARSHIP_CONFIG=home/dot_config/starship.toml starship prompt    # preview prompt
```

On a new machine: `sh -c "$(curl -fsLS get.chezmoi.io)" -- -b ~/.local/bin init --apply <github-user>`

## Cross-platform rules

- Branch on OS with `{{ if eq .chezmoi.os "darwin" }}` / `{{ else if eq .chezmoi.osRelease.id "ubuntu" }}`. `.chezmoi.osRelease` doesn't exist on macOS, so always test `.chezmoi.os` first.
- Don't hardcode home paths, usernames or tool versions. Use `$HOME`, derive paths at runtime (as `NODE_PATH` does from `${commands[node]}`), and check that a command exists with `(( $+commands[x] ))`.
- On macOS, Homebrew is initialized at the top of `.zshrc` (`/opt/homebrew`, falling back to `/usr/local`), followed by `typeset +x FPATH`: `brew shellenv` exports `FPATH`, and an inherited `FPATH` from a different or removed zsh breaks `compinit` with "function definition file not found". Use the system `/bin/zsh` on macOS, not Homebrew's zsh. Keg-only formulas like `libpq` need explicit `PATH` entries there. On Ubuntu, `bat` is named `batcat`, and the zsh-bat plugin handles that.
- Starship is initialized last in `.zshrc`. The Oh My Zsh theme is intentionally empty.

## Secrets and privacy

This repo is meant to be **public**. Keep secrets, private IPs and hostnames out of it. For example, `tsdb` connects with `psql "service=tsdb"`, which reads the connection details from the untracked `~/.pg_service.conf` and the password from `~/.pgpass`. `~/.zshenv` and `~/.zprofile` are deliberately unmanaged and machine-local. On the macOS host, `~/.zshenv` (mode 600) unlocks the login keychain for every SSH session, so automations work. If a secret ever needs to be deployed, use chezmoi's age encryption or a password-manager template function. Never commit plaintext.

Per the user's global rules, Python dependencies always go in a virtualenv, never on the host.
