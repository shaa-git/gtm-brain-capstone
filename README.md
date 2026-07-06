# GTM Brain — capstone for *Agentic AI for Product Managers*

**Live app:** https://gtm-agents-web-neon.vercel.app — try the free, no-login [positioning capture](https://gtm-agents-web-neon.vercel.app/positioning) on your own site.

**In this repo:** [PRD](PRD.md) · [How it was built with the course frameworks](BUILD-METHOD.md) · [sample outputs](artifacts/) (production chain, synthetic businesses)

---

## Project summary

**GTM Brain is a persistent, editable model of a small firm's go-to-market that AI agents read from and write back to** — so a consultant who's great at their craft but bad at articulating it can generate on-message, on-persona sales content without re-explaining their business to a chatbot every session.

Earlier work validated that orienting around buyer personas was useful — consultants care about getting their positioning right. But that alone wasn't enough: users needed to **manage their positioning over time and create content from it**, not just get a one-session read. GTM Brain was built to close that gap: a persistent, editable model that generators read from, so the work of articulating your business compounds instead of resetting every session.

The v1 wedge manufactures the demand it needs to test: a free, anonymous **positioning capture** — drop your URL and two competitors, get a framework-mapped read (April Dunford's *Obviously Awesome*) of what's table stakes vs. your evidence-backed wedge. Save it, and it graduates into your brain: editable Messaging/POV, competitors, and buyer personas that can be **grounded in real buyers** with cited evidence. Then the generator: a case-study **one-pager** that reads the brain — five templates, a ≤3-question interview designed to prevent fabrication, brand colors pulled from your own site — and never asks you to describe your business again.

The build itself is the course applied end-to-end: two `CLAUDE.md`-anchored repos (method vs. product), six skills wrapping a 5-phase builder kit, five delegated sub-agents, and a two-layer eval loop — baseline parity against captured ground truth, plus headless rubric evals that drove real decisions (including catching a failure class that survived two rounds of prompt engineering and needs a deterministic fix instead). Every deliberate trade-off — what shipped, what's demand-gated behind in-product votes, what stays private and why — is documented in the PRD and build-method docs.

**Status:** live on Vercel; supervised beta next. The falsification metric: does anyone come back for a *second* asset?

---

## Feedback I'm looking for

1. **Run the [positioning capture](https://gtm-agents-web-neon.vercel.app/positioning) on your own site** (or a consultancy you know). Did the read surface anything you hadn't articulated — or did it feel generic? Be harsh; "generic" is the failure mode that kills the thesis.
2. **Look at a [sample one-pager](artifacts/)** — would you send this to a real prospect? What's the first thing you'd change?
3. **Which generator should be next** — web page, ad set, LinkedIn posts, or cold email? (There's a vote for this inside the product, too.)
