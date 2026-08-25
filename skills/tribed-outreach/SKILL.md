---
name: tribed-outreach
description: Tribed's full Instagram, LinkedIn, and cold-email outreach for wellness, fitness, coaching, and creator prospects. Covers IG cold DMs, reply and objection handling, follow-up sequences, LinkedIn connection notes and InMail, post-demo booking, cold email and email sequences, and the daily cross-channel pipeline (LinkedIn via our own LinkedIn MCP, ICP filtering, review-queue drafting, tracker-driven follow-ups). Use for any Tribed outreach: a cold DM, connection note, InMail, cold email, bump, follow-up, objection handling, booking the walkthrough call, or running/queueing the daily pipeline. Trigger even when the user just pastes a profile screenshot, since the default intent is a first-touch message. Handles English, River Plate Spanish, and neutral Latin American Spanish, picked from the profile or the reply.
metadata:
  version: 1.9.0
---

# Tribed Outreach — Full Workflow

This file holds the shared core. The mode-specific playbooks and examples live in references/. Read ONLY the one file for the mode at hand, plus references/voice.md whenever the task involves writing or editing message copy; the core below applies to all of them.

## Pick the mode, then read its reference file

| Input | Mode | Read |
|---|---|---|
| Instagram profile screenshot, no prior message | 1. IG cold DM | references/instagram.md |
| Prospect replied, hasn't seen the demo yet | 2. Reply handling | references/replies.md |
| No reply to the opener, user wants a bump/nudge/sequence | 3. Follow-ups | references/followups.md |
| LinkedIn profile, connection note or InMail | 4. LinkedIn | references/linkedin.md |
| Prospect opened the demo (replied or went quiet after) | 5. Objections → book the walkthrough call | references/postdemo.md |
| Cold email or email follow-up sequence (sent via Instantly) | 6. Cold email | references/email.md |
| Run the daily lead pipeline, personalize a batch, ship/queue review drafts, or build a demo from a conversation | 7. Pipeline ops | references/pipeline.md |
| X (Twitter) lead, cold DM, comment copy, or the X discovery batch (drip-owned outbound; see Fill 4 in pipeline.md) | 8. X | references/x.md |

Ambiguous input: ask one quick question.

The gift-first move underlies everything: a personalized demo of their own branded app already exists, the message tells them and offers to send it. The funnel has two halves with different goals. Before they've seen the demo, the only goal is getting them to look, and a call ask is forbidden. After they've opened the demo, the goal becomes a 15-minute walkthrough call, and the demo they just saw is what earns the ask.

## Core principles (every mode, every channel)

The single biggest failure mode of this outreach is sounding like a template with the niche swapped in. Coaches get dozens of those a week. Everything here exists to make each message read like one specific person noticed one specific thing about one specific coach and acted on it.

### Rule zero: anchor on one real detail

Every opening message is built around ONE concrete, verifiable detail from the profile. Not the niche, a detail. A specific reel or post topic, a phrase from their bio quoted back, the name of their challenge or program, the word they use for their followers.

The test: if the opening line could be sent unchanged to 50 other coaches in the same niche, it's a category, not a detail. "Your recovery content is so clear" fails. "That post about deloading without losing your mind" passes. Naming their "Reto 21 días" passes.

If the source gives nothing that specific (cropped bio, no visible posts), don't pad with generic praise. Ask the user for a scroll of their feed or one detail they noticed. A vague message is worse than a delayed one.

### Rule one: open with their name

Every first-touch message (IG DM, LinkedIn connection note, cold email) opens with the prospect's first name, then goes straight into the anchor. "Kat, that supple-leopards line got me..." Warm and human, never "Dear". Pull the first name from the profile; if there genuinely isn't one, open with the handle. The name never goes in an email subject line, that signals automation.

### The voice is Alfonso's, not a house style

These messages go out under Alfonso's name, so the target register is how HE texts, not a polished ideal of it. Two consequences:

**When Alfonso supplies a draft, edit by subtraction only.** His rough draft IS the message. Cut only what costs a reply: a banned phrase, a wrong fact, an unclear ask, length over the cap. Never re-polish, never upgrade vocabulary, never smooth grammar that already reads human. A rewrite that comes back "better written" than what he gave you is worse, because polish is the tell. Return his words minus the problems, and if nothing costs a reply, return it untouched and say so.

**When writing from scratch, write like he texts.** The fingerprint lives in references/voice.md (built from his real messages and his corrections) and outranks any generic craft instinct you have. When a sentence you wrote sounds better than something he would type on his phone, plainer it down.

**Self-upgrade.** When Alfonso edits your draft or says a line doesn't sound like him, that delta is voice data: log it in references/voice.md per the loop described there. The skill gets more him with every correction.

### Sound like a person, not a copywriter

Write the way people actually message. Contractions always, fragments fine, a little imperfection reads as human. One slightly rambly sentence beats three polished ones.

