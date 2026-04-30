# termio

A terminal-native SSH alias and connection manager for Linux and macOS. A lightweight, POSIX sh replacement for GUI tools like Termius — no Electron, no subscription, no cloud account required.

Works as a full TUI (whiptail menus) or a plain CLI. One shell script, no compiled dependencies.

<img width="1207" height="876" alt="Screenshot_20260430_114144" src="https://github.com/user-attachments/assets/7e06156d-26bb-4d23-9aaa-beda666444ab" />

---

## Features

- **Named SSH aliases** stored in `~/.ssh/config` — aliases work directly with `ssh` too
- **Per-alias key generation** — unique key per alias (ed25519, RSA-4096, or ECDSA-521), copied to remote on add
- **Key rotation** — generate and deploy a new key without touching the old one until confirmed
- **Key age audit** — flag keys older than a configurable threshold; warns at startup
- **Groups / tags** — tag aliases (`homelab`, `work`, `prod`) and filter with `termio ls --group <g>`
- **Snippets** — save and run reusable remote commands across one or more hosts, with optional sudo and group filtering
- **SSH tunnels** — named port-forward profiles with start/stop/status management
- **SSH agent** — load, inspect, and manage per-alias keys in `ssh-agent`
- **One-off commands** — `termio run <alias> <cmd>` without saving a snippet
- **File copy** — `termio copy <alias>:/path /local` using alias credentials automatically
- **Status dashboard** — test all connections in parallel with a live reachability table
- **Connection history** — last-connected timestamp tracked per alias
- **Sync** — mirror aliases, snippets, and preferences to any mounted folder (NAS, cloud drive, USB)
- **Encrypted key sync** — opt-in AES-256 archive of private keys, passphrase never stored
- **Export / import** — portable plaintext format for moving aliases between machines
- **Interactive TUI** via whiptail/newt, with automatic fallback to CLI prompts
- **Tab completion** for bash and zsh — source `termio-completion`
- **Bootstrap** — deploy a snippet widget (`Ctrl+X s`) to any remote host so your saved snippets are available as a fuzzy-searchable command palette in the remote shell (zsh ZLE + bash readline); persistent install or ephemeral `--profile` sessions
- **Per-alias preferences** — auto agent-add and optional password storage per alias
- **Verbose mode** — `-v` flag exposes all background operations

---

## Requirements

| Dependency | Required | Purpose |
|---|---|---|
| `openssh-client` (`ssh`, `ssh-keygen`, `ssh-copy-id`, `ssh-add`) | **Yes** | Everything |
| `whiptail` / `newt` | No | TUI menus (auto-detected; falls back to CLI) |
| `sshpass` | No | Non-interactive key copy on `add`; optional password auth |
| `openssl` | No | Encrypted key sync |
| `wakeonlan` or `python3` | No | Wake-on-LAN (`termio wake`) |
| `fzf` *(on the remote host)* | No | Snippet widget (`termio bootstrap`) — offered for install if missing |

---

## Installation

```sh
sudo curl -fsSL https://raw.githubusercontent.com/dcwestra/termio/main/termio \
    -o /usr/local/bin/termio
sudo chmod +x /usr/local/bin/termio
```

Or manually:

```sh
sudo cp termio /usr/local/bin/termio
sudo chmod +x /usr/local/bin/termio
```

### Tab completion

```sh
# Bash
echo '. /path/to/termio-completion' >> ~/.bashrc

# Zsh
echo '. /path/to/termio-completion' >> ~/.zshrc

# Or system-wide
sudo cp termio-completion /etc/bash_completion.d/termio
```

### TUI (whiptail)

| Distro | Command |
|---|---|
| Debian / Ubuntu / Mint | `sudo apt install whiptail` |
| Fedora / RHEL | `sudo dnf install newt` |
| Arch / Manjaro | `sudo pacman -S libnewt` |
| openSUSE | `sudo zypper install newt` |
| macOS | `brew install newt` |

Or let termio install it: `termio prefer whiptail`

---

## Usage

Run `termio` with no arguments to open the interactive menu.

```
termio [-v] <command> [args]
```

### Alias management

```sh
termio add                        # Guided wizard — alias, host, user, port, group, key type
termio connect <alias>            # Connect to a saved alias
termio connect <alias> --agent    # Connect and auto-load key into ssh-agent first
termio connect <alias> --sshpass   # Connect using a prompted (or stored) password
termio connect <alias> --profile   # Connect with snippet widget active (ephemeral, no install)
termio ls                         # List all aliases with group and last-connected time
termio ls --group <name>          # Filter list to a specific group
termio info <alias>               # Full details: host, key fingerprint, group, last seen
termio edit <alias>               # Edit any saved field
termio rm <alias>                 # Remove alias and optionally delete its key
termio test <alias>               # Test connectivity (non-interactive)
termio rotate <alias>             # Rotate the SSH key for an alias
termio export [file]              # Export aliases to a portable file
termio import <file>              # Import aliases from an export file
```

