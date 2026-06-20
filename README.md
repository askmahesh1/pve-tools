# Proxmox LXC Container Update Automation Script

A robust, zero-dependency Python 3 script to automate running update commands (or any custom shell commands) across running LXC containers in a Proxmox VE (PVE) cluster.

## Features

- **Zero Dependencies**: Written purely in Python 3 using standard library modules. No need to install `pip` packages.
- **SSH Shell Integration**: Automatically runs via your system's native `ssh` client. It seamlessly leverages your active `ssh-agent`, SSH key pairs, and custom SSH configs (`~/.ssh/config`).
- **Cluster-Wide Execution**: Uses Proxmox API (`pvesh get /cluster/resources`) to query and map containers across all nodes in the cluster.
- **Auto-Routing**: Automatically runs commands on the target cluster node where the container resides, leveraging Proxmox's built-in passwordless inter-node SSH.
- **Configurable Container List**:
  - Whitelist: Update only specified VMIDs or names (`-c` / `--containers`).
  - Blacklist: Exclude specific VMIDs or names (`-e` / `--exclude`).
  - Node filtering: Target only containers on a specific node (`-n` / `--node`).
- **Sudo Support**: If your entrypoint SSH user is not `root`, the `-s` / `--sudo` flag runs an interactive password verification (`sudo -v`) at startup, caching your sudo privileges so the script can capture output cleanly.
- **Dry-Run Mode**: Safely preview the container list and exact commands before committing to execution.
- **Detailed Reporting**: Displays stdout/stderr outputs and prints a clean execution time summary at the end.

---

## Installation

Ensure the script is executable:

```bash
chmod +x pveup
```

---

## Shell Integration

To run `pveup` from any directory without needing an alias, add the tool's directory to your shell's `PATH` configuration (e.g., in `~/.zshrc` or `~/.bashrc`):

```bash
# Add pve-tools to PATH
export PATH="/path/to/pve-tools:$PATH"
```

Once loaded (run `source ~/.zshrc`), you can use these commands from anywhere:
- **`pveup lxc`** (or just **`pveup`**): Run with the default container update command.
- **`pveup apt`**: Run package manager updates (`apt-get update && apt-get upgrade -y`) on all running containers.
- **`pveup lxc --dry-run` / `pveup apt -c 101`**: Forward any additional options (like dry-run, container selectors) directly to the script.

---

## Usage Examples

### 1. Run a Dry Run (Recommended first step)
Verify the connection to your Proxmox host and see which containers will be targeted:
```bash
pveup --dry-run
```

### 2. Run Default "update" Command on All Containers
Executes the literal command `update` inside all running LXC containers in the cluster:
```bash
pveup
```

### 3. Run Package Manager Updates
To run standard package updates (e.g., APT updates and upgrades on Debian/Ubuntu):
```bash
pveup apt
```
Or run with a custom command:
```bash
pveup --cmd "apt-get update && apt-get upgrade -y"
```

### 4. Update Specific Containers
Update only VMID `100` and `101`:
```bash
pveup -c 100,101
```

Exclude container named `pihole`:
```bash
pveup -e pihole
```

### 5. Running directly on Proxmox VE Host (Local Mode)
If you copy the script onto your Proxmox host directly, run it with the local flag:
```bash
pveup --local
```

### 6. Using a Non-Root SSH User with Sudo
If your default SSH configuration logs in as a non-root user, use `--sudo` to authenticate:
```bash
pveup --host myuser@pve.home.local --sudo
```

---

## CLI Options

```text
usage: pveup [-h] [--host HOST] [--cmd CMD] [--containers CONTAINERS]
             [--exclude EXCLUDE] [--node NODE] [--dry-run] [--local] [--sudo]
             [action]

Automate command execution (like 'update') across Proxmox LXC containers.

positional arguments:
  action                Type of update to perform: 'lxc' (default container
                        update command) or 'apt' (apt-get update && upgrade).

optional arguments:
  -h, --help            show this help message and exit
  --host HOST, -H HOST  Proxmox host to SSH into (default: root@pve.home.local). 
                        Pass 'local' to run locally on the PVE host.
  --cmd CMD, -C CMD     Command to run inside each container (defaults to
                        'update' for 'lxc' and 'apt-get update && apt-get
                        upgrade -y' for 'apt').
  --containers CONTAINERS, -c CONTAINERS
                        Comma-separated list of container VMIDs or names to update (whitelist).
  --exclude EXCLUDE, -e EXCLUDE
                        Comma-separated list of container VMIDs or names to exclude (blacklist).
  --node NODE, -n NODE  Only update containers on a specific Proxmox cluster node.
  --dry-run, -d         Show what containers would be updated and what commands
                        would run without executing them.
  --local               Shortcut to run locally on the Proxmox host (equivalent to --host local).
  --sudo, -s            Use sudo for Proxmox commands (needed if SSH user is not root).
```

---

## License

MIT License. Feel free to modify and customize.
