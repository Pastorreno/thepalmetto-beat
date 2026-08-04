# Agent Context — The Palmetto Beat

Project-scoped context. Auto-loaded by Claude Code, Codex, Hermes, and OpenCode when cwd is this repo.

**If you are picking this project up cold, read this file, then [`PROJECT-LOG.md`](PROJECT-LOG.md) for what has actually happened.**

---

## What this is

A local community news newsletter for **Palmetto, Florida** — the small city north of the Manatee River, next to Ellenton, in the Bradenton–Sarasota metro. The user (Mareneo "Reno" Flournoy) **lives there**, which drives most of the strategy.

**The wedge:** Bradenton Herald, The Bradenton Times, Bay News 9, and ABC7 all cover *Manatee County* from Bradenton's side of the river — Palmetto is always the second item. Patch and NewsBreak have Palmetto pages but they're unstaffed aggregators. Nobody does a human-curated, Palmetto-first digest.

**Repo:** `Pastorreno/thepalmetto-beat` · **Working copy:** `~/thepalmetto-beat`

## Current state (as of 2026-08-03)

| Thing | State |
|---|---|
| Repo | Unparked. PR [#1](https://github.com/Pastorreno/thepalmetto-beat/pull/1) open on branch `launch-newsroom` — **not yet merged** |
| beehiiv account | ❌ Not created. User must sign up — agents cannot. |
| Domain `thepalmettobeat.com` | ❌ Not registered. **Do this before the name goes public.** |
| Social handles `@thepalmettobeat` | ❌ Not claimed |
| PO Box (needed for CAN-SPAM footer) | ❌ Not obtained |
| n8n newsroom workflow | ✅ Built + tested, ❌ **not imported, not activated** |
| Issue #1 | ❌ Not written. Template only. |
| Subscribers | 0 |

## Architecture — decided, do not relitigate without cause

**Rent the post office, build the newsroom.** Full reasoning in [`docs/00-ARCHITECTURE-DECISION.md`](docs/00-ARCHITECTURE-DECISION.md).

- **beehiiv** = delivery, list, compliance, growth network. Launch (free) → Scale ($43/mo).
- **n8n** = sourcing, filtering, AI drafting. Runs on the **MacBook** (Docker `newproject-n8n-1`, `localhost:5678`), **not** the Mac mini.
- **Seam is a manual paste**, ~5 min/issue.

The user asked about self-hosting the whole thing on the Mac mini. It was considered seriously and rejected — three reasons that still hold:

1. beehiiv's **recommendation network cannot be self-hosted** — the asset is other newsletters' audiences, not software. Biggest early growth lever.
2. **You cannot send bulk email from a residential IP** (port 25 blocked, no sender reputation). You'd relay through Resend/SES anyway.
3. **Bulk-sender compliance fails silently** — get SPF/DKIM/DMARC, one-click unsubscribe, or complaint rate wrong and you're quietly deprioritized by Gmail with no error.

Superseded: the original concept's Google Forms → Sheets → Mailchimp → Google Sites stack.

## Hard-won facts — verify before trusting, but don't re-derive blindly

These cost real debugging time. Re-check if behaviour changed; don't assume they're wrong.

- **beehiiv's Create Post API endpoint is Enterprise-only.** Not on Launch, not on Scale. This is *why* the seam is manual. The Subscriptions API *is* available on Launch.
- **beehiiv Launch (free) excludes** referral program, polls, and automations — so **no automated welcome email**. Includes recommendation network, custom domains, segmentation, non-Send API.
- **`palmettofl.org` has no News Flash RSS.** `ModID=76` looks like it and returns 50 items, but its channel is "Palmetto - Pages", a site directory. 18 module IDs were probed; only Calendar (`ModID=58`) and Agendas (`ModID=65`) work.
- **Google News RSS returns items back to 2015** unless you append `when:7d` / `when:14d` to the query. This silently destroyed the first filter build (304 of 336 items dropped as stale).
- **"Palmetto" has severe false positives**: Palmetto **Bay** FL, Palmetto **Expressway** / Senior High / Panthers (all Miami-Dade, ~200 mi away), Palmetto **GA** (Fulton County), Palmetto Bluff SC, the Palmetto State (SC), palmetto bugs.
- **No RSS available:** Bradenton Times (403/404), Patch (0 items), Manatee County, Manatee Chamber, Manatee County Schools, Bay News 9.

## Editorial rules — these are not style preferences

1. **Verify every fact against a primary source before publishing.** The AI drafts from headlines and summaries, not full articles. It can misread a headline.
2. **Corrections run at the top of the next issue.** Local trust is the entire asset.
3. **Sponsored content is always labeled. Editorial is never for sale.**
4. **This is a community publication, not a ministry publication.** Church content sits in Calendar/Around Town alongside everyone else's. That neutrality is what lets it serve the whole town and sell ads to businesses that would never sponsor a ministry newsletter. The user runs JGM/JGC — keep them separate.
5. **Sensitive stories** (death, crime, hate crime, litigation): plain and factual, attributed to the reporting outlet, **no speculation about motive or blame**.

## Rules

- Do not print secrets.
- Do not delete files without explicit user approval.
- **Always notate. Never silently overwrite the record** — see below.

## Notation & amending notes (standing convention)

**Append to [`PROJECT-LOG.md`](PROJECT-LOG.md) every time you do meaningful work here.**

When something previously recorded turns out to be wrong or outdated, **do not edit or delete the original entry.** Append an **AMENDMENT** that points back to it. The record of what we believed and when is often more useful than the current answer alone — it's how you avoid re-making a decision that was already made for good reasons.

See `PROJECT-LOG.md` for the entry format.

## Docs

| Doc | Contents |
|---|---|
| [`docs/00-ARCHITECTURE-DECISION.md`](docs/00-ARCHITECTURE-DECISION.md) | Build vs. rent |
| [`docs/01-BRAND-AND-FORMAT.md`](docs/01-BRAND-AND-FORMAT.md) | Positioning, voice, six-section issue format, cadence |
| [`docs/02-BEEHIIV-SETUP.md`](docs/02-BEEHIIV-SETUP.md) | Setup, plan comparison, deliverability protection |
| [`docs/03-SOURCES.md`](docs/03-SOURCES.md) | Tiered sourcing playbook, RSS status per source |
| [`docs/04-GROWTH.md`](docs/04-GROWTH.md) | First 1,000 subscribers |
| [`docs/05-SPONSORS.md`](docs/05-SPONSORS.md) | Rate card, prospects, pitch, revenue math |
| [`docs/06-NEWSROOM-N8N.md`](docs/06-NEWSROOM-N8N.md) | Workflow: feeds, filter tuning, setup, limitations |
| [`issues/issue-001-TEMPLATE.md`](issues/issue-001-TEMPLATE.md) | Issue #1 skeleton |
| [`n8n/palmetto-beat-newsroom.json`](n8n/palmetto-beat-newsroom.json) | The workflow, importable |