### One-off commands and file copy

```sh
termio run <alias> <cmd>          # Run a remote command without saving a snippet
termio copy <alias>:/remote /local  # Copy files using alias credentials (sftp)
termio copy /local <alias>:/remote  # Upload files
```

### Status and audit

```sh
termio status                     # Parallel reachability dashboard for all aliases
termio test                       # Same as status — test all connections at once
termio test <alias>               # Test a single alias with detailed output
termio audit                      # Audit SSH key ages — flag keys past rotation threshold
```

### Groups

Tag aliases at add/edit time with comma-separated group names (e.g. `homelab,vpn`). Then:

```sh
termio ls --group homelab
termio snip run deploy --group prod
```

### Snippets

Snippets are saved remote shell commands you can run across multiple hosts at once.

```sh
termio snip                       # List all snippets
termio snip add                   # Add a new snippet
termio snip run <name>            # Run on one or more hosts (interactive selection)
termio snip run <name> --group <g>  # Run on all hosts in a group
termio snip edit <name>           # Edit a snippet
termio snip rm <name>             # Remove a snippet
termio snip info <name>           # Show snippet details
```

Snippets with sudo enabled prompt for the sudo password before each host (or once to share across all targets). Password is never stored.

### SSH tunnels

Named port-forward profiles that survive termio restarts.

```sh
termio tunnel                        # List all tunnels and their status
termio tunnel add <name> <alias> <local>:<host>:<port>  # Define a tunnel
termio tunnel start <name>           # Start a tunnel in the background
termio tunnel stop <name>            # Stop a running tunnel
termio tunnel rm <name>              # Remove a tunnel definition
```

Example:

```sh
termio tunnel add db-tunnel myserver 5432:localhost:5432
termio tunnel start db-tunnel
# Now connect to localhost:5432 to reach the remote database
termio tunnel stop db-tunnel
```

### SSH agent

```sh
termio agent                         # List which alias keys are loaded in ssh-agent
termio agent add [alias]             # Load a specific (or all) alias key(s) into agent
termio agent rm <alias>              # Remove an alias key from the agent
termio agent clear                   # Remove all keys from ssh-agent
```

### Sync

Sync keeps aliases, snippets, and preferences in a folder of your choosing — a NAS share, Nextcloud, Syncthing, or USB drive.

```sh
termio sync                       # Show sync status
termio sync init                  # Configure the sync folder
termio sync detach                # Remove sync, restore files locally
termio sync keys                  # Show key sync status
termio sync keys enable           # Opt in to encrypted key sync
termio sync keys disable          # Remove the encrypted key archive
termio sync keys update           # Rebuild the encrypted key archive
termio sync keys restore          # Restore keys from the archive
```

Key sync uses AES-256-CBC with PBKDF2. The passphrase is never stored — you enter it each time the archive is rebuilt.

### Preferences

```sh
termio prefer                        # Show all current preferences
termio prefer whiptail               # Use TUI dialogs
termio prefer cli                    # Use plain CLI prompts always
termio prefer key_type <type>        # Default key type: ed25519 | rsa | ecdsa
termio prefer audit_threshold <N>    # Flag keys older than N days (default: 90)

# Per-alias preferences
termio prefer <alias>                # Show per-alias settings
termio prefer <alias> auto_agent on  # Auto-add key to ssh-agent on every connect
termio prefer <alias> auto_agent off # Disable
termio prefer <alias> sshpass        # Store an obfuscated password for this alias
termio prefer <alias> sshpass clear  # Remove stored password
```

### Templates

Save a set of defaults (user, port, group, key type) as a named template to pre-fill the `add` wizard.

```sh
termio template list              # Show saved templates
termio template save <name>       # Save a new template (interactive prompts)
termio template rm <name>         # Remove a template
```

When running `termio add`, if templates exist you'll be offered a picker to pre-fill the wizard.

### Clone and rename

```sh
termio clone <alias> [<new-alias>]   # Copy an alias as a starting point — prompts to override fields, generates a new key
termio rename <old> <new>            # Rename an alias cleanly — updates key files and all stored preferences
```

### Tags

```sh
termio tag <alias> <group>        # Add a group tag without going through full edit
termio untag <alias> <group>      # Remove a specific group tag
```

### Open (SFTP)

```sh
termio open <alias>               # Open an interactive SFTP session to an alias
```

### Pinned aliases

Pin aliases to always appear at the top of `termio ls`, regardless of last-connect order.

