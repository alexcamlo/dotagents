# Issue Loop Examples

These are compact golden examples for the contract in `~/.agents/contracts/issue-loop.md`.

## Happy Path

### Orchestrator -> coder

```md
Role: issue-orchestrator
Status: selected
Stage: selection
Issue: #42
Attempt: 1
Workspace Root: /Users/win/src/project

## Summary
Issue `#42` is the highest-priority actionable bug because it carries a `priority:high` label and blocks the next release milestone. The repository already has a failing test path around the broken serializer, so the coder can work from a concrete target.

## Priority Rationale
- `priority:high` outranks the rest of the open queue.
- The issue is assigned to the current release milestone.
- The issue is labeled `autonomy:afk` and provides its own success criteria and verification commands.

## Success Criteria
- Fix the serializer bug for empty payloads.
- Preserve existing non-empty payload behavior.
- Add or update targeted regression coverage.

## Repo Context
- `src/serializer.ts` owns the failing path.
- `tests/serializer.test.ts` already covers adjacent cases.

## Raw Issue Context
- Title: Empty payload crashes serializer
- Labels: `bug`, `priority:high`, `autonomy:afk`
- Milestone: `v1.8.0`
- Body: `Autonomy: AFK`, `Parent: None`, `Context to Read: issue comment 2`, `Blocked by: None`, targeted success criteria, `Verification: pnpm test tests/serializer.test.ts && pnpm typecheck`, `Out of Scope: serializer API redesign`.
- Comments: Comment 2 includes a stack trace and a minimal reproduction.
- Linked Context: Stack trace in issue comment 2.

## Open Questions
- None
```

### Coder -> reviewer

```md
Role: coder
Status: implemented
Stage: implementation
Issue: #42
Attempt: 1
Workspace Root: /Users/win/src/project
Branch: issue/42-empty-payload-serializer
Worktree: /Users/win/src/project/.worktrees/42-empty-payload-serializer
Verification: passed

## Summary
I fixed the empty-payload branch in the serializer and kept the existing non-empty behavior intact. I also added a focused regression test so the crash path stays covered.

## Changed Files
- src/serializer.ts
- tests/serializer.test.ts

## Verification Details
- `pnpm test tests/serializer.test.ts` - passed: serializer regression coverage is green
- `pnpm typecheck` - passed

## Implementation Decisions
- Returned the existing empty-result shape instead of introducing a new sentinel type.
- Kept the guard local to the serializer to avoid widening downstream API changes.

## Feedback Resolution
- None

## Known Risks / Follow-ups
- None

## Open Questions
- None
```

### Reviewer -> coder

```md
Role: reviewer
Status: approved
Stage: review
Issue: #42
Attempt: 1
Workspace Root: /Users/win/src/project
Reason: The fix matches the issue, stays in scope, and passes verification.
Branch: issue/42-empty-payload-serializer
Worktree: /Users/win/src/project/.worktrees/42-empty-payload-serializer
Verification: passed

## Summary
The implementation fixes the crash path without broadening scope. The regression test and rerun verification make the change credible to commit.

## Changed Files
- src/serializer.ts
- tests/serializer.test.ts

## Verification Details
- `pnpm test tests/serializer.test.ts` - passed
- `pnpm typecheck` - passed

## Approval Rationale
The changed code is small, targeted, and directly covers the reported failure mode.

## Blocking Findings
- None

## Non-blocking Notes
- None

## Open Questions
- None
```

### Coder finalization

```md
Role: coder
Status: committed
Stage: finalization
Issue: #42
Attempt: 1
Workspace Root: /Users/win/src/project
Branch: issue/42-empty-payload-serializer
Worktree: /Users/win/src/project/.worktrees/42-empty-payload-serializer
Verification: passed
Commit: 0123456789abcdef0123456789abcdef01234567

## Summary
I committed the approved serializer fix and reran the approval-critical checks against the committed artifact. The committed tree matches the reviewed change.

## Changed Files
- src/serializer.ts
- tests/serializer.test.ts

## Verification Details
- `git checkout 0123456789abcdef0123456789abcdef01234567 -- . && pnpm test tests/serializer.test.ts` - passed
- `git checkout 0123456789abcdef0123456789abcdef01234567 -- . && pnpm typecheck` - passed

## Commit Message
Fix empty payload serializer crash

## Known Risks / Follow-ups
- None

