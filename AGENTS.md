## Boundaries

🚫 Never:

- Commit secrets, API keys, or tokens
- Use rm/rmdir — use `trash` instead
- Use `git rm` except for tracked files intentionally being removed

⚠️ Ask first:

- Changes that affect system-wide config or bootstrap scripts
- Adding new system-wide dependencies
- Modify files outside the current project repo

✅ Always:

- Typecheck when done making a series of code changes
- Prefer running single tests, not the whole test suite
- tmux path: `/opt/homebrew/bin/tmux`

If Claude Code:

- For npm/pnpm builds, dev servers, database ops, downloads: retry with `dangerouslyDisableSandbox: true` on sandbox failure

## Git Workflow

- Branch from `main` for changes
- Commit messages: imperative, concise (e.g., "Add user auth middleware")
- Don't merge long-lived branches without review

### Plans

- Make the plan extremely concise. Sacrifice grammar for concision.
- At the end of each plan, list unresolved questions, if any.

## Dev Server Convention

A dev server (`pnpm dev`) typically runs in a tmux window within the current session. Before starting a new dev server: 0. Check if something is running in the expected port (for pnpm and npm port is 3000) - If yes:
a. Check for an existing tmux window/pane running the dev server (`tmux list-windows`, `tmux list-panes`).
b. If found, restart it there (send `C-c` then re-run the command in that pane).
c. If not found, create a new tmux window in the current session for it. - If not:
a. If not found, create a new tmux window in the current session for it.
Do not start the dev server as a background shell process — use tmux so it persists and is visible.

## Browser Automation

Use `agent-browser` for web automation. Run `agent-browser --help` for all commands.

Core workflow:

1. `agent-browser open <url>` - Navigate to page
2. `agent-browser snapshot -i` - Get interactive elements with refs (@e1, @e2)
3. `agent-browser click @e1` / `fill @e2 "text"` - Interact using refs
4. Re-snapshot after page changes

## Philosophy

This codebase will outlive you. Every shortcut becomes someone else's burden. Every hack compounds into technical debt that slows the whole team down.

You are not just writing code. You are shaping the future of this project. The patterns you establish will be copied. The corners you cut will be cut again.

Fight entropy. Leave the codebase better than you found it.
