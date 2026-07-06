# PRD — GTM Brain

**Product:** GTM Brain (working title) · **Live:** https://gtm-agents-web-neon.vercel.app
**Owner:** Shaalin Parekh, Industry GTM · **Status:** v1 shipped to production; supervised beta next
**Capstone for:** Hamza Farooq's *Agentic AI for Product Managers*

---

## 1. Problem statement

Solopreneurs and boutique consultancies are excellent at their craft and weak at articulating it. To sell, they need content — one-pagers, landing pages, emails — but good content requires knowing *who you sell to* and *what makes you different* first. Most skip that step and ship generic content.

Their current workaround is raw ChatGPT: they **re-explain their business every session** and still get generic output, because a chat thread has no durable, structured model of their go-to-market.

**Diagnostic evidence over enthusiasm.** Earlier work (a persona builder with 17,000+ uses) validated that orienting around buyer personas resonates with consultants. But personas alone weren't actionable — users couldn't manage their positioning over time or create content from it. A YC-style interrogation of the job-to-be-done surfaced the real gap: a durable, editable GTM model that generators could read, so users never re-explain their business again.

**The bet:** users don't need more content tools. They need a **shared, persistent, editable GTM brain** that agents read from and write back to — so every generated asset is on-message and on-persona without re-explaining anything. Persistence alone is not a moat (ChatGPT has memory); the moat is **structure plus a GTM opinion** — positioning mapped to real product-marketing frameworks (April Dunford's *Obviously Awesome*) that teaches users something about their own business.

## 2. Target users

**Primary (v1 beachhead): the boutique/advisory consultant.** 1–5 person firms selling projects against larger firms (e.g., a post-merger-integration shop competing with Big-4-alumni firms). Strong craft, weak differentiation language, deal-driven urgency: they lose when they can't articulate why them.

**Secondary (explicitly deferred): solopreneur creatives** — e.g., an independent design-studio founder (our earliest real tester). Demoted post-diagnosis: consultants have a sharper job-to-be-done (winning specific deals) and higher willingness to act on positioning.

## 3. Success metrics & business impact

The v1 wedge is a free, **no-login positioning capture**: drop your URL + 2 competitor URLs → get a framework-mapped read of your positioning that surfaces an "I've never articulated this about my firm" insight. It manufactures demand while producing the brain's intake material.

| Gate | Metric | Target |
|---|---|---|
| Aha (pre-build gate) | Prompt chain produces a table-stakes-vs-wedge read, not generic advice | ≥2 of 3 real test sites (**passed 4/4**) |
| Top of funnel | Consultants who run the capture | ≥10 |
| Conversion | Click "save your positioning" (→ account) | ≥30% |
| **True demand signal** | A user returns to generate a **second** asset | ≥1 |

The third metric is the falsification test for the whole thesis. If people love the free read and never save, the brain thesis is wrong — and the design deliberately makes that cheap to learn. Business impact model: free with rate limits → validated demand → paid tiers (deferred until the return-user signal exists).

## 4. Personas & use cases

1. **The anonymous prospect.** Drops their site + 2 competitor URLs → gets a positioning read separating table stakes from their evidence-backed wedge. Success: visible "aha" + save click. *(Tone rule learned in testing: the read is an affirming distillation — "capture your positioning," never a "teardown." ≤3 proposed tweaks, side-by-side and editable.)*
2. **The brain owner.** Signs up; the capture graduates into a persistent brain (Messaging/POV, Competitors, Personas). Edits and locks fields; generates 2–3 buyer personas from the positioning; optionally **grounds** a persona in real buyers (LinkedIn profiles → evidence-cited decision-making analysis).
3. **The deal-chaser.** Picks a persona + template, answers ≤3 interview questions, generates a **case-study one-pager** that reads the brain — zero re-entry of business context — and downloads the PDF for a live deal.

**Fabrication guardrails (from real-tester feedback):** an early one-pager invented engagement-model "buckets" the user didn't offer and misattributed a client statistic. This produced three binding rules: no invented structure, every metric attributed or unattributed-by-construction, and a pre-generation interview designed to close exactly the information gaps the model would otherwise fill by guessing.

## 5. Technical architecture

```
ANONYMOUS (public, rate-limited)          AUTHED (Clerk)
┌─────────────────────────────┐   save    ┌──────────────────────────────┐
│ /positioning capture         │  ──────▶ │ THE BRAIN (Neon, per-user)   │
│ scrape (Jina→Readability)    │  claim-  │ messaging_pov · competitors  │
│ → extract → Dunford map      │  token   │ · personas (durable, TTL-    │
│ → rendered read (ephemeral,  │  gradu-  │ exempt) — pull → propose →   │
│   24h TTL, no PII)           │  ation   │ refine → lock, always        │
└─────────────────────────────┘           │ editable                     │
                                          └──────────┬───────────────────┘
                                                     │ reads (never re-asks)
                                                     ▼
                                          ┌──────────────────────────────┐
                                          │ GENERATORS: one-pager (5     │
                                          │ templates, interview, brand  │
                                          │ colors, persona-angled) —    │
                                          │ fire-and-forget run model,   │
                                          │ poll, HTML artifact → PDF    │
                                          └──────────────────────────────┘
```

| Layer | Choice |
|---|---|
| App | Next.js 15 (App Router) monorepo on Vercel |
| Auth | Clerk |
| Data | Neon Postgres + Drizzle (brain objects durable; run artifacts on 48h TTL unless pinned) |
| Rate limiting / cost | Upstash Redis — per-IP anonymous limits (fail closed), weighted per-user cost units, global daily spend cap |
| LLM | Anthropic Claude (Opus) — structured-output chains with zod validation + one repair retry |
| Scraping | Jina Reader → Readability fallback; thin-site degradation path (paste-a-blurb) |
| Design system | Shell + data injection — the LLM fills typed fields; it never generates CSS. Brand accent extracted deterministically from the user's site; applied only on explicit user choice |

Key security/cost decisions: the anonymous capture is the only public LLM surface — rate-limited from the first deploy, spend cap fails closed. Claim-token graduation is an atomic GETDEL (first claim wins). No anonymous write path to brain tables.

## 6. Implementation & deployment plan

Phased plan with acceptance criteria per phase; each code phase passed an external AI code review (iterate-review) before merging.

| Phase | Scope | Status |
|---|---|---|
| 0 | Aha gate — prompt spike vs. real consultant sites before any build | ✅ 4/4 |
| 1 | Anonymous positioning capture: scrape → map → render; abuse controls | ✅ 20/20 smoke incl. spend-cap fail-closed |
| 2 | Brain schema + claim-token graduation (additive-only migrations) | ✅ |
| 3 | Editable brain UI: review queue → brain-as-document; explicit-choice rule (machine suggestions never apply silently) | ✅ |
| 4 | One-pager generator reading the brain | ✅ eval'd 56/60 |
| 4b–4c | Persona builder + grounding; templates, interview, section schema | ✅ eval'd across 15 scenarios |
| 5 | E2E suite | 🟡 automatable specs green; signed-in prod pass in progress |
| Next | Supervised beta (first external testers) → demand-gate readout | — |

**Deployment:** GitHub → Vercel (push-to-main deploys). Rollback is phase-isolated: capture is a route removal; schema changes were additive-only; the generator is an independent module. **Deliberate non-goals for v1:** scheduled/proactive runs, additional generators (demand-voted tiles instead — "coming soon" tiles collect build-this-next votes in-product), paid tiers, the full 5-object brain (Brand/Design and GTM-framework mapping are post-validation).
