# Start Loop

Kick off the autonomous issue loop for the current repository with the `issue-orchestrator` agent.

## Arguments

`$ARGUMENTS`

- If no argument is provided, default to `1` issue.
- If an argument is provided, it must be a positive integer representing how many issues to process.
- If the argument is invalid, stop and tell the user to run `/start-loop <number-of-issues>`.

## Process

1. Parse the requested issue count.
2. **Check for issue-loop labels** using `gh label list --json name | jq -e 'map(select(.name == "type:parent")) | length > 0'` and similar for `autonomy:afk` and `autonomy:hitl` (exit code 0 means exists, non-zero means missing).
3. If any expected labels are missing:
   - **Warn** that label-based filtering is unavailable.
   - **Fall back** to slower body-based and comment-based classification for autonomy detection.
   - **Recommend** running `/setup-issue-loop` to create the labels for faster, more reliable filtering.
4. Use the `issue-orchestrator` agent for the current repository.
5. Instruct it to process the highest-priority actionable child issues for the current repo, up to the requested count, re-evaluating the remaining queue after each completed or skipped issue.
6. Let the orchestrator run the full selection -> implementation -> review -> finalization loop.
7. Return a concise outcome for each issue, including why it was selected or skipped, what verification ran, whether a commit was produced, and the manual next step for skipped HITL or blocked issues when known.

## Notes

- The current working directory is the workspace boundary for the loop.
- Parent PRD or epic issues labeled `type:parent` are context sources, not direct loop candidates.
- Prefer a two-stage GitHub fetch: first prefilter metadata with `gh issue list --search 'is:open label:"autonomy:afk" -label:"type:parent" -label:"autonomy:hitl"'`, then fetch full bodies/comments only for the highest-ranked candidate set.
- **If labels are missing**, the orchestrator must fall back to slower body/comment inspection instead of creating labels automatically.
- Read a parent issue only when the child issue explicitly points to it in `Parent` or `Context to Read`.
- If a child issue depends on missing or implicit parent context, skip it as under-specified instead of guessing.
- Issues skipped as HITL, blocked, conflicting, or already in progress should not count toward the requested total.
- Run `/setup-issue-loop` to create or update the required labels for faster, more reliable filtering.
