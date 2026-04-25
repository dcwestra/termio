# termio

A terminal-native SSH alias and connection manager for Linux. A lightweight, POSIX sh replacement for GUI tools like Termius — no Electron, no subscription, no cloud account required.

Works as a full TUI (whiptail menus) or a plain CLI. One shell script, no compiled dependencies.

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
termio connect <alias> --sshpass  # Connect using a prompted (or stored) password
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
termio copy <alias>:/remote /local  # Copy files using alias credentials (scp)
termio copy /local <alias>:/remote  # Upload files
```

### Status and audit

```sh
termio status                     # Parallel reachability dashboard for all aliases
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
~/.config/termio/tunnel-pids/        PID files for running tunnels
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
