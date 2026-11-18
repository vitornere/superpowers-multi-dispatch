# Independent review workflow

## Applicability

Run this review workflow for non-trivial behavioral code changes produced through `superpowers-multi-dispatch`.

Skip it for:

* Documentation-only changes.
* Formatting-only changes.
* Comments.
* Trivial renames.
* Changes that do not modify behavior.

Run it whenever the user explicitly requests the full review workflow.

## Relationship to per-task reviews

`subagent-driven-development` requires a review after each plan task. Those per-task reviews belong to that skill, run through its own subagent, and are not counted against the budget below.

This workflow is the broad final review over the whole branch or working tree. Run it once, after the last task, not after each one.

## Reviewer

Both review rounds use tier `review` from [models.md](models.md): `multi:cursor-explore`, read-only, with the tier's `--model`, its fallback chain, and its Claude fallback.

Every review dispatch carries `--model <id> --read-only --fresh`. `--fresh` keeps the reviewer out of any earlier Cursor session, including the one that implemented the change.

Announce it: `Dispatch: requesting-code-review -> multi:cursor-explore (review: <model>)`.

## Review budget

Use at most two successful independent reviews:

1. One first review.
2. One final review, only when first-review findings caused code changes.

Do not run reviewers in parallel.

Fixing findings does not reset the review budget.

Do not create an automatic review loop.

A dispatch that failed for a Cursor reason (quota, model, auth, transport) is not a completed review. Follow the fallback in [models.md](models.md) and run the review on the replacement. It still counts as the same round.

## First review

The first-review prompt contains, in this order:

1. Role line: `You are an independent code reviewer. Read-only: do not modify any file and do not run version-control actions.`
2. Scope: the base ref, the list of changed and untracked files, and the diff inline when it fits.
3. The original request and its acceptance criteria.
4. Paths of the applicable repository instruction files (`AGENTS.md`, `CLAUDE.md`, contribution and testing guides), for the reviewer to read.
5. Focus: correctness, regressions, security and authorization boundaries, tenant isolation, sensitive-data handling, data integrity, error handling, concurrency, retries and idempotency, performance, missing tests, repository conventions.
6. Output contract: findings ordered by severity, each with severity, file path, line where possible, concrete evidence, impact, and a concise recommended correction. End with `No findings` if there are none.

The first-review prompt contains only those six parts. The reviewer forms its own view from the code.

## First-review findings

Claude must validate every finding against:

* The code.
* The original request.
* Acceptance criteria.
* Repository instructions.
* Existing tests.
* The approved design.

Classify each finding as:

* Confirmed and in scope.
* Incorrect.
* Duplicate.
* Already addressed.
* Outside scope.
* Requires user decision.

Do not silently accept or discard findings.

If no confirmed finding requires code changes:

1. Record the result.
2. Skip the final review.
3. Perform final verification.
4. Complete the task.

If confirmed findings require changes:

1. Delegate corrections through [execution.md](execution.md), choosing the tier there.
2. Inspect the resulting diff.
3. Run all affected verification.
4. Dispatch the final review.

## Final review

Run the final review only when files changed in response to first-review findings.

Same reviewer tier and flags as the first review, in a new `--fresh` session.

The final-review prompt contains, in this order:

1. The same role line, scope, request, acceptance criteria, instruction paths, focus, and output contract as the first review, with the scope updated to the current diff.
2. The implementation plan.
3. Verification commands run and their results.
4. First-review findings, each with Claude's classification and, for fixed ones, what changed.
5. An added focus item: whether each review-driven fix is correct and complete, and whether it introduced regressions.

Run the final review at most once.

## Final findings

Claude must validate every final-review finding.

Automatically delegate confirmed, in-scope corrections through [execution.md](execution.md).

After corrections:

1. Inspect the diff.
2. Run affected verification commands.
3. Do not dispatch another review automatically.
4. Perform final verification.
5. Report residual risks created by completing without another independent review.

Additional reviews require explicit user authorization.

## Final report

Report:

* What changed.
* Verification commands and results.
* Which reviewer model ran each round, and any fallback with its reason.
* First-review findings fixed or rejected.
* Whether the final review was required.
* Final-review findings fixed or rejected.
* Remaining risks and unverified checks.
* Actions still requiring user authorization.
