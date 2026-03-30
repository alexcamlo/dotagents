# Issue Loop Contract

This file is the single source of truth for the shared issue workflow used by `issue-orchestrator`, `coder`, and `reviewer`.

For compact worked examples, see `~/.agents/contracts/issue-loop-examples.md`.

## Core Invariants

- One handoff represents exactly one issue.
- Use the exact metadata fields, exact section names, and exact section order defined here.
- Required sections must always appear. If a section is empty, write `None`.
- Route work from `Stage` + `Status`. Do not add `Next Action` or `Next Agent` fields.
- `Issue` is the canonical issue reference only, such as `#123`.
- `Attempt` counts implementation cycles, not message count. It starts at `1` and stays constant through selection, implementation, review, and finalization for the same reviewed artifact.
- `Workspace Root` is the absolute current working directory captured at run start. It stays constant for the same issue across all stages.
- Increment `Attempt` only when blocking reviewer feedback sends work back to the coder, or when commit-time hooks change tracked files and force re-review.
- Allow up to five reviewer-driven correction rounds after the initial implementation review. If the work still does not converge, stop the issue as blocked instead of looping forever.
- `Branch` and `Worktree` names must include the issue number and a short slug.
- `Worktree` must be an absolute path.
- The resolved `Worktree` path must stay inside the resolved `Workspace Root`.
- Use a deterministic worktree path inside the workspace, preferably `Workspace Root/.worktrees/<issue-number>-<slug>`.
- `Changed Files` must use repo-relative paths only.
- `Verification: not_run` is allowed only for blocked handoffs.
- If git hooks modify tracked files during final commit, prior approval is invalidated. The workflow must re-enter review on a new `Attempt`.

## Roles By Stage

- `selection` -> `issue-orchestrator`
- `implementation` -> `coder`
- `review` -> `reviewer`
- `finalization` -> `coder`

## Issue Labels And Fields

- Use `type:parent` for PRDs, epics, and other container issues that provide context but are not direct `/start-loop` selection candidates.
- Use `autonomy:afk` for implementation-ready child issues that are eligible for the autonomous loop.
- Use `autonomy:hitl` for issues that intentionally require human decisions before coding.
- Keep priority, severity, and release labels as ranking signals after the candidate set is filtered.
- Builder-ready child issues should provide these body fields explicitly: `Autonomy`, `Parent`, `Context to Read`, `Blocked by`, `Success Criteria`, `Verification`, and `Out of Scope`.
- `Parent` must be an issue reference such as `#123` or `None`.
- `Context to Read` must enumerate the exact additional material the orchestrator should open, such as issue numbers, comment URLs, or repo file paths. Use `None` when the child issue is self-contained.
- `Blocked by` must enumerate explicit issue dependencies or `None`.
- `Success Criteria`, `Verification`, and `Out of Scope` should be concrete enough for the coder and reviewer to stay in scope without inferring missing product intent.

## Selection Rules

- The orchestrator must build its candidate set from actionable child issues before ranking them.
- Prefer a cheap metadata pass first, for example via `gh issue list --search 'is:open label:"autonomy:afk" -label:"type:parent" -label:"autonomy:hitl"'`, before fetching full issue bodies or comments.
- Full issue bodies and comments should be fetched only for the top candidate or short list of top candidates that survive metadata ranking.
- Issues labeled `type:parent` are context sources, not direct selection candidates for `/start-loop`.
- Treat `autonomy:afk` as the explicit include signal for autonomous selection.
- Use explicit `AFK` or `HITL` markers in the issue body, issue comments, and labels as strong signals.
- If neither marker is explicit, infer `AFK` for clear bug fixes, narrow implementation tasks, and issues with concrete acceptance criteria.
- Infer `HITL` for issues that require architectural choice, product or design judgment, missing constraints, or ambiguous scope.
- Issues marked or inferred as `HITL` are not actionable for the autonomous loop unless the user explicitly says to include them.
- When skipping `HITL` work, record the concrete manual next step or human decision needed to make it actionable again.
- Read a parent PRD or epic issue only when the selected child issue explicitly references it in `Parent` or `Context to Read`.
- Do not automatically ingest a parent PRD or epic issue just because one exists.
- Parse explicit `Blocked by` sections in issue bodies. An issue is not actionable while any referenced blocker is still open.
- If a builder-ready child issue appears to depend on parent or linked context but does not explicitly identify that context in `Parent` or `Context to Read`, stop it as under-specified instead of guessing.
- Treat missing required builder-ready fields as an autonomy stop when they prevent deterministic selection or bounded implementation.
- Treat issues as non-actionable when concrete evidence sources such as open PRs, local or remote branches, linked issues, or recent issue comments show the work is already in progress.
- In multi-issue runs, avoid selecting issues that likely conflict because they touch the same subsystem, same files, or the same unresolved dependency chain.
- In multi-issue runs, re-evaluate the remaining candidate set after each issue is committed, blocked, or skipped instead of locking the whole batch from the initial ranking.
- When two otherwise actionable feature issues compete, prefer the smallest end-to-end tracer bullet.

