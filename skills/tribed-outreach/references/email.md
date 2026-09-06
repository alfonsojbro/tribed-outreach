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
