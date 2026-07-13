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

**The full vision:** a consultant drops their URL once and the brain generates every sales asset they need — one-pagers, web copy, ad sets, LinkedIn posts, cold emails — each personalized to a specific buyer persona and consistent with their positioning, with no re-entry of context. Every generator reads the same brain, so output compounds instead of resetting.

**v1 tests the foundation of that thesis.** The entry point is a free, **no-login positioning capture**: drop your URL + 2 competitor URLs → get a framework-mapped read that surfaces an "I've never articulated this about my firm" insight. Save it, and it becomes the brain every generator reads from. The first generator — a case-study one-pager — is the proof of concept; the rest are demand-gated behind in-product votes.

| Gate | Metric | Target |
|---|---|---|
| Aha (pre-build gate) | Prompt chain produces a table-stakes-vs-wedge read, not generic advice | ≥2 of 3 real test sites (**passed 4/4**) |
| Top of funnel | Consultants who run the capture | ≥10 |
| Conversion | Click "save your positioning" (→ account) | ≥30% |
| **True demand signal** | A user returns to generate a **second** asset | ≥1 |

The third metric is the falsification test for the whole thesis. If people love the free read and never save, the brain thesis is wrong — and the design deliberately makes that cheap to learn. Business impact model: free with rate limits → validated demand → paid tiers (deferred until the return-user signal exists).

*v2 scope and its own metrics table: see §7.*

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

*v2 phases (6–9) continue this table's numbering — see §7.*

## 7. v2 Enhancements

v1 validated the foundation: an anonymous read that surfaces a wedge, a save, and one generator that reads the brain. v2 deepens the input side (the brain learns from more than a website) and broadens the output side (a second generator plus user-runnable research), and it is the point at which some v1 non-goals were always meant to graduate.

**What graduates from v1 non-goal to v2 scope:** *additional generators* (v1 gated these behind in-product votes; the LinkedIn Content Strategy generator is the first tile to graduate on demand signal) and *agent-run research* (implicitly deferred; now a first-class, user-runnable capability). **What stays a non-goal:** scheduled/proactive runs, paid tiers, and the full 5-object brain (Brand/Design and GTM-framework mapping remain post-validation). v2 does *not* reverse those — the two new objects it adds (§7.2) are input stores, not the deferred brain objects.

### 7.1 Multi-source positioning intake ("Evidence Locker")

**The bet:** a website is what a firm says *publicly*; a sales deck or a proposal is what it says *when money is on the line*. The gap between the two is where the real wedge lives. v1 read only the public surface; v2 captures both and surfaces the delta.

**Entry model.** The website URL stays the anonymous fast path — the cost and abuse posture from v1 is unchanged (it remains the only public LLM surface). Uploads are *complementary*, offered in two places: (a) immediately after the anonymous read, as "sharpen this read" enrichment, and (b) in the authed brain as a persistent Evidence Locker. Thin-site users — who in v1 fell back to paste-a-blurb — can now lead with uploads, upgrading that degradation path.

**What to upload — opinionated, not a dropzone.** A checklist UI, each item labeled with *why it helps* and a per-type expected-signal tag:

| Upload | Why it helps | Signal |
|---|---|---|
| Sales/pitch deck (the one actually sent) | How you pitch when it matters | Highest |
| Proposal / SOW | Scope language, pricing framing | High |
| Cold/intro email or outreach sequence | How you open and frame value | High |
| Case study / testimonial docs | Named proof points and outcomes | Medium |
| Bio, speaker one-liner, LinkedIn About (paste) | Self-description in your words | Medium |
| Discovery-call notes / transcript (optional) | Unfiltered buyer language | Highest-variance |

Formats: PDF, PPTX, DOCX, TXT/MD, pasted text, `.eml`. An explicit **"don't upload"** list — contracts, financials, anything with client PII they can't share — plus an automatic PII-scrub pass before extraction.

**Parsing pipeline.** file → text/structure extraction (slides preserved as ordered sections; email threads split by sender) → per-document typed extraction (claims, proof points, named clients/metrics, offer structure, pricing signals, tone) → **source-attributed evidence units** written to a new `evidence` object with per-unit provenance (document, page/slide) → a synthesis pass that re-runs the Dunford map over website + evidence *together*.

**Synthesis is delta-first.** The read explicitly surfaces: (1) claims consistent across sources → confidence up; (2) claims the deck makes that the website doesn't → *"your site undersells this"*; (3) contradictions → review-queue items, never silently resolved (the v1 explicit-choice rule holds).

**Guardrails.** Provenance is mandatory on every extracted claim — this extends the v1 fabrication rules: every metric is attributed *to a specific source document*. Uploads never auto-overwrite locked fields. Anonymous uploads are size/count/rate-limited and carry the same 24h TTL as the anonymous capture.

### 7.2 Richer positioning capture

Elaborations that stay on-thesis (structure + GTM opinion, affirming tone, the ≤3-tweaks discipline):

