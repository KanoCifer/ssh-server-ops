# ssh-server-ops

Perform remote server operations via SSH. All credentials are managed through environment variables — no plaintext IPs or passwords stored in the repo.

## Skills

| Skill            | Purpose                                                                              | Trigger                                                        |
| ---------------- | ------------------------------------------------------------------------------------ | -------------------------------------------------------------- |
| `ssh-server-ops` | Run ops on a remote server (status / logs / restart / containers / nginx / database) | User mentions "server / ops / restart / logs / containers / …" |
| `setup-ssh`      | Initialize / update SSH connection info, guide setting env vars                      | User mentions "configure connection / add server / init ssh"   |

## Installation

### Claude Code Plugin (recommended)

Single plugin containing both `setup-ssh` + `ssh-server-ops` skills — one install covers everything:

```bash
# Add the marketplace
claude plugins marketplace add KanoCifer/ssh-server-ops

# Install plugin
claude plugins install ssh-server-ops
```

Both skills become available automatically after install — no extra configuration needed.

Or use `/plugin` interactively in the Claude prompt:

```
/plugin marketplace add KanoCifer/ssh-server-ops
/plugin install ssh-server-ops
```

Local dev load:

```bash
claude --plugin-dir /path/to/ssh-server-ops
```

### Manual (without plugin)

Copy the skill directories into Claude Code's skills path:

```bash
# As user-level skills (available globally)
cp -r skills/ssh-server-ops ~/.claude/skills/
cp -r skills/setup-ssh ~/.claude/skills/

# Or as project-level skills (in the project's .claude/skills/)
mkdir -p .claude/skills
cp -r skills/ssh-server-ops .claude/skills/
cp -r skills/setup-ssh .claude/skills/
```

### Cross-Agent via npx skills

Works with 70+ coding agents (Claude Code, Codex, Cursor, etc.):

```bash
npx skills add KanoCifer/ssh-server-ops
```

## Configuration

Run `/setup-ssh` to initialize the server connection.

Credentials are provided via environment variables. Set them in `~/.zshrc` (or `~/.bash_profile`):

```bash
export SERVER_IP='<server IP or domain>'
export SERVER_USER='<SSH username>'
export SERVER_SSH_KEY='~/.ssh/id_ed25519'   # key path
export SERVER_SSH_PORT='22'                  # optional, default 22
export SUDO_SSH_PASSWORD='<sudo password>'   # needed for root access
```

To switch servers, simply override the relevant `export`.

## Usage

```text
# First, configure the connection
> Add a new server                   # triggers /setup-ssh

# Then run ops
> Show me what Docker containers are running   # triggers ssh-server-ops → environment discovery
> nginx is returning 502, help me check         # triggers ssh-server-ops → log triage + incident response
> Restart the backend service                   # triggers ssh-server-ops → service restart
```

## Workflow

```
shell env ($SERVER_IP / $SERVER_USER / $SERVER_SSH_KEY / $SUDO_SSH_PASSWORD)
        │
        ▼
ssh-server-ops skill
        │
        ├── Environment discovery — docker / systemd / supervisor / disk
        ├── Log triage            — site logs / docker logs / journalctl
        ├── Service restart       — docker compose / systemctl / supervisor
        ├── Container ops         — exec / inspect / export compose
        ├── Nginx config          — nginx -t first, then reload
        ├── Database              — mysql / psql (password asked at runtime)
        └── Incident response     — parallel legwork → root cause → recovery
```

## Project Structure

```
.
├── .claude-plugin/
│   ├── plugin.json         # single plugin definition (skills array)
│   └── marketplace.json    # marketplace publish config
├── LICENSE
├── CLAUDE.md              # project-internal dev conventions
├── README.md
├── README.en.md           # this file
└── skills/
    ├── setup-ssh/
    │   └── SKILL.md      # connection info initialization
    └── ssh-server-ops/
        ├── SKILL.md      # ops workflow + RED gate
        ├── evals/
        │   └── evals.json # evaluation cases
        └── references/
            ├── server-info.md       # connection template (placeholders, committed)
            ├── server-info.local.md # real credentials (gitignored)
            └── bt-panel.md          # BT Panel command reference
```

## Security

- IP / username / key path / sudo password are **all environment variables**, never stored in the repo
- `[RED]` operations (`rm -rf` / `DROP` / `iptables`) require explicit user confirmation
- Always run `nginx -t` before reloading nginx

## License

[MIT](LICENSE)