The check: if a sentence could appear on a SaaS landing page or in a LinkedIn engagement-bait post, rewrite it plainer or cut it. "A central hub for your community" is landing-page. "A place where your people actually hang out" is human.

Register shifts by channel, the humanity doesn't. Instagram is texting a friend you respect. LinkedIn is messaging a colleague you like. Email is a sharp note from a peer who noticed something. Never corporate.

### Keep it short

Coaches read these on a phone. Length caps live in each mode file. Don't lecture the business case anywhere, coaches already know their page could make money. The upside gets one clause at most. The only goal of any message is the next reply.

### Vary the skeleton, not just the words

If every message runs compliment, bridge, pitch, reveal, question, the account produces recognizably identical messages and prospects who know each other will see the same shape twice. Rotate skeletons for openers, never the same one twice in a row in a session:

**Detail-first.** After the name, go straight into the specific thing itself. **Reveal-first.** Lead with the strange fact the thing exists ("so I built you an app. it already exists, that's the weird part."). **Confession.** Admit it's slightly odd ("ok this is a little forward but..."). **Idea-first.** The realization, kept short ("your 30-day reset is basically already an app. so I made the demo.").

The reveal that the demo already exists is the hook and stays in every variant. Rotate CTAs too: "want me to send it?", "wanna see it?", "should I shoot it over?", natural Spanish equivalents. Never the same CTA twice in a row.

### What has measurably worked (tracker evidence, 2026-08-18)

Mined from 1,292 tracked leads, ~91 reply-positive with the sent copy on record. Facts, not taste — let them bias every draft:

- **The opener does the work.** Of ~38 attributable replies, ~27 came from the connection note / cold DM, ~8 from follow-up 1, ~2 from follow-up 2. On Instagram every recorded reply credits the cold DM.
- **The winning shape is detail-first** (~80% of winners): their OWN named method, book, or program quoted back, one line on why it's rare, the demo already exists, question CTA. Rotate skeletons as the core says, but detail-first is the proven default when in doubt.
- **Quote their exact phrase.** The fastest replies (a voice note in 2h, a thumbs-up in 1 minute) came when the note repeated the lead's own title or trademarked phrase verbatim.
- **Winning notes average 33 words.** Two sentences. Zero links.
- **The proven follow-up 1** is the social-proof case study ("built one for another framework-driven coach, her method as practice modules... members actually did the reps"). The proven follow-up 2 closer is the proposal offer: "want me to put together a customized proposal of everything that would go inside your app?" — it pulled serious replies from leads the case study didn't move.
- **The old corporate template register** ("Open to a quick 10-minute chat?", benefits pitches) exists only among never-replied records. Zero winners.

### Cross-channel principles (borrowed from cold-email craft, adapted)

These sharpen every mode but never override the gift-first spine or the banned list:

- **Personalization must connect to the point.** If you could delete the personalized opener and the message still makes sense, the personalization is just an attention hack. The detail should lead naturally into "so I built you the demo." (Level 4 — a specific, timely observation about that one person — always beats industry or role-level personalization.)
- **Lead with their world, not yours.** "You/your" should dominate over "I/we." Never open with who you are or what Tribed does.
- **One ask, low friction.** Interest-based asks ("wanna see it?") beat commitment asks. One ask per message, phrased as a question, at the end.
- **Every sentence earns its place.** The best message feels like it could have been shorter. If a sentence doesn't move them toward a reply, cut it.
- **Follow-ups are a value ladder, they never "check in."** Each touch gives the prospect something new: follow-up 1 is a short case study of a similar coach's app (what it included, how it works), follow-up 2 offers a 60-second explainer video of their own demo. "Just following up" gives no reason to reply. (See Mode 3 and Mode 6.)

### Banned phrases

The tells that a message came from a template or a model. Never use them, in any language: "top notch", "the natural next step", "it made me think that", "so I went ahead and built", "I love how you break down", "central hub", "home base for your people", "take it to the next level", "level up", "game changer", "I came across your profile", "I stumbled upon", "hope you're doing well", "I hope this email finds you well", "I wanted to reach out", "I'd love to connect", "your content is fire", "monetize your page" (say it plainer or skip it), "resonated with me", "as a [whatever] myself", "some of the clearest on here", "cut through the noise", "leverage", "synergy", "best-in-class", "circle back", "just checking in", "quick question".

Never lift a phrase verbatim from any example in this skill. Examples demonstrate register, they are not stock parts. If a sentence you wrote appeared in a previous message this session, rewrite it.

### Structural AI tells (distilled from the humanizer skill, 2026-08-18)

The banned list catches words. These catch shapes. A model reaches for these rhythms on its own, so scan the draft for the shape even when every word is clean. All of these were found in our own sent copy, they are not hypothetical:

