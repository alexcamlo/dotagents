## Boundaries

🚫 Never:

- Commit secrets, API keys, or tokens
- Use rm/rmdir on project or home files — use `trash` instead (scratch files in /tmp are fine)
- Use `git rm` except for tracked files intentionally being removed
- Touch files, components or features I didn't name. Ask before widening scope.
- Discard, stage or commit unrelated dirty work.

⚠️ Ask first:

- Changes that affect system-wide config or bootstrap scripts
- Adding new system-wide dependencies
- Modify files outside the current project repo

✅ Always:

- Typecheck when done making a series of code changes
- Prefer running single tests, not the whole test suite
- Wrap long or browser commands in `timeout` (e.g. `timeout 120 agent-browser …`); give each agent-browser run a unique `--session` and close it afterwards.
- If ~15 tool calls of reading haven't produced an edit, stop and either edit or report the blocker.

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

## Git Workflow

- Use Conventional Commit specification for commit messages
- Don't merge long-lived branches without review

## Dev Server

- Before starting one, check whether a dev server for this project is already running, and reuse it or restart it on the same port. Don't start another one on a new port.
- Run it however the harness supports (background task, or a herdr/tmux pane if I'm in one). Use `zsh -ic` if it needs env vars from `.zshrc`.
- Stop servers you started when the task is done, unless I'm still using them.

## Browser Automation

Use the harness's built-in preview/browser tools when they exist (e.g. T3 Code preview); otherwise use `agent-browser`. Run `agent-browser --help` for all commands.

Core workflow:

1. `agent-browser open <url>` - Navigate to page
2. `agent-browser snapshot -i` - Get interactive elements with refs (@e1, @e2)
3. `agent-browser click @e1` / `fill @e2 "text"` - Interact using refs
4. Re-snapshot after page changes

## Principles

- Think before coding. State assumptions, surface tradeoffs, push back when warranted.
- Simplicity first. Minimum code that solves the problem. Nothing speculative.
- Surgical changes. Touch only what you must. Clean up only your own mess.
- Goal-driven execution. Define success criteria. Loop until verified.

## Response Endings

When replying to me (not when returning output to a parent agent), end every completed workflow or implementation with exactly one of:

- `Next: <single concrete action>` when follow-up work remains.
- `Next: none — <brief reason>` when the task is complete.

Make the next action executable and specific; never use vague phrases like “continue testing” or “consider improvements.”
