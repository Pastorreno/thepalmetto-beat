# Architecture Decision: What to build vs. what to rent

**Date:** 2026-08-03
**Question:** Should The Palmetto Beat run on a self-hosted n8n system on the Mac mini instead of beehiiv?
**Decision:** Split the system. Build the newsroom, rent the post office.

---

## The instinct is right, but it's aimed at the wrong layer

"Build my own" is correct for the part that is **differentiated and reusable**, and wrong for the part that is **commodity and reputation-bound**.

A newsletter is two systems that look like one:

| Layer | What it does | Build or rent |
|---|---|---|
| **Newsroom** | Collect local sources, take reader tips, draft the issue with AI, assemble the calendar | **BUILD on n8n** — this is the leverage, and it's resellable |
| **Post office** | Deliver 5,000 emails to Gmail inboxes, manage the list, handle unsubscribes and bounces, comply with bulk-sender law, grow via network effects | **RENT from beehiiv** |

## Why the post office is not worth building

### 1. You cannot send bulk email from a Mac mini. At all.
Residential ISPs block outbound port 25. A residential IP has no sender reputation and gets rejected or spam-foldered by Gmail, Yahoo, and Outlook essentially on sight. To send real email you would relay through Resend, SES, or Postmark **anyway** — so the "self-hosted" version still rents delivery. You'd own the app and rent the hard part.

### 2. Bulk sender compliance is legally binding, not best-practice
Since Google and Yahoo's bulk sender requirements, you must have: an authenticated domain (SPF + DKIM + DMARC alignment), a one-click `List-Unsubscribe-Post` header, spam complaint rate held under 0.3%, and automatic bounce suppression. Getting these wrong doesn't produce an error message — it produces silence. Your domain gets quietly deprioritized and your newsletter stops reaching Gmail without telling you.

### 3. The growth network is literally unbuildable
beehiiv's recommendation network is *other newsletters' audiences* recommending you to their new subscribers. It is the single largest early growth lever for a new local publication, it's free on the Launch plan, and no amount of n8n replicates it — because the asset isn't software, it's other people's readers. **This is the strongest argument in the whole document.**

### 4. Uptime you don't control, on a deadline you promised
The mini sits at your house on Spectrum. A power blip, an ISP outage, or a macOS update rebooting at 5:52 AM means no issue goes out. You've told 2,000 neighbors it arrives at 6:00. beehiiv does not go down.

### 5. Opportunity cost — the real one
The Palmetto Beat succeeds or fails on **content quality and local relationships**, not infrastructure. Every hour spent on bounce-handling logic is an hour not spent walking into a business on 8th Ave to sell a $300 sponsorship. Rebuilding delivery costs roughly 40-60 hours to save $43/month.

## Why the newsroom IS worth building

This is where the actual value is, and n8n is genuinely the right tool:

- **Nobody else has it.** Any town can start a newsletter. An AI pipeline that watches every civic source in the city and assembles a draft is a real moat.
- **It's the resellable Melvin vertical.** A "local news engine" productized for other small towns is a legitimate GGI Hub product. That IP lives in the newsroom layer, not the sending layer.
- **It's the part that burns you out.** Sourcing content twice a week, forever, is what kills local newsletters. Automating sourcing is automating the failure mode.

## The seam (and its honest limitation)

⚠️ **beehiiv's Create Post API endpoint is Enterprise-only** — not available on Launch or Scale. n8n cannot push a finished draft into beehiiv programmatically at a realistic price point.

**What n8n CAN do via the beehiiv API on Launch:** manage subscriptions (create, update, tag, segment) and read analytics.

**So the seam is manual, and that's fine:** n8n assembles the complete issue and delivers it to you as formatted HTML via Telegram at 5:00 AM on publication days. You review it, paste it into the beehiiv editor, and schedule. **That's a five-minute step, twice a week** — and the human review is a feature, not a workaround. You do not want an unreviewed AI-drafted local news story going out under your name.

## Current state to be aware of

- n8n is running on **the MacBook**, not the mini — Docker container `newproject-n8n-1` on `localhost:5678`.
- The standing constraint is **no Docker on the Mac mini** (hardware too old). Moving n8n there requires a native install, which is more fragile to maintain.
- **Recommendation:** leave n8n where it is for now. Moving it is a separate project with no benefit to the newsletter launch. Revisit only if the MacBook's uptime becomes the bottleneck.

## Decision summary

| Phase | What | When |
|---|---|---|
| **1** | Launch on beehiiv. Source and write issues manually. Reach 500 subscribers. | Now — weeks 1-4 |
| **2** | Build the n8n newsroom: source scrapers, tip intake, AI drafting, Telegram delivery of the draft. | Weeks 2-6, in parallel |
| **3** | Revisit owning delivery **only** if you productize this for other towns — and even then, relay through Resend, never the mini. | Later, if ever |

**Bottom line:** rent the commodity, build the moat. Spending your first month on send infrastructure would be building the one part of this that is already solved, while the part that actually wins — being the only person in Palmetto who shows up to the commission meeting — goes undone.