## Legal States And Routing

| Stage | Status | Next step |
| --- | --- | --- |
| `selection` | `selected` | hand off to `coder` |
| `selection` | `blocked` | stop or skip issue |
| `implementation` | `implemented` | hand off to `reviewer` |
| `implementation` | `blocked` | stop or skip issue |
| `review` | `changes_requested` | hand back to `coder` |
| `review` | `approved` | hand off to `coder` for finalization |
| `review` | `blocked` | stop or skip issue |
| `finalization` | `committed` | done |
| `finalization` | `blocked` | stop or skip issue |

`review + blocked` is reserved for true autonomy-stop states where the reviewer cannot give the coder a normal correction loop. Use `review + changes_requested` when the reviewer can state concrete required outcomes.

## Metadata Block

Write metadata at the top of every handoff as plain Markdown `Key: value` lines in this exact order.

1. `Role`
2. `Status`
3. `Stage`
4. `Issue`
5. `Attempt`
6. `Workspace Root`
7. `Reason` when `Status` is `blocked`, `changes_requested`, or `approved`
8. `Branch` for `implementation`, `review`, and `finalization`
9. `Worktree` for `implementation`, `review`, and `finalization`
10. `Verification` for `implementation`, `review`, and `finalization`
11. `Commit` for `finalization + committed`, and for `finalization + blocked` only when a real commit exists

## Verification Vocabulary

Allowed values for `Verification` are:

- `passed`
- `failed`
- `partial`
- `not_run`

Per-state verification rules:

| Stage | Status | Allowed `Verification` |
| --- | --- | --- |
| `selection` | `selected` | none |
| `selection` | `blocked` | none |
| `implementation` | `implemented` | `passed`, `partial` |
| `implementation` | `blocked` | `passed`, `partial`, `failed`, `not_run` |
| `review` | `changes_requested` | `passed`, `partial`, `failed` |
| `review` | `approved` | `passed` |
| `review` | `blocked` | `passed`, `partial`, `failed`, `not_run` |
| `finalization` | `committed` | `passed` |
| `finalization` | `blocked` | `passed`, `partial`, `failed`, `not_run` |

Additional verification rules:

- `implementation + implemented` must never use `failed` or `not_run`.
- `review + approved` requires `Verification: passed`.
- `finalization + committed` requires `Verification: passed`.
- `finalization + committed` must report commands rerun against the exact committed artifact, not only the pre-commit worktree.

## Section Formatting Rules

- `Summary`, `Approval Rationale`, and `Commit Message` use short prose.
- `Summary` should be about 2-4 sentences.
- Most other sections use bullet lists.
- `Verification Details` uses one bullet per command with the command and concise outcome.
- `Changed Files` uses one bullet per repo-relative path.
- Empty required sections use `None`.

Example `Verification Details` bullet:

```md
- `pnpm test src/foo.test.ts` - passed: 12 tests
```

## Canonical Templates

### `selection + selected`

```md
Role: issue-orchestrator
Status: selected
Stage: selection
Issue: #123
Attempt: 1
Workspace Root: /absolute/path/to/current-cwd

## Summary
Short prose.

## Priority Rationale
- Why this issue beat other open candidates.
- Why it was actionable now and not `HITL`, blocked, conflicting, or already in progress.

## Success Criteria
- Canonical acceptance conditions for this issue.

## Repo Context
- Relevant repo facts already inspected.

## Raw Issue Context
- Title: ...
- Labels: ...
- Milestone: ...
- Body: ...
- Comments: ...
- Linked Context: ...

## Open Questions
- None
```

### `selection + blocked`

```md
Role: issue-orchestrator
Status: blocked
Stage: selection
Issue: #123
Attempt: 1
Workspace Root: /absolute/path/to/current-cwd
Reason: Short reason the issue cannot be selected for autonomous work.

## Summary
Short prose.

## Priority Rationale
- Why this issue was considered.
- Why it is `HITL`, blocked by open work, conflicting, or already in progress.

## Success Criteria
- None

## Repo Context
- Relevant repo facts already inspected.

## Raw Issue Context
- Title: ...
- Labels: ...
- Milestone: ...
- Body: ...
- Comments: ...
- Linked Context: ...

## Unblock Conditions
- What manual next step, decision, or repo state change must be true before this issue can be retried.

## Open Questions
- Any unresolved ambiguity.
```

### `implementation + implemented`

