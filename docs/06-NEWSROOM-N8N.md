# The Newsroom — n8n Workflow

**File:** [`n8n/palmetto-beat-newsroom.json`](../n8n/palmetto-beat-newsroom.json)
**Built and tested against live feeds:** 2026-08-03
**Target:** n8n 2.20.6 (container `newproject-n8n-1`, `localhost:5678`, TZ `America/New_York`)

## What it does

Every morning at 5:00 AM it pulls 8 RSS feeds, throws away the noise, drops anything it has already shown you, has Claude draft the issue in house style, and Telegrams you an HTML file to review.

```
5:00 AM  →  Feed List  →  Fetch Feed  →  Parse & Normalise  →  Filter & Score
              (8 feeds)    (HTTP, XML)     (RSS + Atom)         (noise + relevance)
                                                                      ↓
Telegram  ←  To File  ←  Wrap HTML  ←  Draft the Issue  ←  Anything New?  ←  Collect  ←  Drop Already Seen
 (.html)                                (Claude)         (else: "nothing new" ping)      (cross-run memory)
```

You review the draft, verify every `[VERIFY]` tag, and paste it into beehiiv. **That manual step is deliberate** — beehiiv's Create Post API is Enterprise-only, and you would want a human read on local news under your name regardless.

## The feeds

All 8 were fetched and confirmed returning items before being included.

| Source | Tier | Official | Note |
|---|---|---|---|
| City of Palmetto — Calendar | 1 | ✅ | `ModID=58` — commission & CRA meetings |
| City of Palmetto — Agendas | 1 | ✅ | `ModID=65` — empty when nothing is posted; that's normal |
| Google News — Palmetto government | 1 | | `"City of Palmetto" OR "Palmetto commission"`, `when:14d` |
| Google News — Palmetto FL | 2 | | `when:7d` |
| The Suncoast Post | 2 | | WordPress feed, covers local events |
| Google News — Manatee County | 3 | | `when:7d` |
| ABC7 WWSB / MySuncoast | 3 | | Regional TV |
| The Bradenton Journal | 3 | | Substack, good county government coverage |

**`official: true`** means an authoritative city feed — everything on it is Palmetto by definition, so it skips the local-place-name test. Google News tier-1 queries are *searches*, not authoritative, so they still have to name a local place.

### Feeds tested and rejected

| Source | Why |
|---|---|
| `palmettofl.org ModID=76` | **Looks like News Flash but isn't.** Channel title is "Palmetto - Pages" — a site directory returning "Agendas", "Minutes", "Human Resources", "Financial Reports". It was flooding tier 1 with junk. Removed. No News Flash RSS module exists on this site — probed 18 module IDs, only 76 returned anything. **Check `palmettofl.org/m/NewsFlash` by hand.** |
| The Bradenton Times | 404 on `/rss/`, `/feed`, 403 on `/rss.xml`. Covered indirectly via Google News. |
| Patch — Palmetto | Feed returns 0 items |
| Manatee County (`mymanatee.org`) | No RSS found |
| Manatee Chamber | 404 |
| Manatee County Schools | No RSS |
| Bay News 9 | No usable feed |

### Google News `when:` operator

Appending `when:7d` to the query filters at the source. This matters — **without it the feeds return items back to 2015**, and the recency filter was throwing away 304 of 336 items. With it, 100 items becomes 34 current ones.

## The filter (tuned against real output)

Live results: **137 items in → 13 kept.**

```
dropped: title 8 · wrongPalmetto 7 · stale 7 · notLocal 74 · lowScore 28
```

Four rejection stages, cheapest first:

1. **Blocked domains** — obituary sites (tributearchive, legacy, echovita), real-estate listings (realtor, zillow, redfin), maxpreps, youtube, iqair
2. **Blocked titles** — obituaries, "for sale", "open house", MLS, bare street-address listings
3. **The wrong Palmetto** — there are several, and they generate real false positives:
   - **Palmetto Bay, FL** (Miami-Dade)
   - **Palmetto Expressway** (SR 826, Miami) and **Palmetto Senior High / Panthers** (Miami)
   - **Palmetto, Georgia** — a Fulton County property-tax story got through before this was added
   - Palmetto Bluff SC, the Palmetto State, palmetto bugs
4. **Local relevance** — non-official sources must name a local place (Palmetto, Ellenton, Manatee County, Terra Ceia, Rubonia, Old Memphis, Green Bridge, Snead Island, Palmetto High…)

