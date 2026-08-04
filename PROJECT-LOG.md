# Project Log — The Palmetto Beat

Append-only running record of what happened, what was decided, and what was learned.

**Read this before starting work.** It exists so you don't re-make a settled decision or re-derive a fact that already cost someone an hour.

---

## How to use this log

**Append an entry every time you do meaningful work.** Newest at the bottom.

**Never edit or delete a past entry.** If something recorded here turns out to be wrong, outdated, or superseded, append an **AMENDMENT** referencing the original date and heading. What we believed and when is part of the record — it's how the next person understands why a decision looked right at the time.

### Entry format

```markdown
## YYYY-MM-DD — Short title (who did it)

**What happened:**
**Decisions made:**
**Learned / verified:**
**Left undone, and why:**
**How to verify:**
```

### Amendment format

```markdown
## YYYY-MM-DD — AMENDMENT to "YYYY-MM-DD — Original title"

**What changed:** what we now know
**Why the original was wrong (or is now stale):**
**What to do differently:**
```

### What belongs here vs. elsewhere

| Goes here | Goes elsewhere |
|---|---|
| Decisions and why | Durable how-to → `docs/` |
| Facts verified the hard way | Current project state → `AGENTS.md` |
| Things tried that failed | Machine/agent-context changes → `~/AGENTS_CHANGELOG.md` |
| Open questions | Secrets → **nowhere in this repo** |

---

## 2026-08-03 — Unparked the project; built launch kit + n8n newsroom (Claude Opus 5, with Reno)

**What happened:**

Repo was a README-only parked concept ("preserved as a future idea, not an active build"). User said they were ready to launch. Converted it into an active build: 7 docs, an importable n8n workflow, and an issue template. Opened PR #1 on branch `launch-newsroom`.

**Decisions made:**

- **Platform: beehiiv**, chosen by the user from four options. Free Launch tier to start, upgrade to Scale ($43/mo) at ~500 subscribers or first sponsor.
- **Launching on the beehiiv subdomain**, custom domain later (user's call).
- **Architecture: rent delivery, build the newsroom.** User asked mid-build about self-hosting the whole thing on the Mac mini with n8n. Considered seriously, rejected for delivery, accepted for sourcing. Full reasoning in `docs/00-ARCHITECTURE-DECISION.md`.
- **Cadence: Tuesday/Thursday 6:00 AM.** Not daily — burnout kills local newsletters more often than low subscriber counts.
- **Positioning: community publication, NOT a ministry publication.** Deliberate call given the user runs JGM/JGC. Church content sits alongside everyone else's in Calendar and Around Town. This neutrality is what lets it sell ads to businesses that would never sponsor a ministry newsletter. **Flagged to the user for override; not overridden.**

**Learned / verified:**

*Platform:*
- beehiiv **Create Post API endpoint is Enterprise-only** — not Launch, not Scale. This is why the n8n→beehiiv seam is a manual paste rather than an API call. Subscriptions API *is* on Launch.
- beehiiv **Launch (free) excludes referral program, polls, and automations** → **no automated welcome email**. Corrected an earlier claim made to the user before checking. Recommendation network, custom domains, segmentation, non-Send API *are* included.

*RSS — all found by running the pipeline, not by reading docs:*
- **`palmettofl.org` has no News Flash RSS feed.** `ModID=76` looks like one and returns 50 items, but its channel title is "Palmetto - Pages" — a site directory serving "Agendas", "Minutes", "Human Resources". Probed 18 module IDs; only Calendar (`58`) and Agendas (`65`) return real content. News Flash must be checked manually.
- **Google News RSS returns items back to 2015** without a `when:` operator. First filter build kept only 8 of 336 items because the recency rule was doing all the work. Adding `when:7d` cut a feed from 100 items to 34, all current.
- **"Palmetto" false positives are severe.** Caught in live output: Palmetto **GA** (a Fulton County property-tax story), Palmetto **Expressway** and Palmetto **Panthers/Senior High** (all Miami-Dade), plus Palmetto Bay FL, Palmetto Bluff SC, the Palmetto State, palmetto bugs.
- **Dead ends:** Bradenton Times (404 on `/rss/` and `/feed`, 403 on `/rss.xml`), Patch Palmetto (feed returns 0 items), Manatee County, Manatee Chamber, Manatee County Schools, Bay News 9.

*Market:*
- Confirmed the wedge — every outlet covers Manatee County from Bradenton's side. No Palmetto-first publication exists.
- **Warm sponsor leads, publicly documented:** Stewart Title, Revolution Mortgage, and Blue Collar Roofing already sponsor Palmetto Downtown Main Street's Farmer's Market & Movie Night. They've proven they pay for local visibility.
- **Live storylines at time of writing** (all unverified, from public reporting): Old Memphis Cemetery vandalism (17 graves, national coverage, community cleanup planned); CRA sale of four downtown parcels for a six-story 152-unit development; SWFWMD Phase II water restrictions; city branding campaign; CodeRED breach fallout; new Veterans Elementary opening; Palmetto pocket park county support.

*Workflow test results:*
- Final live run: **137 items in → 13 relevant local stories out.** Drop breakdown: title 8, wrongPalmetto 7, stale 7, notLocal 74, lowScore 28.

**Left undone, and why:**

- **beehiiv account, domain, handles, PO Box** — all require the user; an agent can't sign up or pay.
- **Workflow not imported or activated** — needs an Anthropic credential, a Telegram credential, and `PALMETTO_TELEGRAM_CHAT_ID`. User should execute manually once and read the filter log before activating.
- **Tip intake not built** — submission form + Telegram bot. Next obvious build; reader tips are the best source.
- **`/m/NewsFlash` scraper not built** — would recover the city announcements feed that has no RSS.
- **Issue #1 not written** — template only, and every storyline in it is marked unverified.
- **PR #1 not merged.**

**How to verify:**

```bash
gh pr view 1 --repo Pastorreno/thepalmetto-beat
# re-run the pipeline logic against live feeds:
#   see docs/06-NEWSROOM-N8N.md; the Filter & Score node logs
#   "Filter: N in -> M kept" with a per-rule drop breakdown
```

**Open questions for the user:**

1. Upgrade to beehiiv Scale at launch rather than at 500 subs, to get a welcome email? (Highest-open email you'll ever send; losing it is a real cost.)
2. Merge PR #1 as-is, or push directly to `main` on this repo going forward?

---

## 2026-08-03 — Established notation convention (Claude Opus 5, at user request)

**What happened:**

User asked that all of this be stored durably so they or another agent can pick it up later, and that there be a standing instruction to always notate and leave amending notes.

**Decisions made:**

- This log is **append-only**. Past entries are never edited or deleted; corrections are appended as **AMENDMENT** entries referencing the original.
- `AGENTS.md` in this repo holds **current state**; `PROJECT-LOG.md` holds **history and reasoning**. Keep them separate — state gets stale and should be updated in place, history should not.
- The convention was also added to `~/AGENTS.md`, `~/CLAUDE.md`, and `~/GEMINI.md` so it applies to every agent on this machine, not just this project. Recorded in `~/AGENTS_CHANGELOG.md` per the existing rule.

**Left undone, and why:**

- Not propagated to the Mac mini. This machine's agent-context files are machine-specific by existing convention, and the mini has its own.

**How to verify:**

```bash
grep -n "amending" ~/AGENTS.md ~/CLAUDE.md ~/GEMINI.md
tail -40 ~/AGENTS_CHANGELOG.md
```
