# Mode 6 — Cold email + email follow-ups

Same gift-first move as the DMs, but this is a real inbox, not a chat. It earns deletion just as fast as a DM earns a scroll-past, so the craft is tighter. Adapted from Corey Haines' cold-email method, kept in Tribed's voice; where the two conflict, the Tribed core (gift-first, no banned phrases, no call ask pre-demo) wins.

## The rail: Alfonso's own Gmail, direct-send, no drafts (corrected 2026-09-06)

**Instantly is retired and `kind: "email"` drafts are LEGACY.** Do not queue one and do not ship one; the daily run rejects any pending email draft with "superseded by Gmail podcast leg". The live leg is phase 1e of the daily run: it reads enriched leads (`data.email`, stamped by `enrich_outreach_lead_emails` off the coach's own site) and sends **directly from Alfonso's personal Gmail** via the Gmail MCP, one message at a time.

What that rail changes about the craft:

- **The cap is 5 a day, seven days a week**, hard ceiling 15. This is a personal mailbox, not a cold-email tool, so the low volume and the one-to-one quality of each message ARE the deliverability strategy. There is no warm-up pool and no sending domain to burn but his own.
- **A bounce is expensive.** Two bounces in one day halt the leg (the `email_day_ledger` circuit breaker). A bounce means the extractor stamped a stale address, so never guess or pattern-build an address.
- **Follow-ups are same-thread replies**, authored fresh, not sequence steps someone else's tool fires.
- The current opener is the **podcast invite**, not the app reveal. When writing an app-reveal cold email instead, the shapes below still apply.

Older text in this file and in pipeline.md described an Instantly campaign with a `shipTarget` campaign id. That rail is gone; if you find a paragraph that still assumes it, it is stale.

## The address has to be theirs, and it has to exist (2026-09-18)

Three bad addresses reached the pool in one week, and one of them hard-bounced. `enrich_outreach_lead_emails` now carries two guards, and one sticky flag that no run may clear.

**1. Ownership — is this site even theirs?** Before the crawler fetches a single page it judges the bio link. A url tagged as an affiliate or referral campaign is refused outright, reason `bio link is an affiliate or referral url, not their own site`:

- a `utm_source` / `utm_medium` / `utm_campaign` / `utm_id` / `utm_content` whose value reads *influencer, affiliate, partner, ambassador, creator, referral, promo*;
- a `?ref=` / `?aff=` / `?affiliate=` / `?via=` / `?partner=` parameter;
- a `/referral-portal`, `/affiliate`, `/partners` path segment;
- a utm campaign named after the lead — a site does not tag its own owner as a campaign.

That is Danai Maraire's case exactly: her bio link was `tslhg.com/referral-portal/?utm_source=influencer&utm_campaign=Danai-May`, an affiliate portal for The Student Loan Help Group, and the crawl stamped `info@tslhg.com` as her own role address at confidence `high`. Mailing it mails a student-loan company. The refusal now happens **before the fetch**, so no `emailSiteText` describing a stranger's business is ever captured for a message to anchor on.

A site we simply cannot tie to the lead — no name, handle or notes match — is **not** refused. It is capped: an address found there can never be graded `high`, so it can never outrank a corroborated one in the `high → medium → low` selection order.

Ownership is judged **twice**: once on the bio link, and again on whatever url actually answers, because redirects are followed. A link on the lead's own domain that 302s into an affiliate portal — or a link shortener hiding one, and no shortener is in the aggregator list — is caught by the second verdict, before the landing page's words are captured.

The second judgement is deliberately **weaker** than the first. A refusal there also needs the landing HOST to be somebody else's: a redirect back onto the same host, or onto a host we can corroborate as the lead's, is never an affiliate refusal however the query string reads. Sites append their own tracking (`tinawellsfit.com/` → `tinawellsfit.com/?ref=bio`), and since an affiliate refusal is sticky, the strict rule applied to a landing url would park good leads behind a human over a cosmetic parameter. The pre-fetch judgement keeps the strict rule unchanged.

**2. Resolution — does the domain exist?** The verdict on an address's own domain:

| DNS | Verdict |
|---|---|
| MX records present | keep, unchanged |
| A record but no MX | keep, but **forced to confidence `low`** |
| nothing resolves | **refused** / flagged unusable |
| malformed domain (`coach example.com`, `x.com>`, empty) | **refused** — this is bad input, not a resolver failure |
| resolver error / timeout | keep — **fails open** |

Unusual is not malformed, and the shape check is careful about the difference, because a refusal is sticky and parking a lead over a legal domain costs a human to undo. A trailing dot (`gmail.com.`) is the root-anchored form of a good name; an internationalised domain (`münchen.de`) is normalised to the punycode DNS is actually asked for; an IDN TLD (`example.xn--p1ai`) matches no plain letters-only tail and must not be refused for it. All three resolve normally.

Failing open on a resolver error is deliberate. A SERVFAIL is our problem, not evidence against the lead, and one flaky afternoon must not quietly empty the email pool. It is *not* good enough to un-park a lead, though — see the flag below.

**The check that matters is the one on STORED addresses, not the one inside the crawl.** Be clear about this, because the crawl-side check reads more important than it is: the crawler only ever promotes an address on the host that just served it HTTP, or a freemail address, and both of those resolve by definition. It is a backstop, near-dead by construction.

The guard that does the work runs on `data.email` — addresses that came from an Instagram bio, or were typed into the data bag by hand, and were never crawled at all. **That is where both bounces came from.** Coach Tho's `thomeisha@infinitecurvesrva.com` and Megan Long's `megan@ateamathletes.com` were both hand-written, both on NXDOMAIN domains, and Tho's bounced on 2026-09-16.

So: a lead that already has an address is verified rather than skipped — one DNS query, no crawl. That happens without `force`, and **also under `force`** whenever the re-resolve misses and the stored address survives, which is precisely the case `force` is advertised for. `verifyOnly: true` runs that pass and nothing else; that is the pre-send check the daily run does over the day's candidates before writing any copy.

**3. `data.email_unusable` is sticky, and `force` does not override it.** A flagged lead is skipped entirely, reported with status `skipped-unusable` and the stored reason. Only `overrideUnusable: true` re-checks one, and the flag clears **only** when the re-check comes back clean — an address found, ownership not affiliate, domain resolving — stamping `data.email_unusable_cleared_at`. Anything less leaves the lead parked.

"Clean" is strict. The address's domain must positively resolve: a `no-mx` domain is good enough to stamp an address at `low` and nowhere near good enough to un-park a lead, and a resolver that could not answer is not an answer.

Nothing automatic ever lowers the flag or rewrites a reason already stored. Danai's flag carries a human's paragraph of reasoning; re-deriving it would be strictly worse than keeping it. On a clear the reason is **moved** to `data.email_unusable_prior_reason`, never deleted, so a lead that has to be re-parked has not lost its history. When the extractor raises the flag itself it also writes `data.email_unusable_source: "extractor"`.

The response counts these apart from misses, under `unusable`, and they are **excluded** from `missed` so the digest's miss rate stays comparable to the ~35% baseline. A miss is a lead we could not find an address for. An `unusable` is a lead whose address we have and must actively refuse to use. Per lead, the marker is `results[].status`: `skipped-unusable` for the sticky flag, `unusable-domain` for a dead domain.

What to gather: name, niche, the anchor detail (rule zero — a post, a program name, a book, a launch), follower/creator context for the price tier if it ever comes up, language. If no anchor is visible, ask; don't pad.

## What makes email different from a DM

- It has a **subject line** (a DM doesn't). See below.
- Slightly more context is tolerated than a DM, but not much. **40–75 words** is the target for the opener; under 75 gets ~83% more replies.
- "You/your" dominates over "I/we." Lead with their world.
- One low-friction, interest-based ask. Never a call ask before they've seen the demo.
- Reading level 3rd–5th grade. Short words, short sentences.
- No HTML, no images, one link max (the demo link the user drops in), never a fake "Re:" or "Fwd:".

## Subject lines

Short, boring, internal-looking, the subject's only job is the open, not the sell.

- **2–4 words, lowercase, no punctuation tricks, no emoji, no first name.** (First name in the subject signals automation and costs replies.)
- Should read like a note from a peer, not a pitch. Anchor it on their world when you can: `your 30-day reset`, `deload post`, `your community`, `built you something`, `an app for [program]`.
- Avoid salesy words ("boost", "increase", "ROI"), urgency ("ASAP"), and excessive punctuation.

## Openers — pick a shape, keep the reveal

The gift-first reveal (the demo already exists, built around their work, free to look at) is the spine of every email. Choose the shape that fits the anchor:

**Detail → reveal → ask (default).** Name the specific thing, reveal you built the demo around it, one soft ask.
> subject: your deload post
>
> Dave, your post on deloading without losing your mind is the clearest take on it I've read. I build branded apps for coaches, and I ended up making a demo of one around your method, your name on it, AI trained on your content, free to look at. Want me to send it over?

**Reveal → why them → ask.** Lead with the odd fact, then the anchor.
> subject: built you an app
>
> Maya, slightly strange intro: I already built you a demo app. Your 5-Minute Reset series gave me the idea, so it's your method inside it, not a template. It's done and it's free to look at. Want to see it?

**Their world → bridge → ask** (for a named program/challenge/book).
> subject: your reto 21 días
>
> Sofía, tu Reto 21 Días ya funciona como un programa completo, se nota el método detrás. Se me ocurrió que eso es prácticamente una app, así que armé una demo con tu contenido adentro, coaching con IA y seguimiento para tu gente. Ya existe y verla no cuesta nada. ¿Te la mando?

Rotate shapes across prospects the same way you rotate DM skeletons. Three emails that all read "your X is great, I built an app around your method" are one template, not three.

## Email follow-up sequence (same-thread bumps, authored fresh)

Follow-ups carry a large share of replies, and each one is a value ladder, never "just checking in." Open with their first name. Cap the sequence; stop at any reply, positive signal, or no.

**The ladder is the one in references/followups.md.** That file is the single source for what each touch says and when it fires; this table is the email-channel rendering of it, not a second answer. If the two ever disagree, followups.md wins and this table is the bug.

| Touch | Timing | Angle |
|---|---|---|
| Email 1 (opener) | day 0 | Anchor + gift-first reveal + soft ask |
| Email 2 (follow-up 1, case study) | 3-4 days later | How a similar coach in their niche got a branded app: what it included (their content as guided programs, AI trained on their material, a community space, tracking) and how it works. Honest, no invented names or metrics; a [case study link] placeholder if useful. One soft ask. |
| Email 3 (follow-up 2, proposal offer) | 4 days after email 2 | Offer to put together a customized proposal of everything that would go inside their app, named around one thing of theirs. No price, no link, one soft ask. This is the proven closer; the 60-second video is a post-demo asset and belongs to Mode 5. |

Each bump is a **reply on the same thread**, sent by hand from Alfonso's Gmail, and written fresh for that person. There is no sequencing tool firing these, so a bump that never gets authored simply never goes out. Stamp `data.email_bumped_at` after each one.

Best days Tuesday to Thursday, 9-11am or 1-3pm their local time. Each email must stand alone, assume they never read the previous one.

**Follow-up 1 example (case study, English):**
> subject: your reset app
>
> Dave, quick one, I built a similar habit coach her own app last month: her programs became guided tracks, the AI answered client questions in her voice, and her community lived in one place instead of scattered DMs. yours would work the same way with your Reset inside. want a look?

**Follow-up 2 example (proposal offer, English):**
> subject: your reset app
>
> Dave, last one from me. Want me to put together a customized proposal of everything that would go inside the Reset app? Costs you nothing to read it.

## Frameworks to borrow (for the body, not for pasting)

When an anchor calls for more structure, these B2B shapes can inform the body — but keep Tribed's gift-first reveal, not a problem-agitate pitch:

- **PPP (Praise, Picture, Push)** fits best: genuine specific praise → a light picture of their work living as an app → soft push to look. Requires a real trigger, which rule zero already demands.
- **BAB (Before, After, Bridge)** works when there's a clear before/after ("scattered across posts" → "one place your people open daily"), with the demo as the bridge.
- Avoid PAS/AIDA-style agitation **in email specifically**: manufacturing a pain point in a cold inbox clashes with a gift, and it reads as a sales sequence, which is the one thing a personal Gmail must never look like.

**This is a channel scope rule, not a contradiction of instagram.md.** The Instagram format library (references/instagram.md) does include `pas` and `aida`, and they work there: a DM is a chat, the register is casual, and the problem named is usually the coach's own workflow rather than an invented pain. Email is a colder surface with a spam filter and a reply-all button, so it keeps the gentler shapes. Use the library on IG and X, use PPP or BAB here.

## Banned in email (on top of the global list)

"I hope this email finds you well", "My name is X and I work at Y", "leverage", "synergy", "circle back", "best-in-class", "leading provider", "just checking in", fake "Re:"/"Fwd:" subjects, {{FirstName}} in the subject line, any 30-minute-call ask in a first touch.

## Self-check

Read it aloud — would a busy coach reply? Delete the personalized opener: does the email collapse? (Good — that means the anchor is load-bearing.) Under 75 words? One link, one ask? Subject 2–4 lowercase words with no name? Different shape from the last email this session?
