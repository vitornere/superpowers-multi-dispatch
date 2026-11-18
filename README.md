# superpowers-multi-dispatch

A Claude Code skill that sits on top of [Superpowers](https://github.com/obra/superpowers) and decides **who executes each unit of work**: Claude itself, a Claude subagent, or [Cursor](https://cursor.com) through the [`multi`](https://github.com/greenpolo/cc-multi-cli-plugin) plugin.

Superpowers decides *what* happens and *in what order*. This skill only reassigns execution. It never skips, reorders, or shortens a Superpowers step, and delegated output is always treated as a proposal: Claude still reads the diff, runs the verification, observes RED and GREEN, and validates every review finding.

> This is an independent extension. It is not part of, or endorsed by, the Superpowers or `multi` projects.

## What it does

| Superpowers phase | Routed to |
|---|---|
| `brainstorming`, `writing-plans`, `systematic-debugging` | Parallel Claude `Explore` agents, plus `multi:cursor-explore` / `multi:cursor-research` for wide sweeps, external research, and adversarial plan critique |
| `executing-plans`, `subagent-driven-development`, `test-driven-development`, `receiving-code-review` | Implementer by tier: `multi:cursor-delegate` for routine work, a Claude `general-purpose` Opus subagent for complex work |
| `requesting-code-review` | `multi:cursor-explore` in review mode, at most two rounds |
| `using-git-worktrees`, `verification-before-completion`, `finishing-a-development-branch` | Claude inline |

Lightweight work (docs, typos, trivial renames, one-line config) goes to an explicit inline lane. Claude always keeps the conversation, design decisions, plan authoring, diff inspection, verification, and every version-control action.

Every phase announces its routing in one line, for example:

```text
Dispatch: executing-plans -> multi:cursor-delegate (implement: composer-2.5)
```

## Requirements

1. **Claude Code** with the **Superpowers** plugin installed.
2. The **`multi`** plugin (provides the `multi:cursor-*` subagents):
   ```text
   /plugin marketplace add greenpolo/cc-multi-cli-plugin
   /plugin install multi@cc-multi-cli-plugin
   ```
3. The **Cursor CLI** (`agent`), installed and signed in. `agent --list-models` must work.

### Without Cursor

The skill still works. If the `multi` agents are missing, the Cursor CLI is not installed, or Cursor runs out of quota, every tier falls back to a Claude subagent (see `references/models.md`), and the fallback is announced in chat and in the final report. You lose the main benefit, which is moving mechanical implementation off Claude.

Other `multi` backends (Codex, OpenCode, Antigravity) are not routed yet.

## Install

Clone into your personal skills directory:

```bash
git clone https://github.com/vitornere/superpowers-multi-dispatch ~/.claude/skills/superpowers-multi-dispatch
```

The skill description already asks Claude to load it whenever a Superpowers skill is active. To make that reliable, add this to `~/.claude/CLAUDE.md`:

```markdown
Whenever any Superpowers skill is invoked or active, invoke `superpowers-multi-dispatch` through the Skill tool, at every phase, including brainstorming, planning, debugging, review, and read-only phases.

Do not invoke `superpowers-multi-dispatch` when no Superpowers skill is active.
```

## Configure models

`references/models.md` is the single source of truth for which model each tier uses, its fallback chain, and the Cursor quota pools. The shipped values reflect one Cursor plan. Review them and replace the model IDs with ones your plan offers (`agent --list-models`). The skill never edits this file on its own: when a pinned model disappears, it tells you and moves down the chain.

## Layout

```text
SKILL.md                    Routing table, non-delegable work, completion rules
references/models.md        Tiers, pinned models, availability check, fallback
references/execution.md     Implementer tiers and per-task dispatch
references/investigation.md Exploration, research, and debugging fan-out
references/review.md        Review rounds and finding validation
```

## License

[MIT](LICENSE)
