# Cortex, a PM Chief-of-Staff Agent

> When the CPO asks (or Monday 9am if nobody has), Cortex pulls the project's state, engineering activity, roadmap, past updates and team norms, drafts the weekly L10 status update, and proposes up to 10 next-sprint stories from the PRD. An independent critic checks every draft; then it stops. Nothing is posted, created or committed without me.

_Anthony · Agentic Workflows & Loops Cohort · November 2026_

This repo is my final project for the Agentic Workflows & Loops Certification, **Cortex**. Each module's artifact lives in its own folder; this README is the dashboard and the pitch. Repo: https://github.com/Anthony-DeK/Cortex · Pitch deck: [`pitch.html`](pitch.html)

---

## Module artifacts

### M1 · The Agent Line
- **Agent-line map**: [`01-agent-line/agent-line-map.md`](01-agent-line/agent-line-map.md)

### M2 · Loop Engineering
- **Loop spec**: [`02-loop-design/loop-spec.md`](02-loop-design/loop-spec.md)

### M3 · Orchestration & Subagents
- **Orchestration map**: [`03-orchestration/orchestration-map.md`](03-orchestration/orchestration-map.md)

### M4 · Choose What Each Agent Step Is Allowed to See
- **Memory & context plan**: [`04-memory-context/memory-and-context.md`](04-memory-context/memory-and-context.md)

### M5 · Bounds & Evals
- **Bounds & evals**: [`05-bounds-evals/bounds-and-evals.md`](05-bounds-evals/bounds-and-evals.md)

### M6 · Autonomy & Production
- **Production & autonomy plan**: [`06-autonomy/production-and-autonomy.md`](06-autonomy/production-and-autonomy.md)
- **Prototype + six run transcripts**: [`06-autonomy/prototype.md`](06-autonomy/prototype.md)
- **Build insights**: [`06-autonomy/build-insights.md`](06-autonomy/build-insights.md)

**Build:** Claude Code · `gpt-4o-mini` · runnable agent in [`00-build/`](00-build/) (see `prototype.md` → *How to run it*).

---

## Ship plan

### Autonomy dial (per segment)
- Seasoned PM / project owner → Bounded-autonomous: routine weekly updates are low value to hand-check; I only review the post.
- New PM → Supervised: I'd treat their Cortex output like a junior PM's work, review it and give extra instructions before anything goes out.
- Eng lead (backlog) → Assisted: Cortex retrieves the right backlog items; deciding what goes into a sprint is the eng lead's job.
- The dial never moves the agent line: flagging at-risk, choosing escalations and approving a company-wide update stay human for everyone.

### Trust Ladder rung + eval gate
Current rung: **Supervised**, every draft held for my approval. **0 production runs so far**; all evidence is offline fixture runs.
Eval gate to climb to Bounded-autonomous: EV-1 to EV-6 pass 100% in CI on every change (EV-5 jailbreak 3/3), AND over **8 weeks and ≥ 20 supervised runs**, **≥ 95%** of drafts approved with no factual edits and **0** EV-5/EV-6-class incidents. The 20 approved runs become the golden set.
Clean incident record: zero EV-5/EV-6-class incidents, zero kill-switch activations caused by Cortex misbehaving, every bound trip escalated with nothing posted.

### Deployment plan
- Runtime: serverless. The M2 loop is hook + cron (backup): short, bursty runs (≤ 120s) with nothing running in between.
- Operator: me (PM-builder). Cortex is turned off (`CORTEX_DISABLED` + cron disabled) when I'm away. Engineering ops takes over if/when it becomes a company-wide tool.
- Rollback: flip `CORTEX_DISABLED` (no write tool, so nothing to undo) → revert to the last commit that passed CI → drop the dial a rung.
- Monitoring: eval pass %, approved-with-no-factual-edits %, escalation rate, cost per run, trust incidents.

### ROI metrics + widen-autonomy rule
- Outcome: PM time per weekly update, 30 min by hand → ≤ 10 min review while supervised → ≤ 2 min once bounded-autonomous.
- Cost-to-serve: ≤ $0.50 model spend per approved update.
- Trust incidents: 0 EV-5/EV-6-class per 20 runs.
Widen rule: up one rung for one segment at a time, only after ≥ 20 supervised runs over ≥ 8 weeks with ≥ 95% approved with no factual edits, 0 incidents, EV-1 to EV-6 at 100% in CI, and median review ≤ 10 min. Any single trust incident drops it back one rung.

### Governance & strategy
- Compliance: API keys and customer PII never enter a prompt; embargoed roadmap items (Orbit, Pulsar) never leave in an output.
- Safety: posting, company-wide approval, GA dates / launch gates, at-risk flags and escalation choices stay above the line for everyone; Cortex has no post tool; kill switch `CORTEX_DISABLED`.
- Reliability: caps (8 iterations, $0.50/run + $5/day, 10 stories, 2 revisions); escalate-on-stuck; if the model is down, halt + escalate and I write the update by hand.
- Strategy: promote the seasoned-PM segment to bounded-autonomous first, gated by the eval gate above.

---

## Build insights

- **Friction.** The build went really well; the evals were the friction. Approving each artifact for a golden set is really time-consuming.
- **Key learning.** Building the agent is easy; the evals work is tedious, but a golden set of approved outputs is what keeps quality high at a reasonable cost to serve.
- **Aha moment.** An agent is a delicate balance between a prompt, a skill, a tool and evals. Some agents could just be skills, but you need to tell the agent to use that skill, or it never will.

---

_Certification submission, Agentic Workflows & Loops Certification._
