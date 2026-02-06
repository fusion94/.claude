# .claude

Personal [Claude Code](https://docs.anthropic.com/en/docs/claude-code) configuration repository containing reusable slash commands for software development workflows.

## What Is This?

Claude Code supports user-level custom commands stored in `~/.claude/commands/`. These commands are available across all projects and can be invoked as slash commands (e.g., `/create-pr`, `/code-stats`) within any Claude Code session.

This repository tracks those commands in version control so they can be shared, versioned, and restored across machines.

## Commands

| Command | Description |
|---------|-------------|
| `/add-to-changelog` | Adds a new entry to a project's `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/) conventions. Accepts a version, change type, and message. |
| `/analyze-issue` | Fetches a GitHub issue by number and generates a full technical specification including problem statement, implementation plan, test plan, and affected files. |
| `/code-stats` | Generates a comprehensive code statistics report across all git repositories in the current directory, including commit activity, test vs. application code breakdown, weekly velocity, collaboration metrics, and repo health. |
| `/create-command` | A meta-command that assists in creating new Claude Code commands through a guided interview process covering purpose, category, location, and pattern selection. |
| `/create-issue` | Researches the codebase and creates a well-structured GitHub issue with classification, affected areas, acceptance criteria, and proper labeling using `gh`. |
| `/create-pr` | Creates a branch, formats code, splits changes into logical conventional commits, pushes, and opens a pull request with a structured description via `gh`. |
| `/create-prd` | Same guided workflow as `/create-command`, oriented toward generating product requirement documents. |
| `/dev` | Starts npm dev servers, checking for port conflicts first and falling back to alternative package managers if needed. |

## Repository Structure

```
~/.claude/
├── .gitignore        # Tracks .gitignore, LICENSE, README, commands/, and scripts/
├── LICENSE           # Apache License 2.0
├── commands/         # Slash command definitions (markdown files)
│   ├── add-to-changelog.md
│   ├── analyze-issue.md
│   ├── code-stats.md
│   ├── create-command.md
│   ├── create-issue.md
│   ├── create-pr.md
│   ├── create-prd.md
│   └── dev.md
├── scripts/          # Helper scripts used by commands
│   └── fetch-github-issue.sh
└── README.md
```

## Setup

Clone into your home directory so Claude Code discovers the commands automatically:

```bash
git clone git@github.com:fusion94/.claude.git ~/.claude
```

If you already have a `~/.claude` directory, back it up first or merge the `commands/` directory manually.

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- [GitHub CLI (`gh`)](https://cli.github.com/) authenticated (required by `/create-pr`, `/create-issue`, `/analyze-issue`, `/code-stats`)

## Usage

From any Claude Code session, invoke a command with its slash name:

```
/create-pr #42
/analyze-issue 15
/add-to-changelog 1.2.0 added "New search feature"
/code-stats 2026-01-01
/create-issue Add rate limiting to the API
/dev
```

## Scripts

The `scripts/` directory contains helper scripts used by commands:

| Script | Used By | Description |
|--------|---------|-------------|
| `fetch-github-issue.sh` | `/analyze-issue` | Fetches GitHub issue details via the REST API using `curl`, bypassing interactive permission prompts. |

## Adding New Commands

Create a new markdown file in `commands/` and it will automatically be available as a slash command. Use `/create-command` for a guided workflow, or add the file directly:

```bash
vim ~/.claude/commands/my-command.md
```

## License

This project is licensed under the [Apache License 2.0](LICENSE).
