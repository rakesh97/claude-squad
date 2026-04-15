# Claude Squad [![CI](https://github.com/smtg-ai/claude-squad/actions/workflows/build.yml/badge.svg)](https://github.com/smtg-ai/claude-squad/actions/workflows/build.yml) [![GitHub Release](https://img.shields.io/github/v/release/smtg-ai/claude-squad)](https://github.com/smtg-ai/claude-squad/releases/latest)

[Claude Squad](https://smtg-ai.github.io/claude-squad/) is a terminal app that manages multiple [Claude Code](https://github.com/anthropics/claude-code), [Codex](https://github.com/openai/codex), [Gemini](https://github.com/google-gemini/gemini-cli) (and other local agents including [Aider](https://github.com/Aider-AI/aider)) in separate workspaces, allowing you to work on multiple tasks simultaneously.


![Claude Squad Screenshot](assets/screenshot.png)

### Highlights
- **Multi-project dashboard** — manage sessions across any number of repos from one window
- **Import existing sessions** — bring in running tmux sessions, or resume saved conversations from Claude Code and Codex
- Complete tasks in the background (including yolo / auto-accept mode!)
- Manage instances and tasks in one terminal window
- Review changes before applying them, checkout changes before pushing them
- Each task gets its own isolated git workspace, so no conflicts

<br />

https://github.com/user-attachments/assets/aef18253-e58f-4525-9032-f5a3d66c975a

<br />

> **Note:** This is a personal fork of [smtg-ai/claude-squad](https://github.com/smtg-ai/claude-squad), maintained because upstream development has stopped. Install from this fork rather than Homebrew or the upstream install script.

### Installation

Install with a single command (clones the fork, builds from source, installs to `~/.local/bin`, and sets up the `cs` alias):

```bash
curl -fsSL https://raw.githubusercontent.com/rakesh97/claude-squad/main/install.sh | bash
```

The script installs the required dependencies (`go`, `git`, `tmux`, `gh`) via your system package manager if they're missing, caches the source tree at `~/.cache/claude-squad-src`, and builds the binary with `go build`.

Re-run the same command at any time to update to the latest `main`.

Environment overrides:

| Variable | Default | Description |
|----------|---------|-------------|
| `BIN_DIR` | `$HOME/.local/bin` | Where the binary is installed |
| `SRC_DIR` | `$HOME/.cache/claude-squad-src` | Where the source is cached |
| `INSTALL_NAME` | `claude-squad` | Binary name |
| `BRANCH` | `main` | Branch to build from |
| `NO_ALIAS` | _(unset)_ | Set to any value to skip creating the `cs` alias |

Example — skip the alias and install under a custom name:

```bash
curl -fsSL https://raw.githubusercontent.com/rakesh97/claude-squad/main/install.sh \
  | NO_ALIAS=1 INSTALL_NAME=my-cs bash
```

#### Migrating from the Homebrew version

If you previously ran `brew install claude-squad`, remove it and its `cs` symlink first, otherwise the old version will shadow this one:

```bash
brew uninstall claude-squad
rm -f "$(brew --prefix)/bin/cs"
```

Also remove any existing `alias cs=...` line from your shell rc file — the install script will add a fresh one pointing at the new binary.

### Prerequisites

- [tmux](https://github.com/tmux/tmux/wiki/Installing)
- [gh](https://cli.github.com/)

### Usage

```
Usage:
  cs [flags]
  cs [command]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  debug       Print debug information like config paths
  help        Help about any command
  reset       Reset all stored instances
  version     Print the version number of claude-squad

Flags:
  -y, --autoyes          [experimental] If enabled, all instances will automatically accept prompts for claude code & aider
  -h, --help             help for claude-squad
  -p, --program string   Program to run in new instances (e.g. 'aider --model ollama_chat/gemma3:1b')
```

Run the application with:

```bash
cs
```
NOTE: The default program is `claude` and we recommend using the latest version.

<br />

<b>Using Claude Squad with other AI assistants:</b>
- For [Codex](https://github.com/openai/codex): Set your API key with `export OPENAI_API_KEY=<your_key>`
- Launch with specific assistants:
   - Codex: `cs -p "codex"`
   - Aider: `cs -p "aider ..."`
   - Gemini: `cs -p "gemini"`
- Make this the default, by modifying the config file (locate with `cs debug`)

<br />

#### Menu
The menu at the bottom of the screen shows available commands: 

##### Instance/Session Management
- `n` - Create a new session (quick — title only)
- `N` - Create a new session with full options (title, prompt, working directory, branch name, profile)
- `I` - Import an existing session (tmux, Claude Code, or Codex conversations)
- `R` - Rename the selected session
- `D` - Kill (delete) the selected session
- `↑/j`, `↓/k` - Navigate between sessions

##### Actions
- `↵/o` - Attach to the selected session to reprompt
- `ctrl-q` - Detach from session
- `s` - Commit and push branch to github
- `c` - Checkout. Commits changes and pauses the session
- `r` - Resume a paused session
- `?` - Show help menu

##### Navigation
- `tab` - Switch between preview tab and diff tab
- `q` - Quit the application
- `shift-↓/↑` - scroll in diff view

### Creating Sessions with Shift+N

The `N` (Shift+N) overlay provides full control over new sessions:

| Field | Description |
|-------|-------------|
| **Profile** | Select which agent to use (if multiple profiles configured) |
| **Session Title** | Display name (up to 100 characters, visual-only) |
| **Prompt** | Initial prompt to send to the agent |
| **Working Directory** | Path to the git repo (defaults to CWD, can target any repo) |
| **Branch Name** | Custom branch name, or leave empty to auto-generate from title |
| **Branch Picker** | Check out an existing branch instead of creating a new one |

Titles are decoupled from branch names — you can use any characters in titles without affecting git operations.

### Importing Sessions

Press `I` to open the import overlay with three sources:

- **tmux** — lists running tmux sessions not managed by Claude Squad
- **claude code** — discovers saved conversations from `~/.claude/projects/` and resumes them via `claude --resume`
- **codex** — discovers saved conversations from `~/.codex/session_index.jsonl` and resumes them via `codex resume`

Use `←`/`→` to switch sources, `↑`/`↓` to select a session, and `Tab` to the Import button. Already-imported sessions are automatically filtered out.

### Multi-Project Support

Claude Squad can be launched from any directory — it no longer requires a git repository in the current working directory. Each session independently tracks its own repo and working directory. When sessions span multiple repos, the repo name is shown next to each session in the list.

### Configuration

Claude Squad stores its configuration in `~/.claude-squad/config.json`. You can find the exact path by running `cs debug`.

#### Profiles

Profiles let you define multiple named program configurations and switch between them when creating a new session. When more than one profile is defined, the session creation overlay shows a profile picker that you can navigate with `←`/`→`.

To configure profiles, add a `profiles` array to your config file and set `default_program` to the name of the profile to select by default:

```json
{
  "default_program": "claude",
  "profiles": [
    { "name": "claude", "program": "claude" },
    { "name": "codex", "program": "codex" },
    { "name": "aider", "program": "aider --model ollama_chat/gemma3:1b" }
  ]
}
```

Each profile has two fields:

| Field     | Description                                              |
|-----------|----------------------------------------------------------|
| `name`    | Display name shown in the profile picker                 |
| `program` | Shell command used to launch the agent for that profile  |

If no profiles are defined, Claude Squad uses `default_program` directly as the launch command (the default is `claude`).

### FAQs

#### Failed to start new session

If you get an error like `failed to start new session: timed out waiting for tmux session`, update the
underlying program (ex. `claude`) to the latest version.

### How It Works

1. **tmux** to create isolated terminal sessions for each agent
2. **git worktrees** to isolate codebases so each session works on its own branch
3. A simple TUI interface for easy navigation and management

### What's New in v2.0.0

**Performance** — ~85% reduction in subprocess spawning for smoother UI on macOS:
- Preview refresh reduced from 10/sec to 2/sec
- SHA256 hashing replaced with direct byte comparison
- Git diff only computed when content actually changes
- Redundant preview captures eliminated via dirty flag

**Session management improvements:**
- Rename sessions with `R` (works on running sessions)
- Titles increased to 100 characters, decoupled from branch names
- Custom branch names in Shift+N overlay
- Live progress messages during session startup
- Agent sessions auto-configured with recommended flags

**Bug fixes:**
- Kill/push confirmations no longer freeze the UI
- Sessions start in the correct working directory
- Resumed conversations find their saved session data
- Branch names preserve uppercase letters

### License

[AGPL-3.0](LICENSE.md)

### Star History

[![Star History Chart](https://api.star-history.com/svg?repos=smtg-ai/claude-squad&type=Date)](https://www.star-history.com/#smtg-ai/claude-squad&Date)
