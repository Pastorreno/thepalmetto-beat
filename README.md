# The Palmetto Beat

**Status: Active build — launching.** *(Unparked 2026-08-03.)*

A local community news and events newsletter for **Palmetto, Florida**.

> Everything happening in Palmetto, in a five-minute read, three mornings a week.

## The opportunity

Every outlet covering this area — Bradenton Herald, The Bradenton Times, Bay News 9, ABC7 — covers *Manatee County* from Bradenton's side of the river. Palmetto is always the second item in someone else's story. Patch and NewsBreak have Palmetto pages, but they're unstaffed algorithmic aggregators.

**Nobody is doing a human-curated, Palmetto-first digest.** That's the wedge.

## Architecture

Decided 2026-08-03. See [`docs/00-ARCHITECTURE-DECISION.md`](docs/00-ARCHITECTURE-DECISION.md).

**Rent the post office, build the newsroom.**

| Layer | Choice | Why |
|---|---|---|
| **Delivery, list, compliance, growth network** | **beehiiv** (Launch → Scale) | Bulk email deliverability and the recommendation network can't be self-hosted |
| **Sourcing, filtering, AI drafting** | **n8n** ✅ built | The differentiated, reusable part — a potential Melvin vertical for other towns |
| **Seam** | Manual paste, ~5 min/issue | beehiiv's Create Post API is Enterprise-only. Human review stays permanently regardless. |

> ⚠️ Superseded: the original concept's Google Forms → Sheets → Mailchimp → Google Sites stack. See the architecture decision for reasoning.

## Launch plan

| Phase | Focus | Timeline |
|---|---|---|
| **0** | Domain, handles, PO Box, beehiiv setup | This week |
| **1** | Issues #1-8 written manually. Warm list → 500 subscribers. | Weeks 1-6 |
| **2** | ✅ n8n newsroom **built & tested** — 8 RSS feeds → filter → Claude draft → Telegram. Tip intake still to do. | Done 2026-08-03 |
| **3** | Sponsorship sales begin at 500+ subscribers | Week 6+ |

## Docs

| Doc | Contents |
|---|---|
| [`00-ARCHITECTURE-DECISION.md`](docs/00-ARCHITECTURE-DECISION.md) | Build vs. rent, and why |
| [`01-BRAND-AND-FORMAT.md`](docs/01-BRAND-AND-FORMAT.md) | Positioning, voice, the six-section issue format, cadence |
| [`02-BEEHIIV-SETUP.md`](docs/02-BEEHIIV-SETUP.md) | Click-by-click setup, plan comparison, deliverability protection |
| [`03-SOURCES.md`](docs/03-SOURCES.md) | Every Palmetto content source, tiered, with the weekly routine |
| [`04-GROWTH.md`](docs/04-GROWTH.md) | First 1,000 subscribers, in-person tactics, metrics |
| [`05-SPONSORS.md`](docs/05-SPONSORS.md) | Rate card, who to sell to, the pitch, revenue math |
| [`06-NEWSROOM-N8N.md`](docs/06-NEWSROOM-N8N.md) | The n8n workflow — feeds, filter tuning, setup, limitations |
| [`issues/issue-001-TEMPLATE.md`](issues/issue-001-TEMPLATE.md) | Issue #1 skeleton with real storylines to report against |

## Business model

Free newsletter, monetized by local business sponsorships. At ~2,500 subscribers with two sponsor slots per issue: **≈$2,050/month** against ~$43/month in platform cost.

Realistic mature list for Palmetto + Ellenton: **3,000-6,000 subscribers.**

## Non-negotiables

1. Every fact verified against a primary source before publishing.
2. Corrections run at the top of the next issue.
3. Sponsored content is always labeled. Editorial is never for sale.
4. Community publication, not a ministry publication — that neutrality is what lets it serve the whole town.
