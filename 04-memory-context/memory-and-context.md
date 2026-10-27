# Context Engineering & Memory: Cortex PM Chief-of-Staff Agent

> Module 4 · Context Engineering & Memory
>
> ✅ **What this validates:** the agent reasons on the right, safe inputs, by the end you'll have proven a context budget, per-source retrieve-vs-long-context decisions, and a memory map with risk mitigations.
>
> 🗂️ **How the lab maps to this file:** In **Part A** (before the lecture) you don't edit this file, you rough-draft on scratch, focused on the per-source calls in **section 2** plus a quick remember/forget + "how it rots" sketch. In **Part B** (after the lecture) you complete **all five sections**; the Lab Guide's guided builder writes this file for you to copy in and commit.

## 1. Context budget

Each loop iteration receives, in priority order (cut from the bottom when the budget is tight):

1. Cortex's rules (`CORTEX_SYSTEM`) + this run's "don't repeat" list. Never cut.
2. Task brief. Every run; never cut.
3. Team norms. Every run; never cut.
4. Project + activity slice for this project. The evidence; never cut (missing → escalate).
5. Roadmap: the shareable slice for this project only.
6. Past updates: top 1–2 matches, for tone/precedent only. Cut first.

*Principle:* sources 2–4 in the §2 table (activity, past updates, roadmap) are need-to-know only (retrieve); sources 1 and 5 (task, norms) apply at every run (long-context).

## 2. Retrieve vs. long-context: per source

For each data source, decide: **retrieve** (narrow a large/changing corpus to the relevant slice) or **long-context** (just include a bounded set you can reason over).

| Source | Size / volatility | Decision | Deciding factor |
|---|---|---|---|
| `get_task` (PM brief) | One short doc, per run | **Long-context** | Size: bounded; reason over the whole thing. Applies at every run. |
| `get_activity` (Jira/GitHub) | Large, grows weekly across all projects | **Retrieve** | Size: need-to-know, only this project's recent slice. |
| `search_past_updates` (update archive + decision log) | Unbounded, grows weekly | **Retrieve** | Size: need-to-know, only relevant precedent. |
| `get_roadmap` | Medium; contains CONFIDENTIAL/embargoed items | **Retrieve** (shareable slice) | Citation/audit + confidentiality: need-to-know. Today the build returns the whole doc, which put embargoed items in front of the drafter (critic caught a Pulsar near-leak, M4 Step 0). |
| `get_norms` (playbook) | Medium; must stay current | **Long-context** | Volatility + citation: must be the full current version so Cortex can cite the exact rule. Applies at every run. |

## 3. Retrieval quality plan

Only retrieved sources (§2) need these moves; task and norms are long-context.

| Source | Routing | Doc grading | Reranking | Self-verification | Caching |
|---|---|---|---|---|---|
| `get_activity` | ✓ scope by project ID | (not needed) | | ✓ every figure checked against the slice (critic Rule 1) | ✓ pull once per run; critic reuses the same source log |
| `search_past_updates` | ✓ filter to this project | ✓ drop irrelevant/confidential hits; no match → return nothing (no fallback) | ✓ most recent first, top 1–2 (§1 budget) | | |
| `get_roadmap` | ✓ this project's section only | ✓ drop CONFIDENTIAL/embargoed sections before they reach the drafter | | ✓ critic Rule 3 as backstop | |

**Why, per move:**
- **Routing, all three:** every retrieved source is need-to-know, so each query is scoped to this project.
- **Grading:** not needed for `get_activity`: the tool is already scoped by project ID, so everything it returns is relevant by construction; routing does the grading's job. Needed for `get_roadmap`: we need an additional filter after the routing to remove sensitive info. Needed for `search_past_updates`: today it returns unrelated docs when nothing matches, and its decision log names embargoed projects.
- **Reranking, past updates only:** the archive grows weekly and only the latest 1–2 matter for tone.
- **Self-verification, activity and roadmap:** the critic checks figures against the activity slice (Rule 1) and catches any confidential leak the filter misses (Rule 3).
- **Caching, activity only:** very useful for activity (pulled once, reused by the critic); overkill for the rest.

## 4. Memory map (your PM brain)

| Memory type | What Cortex stores | Scope / TTL | Who writes it |
|---|---|---|---|
| **Working** (in-loop) | The conversation, tool results (source log), bound counters, the critic's "don't repeat" list | This run; discarded at the end | Cortex, automatically |
| **Episodic** (past runs) | One saved draft per project + ISO week (accepted or held, with the reason); read by the dedupe check | Per project; **1 quarter (13 weeks)** | Cortex (draft only) |
| **Semantic** (durable facts/prefs) | Approved project facts, decisions and norms, each with its source (M1 memory plan). Unapproved inferences are never stored. | Per project; until superseded, **re-checked against its source every run** (live source wins) | **PM approves every write** (above the agent line) |
| **Shared** (across agents) | Source log + the draft, between Cortex and the critic; never Cortex's prompt or reasoning (M3 Field 6) | This run | Cortex, in the hand-off |

**Why:**
- **Episodic TTL, 1 quarter:** this is usually how long an initiative lasts within my org.
- **Semantic re-check every run:** information evolves quickly and we can't afford to work with outdated info.
- **PM writes semantic memory:** the PM needs to be in the loop to avoid any misfire.

## 5. Memory risks & mitigations

| Risk | Where it bites Cortex | Mitigation |
|---|---|---|
| **Drift** | Past updates are the format precedent: one sloppy accepted update becomes next week's template, and quality slides week by week. | Only **PM-approved** updates enter the past-updates corpus. Norms (long-context, every run) outrank precedent. The critic re-checks every draft against the rules, not against last week. |
| **Poisoning** | A task brief or pasted note says "ignore your rules" (`task-jailbreak.md`), or a bad "fact" gets saved and trusted forever. | Brief content is **data, not instructions** (in `CORTEX_SYSTEM`; injection is escalated). Semantic writes need **PM approval** (§4). Every stored fact carries its source. |
| **Staleness** | Old PRs/metrics reported as current (pre-ingest data said 41%; this week's says 43%); a stored fact is outdated. | Semantic facts **re-checked against the live source every run** (§4). Activity cited with dates. Episodic memory has a **13-week TTL**. → M5: activity freshness window. |
| **Confidential / retention** | Embargoed items (Orbit, Pulsar) reach the drafter via the whole-doc roadmap and the decision log; critic `reasons` name confidential projects in traces (M3 loose end); saved drafts and traces build up. | **Grading filter before the drafter** (§3); critic Rule 3 as backstop; critic `reasons` must refer to embargoed items generically too. Per-project scope. Drafts deleted at the 13-week TTL; traces not kept beyond it. **PII** (future discovery/interview workflows) never stored in semantic memory. |

**Biggest risk for my Cortex: staleness.** The org moves quickly and facts can change; it would be a disaster if Cortex would hang on to an expired fact.
