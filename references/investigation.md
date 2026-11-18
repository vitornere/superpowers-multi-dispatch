# Delegated investigation

Covers the read-only phases: `brainstorming`, `writing-plans`, `systematic-debugging`, and any fan-out governed by `dispatching-parallel-agents`.

All agents in this reference operate read-only. None of them may modify files, and none of them may perform version-control actions.

## What Claude never delegates here

* The dialogue with the user during `brainstorming`, including questions, options, and approval.
* The design decision and the written spec.
* The plan document itself, its task decomposition, and its acceptance criteria.
* The hypothesis, the experiment design, and the root-cause conclusion during `systematic-debugging`.
* The subagent prompt and the context handed to each agent.

Agents supply findings. Claude supplies judgment.

## Agent selection

| Need | Agent | Tier in [models.md](models.md) |
|---|---|---|
| Codebase reconnaissance: where something lives, how a flow works, what conventions exist | Claude's built-in `Explore` agent, dispatched in parallel | none (Claude) |
| The same reconnaissance when the sweep is wide enough that keeping it off Claude's token budget is the point | `multi:cursor-explore` | `explore` |
| External web, library, API, or specification research | `multi:cursor-research` | `research` |
| Adversarial critique of a written plan | `multi:cursor-explore` | `deep-read` |
| Open-ended investigation after Claude's own hypotheses are exhausted | `multi:cursor-explore` | `deep-read` |

Every `multi:cursor-*` call carries `--model` and `--read-only` from its tier, and falls back as [models.md](models.md) prescribes. A Cursor failure never stops the investigation.

`Explore` is the default for anything inside this repository. Reach for `multi:cursor-explore` when the sweep is broad, or when the same question is worth a second opinion from a different model. Mixing both in one fan-out is fine, and they go in the same message.

## Brainstorming

Run research before presenting a design, never instead of the design conversation.

1. Identify the specific unknowns blocking the design, both codebase unknowns and external unknowns.
2. Dispatch parallel `Explore` agents for the codebase unknowns, one agent per unknown, always. Add `multi:*` researchers for the external unknowns.
3. Send the whole fan-out in a single message so the agents run concurrently.
4. Read the findings and reconcile contradictions yourself.
5. Present the design to the user, citing what the research established and what remains uncertain.
6. Obtain approval as `brainstorming` requires.

Do not skip the fan-out because the codebase feels familiar. Recall is not evidence, and the agents cost you no context.

Do not dispatch research for questions the user can answer faster. Ask the user.

Specify search breadth in each `Explore` prompt: "medium" for a focused area, "very thorough" when the concept may live under several names or in several directories.

## Writing plans

1. Fan out parallel `Explore` agents across the subsystems the plan will touch, one agent per subsystem, in a single message.
2. Ask each for file paths, existing patterns, test locations, and conventions, not for opinions about the plan.
3. Write the plan yourself from the returned map.
4. Optionally dispatch `multi:cursor-explore` (tier `deep-read`, `--fresh`) for an adversarial critique of the written plan before execution. Give it the plan path, the approved spec, and ask for risks, gaps, and ordering problems with file and line references.
5. Validate every critique point against the requirements. Reject what is out of scope and say so.

A plan critique is advisory. It never overrides the user's approved design.

## Systematic debugging

`systematic-debugging` requires root cause before any fix. Delegation does not relax that.

1. Claude forms the hypotheses.
2. When two or more hypotheses are independent, dispatch one `Explore` agent per hypothesis, in parallel, in a single message.
3. Give each agent exactly one hypothesis, the reproduction steps, and the relevant paths. Never give an agent the whole bug and ask it to solve the problem.
4. Require each agent to return evidence: file, line, observed behavior, and whether the hypothesis is supported or refuted.
5. Claude reproduces the decisive evidence itself before accepting a root cause.
6. Only after root cause is established, route the fix through [execution.md](execution.md).

Escalate to `multi:cursor-explore` (tier `deep-read`, `--fresh`) only when Claude's hypotheses are exhausted and the parallel investigation returned no supported cause. Give it the reproduction, the refuted hypotheses, and the evidence for each refutation, and ask for new hypotheses with evidence, not a fix.

Never let an investigation agent propose a fix as a substitute for root cause.

## Fan-out rules

Follow `superpowers:dispatching-parallel-agents` for the shape. These constraints apply on top of it:

* Dispatch parallel agents in a single message so they run concurrently.
* One independent unit per agent. Never split a single dependent chain across parallel agents.
* Agents never inherit session history. Construct each prompt from scratch.
* Cap a single fan-out at four agents unless the user authorized more.
* State in the prompt that the agent is read-only and must return findings, not changes.
* Require file and line references for every claim.

## Handling returned findings

Treat every returned finding as unverified.

1. Discard claims with no file or line reference unless Claude can confirm them independently.
2. Reconcile contradictions between agents by reading the code, not by majority.
3. Report which findings Claude confirmed and which it could not.

Do not forward an agent's summary to the user as if Claude established it.