- **Confidence + provenance per positioning field.** Each brain field shows what it's based on (site / deck / user-stated) and how well-evidenced it is. Low-confidence fields become the interview questions the generators already ask — unifying the ≤3-question interview with capture gaps.
- **Complete the Dunford loop.** v1 maps attributes, table stakes, and the wedge; v2 adds the remaining components explicitly: market-category framing, target-segment sharpening (feeds personas), and relevant trends (feeds the research agents in §7.3 — trends are where POV comes from).
- **Say/sell delta view.** The website-vs-deck comparison from §7.1 as a first-class screen — the new "aha" candidate for repeat visits.
- **Positioning changelog.** The brain is durable; make change visible over time. This supports the "manage positioning over time" job surfaced in the original diagnosis.

**Data-model note.** v2 adds exactly two adjunct **input-store** objects — `evidence` (§7.1) and `research` (§7.3) — that feed the existing three brain objects (Messaging/POV, Competitors, Personas). They are *not* the deferred 5-object brain; that (Brand/Design, GTM-framework mapping) remains post-validation. Everything else in §7.2 elaborates the existing three.

### 7.3 Broadened outputs: a second generator + research agents

**(a) LinkedIn Content Strategy generator.** Reads the brain (positioning, personas, wedge, trends) and produces a durable, editable *strategy* artifact — not one-off posts:

- 3–5 content pillars derived from wedge + persona pains, each with rationale traced to brain fields
- A POV stance per pillar (what the user believes that the market doesn't say — fed by the research agents below)
- A cadence plan + post-format mix, and 10–15 starter hooks/angles mapped to pillars
- Same pipeline shape as the one-pager: persona pick → ≤3-question interview → fire-and-forget run → artifact. The strategy doc is pinned/durable (not 48h-TTL), because it's a living plan.
- This category was a "coming soon" demand-gated tile in v1; it is the **first demand-gated generator to graduate**.

**(b) Research agents ("Field Scouts").** User-runnable agents that scan what's being said about a topic/industry so the user can stake a differentiated POV.

- **Registry model.** Each agent is a typed definition (name, sources scanned, input schema [topic + optional industry/persona], output schema). Seeded with three ports of the discourse agents behind this project: LinkedIn discourse, podcasts/VC-blogs, and Substack/newsletters. The registry is extensible — *additional research-agent examples TBD from Shaalin.*
- **Output = a discourse map**, stored in the `research` object: dominant frames, contrarian takes, vocabulary in use, and gaps/white space — each item source-cited. The white-space section is the bridge to POV: *"here's what no one with your wedge is saying."*
- **POV synthesis step.** Agent results × the user's wedge → 2–3 candidate POV angles, ranked, each traceable to (brain field + discourse gap). These feed the LinkedIn generator's pillar stances — the two features compound: the agents make content non-generic, the brain makes it on-message.
- **Run model.** Authed-only (never anonymous — cost), fire-and-forget with polling like the generators, weighted cost units (research runs are expensive), results cached per topic with staleness timestamps.
- **Guardrails.** Every discourse claim is cited to a real URL/source; agents report *"not found"* rather than inventing consensus (the fabrication rules extend to research). **Source-access boundary:** agents use only publicly accessible, permitted sources; paywalled/private/auth-gated material (private LinkedIn posts, paywalled newsletters, unavailable transcripts) is reported as unavailable, never summarized from memory, and every citation names the exact accessible artifact used (post URL, article URL, episode page, transcript source). Each seeded agent's coverage is bounded by permitted accessible sources — the LinkedIn agent in particular starts with public posts/pages and user-provided URLs, since much LinkedIn content is login-gated; this is a bounded scan, not a broad platform scrape.

### v2 success metrics

| Signal | Metric | Why it matters |
|---|---|---|
| Intake adoption | Upload attach rate — % of saved brains with ≥1 uploaded document | Tests whether the website-only read left signal on the table |
| Delta aha | User accepts ≥1 "your site undersells this" suggestion | The §7.1 bet, made observable |
| Research → use | Within 7 days of an agent run, user marks a POV angle as *used* or attaches a published-post URL (in-product event; no posting integration exists) | Tests whether research changes what the user actually says |
| Second-generator return | A user returns to generate from the **second** generator | The v1 falsification metric, now with a second generator to return to |

### v2 phases (continuing the §6 numbering)

| Phase | Scope | Acceptance |
|---|---|---|
| 6 | Evidence Locker: intake (checklist UI + PII scrub) → parsing → `evidence` object → delta-first synthesis over site + docs | Upload of each supported type produces source-attributed evidence units; delta view flags ≥1 site-undersells item on a real deck |
| 7 | Capture elaborations: per-field confidence/provenance, completed Dunford loop, say/sell delta screen, changelog | Every brain field shows provenance + confidence; low-confidence fields drive the interview; changelog records edits |
| 8 | Research-agent registry + first 3 agents (LinkedIn, podcasts/VC-blogs, Substack/newsletters) + POV synthesis | Each agent returns a source-cited discourse map; synthesis yields 2–3 traceable POV angles; unavailable sources reported, not fabricated |
| 9 | LinkedIn Content Strategy generator (reads brain + research) | Produces a durable strategy artifact with pillars traced to brain fields and stances fed by §8 research |

Order rationale: intake deepens the brain first; the research agents precede the LinkedIn generator because the generator's pillar stances consume research output.

**v2 non-goals:** auto-posting to LinkedIn, scheduled agent runs, an agent marketplace / user-authored agents, and paid tiers (still demand-gated).
