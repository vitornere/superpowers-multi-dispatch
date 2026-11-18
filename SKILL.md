---
name: superpowers-multi-dispatch
description: Use whenever any Superpowers skill is invoked or active, at every phase, including brainstorming, writing-plans, executing-plans, subagent-driven-development, test-driven-development, systematic-debugging, requesting-code-review, receiving-code-review, verification-before-completion, finishing-a-development-branch, using-git-worktrees, and writing-skills. Also use when the user explicitly requests the Superpowers dispatch routing.
---

# Superpowers multi-dispatch

Apply these instructions as standing guidance for as long as any Superpowers skill is active.

Superpowers decides **what** happens and **in what order**. This skill decides **who executes each unit of work**.

## Non-interference

This skill never changes Superpowers behavior.

* Never skip, reorder, shorten, merge, or replace a step from an active Superpowers skill.
* Never treat delegation as satisfying a Superpowers requirement. An external agent writing a test does not satisfy `test-driven-development`; Claude still runs the test and observes RED before GREEN.
* Never use this skill to bypass brainstorming approval, plan approval, verification evidence, or review.
* On any conflict, the Superpowers skill wins on process. This skill only reassigns execution.

If no Superpowers skill is active, this skill has no effect.

## Activation

Activate at the moment any Superpowers skill is invoked, including read-only and exploratory phases. Do not wait for implementation to begin.

Announce the routing decision once per phase in one line: `Dispatch: <phase> -> <agent or inline>`, adding `(<tier>: <model>)` for every Cursor dispatch.

## Non-delegable work

Claude performs these directly, always, in every phase:

* Conversation with the user, clarifying questions, and approvals.
* Requirements, acceptance criteria, architecture, and design decisions.
* Plan authoring, task decomposition, and rulings during execution.
* Constructing every subagent prompt and its context.
* Reading the working-tree diff.
* Running verification commands and reporting their real output.
* Observing RED and GREEN in TDD.
* Validating, confirming, or rejecting every review finding.
* Branches, commits, tags, pushes, pull requests, and releases, only with explicit user authorization.
* The final report.

Delegated output is a proposal, never evidence.

## Routing table

| Active Superpowers skill | Delegated unit | Agent | Reference |
|---|---|---|---|
| `brainstorming` | External research; codebase reconnaissance | Parallel `Explore` agents, always; plus `multi:cursor-research` for external research and `multi:cursor-explore` for wide sweeps | [investigation.md](references/investigation.md) |
| `writing-plans` | Codebase mapping fan-out; adversarial plan critique | Parallel `Explore` agents; `multi:cursor-explore` (tier `deep-read`) | [investigation.md](references/investigation.md) |
| `using-git-worktrees` | Nothing | Claude inline | — |
| `executing-plans` | Per-task implementation | Implementer per tier: `multi:cursor-delegate` (`implement`) or Claude `general-purpose` opus (`implement-complex`) | [execution.md](references/execution.md) |
| `subagent-driven-development` | Per-task implementation, then per-task review | Implementer per tier, then review lane | [execution.md](references/execution.md), [review.md](references/review.md) |
| `test-driven-development` | One RED increment, then one GREEN increment | Implementer per tier | [execution.md](references/execution.md) |
| `systematic-debugging` | Independent hypothesis investigation; stuck root cause | Parallel `Explore` agents, one per hypothesis; `multi:cursor-explore` (tier `deep-read`) | [investigation.md](references/investigation.md) |
| `requesting-code-review` | The reviews themselves | `multi:cursor-explore` (tier `review`), at most two rounds | [review.md](references/review.md) |
| `receiving-code-review` | Confirmed in-scope corrections | Implementer per tier | [execution.md](references/execution.md), [review.md](references/review.md) |
| `verification-before-completion` | Nothing | Claude inline | — |
| `finishing-a-development-branch` | Nothing | Claude inline | — |
| `writing-skills` | Baseline and pressure-scenario runs | Fresh subagents per the Superpowers testing methodology | — |
| `dispatching-parallel-agents` | Governs fan-out shape for any lane above | — | [investigation.md](references/investigation.md) |

Invoke agents directly through the Agent tool. Do not print `/cursor:*` or `/multi:*` commands for the user to run.

## Models and fallback

[models.md](references/models.md) maps each tier to an executor chain (Cursor or Claude subagent) and a final fallback. Read it before the first dispatch of a session.

* Claude-family models (Opus, Sonnet, Haiku, Fable) always run as Claude subagents through the Agent tool, never through Cursor.
* Every `multi:cursor-*` dispatch carries `--model` from that table. The Cursor CLI otherwise reuses the user's last-picked model.
* Once per session, `agent --list-models` confirms the pinned models still exist. A missing one gets a one-line notice to the user.
* An executor outage never blocks the workflow. On quota, auth, or availability failure, move down the chain, tell the user in one line, and keep going.

## Gated delegation

If the harness requires an explicit user request before the Agent tool may be used, delegation is permitted but not yet requested. That is not unavailability.

Announce the routing decision the table prescribes, say it needs the user's go-ahead, and ask for it. Asking in this case is required, and does not conflict with execution.md's rule against handing the user commands to run.

Never report the Agent tool as disabled, missing, or unavailable unless an actual call to it failed.

## Inline lane

Some work inside an active Superpowers workflow is genuinely lightweight: documentation, formatting, comments, typos, a trivial rename, a one-line configuration change, or a focused test adjustment with an established cause.

Route it to Claude inline. Still announce it: `Dispatch: <phase> -> inline (lightweight)`.

The inline lane changes who executes. It never removes verification or the Superpowers steps around it.

## Red flags

| Thought | Reality |
|---|---|
| "Faster if I just write it myself" | Speed is not the routing criterion. Read the table and route. |
| "The change is small" | Small has its own lane. Choose it explicitly and say so. |
| "Superpowers already says dispatch a subagent" | Superpowers says dispatch. This table says which agent and which model. |
| "I already know this codebase" | Investigation fan-out is independent and cheap. Dispatch it. |
| "The reviewer will just agree" | Independent review is not optional for behavioral changes. |
| "This phase is read-only, so the skill does not apply" | It applies at every phase, including brainstorming and planning. |
| "Cursor reported success" | A report is not a diff and not a test run. Inspect and verify. |
| "Delegation makes verification redundant" | Delegation moves execution, never evidence. |
| "The Agent tool must be unavailable" | You have not called it. An ungranted permission is not a missing tool. Ask. |
| "Cursor's default model is fine here" | The default is the user's last pick. Pin `--model` from models.md. |
| "Cursor is out of quota, so the work must wait" | Fall back per models.md, notify in one line, continue. |

## Standing authorization

Automatically correct review findings that Claude validates as correct, actionable, in scope, consistent with the approved design, safe, reversible, and not requiring a new product decision.

Ask the user first when a finding materially expands scope, requires a new product or architectural decision, conflicts with the original requirements, requires a destructive or irreversible operation, requires external coordination, or admits multiple materially different valid behaviors.

## Completion

Do not claim completion until:

* The routing decision for each phase was announced.
* Every active Superpowers skill completed its own steps.
* The relevant diff was inspected by Claude.
* Required verification ran and its output was read.
* The applicable review workflow completed.
* Confirmed in-scope findings were addressed and verification repeated.
* Remaining risks, failed checks, and rejected findings were disclosed.
* No unauthorized version-control action was performed.
