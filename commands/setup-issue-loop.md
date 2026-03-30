# Setup Issue Loop

Bootstrap or update the issue-loop GitHub labels for the current repository.

## Scope

This command is repo-scoped. It operates on the repository associated with the current working directory.

## Labels Managed

- `type:parent` — for PRDs, epics, and container issues that provide context but are not direct `/start-loop` candidates
- `autonomy:afk` — for implementation-ready child issues eligible for the autonomous loop
- `autonomy:hitl` — for issues that intentionally require human decisions before coding

## Process

1. Check the current repository context using `gh repo view --json nameWithOwner`.
2. For each label above, check if it exists using `gh label list --json name | jq -e 'map(select(.name == "<label-name>")) | length > 0'` (exit code 0 means exists, non-zero means missing).
3. Create or update each label idempotently using `gh label create <name> --description <desc> --color <color> --force`.
4. Return a concise summary of which labels were missing vs already present.

## Colors and Descriptions

| Label | Color | Description |
|-------|-------|-------------|
| `type:parent` | `0052CC` | PRD, epic, or container issue providing context for child issues |
| `autonomy:afk` | `0E8A16` | Ready for autonomous implementation; no human decisions required |
| `autonomy:hitl` | `B60205` | Requires human-in-the-loop decisions before coding |

## Output Format

Return a summary in this format:

```
Issue Loop Labels — <owner>/<repo>

Already present: type:parent, autonomy:afk
Created:         autonomy:hitl

Status:          All 3 labels present and configured
Next step:       Run /start-loop to begin processing AFK issues
```

If any `gh` command fails, stop and report the error with the exact command that failed.