```md
Role: coder
Status: implemented
Stage: implementation
Issue: #123
Attempt: 1
Workspace Root: /absolute/path/to/current-cwd
Branch: issue/123-short-slug
Worktree: /absolute/path/to/current-cwd/.worktrees/123-short-slug
Verification: passed

## Summary
Short prose.

## Changed Files
- path/to/file.ts

## Verification Details
- `command` - outcome

## Implementation Decisions
- Key design choice or tradeoff.

## Feedback Resolution
- None

## Known Risks / Follow-ups
- None

## Open Questions
- None
```

For retry handoffs with `Attempt > 1`, `Feedback Resolution` must map each reviewer `Required Outcome` to what changed, or explain why the work is still blocked.

### `implementation + blocked`

```md
Role: coder
Status: blocked
Stage: implementation
Issue: #123
Attempt: 2
Workspace Root: /absolute/path/to/current-cwd
Reason: Short reason coding could not complete autonomously.
Branch: issue/123-short-slug
Worktree: /absolute/path/to/current-cwd/.worktrees/123-short-slug
Verification: partial

## Summary
Short prose.

## Changed Files
- path/to/file.ts

## Verification Details
- `command` - outcome

## Implementation Decisions
- Key design choice or tradeoff.

## Feedback Resolution
- Required Outcome: ...
  Result: ...

## Known Risks / Follow-ups
- Residual risk or follow-up.

## Unblock Conditions
- What must be true before the coder can continue.

## Open Questions
- Any unresolved ambiguity.
```

### `review + changes_requested`

```md
Role: reviewer
Status: changes_requested
Stage: review
Issue: #123
Attempt: 1
Workspace Root: /absolute/path/to/current-cwd
Reason: Short reason approval is not granted.
Branch: issue/123-short-slug
Worktree: /absolute/path/to/current-cwd/.worktrees/123-short-slug
Verification: failed

## Summary
Short prose.

## Changed Files
- path/to/file.ts

## Verification Details
- `command` - outcome

## Blocking Findings
- Concrete blocking problem.

## Required Outcomes
- What must be true before approval.

## Non-blocking Notes
- Optional follow-up or polish note.

## Open Questions
- None
```

### `review + approved`

```md
Role: reviewer
Status: approved
Stage: review
Issue: #123
Attempt: 1
Workspace Root: /absolute/path/to/current-cwd
Reason: Short reason the work is clear to commit.
Branch: issue/123-short-slug
Worktree: /absolute/path/to/current-cwd/.worktrees/123-short-slug
Verification: passed

## Summary
Short prose.

## Changed Files
- path/to/file.ts

## Verification Details
- `command` - outcome

## Approval Rationale
Short prose.

## Blocking Findings
- None

## Non-blocking Notes
- Optional follow-up or polish note.

## Open Questions
- None
```

### `review + blocked`

```md
Role: reviewer
Status: blocked
Stage: review
Issue: #123
Attempt: 2
Workspace Root: /absolute/path/to/current-cwd
Reason: Short reason the reviewer cannot provide a normal correction loop.
Branch: issue/123-short-slug
Worktree: /absolute/path/to/current-cwd/.worktrees/123-short-slug
Verification: not_run

## Summary
Short prose.

## Changed Files
- path/to/file.ts

## Verification Details
- `command` - outcome

## Blocking Findings
- Blocking evidence observed by the reviewer.

## Required Outcomes
- What must be true to unblock, if known.

## Non-blocking Notes
- None

## Unblock Conditions
- Missing access, missing clarity, or other autonomy stop.

## Open Questions
- Any unresolved ambiguity.
```

### `finalization + committed`

```md
Role: coder
Status: committed
Stage: finalization
Issue: #123
Attempt: 1
Workspace Root: /absolute/path/to/current-cwd
Branch: issue/123-short-slug
Worktree: /absolute/path/to/current-cwd/.worktrees/123-short-slug
Verification: passed
Commit: 0123456789abcdef0123456789abcdef01234567

## Summary
Short prose.

## Changed Files
- path/to/file.ts

## Verification Details
- `command` - outcome against the committed artifact

## Commit Message
Exact commit message text.

## Known Risks / Follow-ups
- None

## Open Questions
- None
```

### `finalization + blocked`

```md
Role: coder
Status: blocked
Stage: finalization
Issue: #123
Attempt: 2
Workspace Root: /absolute/path/to/current-cwd
Reason: Short reason finalization could not complete cleanly.
Branch: issue/123-short-slug
Worktree: /absolute/path/to/current-cwd/.worktrees/123-short-slug
Verification: failed
Commit: 0123456789abcdef0123456789abcdef01234567

## Summary
Short prose.

## Changed Files
- path/to/file.ts

## Verification Details
- `command` - outcome against the current artifact

## Commit Message
Exact commit message text, or intended message if commit was never created.

## Known Risks / Follow-ups
- Residual risk or follow-up.

## Unblock Conditions
- What must be true before finalization can resume.

## Open Questions
- Any unresolved ambiguity.
```

For `finalization + blocked`, include `Commit` only when a real commit exists.