```sh
termio pin <alias>                # Pin an alias to the top of the list
termio unpin <alias>              # Remove pin
termio pin                        # Show all pinned aliases
```

### Backups

termio automatically backs up `~/.ssh/config` before every destructive change.

```sh
termio backup                     # List available backups (most recent first)
termio backup restore <N>         # Restore backup number N
```

### Connection log

Every `termio connect` session is logged with timestamp, alias, duration, and exit code.

```sh
termio log                        # Show full connection history (last 50 entries)
termio log <alias>                # Filter history to a specific alias
```

### Diff

Compare your local aliases against the sync folder's export — useful when termio is on multiple machines.

```sh
termio diff                       # Show what's changed locally vs the last sync export
```

### Bootstrap

Bootstrap deploys a snippet widget to a remote host so your saved termio snippets are available as a fuzzy-searchable palette in the remote shell via `Ctrl+X s`. It installs a small `shell-init.sh` profile on the remote (using ZLE for zsh and readline bindings for bash) and pushes your current snippet list as a TSV file.

```sh
termio bootstrap <alias>             # Install profile on alias (persistent)
termio bootstrap update <alias>      # Push updated snippets to an already-bootstrapped alias
termio bootstrap remove <alias>      # Remove profile files from alias
termio bootstrap list                # Show all bootstrapped aliases and install dates

termio connect <alias> --profile     # Ephemeral session — profile injected for this session only,
                                     # nothing written to the remote permanently
```

**First-connect auto-prompt** — enable with `termio prefer auto_bootstrap on` and termio will offer to bootstrap any alias that hasn't been set up yet.

**Dependencies on the remote host:** the snippet widget uses `fzf`. If it is not present, termio offers three options:
- Install via the remote host's package manager (`apt`, `dnf`, `pacman`, `brew`, etc.)
- Install from source to `~/.fzf/bin` (no sudo required)
- Skip — the widget will be installed but inactive until fzf is available

The widget is also accessible from the interactive menu: **Bootstrap ▶** in the whiptail TUI, or `bs` in the plain CLI menu.

### Wake-on-LAN

```sh
termio prefer <alias> wol <MAC>   # Store a MAC address for an alias (e.g. aa:bb:cc:dd:ee:ff)
termio prefer <alias> wol clear   # Remove stored MAC
termio wake <alias>               # Send a WOL magic packet to wake the host before connecting
```

Requires `wakeonlan` or `python3`. Sends a broadcast magic packet to `255.255.255.255:9`.

### Other

```sh
termio version                    # Show version
termio help                       # Show full command reference
termio -v <command>               # Verbose — show background operations
```

---

## File layout

```
~/.ssh/config                        SSH aliases (standard format, works with ssh directly)
~/.ssh/id_<type>_<alias>             Per-alias private key (ed25519, rsa, or ecdsa)
~/.config/termio/preferences         Key=value settings (including per-alias prefs)
~/.config/termio/snippets            INI-style snippet storage
~/.config/termio/tunnels             INI-style tunnel definitions
~/.config/termio/tunnel-socks/       SSH control sockets for running tunnels

# Remote host (after bootstrap install)
~/.config/termio/shell-init.sh       Snippet widget profile (sourced by remote shell)
~/.config/termio/snippets.tsv        Snippet list pushed from local machine (name\tcmd\tdesc)
```

### Sync folder layout (when configured)

```
<sync-folder>/termio-sync/
  aliases-export.txt                 Auto-updated after every alias change
  snippets-export.txt                Auto-updated after every snippet change
  preferences                        Symlinked from ~/.config/termio/
  keys.enc                           AES-256 encrypted key archive (opt-in)
  .termio-sync                       Marker file with init metadata
```

---

## How `add` works

1. Prompts for alias name, hostname/IP, port, username, optional note, and group tags
2. Optionally configures a jump/bastion host and advanced SSH options
3. Prompts for key type (ed25519 / rsa / ecdsa) — or uses the configured default
4. Generates a unique key pair for this alias
5. Copies the public key to the remote using `ssh-copy-id` (password used once, never stored)
6. Writes a named block to `~/.ssh/config`
7. After setup, connect with `termio connect <alias>` or directly with `ssh <alias>`

---

## Security notes

- By default, **passwords and passphrases are never written to disk**; sensitive variables are `unset` immediately after use
- Each alias gets its own key — revoking one host doesn't affect others
- The sync key archive is encrypted client-side; the passphrase is never stored
- `ssh-agent` forwarding (`ForwardAgent`) is opt-in per alias at add time
- The `termio prefer <alias> sshpass` feature stores passwords in **obfuscated form** (base64), not encrypted — it is a convenience feature, not a security hardening measure. For sensitive hosts, use key-based authentication

---

## License

MIT
