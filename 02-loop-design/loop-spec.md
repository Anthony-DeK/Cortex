# Loop Spec: Cortex PM Chief-of-Staff Agent

> Module 2 · Loop Engineering, ★ Deliverable 2
>
> ✅ **What this validates:** the agent knows when to run and when to stop, by the end you'll have proven a one-page Loop Spec with a trigger, a definition of "done," and explicit stop conditions.
>
> Your one-page blueprint for how the work you handed to the agent (M1) actually *runs*.
> An agent is just a prompt that fires itself, this spec says when it fires, what "done" means, and what it needs to do the job. Living document; refine as the course progresses.

## 1. Trigger & loop type

**Chosen type:** hook + cron (backup)

- **Primary: hook** on an inbound task message from the product lead (e.g. "put together this week's update / propose stories from the PRD").
- **Backup: cron**, Monday 9am, drafts the weekly status update if no one has asked yet that week.

**Why:** It either triggers because the user asked for it or automatically every week.

**Ruled out:**
- **Heartbeat**: nothing changes minute to minute, so polling burns money for nothing.
- **Cron only**: ad-hoc asks (e.g. stories before sprint planning) would wait until Monday.
- **Hook only**: if nobody asks, no update gets made; the weekly cadence would depend on someone remembering.
- **Goal** (as the trigger): it describes how a run iterates until it's validated, not what starts it. Cortex already iterates inside each run (drafter ↔ critic).

**Idempotency / dedupe:** every run is keyed on **project + ISO week**, which is also the draft's filename: `<MMDD>-<project>-status-update.md`, where `<MMDD>` is the Monday of that week (e.g. `run-output/1012-P-NORTH-status-update.md`). Before drafting, Cortex checks whether a draft already exists for that key and skips if so. This covers a duplicated message and the Monday cron firing after a hook run already drafted that week's update.

## 2. Goal / definition of done

**Done =** the weekly status update is drafted from the pulled activity, any proposed stories are queued for approval, and the draft is saved for the PM. Cortex never posts or sends.

**Self-validation:** an independent critic (a separate model call that never saw the drafting context) checks the draft against the source data: traceable claims, norms respected, no confidential leak, nothing posted or committed. Cortex never grades its own draft.

**Critic feedback loop (within a run):** on a rejection, the critic returns, for each problem, the **norm it broke** and a **fix instruction**, without repeating the sensitive content itself. Cortex keeps a running "don't repeat" list across revisions in the same run.
*Why:* What was wrong is not enough, Cortex needs to be told which rule it broke.
*Later (M4 plan):* save approved lessons so future runs start with them.

## 3. Stop conditions

| Condition | What it looks like | What happens |
|---|---|---|
| **Success** | Critic returns `pass` **and** the draft is saved under its project + week key | Stop at the HITL checkpoint; PM is told it's waiting for review |
| **Stuck / give up** | Any one trips: a required source (project, activity, norms) fails after **3 attempts** · critic rejects **2×** (revision cap) · **8 iterations** without an accepted draft · run cost reaches **$0.50** | Halt, log the reason, hold the last draft (marked "held") |
| **Escalate to human** | Open **Sev-1** / at-risk status → *Flag at-risk* (above the line) · **unconfirmed date** or launch gate needed → *Decide tone/commitment level* (HITL) · **confidential / embargoed** item involved → *Approve a company-wide update* (above the line) · story batch **over the cap** → *Propose a capped story batch* (HITL) · **already-queued story that shouldn't have been** (no withdraw tool) → *Propose a capped story batch* (HITL) | Stop and hand off to the PM with the reason and the last draft; nothing posted |

## 4. State

No more context is saved within or across runs than what's described here.

- **Within a run:** the conversation, the bound counters (iterations, revisions, cost), and the critic's "don't repeat" list (§2). Discarded when the run ends.
- **Across runs:** one saved draft per **project + ISO week** (read by the dedupe check in §1), plus approved project facts, decisions and norms with their sources (per the M1 memory plan). Unapproved drafts and unverified inferences are discarded.
- **Scope:** per project. A run never loads another project's state, so confidential items can't leak across projects.

## 5. The five things a loop can lean on

_`state` is always-on. `connectors` only if you already have one wired (e.g. a Jira key or Google MCP), otherwise just note it as a plan. `skills`, `subagents`, `work tree` scale with autonomy; "not needed yet, because…" is a valid answer._

| Component | For Cortex |
|---|---|
| **Work tree** (isolated workspace per run, a git worktree) | Not needed for the weekly update, because it only reads sources and writes a draft. **Needed for the future PRD workflow** (writing PRDs from discovery input), which reads *and* writes documents and should work in an isolated space before a human approves. |
| **Skills** (reusable capabilities) | Not needed for the weekly update, which is one workflow. **Needed for the future PRD workflow**, which requires dedicated skills (e.g. writing a PRD). |
| **Plugins / connectors** (tools & access, optional if you don't have one yet) | None wired to Cortex yet; tools read local fixtures. **Access is granted per workflow, not to Cortex as a whole.** **Weekly update workflow (FullEnrich stack):** **Gmail**, read + create drafts only · **Notion**, read past updates, PRDs and norms, write drafts only · **Slack**, read only. This workflow should only have read and draft access to tools like Gmail and Slack, not write access. **Future workflows:** write access is TBD for now. **Open item: Slack connector missing.** Isolate it and work out how to build it. |
| **Subagents** (independent check when the loop can't grade itself) | The **critic** is already in place as the independent check (§2). **Direction:** Cortex as a swarm of agents for dedicated workflows and tasks, each with its own tools and access. Whether each agent earns its place → M3 orchestration-map.md. |
| **State tracking** | See §4. Per-run counters + "don't repeat" list; per project + week draft; approved facts with sources. |

> Context plan (M4) and the hand-off to bounds & evals (M5) come in later modules, you'll add them to their own deliverables then, not here.

## Link to live loop

[`00-build/agent.py`](../00-build/agent.py) (loop + stop conditions), [`prompts.py`](../00-build/prompts.py) (definition of done, critic rules), [`critic.py`](../00-build/critic.py) (structured feedback). Traces: `00-build/happy-run.txt`, `00-build/missing-data-run.txt`.