Then everything is **scored** — official source +60, "Palmetto" in the headline +30, civic keywords (commission, CRA, rezoning, budget, permit) +25, public-safety/infrastructure +15, business openings +12, events +10, regional −10 — and sorted highest first.

### Tuning

Top of the **Filter & Score** node:

```js
const MAX_AGE_DAYS   = 7;   // raise if a draft comes back thin
const TIER1_AGE_DAYS = 30;  // civic notices stay relevant longer
const MIN_SCORE      = 0;   // raise if a draft comes back noisy
const MAX_ITEMS      = 60;
```

The node logs `Filter: N in -> M kept` with a per-rule breakdown on every run. Read that first when something looks wrong.

## Cross-run memory

**Drop Already Seen** uses n8n's *Remove Items Seen in Previous Executions* with a 3,000-key history, so you never get the same story twice. No database needed.

⚠️ **The first run will be large** — everything is new. Expect a long draft. It settles from run two onward.

To reset (after changing feeds, say), clear the node's stored keys from its panel.

## Setup

### 1. Import
n8n → **Workflows** → **Import from File** → `n8n/palmetto-beat-newsroom.json`

### 2. Anthropic credential
Open the **Claude** node → create an Anthropic credential with your API key.
If the model doesn't populate, just re-pick it from the dropdown — the workflow pins `claude-opus-5`.

### 3. Telegram
Open either Telegram node → select your existing bot credential.

Both nodes read the chat ID from an environment variable, so it isn't committed:

```bash
# add to the n8n container environment, then restart
PALMETTO_TELEGRAM_CHAT_ID=<your chat id>
```

If you'd rather not use an env var, replace `={{ $env.PALMETTO_TELEGRAM_CHAT_ID }}` with the chat ID directly in both nodes.

### 4. Test before activating
Click **Execute Workflow** manually. Check:
- **Fetch Feed** — 8 items out, all with content
- **Filter & Score** — read the console log; if `kept` is 0 or 60, tune before going live
- **Draft the Issue** — sensible HTML, `[VERIFY]` tags present
- Telegram file arrives

Only then toggle **Active**.

### 5. Cadence
Ships as **5:00 AM daily**. For a Tue/Thu newsletter, either leave it daily (collect every day, draft twice a week) or change the Schedule Trigger to Monday and Wednesday so drafts land the day before each issue.

## What the prompt enforces

The drafting prompt has hard rules worth knowing, because they're your legal and reputational protection:

- **Use only facts present in the source items. Invent nothing.**
- Every claim carries its source link.
- Anything needing confirmation gets `[VERIFY: what to check, and with whom]`.
- Sensitive stories (death, crime, hate crime, litigation) get written plainly and factually, attributed to the reporting outlet, **no speculation about motive or blame**, and flagged `[SENSITIVE — editor review required]`.
- Empty sections say "(nothing this issue)" rather than getting padded.

It also returns **editor notes**: three subject lines under 45 characters, a gathered `[VERIFY]` checklist, and any story it judged too thin or unverified to run.

## Known limitations

1. **Order-alignment dependency.** `Parse & Normalise` pairs feeds to responses by index. `Fetch Feed` is set to `alwaysOutputData` + `continueRegularOutput` so a dead feed still emits a slot and alignment holds. **If you add or remove feeds, keep the Feed List order stable** and re-test.
2. **Regex XML parsing.** Deliberate — it handles both RSS and Atom and never throws on malformed feeds. It is not a spec-compliant parser.
3. **No tip intake yet.** Reader tips and the submission form are the obvious next build (see below).
4. **The city's own News Flash isn't covered.** No RSS exists. Manual check until someone scrapes `/m/NewsFlash`.
5. **Never publish unreviewed.** The model drafts from headlines and summaries, not full articles. It can misread a headline. You are the editor.

## Next builds

| Build | Why |
|---|---|
| **Tip intake** | Telegram bot + web form → same pipeline. Reader tips are your best source. |
| **`/m/NewsFlash` scraper** | HTTP + HTML parse to recover the city announcements feed that doesn't exist as RSS |
| **Calendar assembly** | Separate workflow building the 7-10 day events block |
| **Chamber ribbon cuttings** | Scrape the Manatee Chamber events page — new businesses are both content and sponsor leads |
| **beehiiv subscriber sync** | The Subscriptions API *is* available on Launch — pull counts into a dashboard |