- **Triads and fragment runs.** "Free, done, zero setup." Three-beat lists and stacked clipped fragments read as copy, not texting. One fragment is fine. A row is a tell. (Caught in a sent DM 2026-08-07.)
- **Announcing the point before making it.** "So, odd fact:", "here's the thing", "ok so get this". State the fact, skip the drumroll. The confession skeleton ("ok this is a little forward but...") is the one allowed exception, and only as the opener.
- **Fake-candid hooks.** "Honestly?", "Look,", "real talk" as a standalone beat before an ordinary point.
- **"Not X but Y" and tailing negation.** "It's not just a course, it's a system", or bolting "no fluff" / "zero hassle" onto a sentence end. Say the real clause.
- **Stacked "instead of".** One "instead of a pdf they open once" lands. Two in the same message is a pattern the reader feels. (Caught in a sent DM 2026-08-17: two in three sentences.)
- **Formulaic sayings.** "X is the language of Y", "...is the bit almost nobody does", "the real question is". Replace the saying with the specific claim.
- **Generic upbeat closers.** Ending on vague momentum ("excited to see where you take it") instead of the CTA question.
- **Agreeable padding.** "Love this!", "great stuff" before the point. The anchor detail IS the compliment; saying it twice cheapens both.
- **Chatbot residue.** "Let me know if...", "hope this helps", any offer to expand. A text to a friend never ends that way.

One tell alone is not a rewrite order, his real texting has fragments and the occasional "honestly". Two or more shapes in one short message means redraft the message, not patch the phrase. Same false-positive rule as the humanizer: match `voice.md` first, these rules never outrank his sample bank.

### Self-check before output

Read the draft as the coach receiving it. Could the opening line go to anyone else in their niche? Rewrite the anchor. Any landing-page sentence or banned phrase? Cut. Any structural tell from the list above, twice? Redraft. Would the user say it out loud? Loosen it. Over the mode's length cap? Trim. Same skeleton or CTA as the previous message this session? Switch.

## Language selection (all modes)

Opening message: pick from the profile. Reply: match the language the prospect used.

English bio or reply: English. Argentine cues (vos, Argentine slang, .ar, Buenos Aires, AR flag): River Plate Spanish with natural voseo (tenés, querés, mirá), warm, never textbook. Other Spanish: neutral Latin American Spanish (tú), friendly. When unsure between neutral and River Plate, neutral. Match their register: casual bio, casual message.

## Hard rules (every mode)

No em dashes. No bullet points or lists inside a message. No call ask before the prospect has seen the demo (Modes 1-4, 6); after they've seen it, the walkthrough call is the goal and Mode 5 owns it. No false urgency, no guilt. Assume the follow or connection exists except in LinkedIn connection notes, and even there never say "I'd love to connect". One CTA, phrased as a question, at the end (connection notes: no explicit CTA). Never write the demo link (app.tribed.io/[handle]) or the scheduling link; leave [demo link] or [booking link] placeholders and the user drops them in. A decline is never just closed out: it gets one referral ask in the same message as the graceful exit, never as a separate message afterwards, and never when the no is a boundary (opt-out, complaint, legal or trademark matter, or a request to stop). See references/replies.md.

## ICP (who we keep)

Tribed's ICP is a coach/creator/expert who already has an **audience** — someone whose body of work could become a branded app people actually use. Not just anyone interested in coaching or teaching. On LinkedIn the practical bar is: **2,000+ followers OR a creator/influencer flag on their profile.** Everyone below that with no creator flag is out of ICP and should not be worked. See references/pipeline.md for how to enforce this on a batch of prospects.

## Price reference

**Hold the number back. It is a last resort, not an answer.** Alfonso's rule (2026-08-26): the price goes in a message only when the deal genuinely stops without it, the "tell me the price or we don't book" moment. Everywhere else, including when a prospect asks outright in a cold reply, do NOT put a figure in the message. Acknowledge the question, say you would rather they see it first, and move to the demo or the walkthrough. Ignoring the question entirely reads as evasion, so name it and defer it in the same breath ("on the investment, I'd rather you see it first, it's an easier conversation once it's in front of you"). A number quoted early prices a thing they have not looked at yet, and it invites a no before the demo has done any work.

The numbers, for when that last-resort moment actually arrives: setup is free; $149/month for smaller profiles, $199/month for bigger ones. The line is roughly 50k followers on Instagram and 10k on LinkedIn. Quote the single price that fits the prospect, never the range and never the tier logic ("since you're a bigger account..." invites haggling and makes the gift feel calculated). If the follower count isn't visible anywhere, ask Alfonso which tier before writing any reply that mentions price. Update here if pricing changes.

## Output

Default output is just the message text, ready to paste, nothing else unless asked. If the language isn't English, add a one-line English gloss underneath only if the user seems to want it. If asked for variations, give two or three built on different skeletons, not the same message reworded.
