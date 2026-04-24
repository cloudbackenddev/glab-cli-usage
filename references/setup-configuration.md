# Setup & Configuration Reference

## Installation

### Homebrew (macOS/Linux)

```bash
brew install glab
```

### APT (Debian/Ubuntu)

```bash
# Add the GitLab GPG key and repository
curl -fsSL "https://gitlab.com/gitlab-org/cli/-/releases/permalink/latest/downloads/glab_amd64.deb" -o glab.deb
sudo dpkg -i glab.deb
```

### Windows (WinGet / Scoop)

```powershell
# WinGet
winget install GLab.GLab

# Scoop
scoop install glab
```

### Binary Download

Download the appropriate binary from the [releases page](https://gitlab.com/gitlab-org/cli/-/releases) for Linux, macOS, or Windows.

### Go Install

```bash
go install gitlab.com/gitlab-org/cli/cmd/glab@main
```

---

## Authentication

### Interactive Login

```bash
# Login to gitlab.com (default)
glab auth login

# Login to a self-managed instance
glab auth login --hostname gitlab.example.com
```

The interactive flow supports:
- **OAuth** — Opens a browser for GitLab OAuth flow (recommended for gitlab.com)
- **Personal Access Token (PAT)** — Paste a token when prompted

### Token-Based Login (Non-Interactive)

```bash
# Using stdin
echo "glpat-xxxxxxxxxxxx" | glab auth login --hostname gitlab.example.com --stdin

# Using environment variable (skips stored auth)
export GITLAB_TOKEN="glpat-xxxxxxxxxxxx"
```

### Verify Auth Status

```bash
# Check all authenticated hosts
glab auth status

# Check a specific host
glab auth status --hostname gitlab.example.com
```

---

## Authentication Types

| Method | How to Use | Best For |
|--------|-----------|----------|
| **OAuth** | `glab auth login` → select OAuth | gitlab.com users |
| **Personal Access Token** | `glab auth login` → paste token | Self-managed instances |
| **Environment Variable** | `export GITLAB_TOKEN=...` | CI/CD, scripting |
| **CI Job Token** | Auto-detected in GitLab CI | Pipeline automation |

### Required Token Scopes

For full functionality, create a PAT with these scopes:
- `api` — Full API access
- `read_repository` / `write_repository` — Git operations

---

## Multi-Host Configuration

glab supports multiple authenticated GitLab instances simultaneously.

```bash
# Add another host
glab auth login --hostname gitlab.internal.company.com

# Check all authenticated hosts
glab auth status

# Target a specific host for a command
glab issue list --hostname gitlab.internal.company.com

# Or use the GITLAB_HOST environment variable
GITLAB_HOST=gitlab.internal.company.com glab issue list
```

### Host Auto-Detection

When inside a Git directory, glab reads the remotes to determine which GitLab instance to use. If the remote hostname matches an authenticated host, that host is used automatically.

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `GITLAB_TOKEN` | Authentication token (takes precedence over stored auth) |
| `GITLAB_HOST` / `GL_HOST` | Override the GitLab hostname |
| `GITLAB_CLIENT_ID` | Custom OAuth application client ID |
| `NO_PROMPT` | Set to `1` to disable interactive prompts |
| `NO_COLOR` | Disable colored output |
| `GLAB_CONFIG_DIR` | Override config directory location |
| `GLAB_DEBUG_HTTP` | Enable HTTP request debugging |
| `GLAB_CHECK_UPDATE` | Set to `0` to disable update checks |
| `BROWSER` | Override the default browser for `--web` commands |
| `VISUAL` / `EDITOR` | Preferred editor (in precedence order) |
| `GLAMOUR_STYLE` | Markdown rendering style (see charmbracelet/glamour) |
| `REMOTE_ALIAS` / `GIT_REMOTE_URL_VAR` | Override the Git remote to use |

### CI Auto-Login

When `GLAB_ENABLE_CI_AUTOLOGIN=true` is set and running inside GitLab CI (`GITLAB_CI` is present), glab auto-authenticates using:
1. `CI_SERVER_FQDN` as the hostname
2. `CI_JOB_TOKEN` as the token

Note: `CI_JOB_TOKEN` has [limited scope](https://docs.gitlab.com/ci/jobs/ci_job_token/#job-token-access) — not all API endpoints are accessible.

---

## Configuration

### Config Commands

```bash
# Set a config value
glab config set editor vim
glab config set browser firefox

# Get a config value
glab config get editor

# Set config for a specific host
glab config set --hostname gitlab.example.com editor nano
```

### Config File Location

The config file is stored at:
- **macOS/Linux**: `~/.config/glab-cli/config.yml`
- **Windows**: `%APPDATA%\glab-cli\config.yml`

Override with `GLAB_CONFIG_DIR`.

---

## Shell Completion

```bash
# Bash
glab completion -s bash > /etc/bash_completion.d/glab

# Zsh
glab completion -s zsh > "${fpath[1]}/_glab"

# Fish
glab completion -s fish > ~/.config/fish/completions/glab.fish

# PowerShell
glab completion -s powershell > glab.ps1
```

---

## Verifying Setup

```bash
# Check version
glab version

# Check authentication
glab auth status

# List issues to verify connectivity
glab issue list

# Check which repo context is detected
glab repo view
```
