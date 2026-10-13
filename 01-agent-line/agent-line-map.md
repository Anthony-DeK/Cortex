# Agent Line Map: Cortex PM Chief-of-Staff Agent

> Module 1 · The Agent Line
>
> ✅ **What this validates:** every risky action has a clear owner, by the end you'll have proven an above/below-the-line map with HITL checkpoints, scored on reversibility, blast radius, and measurability.

## The workflow, decision by decision

List every discrete decision or action in your agent's workflow, then score each one and place it **above** the line (a human owns it) or **below** (the agent owns it). Borderline calls get an HITL checkpoint.

| Decision / action | Reversibility (H/M/L) | Blast radius (H/M/L) | Measurability (H/M/L) | Above / Below | HITL? |
|---|---|---|---|---|---|
| Pull project state + activity | H | M | H | HITL | Yes |
| Decide relevant context | M | M | M | HITL | Yes |
| Draft the update | H | L | H | Below | No |
| Decide tone/commitment level | M | M | M | HITL | Yes |
| Flag at-risk/escalation | L | H | M | Above | No |
| Choose what to escalate | M | H | M | Above | No |
| Propose a capped story batch | H | M | M | HITL | Yes |
| Post a routine project update | L | H | H | HITL | Yes |
| Approve a company-wide update | L | H | M | Above | No |

## Agent anatomy (sketch)

- **Model:** Use a fast, lower-cost model for routine retrieval and drafting. Escalate to a stronger model when project data conflicts or critical-path impact is ambiguous.
- **Tools:** Read project activity, roadmap, past updates, and team norms; draft updates and capped story proposals. Publishing requires human approval.
- **Memory:** Retain approved project facts, decisions, roadmap, and team norms with their sources. Discard transient drafts and unverified inferences.
- **Loop:** _placeholder, defined in M2 loop-spec.md_
- **Bounds:** _placeholder, defined in M5 bounds-and-evals.md_
- **Evals:** _placeholder, defined in M5 bounds-and-evals.md_

## The golden rule, applied

- Pull project state + activity is HITL because it is easy to reverse, has a medium blast radius, and is easy to verify; measurability is the deciding factor, so a human checks that the data is correct.
- Decide relevant context is HITL because it has medium reversibility, blast radius, and measurability; measurability is the deciding factor, so a human can manually assess the context selection.
- Draft the update sits Below because it is easy to revise, has a low blast radius, and is easy to check; blast radius is the deciding factor, since a human can review the draft before it is posted.
- Decide tone/commitment level is HITL because it has medium reversibility, blast radius, and measurability; measurability is the deciding factor, since a human checks that it is appropriate and realistic for the use case.
- Flag at-risk/escalation stays Above because it is hard to reverse, has a high blast radius, and is moderately measurable; blast radius is the deciding factor, and a human has better context.
- Choose what to escalate stays Above because it has medium reversibility, a high blast radius, and medium measurability; blast radius is the deciding factor, and a human has the context to distinguish what is on the critical path.
- Propose a capped story batch is HITL because it is easy to revise, has a medium blast radius, and is moderately measurable; measurability is the deciding factor, since a human can review and correct it.
- Post a routine project update is HITL because it is hard to reverse, has a high blast radius, and is easy to verify; blast radius is the deciding factor, so a human can review and correct it before it goes out.
- Approve a company-wide update stays Above because it is hard to reverse, has a high blast radius, and is moderately measurable; blast radius is the deciding factor, since it is too important to delegate.

## Hardest call

The hardest call was drafting the update. Blast radius settled it: the draft is not public yet, so a human can review it before it is posted.
