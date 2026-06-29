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
- herdr path: `herdr`

## Worktree / CWD Safety

Before editing files:

- Check `pwd`, repo root, branch, and `git status --short`.
- If the user mentions `.pi/worktrees/...`, operate there, not the main checkout.
- If the intended worktree/repo is ambiguous, ask before editing.
- Use relative paths from the repo root for read/write/edit tools.
- In the final response, mention worktree/branch when relevant.

## Tool Discipline

- Before `edit`, read enough surrounding context so `oldText` is unique.
- For repeated text, use a script or larger unique blocks; do not submit duplicate `oldText` replacements.
- For search terms beginning with `-`, use `rg -- '--flag'` so ripgrep does not treat the pattern as an option.
- Do not pass multiple paths as one string to file tools; search paths separately or use shell.
- Prefer targeted, repo-relative paths.

## Stock / Project Patterns First

Prefer official generators, stock components, and existing project patterns before custom code.

For UI/frontend work:

- Inspect the project setup first: `components.json`, local `components/ui/*` wrappers, nearby screens/components, and package scripts.
- Prefer configured shadcn components/generator when applicable.
- Use existing local primitives, design-system tokens, CSS variables, aliases, and framework/library recommended patterns.
- Match adjacent typography, spacing, border radius, shadows, density, and responsive behavior before inventing styles.
- Avoid raw hex values or one-off classes in components when project tokens/classes exist.
- Keep close to stock shadcn/project output unless the user asks for a custom visual direction.
- Preserve accessibility: labels, focus states, keyboard use, touch targets, contrast.
- If deviating from stock/project patterns, explain why first.

If Claude Code:

- For npm/pnpm builds, dev servers, database ops, downloads: retry with `dangerouslyDisableSandbox: true` on sandbox failure

## Git Workflow

- Branch from `main` for changes
- Commit messages: imperative, concise (e.g., "Add user auth middleware")
- Don't merge long-lived branches without review

### Plans

- Make the plan extremely concise. Sacrifice grammar for concision.
- At the end of each plan, list unresolved questions, if any.

## Subagent Workflow Shortcuts

When the user asks for common review/delegation workflows, prefer existing pi-subagents prompt shortcuts instead of inventing long ad-hoc orchestration:

- `/parallel-review` for fresh-context parallel review of current work.
- `/parallel-review autofix` to synthesize and apply only fixes worth doing now.
- `/review-loop` for parent-controlled worker/reviewer/fix cycles until clean or capped.
- `/gather-context-and-clarify` before editing unclear work.
- `/parallel-context-build` or `/parallel-handoff-plan` for large unknown tasks.
- `/parallel-cleanup` for post-implementation cleanup review.

## Dev Server Convention

A dev server (`pnpm dev`) should run in the current terminal session manager, not as a background shell process.

Before starting a new dev server:

0. Detect the session manager:
   - If inside tmux (`$TMUX` is set), use tmux windows/panes. tmux path: `/opt/homebrew/bin/tmux`.
   - If inside herdr, use the corresponding herdr window/pane workflow.
   - If inside neither, prefer creating/entering a tmux session before starting the server.
1. Check if something is running on the expected port (for pnpm and npm, port 3000).
2. If yes, check for an existing tmux/herdr window or pane running the dev server.
3. If found, restart it there (send `C-c`, then re-run the command in that pane).
4. If not found, create a new tmux/herdr window or pane in the current session and run it there.

When creating a new tmux/herdr window or pane to run any command that depends on shell environment variables (for example API keys from `.zshrc`), start an interactive shell first or use `zsh -ic "..."` so the shell init files are loaded. Do not assume newly created windows/panes inherit `.zshrc` environment.

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
