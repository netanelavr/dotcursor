# dotcursor

Simple rules & commands — smarter Cursor.

## Rules (`.cursor/rules/`)

| Rule | What it does |
|------|-------------|
| `ask-before-acting` | Asks clarifying questions before starting any task |
| `git-safety` | Blocks automatic git commands — always asks first |
| `pr-code-review` | Challenge-first code review before publishing PRs |
| `feature-implementation` | 4-phase planning methodology for complex features |
| `docs-location` | Keeps docs organized by purpose |
| `mcp-tool-definition` | Standardized MCP tool documentation |


## Commands (`.cursor/commands/`)

| Command | What it does |
|---------|-------------|
| `back-to-master-delete-branch` | Clean up after merge |
| `create-pr` | Create a pull request with structured description |
| `handle-pr-comments` | Address PR review comments |
| `optimize-cursor-context` | Read-only audit of always-on agent context (rules, skills, plugins, git noise) plus token estimates and optimization ideas |
| `self-review` | Self-review changes before submitting |

## Usage

Copy `.cursor/` into your project root to activate rules and commands.
