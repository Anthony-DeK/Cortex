# Production & Autonomy: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 5, how you'd ship it, govern it, and widen trust over time
>
> ✅ **What this validates:** you can ship it, govern it, and widen trust deliberately, by the end you'll have proven an autonomy dial, a Trust Ladder rung with its eval gate, and a governance plan.

## Autonomy Dial by segment

_Autonomy is a product decision per user, not one global setting._

| Segment | Desired autonomy | Why |
|---|---|---|
| Seasoned PM / project owner | bounded-autonomous | Routine weekly updates are low value to hand-check; I only review the post. |
| New PM | supervised | I'd treat their Cortex output like a junior PM's work: review it and give extra instructions before anything goes out. |
| Eng lead (backlog) | assisted | Cortex retrieves the right backlog items; deciding what goes into a sprint is the eng lead's job, not the agent's. |

> The dial only changes which HITL checkpoints pause for each segment. It never moves the M1 agent line: flagging at-risk, choosing escalations, and approving a company-wide update stay human for everyone.

## Trust Ladder

- **Current rung:** supervised. I expect Cortex to save me time and use me only for approval, for now. Before it climbs further, I need enough experience with it to build a golden set of approved outputs it can rely on to avoid hallucinations.
- **Next rung:** bounded-autonomous (only for the segments the Autonomy Dial allows; the M1 agent line doesn't move).
- **Eval gate to reach the next rung:**
  - **Offline / CI:** EV-1 to EV-6 pass 100% on every change to `prompts.py`, `tools.py` or `agent.py`; EV-5 (jailbreak) passes 3/3 runs per change.
  - **Production:** over **8 weeks and ≥ 20 supervised runs**, **≥ 95%** of drafts are approved by the PM with no factual edits (EV-4 in production), with **0** EV-5/EV-6-class incidents (no embargoed content leaked, no invented date or metric reaching the PM, no unsafe tool call).
  - **Golden set:** the 20 approved runs become the golden set Cortex is checked against before it climbs.
  - **Why these numbers:** 8 weeks covers almost an entire quarter; ≥ 95% leaves a margin for exceptions.
- **Incident record so far:** 0 production runs; all evidence is offline fixture runs. Offline incidents: EV-5 initially failed (2 of 3 jailbreak runs needed the injection guard added in M5 to be held for a human); the critic's human-only `reasons` trace can still name embargoed projects (known residual, Cortex's output is clean); the bound-trip run stopped correctly but had already queued 2 stories for approval.
- **Clean record for the gate window:** zero EV-5/EV-6-class incidents, zero kill-switch activations caused by Cortex misbehaving, and every bound trip escalated with nothing posted.

## Deployment plan

- **Runtime:** serverless. Cortex's M2 loop is hook + cron (backup): short, bursty runs (≤ 120s, keyed on project + ISO week) with nothing running in between. A managed platform or always-on host would be overkill for this use case.
- **Operator / on-call owner:** me (PM-builder). I'm the main user and Cortex was built without engineering's involvement. All escalations and above-the-line items come to me.
  - **When I'm away:** Cortex is turned off. I set `CORTEX_DISABLED` and disable the Monday cron before I leave, since I won't need it and won't be there to approve its output. Nothing runs, so nothing is waiting on an absent approver.
  - **Handoff trigger:** if/when Cortex becomes a company-wide product tool, engineering ops takes over operating it.
- **Rollback:** unplug Cortex immediately, then roll back:
  1. Flip `CORTEX_DISABLED`: the run halts and holds its draft (no write tool, so nothing to undo).
  2. Revert `prompts.py` / `tools.py` / `agent.py` to the last unaffected commit that passed CI (or pin `CORTEX_MODEL` back to the last good model).
  3. Drop the Autonomy Dial one rung for the affected segment before re-enabling.
- **Monitoring:** a weekly dashboard of five signals, each covering a different risk (hallucination, precision, cost):
  - Eval pass % (CI + sampled production runs)
  - Approved-with-no-factual-edits % (the Trust Ladder gate metric)
  - Escalation rate per run
  - Cost per run vs the $0.50 run cap and $5/day cap
  - Trust incidents (EV-5/EV-6-class)

## ROI metrics (beyond adoption & tokens)

| Metric | Target | How captured |
|---|---|---|
| **Outcome: PM time saved per weekly update** (the best ROI metric; the other two measure risk and cost, not the actual benefit) | Baseline 30 min by hand → **≤ 10 min** review while supervised (67% cut) → **≤ 2 min** once bounded-autonomous (93% cut) | Baseline: my current ~30 min per update. After: log my review-and-approve minutes for each run at approval. |
| **Cost-to-serve: $ per approved update** | **≤ $0.50** (matches the run cap) | Cortex's cost tracker logs spend per run; summed weekly on the dashboard, divided by approved runs so failed/escalated runs count against it. |
| **Trust incidents** (EV-5/EV-6-class) | **0 per 20 runs** (matches the gate) | Critic flags + a weekly search of run logs for embargoed names + anything I catch at review. |

## Widen-autonomy decision rule

Turn the dial up one rung for **one segment at a time** only when that segment has logged **≥ 20 supervised runs over ≥ 8 weeks** with **≥ 95% approved with no factual edits**, **0 EV-5/EV-6-class incidents**, **EV-1 to EV-6 passing 100% in CI**, and **median review time ≤ 10 min**. **Any single trust incident drops it back one rung.**

_Why: it exactly fits the eval gates set in the Trust Ladder._

## Governance & forward strategy

- **Compliance:**
  - **Never enters a prompt:** API keys / credentials (read from env only: `CORTEX_*` + model keys) and customer PII (contact names, emails, phone numbers from customer accounts; FullEnrich's product is contact data, so this is a real risk).
  - **Never leaves in an output:** CONFIDENTIAL / embargoed roadmap items (Orbit, Pulsar), enforced by M4 retrieval grading + critic Rule 3.
  - **PII handling:** status updates are about projects, not people; if PII appears in pulled activity, Cortex drops it and flags it to me.
- **Safety:**
  - **Above the agent line for every segment:** posting / publishing anything, approving a company-wide update, committing a GA date or launch gate, flagging at-risk, and choosing escalations. Cortex never creates, closes or merges tickets or PRs.
  - **Kill switch:** `CORTEX_DISABLED`, checked before every model call.
- **Reliability:**
  - **Caps:** 8 iterations · 60s per model call / 120s per run · $0.50 per run + $5/day · 10 stories · revision cap 2 · 3 attempts per source.
  - **Escalate-on-stuck:** any cap trip escalates to me with the draft held, nothing posted.
  - **Model-down fallback:** halt + escalate, and I write that week's update by hand (the 30-min baseline). No automatic switch to an unvalidated backup model, since that would bypass the CI eval gate.
- **Strategy:** promote the **seasoned-PM segment to bounded-autonomous** first. It fits the eval gates and the overall objective of an increasingly autonomous agent. **Gated by** the Trust Ladder gate + widen-autonomy rule above (≥ 20 supervised runs over ≥ 8 weeks, ≥ 95% approved with no factual edits, 0 EV-5/EV-6-class incidents, EV-1 to EV-6 100% in CI, median review ≤ 10 min).
