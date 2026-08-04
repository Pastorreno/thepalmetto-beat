# The Palmetto Beat — Content Sourcing Playbook

The operational core. This is the list n8n will eventually automate, so build the habit manually first — you'll learn which sources actually produce stories.

## Tier 1 — Primary civic sources (check every issue)

| Source | URL | What you get | Cadence |
|---|---|---|---|
| **City of Palmetto — News Flash** | `palmettofl.org/m/NewsFlash` | Official announcements, closures, notices | Check every issue |
| **City of Palmetto — Commission Agendas** | `palmettofl.org/158/Commission-Agendas` | Agendas + packets **before** meetings | Weekly |
| **City of Palmetto — main site** | `palmettofl.org` | Departments, projects, budget | Weekly |
| **Palmetto Police Department** | `palmettofl.org/117/Police-Department` | Blotter, alerts, community programs | Every issue |
| **Manatee County Government** | `mymanatee.org` | County actions affecting Palmetto | Weekly |
| **Manatee County Schools** | `manateeschools.net` | Palmetto High, Lincoln Middle, calendars | Weekly |

> ⚠️ **RSS status (tested 2026-08-03):** the City has **no News Flash RSS feed**. `ModID=76` looks like one but its channel is "Palmetto - Pages" — a site directory. 18 module IDs were probed; only the **Calendar** (`ModID=58`) and **Agendas** (`ModID=65`) feeds work and both are wired into the n8n newsroom. **News Flash must be checked by hand** at `palmettofl.org/m/NewsFlash`.

**The commission agenda is your single most valuable source.** The packet is posted days before the meeting and nobody reads it. Reading it lets you publish "here's what the commission is voting on Monday" before it happens — which is the difference between a newsletter and a news outlet.

## Tier 2 — Events and business

| Source | URL | What you get |
|---|---|---|
| **Palmetto Downtown Main Street** | search Facebook + their site | Farmer's Market & Movie Night, downtown events, small business news |
| **Manatee Chamber of Commerce** | `manateechamber.chambermaster.com/events` | Ribbon cuttings, business events, new members |
| **Manatee County Fairgrounds** | `manateecountyfair.com` | Major events — this venue is *in Palmetto*, big draw |
| **The Suncoast Post** | `suncoastpost.com` | Event coverage, already covers Palmetto events |
| **Wagner Realty community calendar** | `wagnerrealty.com/PalmettoCommunityEventCalendar` | Pre-aggregated local calendar |

**Chamber ribbon cuttings are gold.** A new business opening is a free "Around Town" item, and the owner is a warm sponsorship lead who just watched you write about them.

## Tier 3 — Regional media (for stories that touch Palmetto)

| Outlet | Note |
|---|---|
| **Bradenton Herald** | McClatchy, paywalled. Covers Manatee broadly, Palmetto thinly. |
| **The Bradenton Times** | Free, online. Good local government coverage. |
| **Bay News 9 (Spectrum)** | `baynews9.com/fl/tampa/news/manatee` — TV, breaking news |
| **ABC7 WWSB / MySuncoast** | `mysuncoast.com` — Sarasota-Manatee TV |
| **Suncoast Searchlight** | Nonprofit investigative — occasional deep Manatee pieces |
| **Patch — Palmetto** | `patch.com/florida/palmetto-fl` — automated, low quality, but useful as a scan |

**Never copy.** Read regional coverage, then report the Palmetto angle yourself with your own reporting and a credit link. Aggregating someone else's paragraphs is both a legal problem and a credibility problem.

## Tier 4 — Human sources (build these deliberately)

The ones that make you irreplaceable. Get a direct contact at each:

- City Clerk's office — agendas, minutes, public records
- Palmetto PD public information officer
- Palmetto Downtown Main Street director
- Manatee Chamber membership director
- Palmetto High athletic director — local sports scores are enormously read
- Two or three pastors, two or three nonprofit directors
- A few longtime business owners on 8th Ave / Riverside Dr

Introduce yourself in person as the publisher of The Palmetto Beat **before** you need anything from them. Bring a printed copy of an issue.

## Tier 5 — Community listening

- Local Facebook groups (Palmetto / Manatee County community groups) — **tips only, never quotes.** Verify every claim before publishing.
- Nextdoor Palmetto
- Reader replies — your best source. Ask for tips in every single issue.

## Storylines live right now (August 2026)

Real, current threads to launch into:

1. **Downtown redevelopment** — the CRA approved the sale of four parcels for a six-story mixed-use development with 152 apartments. This is the biggest ongoing Palmetto story. Own it: track the timeline, interview affected businesses, follow the permits.
2. **City branding campaign** — the city has been discussing a branding initiative. Perfect reader-poll material.
3. **Water restrictions** — SWFWMD Modified Phase II severe water shortage, one-day-per-week watering in Manatee County. High-utility service journalism.
4. **CodeRED breach fallout** — the emergency notification system was hit by a cyberattack in late 2025 exposing user data. Worth a "what you should do" explainer.

## Weekly sourcing routine (about 4 hours total)

| Day | Task | Time |
|---|---|---|
| **Sunday** | Read the commission agenda packet. Scan county + schools. Draft the week's lead. | 60 min |
| **Monday** | Assemble calendar for the next 10 days. Check chamber + Main Street. | 45 min |
| **Monday PM** | Write and schedule Tuesday's issue. | 60 min |
| **Wednesday** | Scan regional media + PD. Check reader replies for tips. | 30 min |
| **Wednesday PM** | Write and schedule Thursday's issue. | 60 min |
| **Ongoing** | One in-person relationship touch per week. | 30 min |

## What n8n automates — BUILT

The newsroom workflow is built and tested: [`n8n/palmetto-beat-newsroom.json`](../n8n/palmetto-beat-newsroom.json), documented in [`06-NEWSROOM-N8N.md`](06-NEWSROOM-N8N.md). It covers 8 verified feeds.

**Not covered by RSS — still manual:** City News Flash, Manatee County, Manatee Chamber, Manatee County Schools, The Bradenton Times (403s; reachable indirectly via Google News).

The workflow:

1. ✅ **5:00 AM daily** — pulls 8 verified RSS feeds
2. ✅ **Filter & score** — strips obituaries, real-estate listings, and the other US towns called Palmetto (137 items → 13 on a live test)
3. ✅ **Dedupe across runs** — never shows you the same story twice
4. ✅ **AI drafting** — assembles the six-section issue with `[VERIFY]` tags on anything unconfirmed
5. ✅ **Telegram delivery** — HTML draft file lands on your phone
6. ⬜ **Tip intake** — submission form + Telegram bot (next build)

You review, paste into beehiiv, schedule. The human review step stays permanently.
