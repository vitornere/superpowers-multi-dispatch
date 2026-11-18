# Delegated implementation

Covers every phase that writes code: `executing-plans`, `subagent-driven-development`, `test-driven-development`, and corrections driven by `receiving-code-review`.

## Phase contracts

The active Superpowers skill keeps its own steps. Only execution moves.

### executing-plans

The implementer is the tier's executor from [models.md](models.md): Cursor `multi:cursor-delegate` or a Claude `general-purpose` subagent.

Delegate one plan task per implementer invocation. Do not hand the implementer the entire plan and ask it to finish.

After each task: Claude inspects the diff, runs the task's verification, and honors the plan's review checkpoints before dispatching the next task.

### subagent-driven-development

One fresh implementer invocation per plan task, then the per-task review required by that skill, then the next task.

Claude keeps the ledger and records every `Ruling:` line itself. Do not delegate rulings.

The broad whole-branch review at the end runs through [review.md](review.md).

### test-driven-development

Delegate RED and GREEN as two separate invocations, never one.

1. Dispatch the implementer to write only the failing test. No implementation.
2. Claude runs the test and reads the failure output. If it did not fail, or failed for the wrong reason, fix the test before continuing.
3. Dispatch the implementer to write the minimal implementation. No test edits.
4. Claude runs the test and reads the passing output.

Claude observes RED and GREEN. The implementer's claim that a test fails or passes is never the evidence.

Never let a single invocation produce both the test and the implementation.

### receiving-code-review

Delegate only findings Claude already validated as confirmed and in scope. Give the implementer the finding, the file, and the intended correction, not the reviewer's full report.

## External-agent invocation

Invoke external agents directly through the Agent tool.

Do not merely print or ask the user to execute `/cursor:*`, `/multi:*`, or `agent` commands when Claude can invoke the corresponding agent.

All code and file modifications in this workflow go to the tier's implementer chain in [models.md](models.md). Claude-family models (Opus, Sonnet, Haiku, Fable) run as Claude `general-purpose` subagents, never through Cursor.

Claude may independently inspect files and run verification commands.

The implementer, Cursor or Claude fallback, must not create branches, commits, tags, pushes, pull requests, or releases.

## Model routing

Choose a tier before every dispatch, then take the executor, model, and fallback chain from [models.md](models.md). Every dispatch sets its model explicitly: `--model` for Cursor, `model` (and `effort`) for a Claude subagent.

Announce it: `Dispatch: <phase> -> <executor> (<tier>: <model>)`, for example `-> multi:cursor-delegate (implement: composer-2.5)` or `-> general-purpose (implement-complex: opus, effort high)`.

### Tier `implement` (default)

Use for most implementation and correction tasks:

* Well-scoped features.
* Ordinary bug fixes.
* CRUD operations.
* API endpoints.
* UI components.
* Form and validation changes.
* Focused refactors.
* Test creation and migration.
* Localized review corrections.
* Mechanical or repetitive changes.
* Execution of a detailed approved plan.
* Changes contained within one application or domain.

File count alone does not make a task complex.

### Tier `implement-complex`

Use when implementation materially requires deeper reasoning, cross-system coordination, failure-state analysis, or adaptation during execution:

* Coordinated changes across repositories, applications, or services.
* Complex migrations, backfills, or data transformations.
* Concurrency, race conditions, locks, retries, or idempotency.
* Payment or financial-state workflows with multiple transitions.
* Authentication, authorization, or tenant isolation across several layers.
* Large architectural refactors across multiple modules.
* Event-driven workflows involving queues, workers, and external services.
* Compatibility migrations preserving old and new behavior.
* Rollback, recovery, partial failure, or degraded-operation handling.
* Long plans with several dependent implementation phases.
* Work requiring substantial adaptation after inspecting intermediate results.

Do not use `implement-complex` merely because:

* Many similar files are affected.
* The implementation is time-consuming but mechanical.
* Tests fail for a clear localized reason.
* Routine tests must be updated.
* The change is important but technically well-scoped.
* A reviewer reports ordinary localized findings.

When uncertain, use `implement`.

Do not run two tiers in parallel for the same implementation.

## Delegation context

Provide the implementer with:

* The specific implementation task.
* Requirements and acceptance criteria.
* Applicable repository instructions.
* The approved plan or assigned plan step.
* Relevant architecture and constraints.
* Tests required by the active Superpowers workflow.
* Existing working-tree context.
* Out-of-scope behavior.
* Prohibited version-control actions.

Use `--plan <path>` when executing a written plan and the subagent supports it.

## Verification

The implementer's report is not proof that implementation is correct.

After it returns, Claude must:

1. Inspect the actual diff.
2. Confirm that the implementation matches the plan.
3. Identify unrelated modifications.
4. Run relevant tests, linting, formatting, type checking, migrations, builds, and project-specific checks.
5. Diagnose failures before delegating additional work.

## Failure handling

Two kinds of failure, handled differently:

* **The executor could not run** (Cursor quota, unavailable model, authentication, CLI missing, transport; or a Claude subagent rate-limited or refused): follow the fallback table in [models.md](models.md). Fall back, notify the user in one line, continue. Never stop the workflow to wait for the user because of an executor outage.
* **The implementation is wrong** (defects, failing tests, wrong approach): the steps below.

If an `implement` run produced a wrong implementation:

1. Inspect the partial implementation and failure evidence.
2. Determine whether the cause is an environmental problem, an incorrect plan, an ordinary code defect, or insufficient implementation reasoning.
3. Stay on `implement` for ordinary localized corrections.
4. Escalate to `implement-complex` only when the remaining work requires deeper reasoning or cross-system coordination.
5. Preserve valid changes and give the new run the current state and remaining work.
6. Do not restart the entire implementation unnecessarily.

Do not escalate tiers merely because a test failed.