## Open Questions
- None
```

## Blocked Path

### Orchestrator skips HITL issue

```md
Role: issue-orchestrator
Status: blocked
Stage: selection
Issue: #58
Attempt: 1
Workspace Root: /Users/win/src/project
Reason: The issue requires human-in-the-loop product and design decisions, so it is out of scope for the autonomous loop.

## Summary
Issue `#58` was considered because it is attached to the current milestone, but it is not actionable for autonomous implementation. The issue body and latest maintainer comment both mark it as `HITL`, so the loop should skip it and continue to the next eligible issue.

## Priority Rationale
- The milestone makes it worth triaging now.
- The issue is explicitly marked `HITL` in both the body and discussion.
- It should not count toward the requested autonomous issue total.

## Success Criteria
- None

## Repo Context
- The proposed work would affect the checkout UI and pricing copy.
- No code investigation is needed until a human decision is made.

## Raw Issue Context
- Title: Redesign checkout upsell panel
- Labels: `enhancement`, `design`, `autonomy:hitl`, `type:parent`
- Milestone: `v1.9.0`
- Body: Includes `Autonomy: HITL` and asks for a final decision on upsell hierarchy before implementation.
- Comments: Maintainer comment 3 says to wait for product review before coding.
- Linked Context: Related mockups are still in review.

## Unblock Conditions
- A human owner finalizes the upsell hierarchy, copy, and acceptance criteria.

## Open Questions
- Which checkout variant should remain the control experience?
```

### Orchestrator skips issue blocked by open dependency

```md
Role: issue-orchestrator
Status: blocked
Stage: selection
Issue: #61
Attempt: 1
Workspace Root: /Users/win/src/project
Reason: The issue depends on `#54`, which is still open, so it is not actionable yet.

## Summary
Issue `#61` was triaged because it is part of the current release slice, but the issue body has an explicit `Blocked by: #54` section and `#54` is still open. The loop should skip `#61` for now and continue with another actionable issue.

## Priority Rationale
- The release label makes it relevant to inspect now.
- The dependency is explicit in the issue body and still unresolved.
- It should not count toward the requested autonomous issue total.

## Success Criteria
- None

## Repo Context
- `#54` owns the shared schema change that `#61` depends on.
- Starting `#61` first would force speculative interface work.

## Raw Issue Context
- Title: Add retry banner after async export completes
- Labels: `enhancement`, `release:blocker`, `autonomy:afk`
- Milestone: `v2.3.0`
- Body: Includes `Autonomy: AFK`, `Parent: #54`, `Context to Read: #54`, and `Blocked by: #54` for the export-status API contract.
- Comments: Latest maintainer comment says UI work should wait for the API shape to settle.
- Linked Context: Related frontend branch exists only for `#54`.

## Unblock Conditions
- `#54` closes and defines the export-status response contract needed by this UI work.

## Open Questions
- None
```

### Orchestrator skips under-specified child issue

```md
Role: issue-orchestrator
Status: blocked
Stage: selection
Issue: #67
Attempt: 1
Workspace Root: /Users/win/src/project
Reason: The issue appears to depend on parent context, but it does not explicitly identify the required parent issue or context to read.

## Summary
Issue `#67` was considered because it carries the autonomous-loop label and a current milestone, but the child issue is under-specified for bounded implementation. It references a new onboarding flow and shared copy decisions without a `Parent` value or any `Context to Read`, so the loop should block it instead of searching for the right PRD automatically.

## Priority Rationale
- `autonomy:afk` and the release milestone make it worth triaging.
- The issue is missing the explicit parent/context links required for deterministic handoff.
- It should not count toward the requested autonomous issue total.

## Success Criteria
- None

## Repo Context
- The repo contains multiple onboarding-related issues and docs, so auto-discovery would be ambiguous.
- The issue text mentions approved copy and a new empty state, but does not say which upstream decision is authoritative.

## Raw Issue Context
- Title: Add empty-state CTA to the new onboarding dashboard
- Labels: `enhancement`, `autonomy:afk`, `priority:medium`
- Milestone: `v2.3.0`
- Body: Includes `Autonomy: AFK` and broad success criteria, but omits `Parent` and `Context to Read` while referring to "the new onboarding flow" and approved copy.
- Comments: No maintainer comment narrows the source of truth.
- Linked Context: None.

## Unblock Conditions
- Update the child issue to include `Parent: #...` or `Parent: None` and an explicit `Context to Read` list, or rewrite the issue so it is self-contained.

