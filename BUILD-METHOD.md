# How this was built — the course frameworks in production

This capstone's build system is Hamza Farooq's Claude Code stack: **CLAUDE.md context, a skills library, delegated sub-agents, and a ground-truth eval loop** — plus a persistent decision log. This doc shows each tier with real receipts. (The product source is private — the prompt chains *are* the product's moat; see "What's public vs. private" below.)

## Two repos, one boundary

The build deliberately splits **source/method** from **product**:

- **Builder repo** (private): the original Agent.ai agent definitions (raw material), a reusable 5-phase builder kit, `_project/` (the editable project charter), captured output baselines, and all Claude Code config.
- **Product repo** (private): the Next.js app that ships to Vercel.

The dependency arrow is one-way — the product reads the source, never the reverse. Both repos carry a `CLAUDE.md`; the builder repo's opens with the boundary so no session ever confuses method with product.

## Tier 1 — CLAUDE.md as the root context

The builder repo's `CLAUDE.md` defines: what the repo is, the two-repo boundary, the three tiers and when each fires, response conventions, and a pointer to the live project definition in `_project/`. The product repo's `CLAUDE.md` carries the stack, run model, API quirks (hard-won: e.g. a vendor API that returns "building" forever on one format and must be polled on another), phase status, and accepted debt — so any session starts with the project's real state, not a guess.

## Tier 2 — Skills (repeatable prompt templates)

Six skills wrap the builder kit's 5 phases plus evaluation:

| Skill | Phase | Does |
|---|---|---|
| `/ideate` | 1 | Problem, persona, quality-bar pre-check before anything gets built |
| `/write-prd` | 2 | Structured PRD from the idea |
| `/build-actions` | 3 | Delegates to the `actions-builder` sub-agent → validated workflow JSON |
| `/polish-voice` | 4 | Brand voice across all user-facing copy |
| `/market-brief` | 5 | Product Marketing Brief + launch hooks |
| `/skill-evaluator` | eval | Scores output against captured ground truth |

## Tier 3 — Sub-agents (isolated-context specialists)

| Sub-agent | Job |
|---|---|
| `actions-builder` | Generate validated agent workflow JSON from a PRD |
| `prd-reviewer` | Critique a PRD against the quality bar before build |
| `actions-to-ts` | Port a legacy agent workflow into the product's TypeScript structure |
| `baseline-evaluator` | Score migrated output against captured Agent.ai baselines (2/1/0 rubric per dimension: parity / drift / missing-or-hallucinated) |
| `brain-evaluator` | Score GTM Brain generator output |

Each is a markdown definition with scoped tools (evaluators are read-only) and an output contract. Sample rubric from `baseline-evaluator`:

> **2** = parity — same structure, equivalent content · **1** = drift — right section, materially different detail · **0** = missing, wrong, or hallucinated

## Tier 4 — The eval loop (the part that changed decisions)

Two layers:

1. **Baseline parity.** Before the old platform shut down, real outputs were captured as ground truth (`docs/baselines/`). Ported agents had to score parity before being called done.
2. **Headless generator evals.** Scripts drive the real prompt chains against **synthetic edge-case businesses** (thin proof, 4-service firms, no-metrics design studios…) across all 5 templates, then score against a rubric: no-padding, metric-once, page-budget, footer-integrity.

**A concrete story of the loop earning its keep:** the `metric-once` check kept failing on two scenarios — the model repeated a career credential ("took two companies to $50M ARR") in both the intro and a proof badge. Two rounds of prompt hardening improved it (intra-section repeats disappeared) but did not eliminate the intro↔badge repeat. The eval's verdict: **this failure class is beyond prompt engineering and needs a deterministic post-generation repair pass** — now logged as scoped debt rather than shipped as a surprise. It also caught its own false positive: an ICP-defining range ("60–90 days of close") flagged as a repeat when the generation rule explicitly permits definitional ranges — the rubric was stricter than the rule. Evals need evals.

Real-tester feedback closed the same loop from the other side: an early one-pager **invented engagement-model structure** and **misattributed a client statistic**. That produced binding generation rules (no invented structure; strict attribution) and a redesigned pre-generation interview that asks exactly the questions whose answers prevent those fabrications.

## Memory & the Rethink Gate

A `memory.md` decision log records every repo-level decision with date and reason. The most consequential entry: the build was **paused at a planned "Rethink Gate"** after infrastructure was up. The diagnostic confirmed that the real job was helping users manage their positioning and create content from it — not just surface a one-session read. The project was re-scoped around GTM Brain accordingly. The prior plan is still in the repo, marked as superseded — the paper trail is the point.

Plans themselves go through an **iterate-review loop**: each implementation phase that touches production surfaces gets an external AI code review pass before merge, with checkpoints tracked in the plan doc.

## Primitives deliberately not used

No hooks, no plugins. The workflow is session-driven (skills + sub-agents cover it); hooks would have added event-driven complexity with no triggering events in this build. Choosing *not* to use a primitive is part of using the framework well.

## What's public vs. private — and why

Public (this repo): the method, architecture, PRD, eval rubric structure, and **real eval artifacts** (`artifacts/` — sample one-pagers for synthetic businesses, generated by the production chain). Private: the prompt chains (positioning extraction, framework mapping, generation, humanizing, interview design). The product's thesis is *"tools commoditize, taste compounds"* — the prompts are the taste. Publishing their structure, their eval scores, and a live product to judge their output demonstrates the system without giving away the one layer that's defensible.
