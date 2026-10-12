# dotfiles

zsh, Oh My Zsh and starship config for **Ubuntu Server 24.04+** and **macOS**, managed by [chezmoi](https://chezmoi.io). One command sets up a fresh host. OS differences are handled by templates, and secrets stay out of the repo.

Only `home/` is deployed (see `.chezmoiroot`). Everything at the repo root is documentation or examples.

## Inventory

**Dotfiles** (managed)

| Repo path | Deploys to | What |
|---|---|---|
| `home/dot_zshrc.tmpl` | `~/.zshrc` | Oh My Zsh, plugins, PATH, nvm, Homebrew (macOS), starship init |
| `home/dot_config/zsh/aliases.zsh.tmpl` | `~/.config/zsh/aliases.zsh` | docker compose, `update`, `psa`, `tsdb`, macOS extras |
| `home/dot_config/starship.toml` | `~/.config/starship.toml` | Two-line, p10k-style prompt |
| `home/.chezmoiexternal.toml` | `~/.oh-my-zsh/…` | Oh My Zsh and plugin git clones, pulled weekly |
| `home/.chezmoiscripts/run_onchange_before_10-install-packages.sh.tmpl` | (runs) | Installs the apps below; re-runs when edited |

**Apps** (installed by the script)

| | Ubuntu (apt) | macOS (Homebrew) |
|---|---|---|
| zsh | `zsh`, set as login shell | system `/bin/zsh` (don't install Homebrew's) |
| starship | official installer → `~/.local/bin` | `starship` |
| bat, zoxide, git | `bat`, `zoxide`, `git` | `bat`, `zoxide`, `git` |
| psql | `postgresql-client` | `libpq` (keg-only; `.zshrc` adds it to PATH) |
| Node | nvm + LTS | nvm + LTS |
| Other | `curl`, `command-not-found` | JetBrains Mono Nerd Font |

**Oh My Zsh plugins**
- Built in: `git`, `sudo`, `extract`, `colored-man-pages`, `command-not-found`, `gitignore`, `git-auto-fetch`, and `zoxide` (loaded only if installed).
- External: `you-should-use`, `zsh-bat`, `zsh-autosuggestions`, `zsh-syntax-highlighting` (loaded last).

**Local-only files** (never committed; create them per host)

| File | Purpose | Template |
|---|---|---|
| `~/.pg_service.conf` | `tsdb` connection details | [`examples/pg_service.conf.example`](examples/pg_service.conf.example) |
| `~/.pgpass` (600) | `tsdb` password | [`examples/pgpass.example`](examples/pgpass.example) |
| `~/.zshenv` (600) | Per-host secrets, e.g. macOS keychain unlock over SSH | — |
| `~/.zprofile` | Per-host login setup | — |

## Fresh host

```sh
# Install chezmoi, clone this repo, install apps and deploy dotfiles (asks for sudo on Ubuntu)
sh -c "$(curl -fsLS get.chezmoi.io)" -- -b ~/.local/bin init --apply trial123Zel

# Optional: tsdb
cp ~/.local/share/chezmoi/examples/pg_service.conf.example ~/.pg_service.conf
cp ~/.local/share/chezmoi/examples/pgpass.example ~/.pgpass
chmod 600 ~/.pg_service.conf ~/.pgpass   # then edit both

exec zsh
```

Preview before applying: replace `init --apply` with `init`, then run `chezmoi diff` and `chezmoi apply`.

## Day to day

```sh
chezmoi edit ~/.zshrc        # edit the source template, not the deployed file
chezmoi diff                 # preview
chezmoi apply                # deploy
chezmoi update               # git pull + apply (other hosts)
chezmoi apply -R             # also force-refresh Oh My Zsh and plugins
chezmoi cd                   # shell in the source repo, to commit and push
chezmoi apply --exclude scripts   # deploy without running the install script
```
