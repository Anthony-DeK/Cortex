# Orchestration Map: Cortex PM Chief-of-Staff Agent

> Module 3 · Orchestration & Subagents, ★ Deliverable 3
>
> ✅ **What this validates:** nothing advances unchecked, by the end you'll have proven a justified topology, a roster, and a validator with a defined fail action.
>
> Builds on your M2 Loop Spec. Only split one agent into a team when there's a real reason, coordination has a cost.

## 1. Why split? (or why not)

**Today:** one agent drafts the weekly update, checked by a separate critic.
**Vision:** a chief-of-staff orchestrator over per-workflow specialists.

| Reason | Applies? | Why / why not |
|---|---|---|
| Separation of concerns | No | Tasks are co-dependent and should be performed by the same agent. |
| Parallelism | No | Each task in the workflow is sequential (gather knowledge, build report, draft message). |
| Independent validator | **Yes** | An independent validator is needed for the report. |
| Context-window pressure | No | Context is weekly and shouldn't be too much. |

**Decision:** split for one reason only, an independent validator. Cortex + one critic subagent today.

## 2. Topology

**Pattern:** single + subagents

```
[PM hook / Monday cron] → [Cortex: pulls project, activity, past updates, roadmap, norms → drafts update + queues stories]
    → [Critic]
        ✗ fail → back to Cortex with rule + fix (max 2 revisions) → 3rd fail → escalate to PM (draft held)
        ✓ pass → [PM review checkpoint] → draft saved, stories queued; nothing sent
```

## 3. Roster

| Agent / subagent | Responsibility | Runs which Loop Spec |
|---|---|---|
| Cortex (drafter) | Pulls data via tools, drafts the update, queues stories, escalates when the norms require it | M2 loop ([`agent.py`](../00-build/agent.py)) |
| Critic (validator) | Checks the draft against the 4 rules (§5); returns pass/fail plus violations | One call per draft; no loop of its own, the revision cap lives in Cortex's loop ([`critic.py`](../00-build/critic.py)) |

## 4. Communication & hand-offs

- **Cortex → Critic:** the proposed output plus a source log (the task brief and every tool call with its result). In-process Python function call; no MCP or A2A.
- **Critic → Cortex:** structured JSON, `{verdict, reasons, violations[{rule, fix}]}`. Only `violations` go back to Cortex, as a growing "don't repeat" list. `reasons` go to the human trace only.
- **Cortex → PM:** the draft is saved to `run-output/<MMDD>-<project>-status-update.md`, marked "accepted" or "held" with the reason.

## 5. The validator

An independent critic: a separate model call with its own instructions that never sees the drafting context.

- **What the critic checks:**
  - **Every figure is traceable:** every metric, %, date and red/yellow/green call matches the pulled data; nothing estimated or projected.
  - **No forbidden commitments:** no GA/ship date committed, no launch gate marked; an unconfirmed date is escalated instead.
  - **No confidential leak:** no CONFIDENTIAL/embargoed roadmap item in the update, by name or detail.
  - **Nothing executed:** nothing posted, sent, created, closed or merged; stories only queued, within the queue cap.

  *Why:* we shouldn't convey wrong data, get caught on commitments we can't ensure, or let confidential info slip out. And we want HITL before executing tasks.
- **Fail action: revise.** The draft goes back to Cortex with the rule it broke and a fix instruction (added to the run's "don't repeat" list). *Why:* as long as the agent isn't blocked, we shouldn't escalate; we should empower the agent to try again.
- **Revision cap: max 2, then escalate.** On a third rejection Cortex stops, holds the last draft, and hands it to the PM with the critic's reasons. Enforced in code (`CORTEX_MAX_REVISIONS`), not by the model. *Why:* if the agent fails again after being told what's wrong and why, twice, something is wrong and escalation is needed.
- **Pass action:** the draft advances to the PM review checkpoint and is saved for review. Nothing is sent automatically.

## 6. State: shared vs isolated

| | Shared (both see it) | Isolated |
|---|---|---|
| Source data (brief + tool results) | ✓ | |
| The draft | ✓ | |
| Cortex's system prompt, conversation and reasoning | | Cortex only |
| Critic's system prompt and its `reasons` | | Critic only (reasons go to the human) |
| "Don't repeat" list | | Cortex only, for this run |

**Decision:** keep it as built: the critic sees only the evidence (brief + tool results) and the draft, never Cortex's prompt, conversation or reasoning. *Why:* we need isolation so the critic remains unbiased.

## 7. Cost & latency budget

- **Per draft:** the validator adds 1 model call (+~25% vs a clean single-agent pass: 3 drafter calls + 1 critic). Each revision adds 2 calls (redraft + fresh review).
- **Worst case at the revision cap (2):** 8 calls, ~$0.01 per update on Haiku 5.5, well under the $0.50 run cost cap. Measured: clean pass $0.0032 (4 calls), one revision $0.0056 (6 calls).
- **Latency:** up to ~5 extra calls before the draft reaches me; acceptable for an async weekly update (hook / Monday cron).
- *Why:* this is the safest path and it comes at a reasonable price. → Enforced as bounds in M5 (`CORTEX_MAX_REVISIONS`, `CORTEX_COST_CAP_USD`).