## Open Questions
- Which parent issue, PRD, or comment defines the approved onboarding copy and empty-state behavior?
```

### Orchestrator skips issue already in progress

```md
Role: issue-orchestrator
Status: blocked
Stage: selection
Issue: #73
Attempt: 1
Workspace Root: /Users/win/src/project
Reason: Existing active work already covers this issue, so starting a second autonomous branch would duplicate effort.

## Summary
Issue `#73` looks actionable on its own, but an open PR and recent issue comments show that the work is already underway. The loop should skip it and move to another issue instead of creating conflicting parallel changes.

## Priority Rationale
- The issue is otherwise AFK and well-scoped.
- Open PR `#412` and branch `issue/73-session-timeout-banner` already target the same acceptance criteria.
- It should not count toward the requested autonomous issue total.

## Success Criteria
- None

## Repo Context
- PR `#412` touches `src/session/banner.tsx` and `tests/session/banner.test.tsx`.
- The latest maintainer comment says review feedback is in progress on the existing branch.

## Raw Issue Context
- Title: Add timeout warning banner before forced logout
- Labels: `bug`, `priority:medium`, `autonomy:afk`
- Milestone: `None`
- Body: Includes `Autonomy: AFK`, `Parent: None`, `Context to Read: None`, `Blocked by: None`, success criteria, and verification commands.
- Comments: Maintainer comment 5 links PR `#412` and says the work is already being revised there.
- Linked Context: Existing branch `issue/73-session-timeout-banner`.

## Unblock Conditions
- The existing PR closes, is abandoned, or the overlap is explicitly resolved before retrying autonomous work.

## Open Questions
- None
```

### Orchestrator skips conflicting issue during multi-issue run

```md
Role: issue-orchestrator
Status: blocked
Stage: selection
Issue: #88
Attempt: 1
Workspace Root: /Users/win/src/project
Reason: This issue conflicts with the already selected higher-priority issue `#84`, so it should wait for the first change to land.

## Summary
Issue `#88` is a plausible autonomous follow-on, but the current multi-issue run already selected `#84`, which changes the same notification pipeline and shared files. The loop should skip `#88` for now, then re-evaluate it after `#84` finishes.

## Priority Rationale
- `#84` and `#88` both modify `src/notifications/pipeline.ts` and the delivery retry config.
- `#84` is higher priority because it fixes production delivery failures.
- `#88` should not count toward the requested autonomous issue total until the queue is re-evaluated.

## Success Criteria
- None

## Repo Context
- `#84` is already selected in this run as the prerequisite production fix.
- Both issues would touch the same retry policy and alert formatting paths.

## Raw Issue Context
- Title: Add digest-mode notification fallback
- Labels: `enhancement`, `priority:low`, `autonomy:afk`
- Milestone: `v2.4.0`
- Body: Includes `Autonomy: AFK`, `Parent: None`, `Context to Read: None`, `Blocked by: None`, success criteria, and verification commands.
- Comments: No explicit blocker, but related discussion references the same retry pipeline touched by `#84`.
- Linked Context: Shares files with selected issue `#84`.

## Unblock Conditions
- Finish `#84`, then re-rank and re-check the remaining queue before considering `#88` again.

## Open Questions
- None
```

### Reviewer autonomy stop

```md
Role: reviewer
Status: blocked
Stage: review
Issue: #77
Attempt: 2
Workspace Root: /Users/win/src/project
Reason: I cannot verify the fix because the required staging credentials are unavailable and the issue cannot be reproduced locally.
Branch: issue/77-oauth-refresh-loop
Worktree: /Users/win/src/project/.worktrees/77-oauth-refresh-loop
Verification: not_run

## Summary
The coder produced a plausible fix, but I cannot complete a normal review loop because the failure only reproduces against a staging identity provider that is not accessible in this environment. I cannot credibly approve or request precise corrective changes without that access.

## Changed Files
- src/auth/refresh.ts
- tests/auth/refresh.test.ts

## Verification Details
- `pnpm test tests/auth/refresh.test.ts` - not run: staging-only dependency blocks meaningful verification

## Blocking Findings
- The reported bug depends on staging-only OAuth behavior.
- Local mocks do not reproduce the failing token refresh sequence from the issue.

## Required Outcomes
- Reproduce the failure with valid staging credentials, or provide a reliable local fixture that matches the real provider behavior.

## Non-blocking Notes
- The added unit test shape looks reasonable, but it is not enough to validate the real failure mode.

## Unblock Conditions
- Provide reviewer access to the staging identity provider, or attach a reproducible fixture that captures the failing refresh exchange.

## Open Questions
- Does the provider rotate refresh tokens on every successful refresh, or only on specific scopes?
```
