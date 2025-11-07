dothub
=====

Minimal dotfile manager with a curated hub browser. Clone and link repos into your config, or browse a YAML-powered index of dotfile setups sorted by GitHub stars.

Features
- Universal linking: replace `~/.config/<target>` with a symlink to a repo in your store
- Fast hub view: table of curated repos with star counts, filters, and a loading spinner
- User-local store by default; configurable via environment
- Shell completions for bash, zsh, fish, PowerShell, and elvish

Installation
- User-local (no sudo):
  - `cargo install --path .`
- System-wide binary:
  - `cargo build --release`
  - `sudo install -m 0755 target/release/dothub /usr/local/bin/dothub`
- Completions:
  - Bash: `dothub completions bash | sudo tee /etc/bash_completion.d/dothub > /dev/null`
  - Zsh: `dothub completions zsh > "${fpath[1]}/_dothub"`
  - Fish: `dothub completions fish > ~/.config/fish/completions/dothub.fish`
  - PowerShell: `dothub completions powershell | Out-String | Set-Content $PROFILE.CurrentUserAllHosts`
  - Elvish: `dothub completions elvish > ~/.elvish/lib/dothub.elv`

Environment
- `DOTHUB_DIR`: override the store path (default resolves to XDG data dir, e.g. `~/.local/share/dothub`).
- `GITHUB_TOKEN`: optional token to speed up hub star fetching via GraphQL and raise rate limits.

Usage
- Browse hub (from default hub.yml):
  - `dothub`
  - `dothub nvim,tmux` (filtered view)
- Install a repo into the store:
  - `dothub install https://github.com/hygo-nvim`
- Link a repo to your config target:
  - `dothub link hygo-nvim nvim`  → `~/.config/nvim -> <store>/hygo-nvim`
- Update all repos in the store:
  - `dothub update`
- Show active links under `~/.config` that point into the store:
  - `dothub active`
- List or remove installed repos:
  - `dothub list`
  - `dothub remove <name>`
  - Install with explicit name: `dothub install https://github.com/foo/bar my-bar`

Commands
- `dothub [TYPE[,TYPE...]] [--url <yaml-url>]`
  - Loads a YAML index (default: `https://raw.githubusercontent.com/huncholane/dothub/main/hub.yml`) and prints a table of repos:
    - Columns: Rank, Stars, Installed (y/n), Source
    - Spinner shown while fetching
    - Post-table tips about `GITHUB_TOKEN` as applicable
  - Filters by type(s) when provided (`nvim,tmux` or `nvim tmux`).
  - Use `--url` to point at your own index file.
- `dothub install <repo-url> [name]`
- `dothub link <name> <target>`
- `dothub update`
- `dothub active`
- `dothub list`
- `dothub remove <name>`
- `dothub completions <shell>`

Hub Index Format (hub.yml)
- A mapping of types to repo URL(s). Example:
  - `nvim: [https://github.com/hygo-nvim, https://github.com/LazyVim/LazyVim]`
  - `tmux: https://github.com/gpakosz/.tmux`
  - `zsh: [https://github.com/ohmyzsh/ohmyzsh, https://github.com/sorin-ionescu/prezto]`

Stars and Performance
- Without `GITHUB_TOKEN`: dothub uses REST per repository. Works everywhere but is slower and rate-limited.
- With `GITHUB_TOKEN`: dothub batches many repos in a single GraphQL request for better speed.
- Heuristic for owner-only URLs (`https://github.com/<owner>`): dothub tries `<owner>/<owner>`. Prefer explicit `owner/repo` URLs for accurate stars.
- Generate a token at https://github.com/settings/personal-access-tokens and export it as `GITHUB_TOKEN`.

Store and Permissions
- Default store is user-local (XDG): `~/.local/share/dothub`.
- Override with `DOTHUB_DIR` if you want a custom path.
- Linking targets in your home never require sudo. Installing/updating only require permissions to your chosen store.

Troubleshooting
- Hub YAML looks stale:
  - dothub sends cache-busting headers; if you still see staleness, try a commit-pinned URL or pass a `?ts=<now>` query param with `--url`.
- Star counts show 0 unexpectedly:
  - Confirm URLs are repository URLs, not org pages. Add `GITHUB_TOKEN` to avoid REST limits.
- Permission denied writing the store:
  - Use the default user-local store or set `DOTHUB_DIR` to a path you own.

Uninstall
- Remove the binary from your PATH (e.g., `/usr/local/bin/dothub`) and delete your local store if desired.
