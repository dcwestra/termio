# termio

A terminal-native SSH alias and connection manager for Linux. A lightweight, POSIX sh replacement for GUI tools like Termius — no Electron, no subscription, no cloud account required.

Works as a full TUI (whiptail menus) or a plain CLI. One shell script, no compiled dependencies.

---

## Features

- **Named SSH aliases** stored in `~/.ssh/config` — aliases work directly with `ssh` too
- **Automatic key generation** — unique ed25519 key per alias, copied to the remote on add
- **Interactive TUI** via whiptail/newt, with automatic fallback to CLI prompts
- **Key rotation** — generate and deploy a new key without touching the old one until confirmed
- **Jump / bastion host** support
- **Advanced SSH options** — `ServerAliveInterval`, `ForwardAgent`, `LocalForward`
- **Snippets** — save and run reusable remote commands across one or more hosts, with optional sudo
- **Sync** — mirror aliases, snippets, and preferences to any mounted folder (NAS, cloud drive, USB)
- **Encrypted key sync** — opt-in AES-256 archive of private keys, passphrase never stored
- **Export / import** — portable plaintext format for moving aliases between machines
- **Verbose mode** — `-v` flag exposes all background operations

---

## Requirements

| Dependency | Required | Purpose |
|---|---|---|
| `openssh-client` (`ssh`, `ssh-keygen`, `ssh-copy-id`) | **Yes** | Everything |
| `whiptail` / `newt` | No | TUI menus (auto-detected; falls back to CLI) |
| `sshpass` | No | Non-interactive key copy on `add` |
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

To enable the TUI, install whiptail for your distro:

| Distro | Command |
|---|---|
| Debian / Ubuntu / Mint | `sudo apt install whiptail` |
| Fedora / RHEL | `sudo dnf install newt` |
| Arch / Manjaro | `sudo pacman -S libnewt` |
| openSUSE | `sudo zypper install newt` |

Or let termio install it for you: `termio prefer whiptail`

---

## Usage

Run `termio` with no arguments to open the interactive menu.

```
termio [-v] <command> [args]
```

### Alias management

```sh
termio add                   # Add a new SSH alias (guided wizard)
termio connect <alias>       # Connect to a saved alias
termio ls                    # List all aliases
termio list                  # Same as ls
termio info <alias>          # Show alias details and key fingerprint
termio edit <alias>          # Edit an alias
termio rm <alias>            # Remove an alias and its key
termio test <alias>          # Test connectivity (non-interactive)
termio rotate <alias>        # Rotate the SSH key for an alias
termio export [file]         # Export aliases to a portable file
termio import <file>         # Import aliases from an export file
```

### Snippets

Snippets are saved remote shell commands you can run across multiple hosts at once.

```sh
termio snip                  # List all snippets
termio snip add              # Add a new snippet
termio snip run <name>       # Run a snippet on one or more hosts
termio snip edit <name>      # Edit a snippet
termio snip rm <name>        # Remove a snippet
termio snip info <name>      # Show snippet details
```

When running a snippet with sudo enabled, termio prompts for the sudo password before each host (or once if you choose to share it across all targets).

### Sync

Sync keeps your aliases, snippets, and preferences in a folder of your choosing — a NAS share, a cloud-synced folder, or a USB drive.

```sh
termio sync                  # Show sync status
termio sync init             # Configure the sync folder
termio sync detach           # Remove sync, restore files locally
termio sync keys             # Show key sync status
termio sync keys enable      # Opt in to encrypted key sync
termio sync keys disable     # Remove the encrypted key archive
termio sync keys update      # Rebuild the encrypted key archive
```

Key sync uses AES-256-CBC with PBKDF2. The passphrase is never stored — you enter it each time the archive is rebuilt.

### Settings

```sh
termio prefer                # Show current UI preference
termio prefer whiptail       # Use TUI dialogs
termio prefer cli            # Use plain CLI prompts
termio version               # Show version
termio help                  # Show help
```

### Flags

```sh
termio -v <command>          # Verbose — show background operations
termio --verbose <command>   # Same
```

---

## File layout

```
~/.ssh/config                        SSH aliases (standard format, compatible with ssh directly)
~/.ssh/id_ed25519_<alias>            Per-alias private key
~/.config/termio/preferences         Key=value settings
~/.config/termio/snippets            INI-style snippet storage
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

1. Prompts for alias name, hostname/IP, port, username, optional note
2. Optionally configures a jump/bastion host and advanced SSH options
3. Generates a unique `ed25519` key pair for this alias
4. Copies the public key to the remote using `ssh-copy-id` (password used once, never stored)
5. Writes a named block to `~/.ssh/config`
6. After setup, connect with `termio connect <alias>` or directly with `ssh <alias>`

---

## Security notes

- Passwords and passphrases are **never written to disk**
- Sensitive variables are `unset` immediately after use
- Each alias gets its own key — revoking access for one host doesn't affect others
- The sync key archive is encrypted client-side; the passphrase is never stored anywhere
- `ssh-agent` forwarding (`ForwardAgent`) is opt-in per alias

---

## License

MIT
