---
name: repo-status-handoff
description: Create a concise, evidence-backed handoff of repo/worktree/session state before continuing, resuming, committing, or delegating work.
---

# Repo Status Handoff

Use this skill when the user asks to resume, continue, check status, summarize what is done, prepare a handoff, decide next action, or understand worktree/branch state.

## Rules

- Do not edit files unless the user separately asks for implementation.
- Prefer command/file evidence over conversation memory.
- First establish location and safety:
  - `pwd`
  - `git rev-parse --show-toplevel`
  - `git branch --show-current`
  - `git status --short`
- If a `.pi/worktrees/...` path is mentioned or detected, say clearly whether the session is in that worktree or in the main checkout.
- Check recent commits with a short log when relevant.
- Look for local state files such as `plan.md`, `progress.md`, `context.md`, issue docs, or subagent artifacts when relevant.
- If dev servers/panes matter, inspect the current session manager when available, but do not start/restart anything unless asked.

## Output format

```md
## Repo Status Handoff

- Location: `<pwd>`
- Repo root: `<root>`
- Branch/worktree: `<branch>` / `<worktree-or-main>`
- Git status: clean | dirty summary

### Completed
- ...

### Pending
- ...

### Risks / Blockers
- ...

### Recommended next safe action
- ...

### Evidence checked
- commands/files inspected
```

Keep it concise. If the next action is ambiguous, provide a small numbered menu instead of guessing.
