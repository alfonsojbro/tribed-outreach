# Mode 3 - Follow-up sequences (no reply to the opener)

Two follow-ups after the opener, then stop. They form a value ladder: each one gives the prospect something, never a bare "just checking in". Never guilt, never reference how many times you've messaged. Open with their first name. If the user doesn't say which touch, assume follow-up 1. Stop immediately at any reply, any positive signal, or any no. (If the prospect went quiet after a delivered demo, that's Mode 5 silence, read "The post-demo ladder (Mode 5 silence)" at the end of this file.)

## The ladder (this file is the single source; pipeline.md and email.md defer to it)

| Touch | When | Angle | Evidence |
|---|---|---|---|
| Opener | day 0 | Anchor + the app already exists + question CTA | ~27 of ~38 attributable replies |
| Follow-up 1 | 3-4 days after the ACCEPTANCE, not after the invite | Case study | ~8 replies |
| Follow-up 2 | 4 days after follow-up 1 | Customized proposal offer | ~2 replies, and they were the serious ones |

Then stop. Silence after a genuine gift is an answer.

**The timing is enforced in code, so do not restate it from memory.** `FOLLOWUP_2_DELAY_DAYS` in `NobleAdmin/mcp/src/repos/linkedinDrip.ts` is 4. If that constant changes, change this table in the same commit. This file said "day 10" until 2026-09-06 while the drip sent on day 4, which is how three documents ended up giving three different answers.

**Follow-up 1, 3-4 days after acceptance, the case study.** Show, don't nudge. Tell them how a similar coach in their niche got a branded Tribed app: what it included (their content turned into guided programs, AI coaching trained on their own material, a community space, habit and progress tracking) and how it works in a sentence or two. Concrete but honest, never invent names or metrics you don't have; use a [case study link] placeholder if a link helps. End with one soft question. This earns the reply because it paints the picture instead of asking again.

**Follow-up 2, 4 days later, the proposal offer.** Offer to put together a customized proposal of everything that would go inside their app. This is the proven closer: it pulled serious replies from leads the case study did not move, which is exactly the job of a last touch. Name one thing of theirs the proposal would be built around, so the offer is about their programme rather than our document. One soft question, no price, no link.

**The 60-second video is NOT follow-up 2.** It is a post-demo asset and it belongs to Mode 5 (references/postdemo.md), where the prospect has already opened their demo and gone quiet. Offering a walkthrough video to someone who has never seen the app is a tour of something they have no reason to care about yet. This file documented the video as follow-up 2 until 2026-09-06, against both the tracker evidence and what the live run actually authors.

Channel register: Instagram casual, LinkedIn a notch more professional, never corporate. Vary the wording across prospects, follow-ups are short, which makes copy-paste sameness even easier to spot. No em dashes.

## Examples

**Follow-up 1 (case study), Instagram:**
Jess, quick one, I built a cycle-based coach her own app recently, her programs became guided tracks, the AI answered client questions in her voice, and her community lived in one place instead of scattered DMs. yours is basically ready to look at the same way, want me to send it?

**Follow-up 1 (case study), LinkedIn:**
Tim, a leadership coach I built one for had his framework turned into guided prompts, AI trained on his articles, and a members space, all under his name. Yours would work the same way with the Five Conversations method inside. Want a look?

**Follow-up 2 (proposal offer), LinkedIn:**
Tim, last one from me. Want me to put together a customized proposal of everything that would go inside the Five Conversations app? Costs you nothing to read it.

**Follow-up 2 (proposal offer), Instagram:**
Kat, one more and then I'll leave it. want me to put together a proposal of everything that would sit inside your app, built around the 12 weeks? no strings, just so you can see it on paper.

**Follow-up 2 (proposal offer), neutral Spanish:**
Lucia, ultima de mi parte. Te armo una propuesta de todo lo que iria adentro de tu app, con tu metodo como base? Leerla no te cuesta nada.

## The post-demo ladder (Mode 5 silence)

The ladder above is for a prospect who never replied to the opener. This one is for a prospect whose demo was delivered and who then went quiet. Same shape: three touches, each one gives something, then stop. The clock starts at the DELIVERY of the demo, not at the reply that asked for it. The same clock runs on LinkedIn, X and Instagram. Email has no ladder.

| Touch | When | Angle | Copy field |
|---|---|---|---|
| Demo delivered | day 0 | the link itself, by the drip's demo rail or by hand | `li_demo_message` on LinkedIn |
| Demo bump 1 | 3 days after delivery, no reply | curiosity, anchored on ONE thing inside THEIR app ("did you find the AI coach, ask it about deloading"). When open tracking is live and `data.demo_opened_at` is absent, ask whether the link opened instead | `li_demo_fu1` on LinkedIn, `x_demo_fu1` on X, `ig_demo_fu1` on Instagram |
| Demo bump 2 | 4 days after bump 1, no reply | THE AUDIT OFFER: one line naming what the audit would look at in THEIR funnel, one question answerable in a word. No link, no page, no price. The page is built only after they say yes | `li_demo_fu2` on LinkedIn, `x_demo_fu2` on X, `ig_demo_fu2` on Instagram |
| Demo bump 3 | 4 days after bump 2, no reply | the video walkthrough: offer to walk them through their app, or to send a short video of it. One soft question, never a `[video link]` placeholder | `li_demo_fu3` on LinkedIn, `x_demo_fu3` on X, `ig_demo_fu3` on Instagram |
| Stop | 14 quiet days after bump 3 | `nextAction: "Park (demo ladder done)"`, stage unchanged, door open. The daily report lists them; nobody auto-marks them lost | none |

The states run in order on every rail: `demo_await_1` -> `demo_await_2` -> `demo_await_3` -> `done`. Bump 2 copy carries no link. It is gated like rungs 1 and 3: not empty, no placeholder.

Any reply at any point stops the ladder. The ladder state field is rail-prefixed, one per channel: `data.li_state` on LinkedIn, `data.x_state` on X, `data.ig_demo_state` on Instagram. A reply parks the lead for a reply with `advanceTo: "replied"` and the stop value of that channel: `reply_due` on LinkedIn, `replied` on X, `reply_due` on Instagram. The run then drafts the answer instead of the next bump. A lead carrying `data.slots_offered_at` is NOT on this ladder: its last message was the two-slot offer, so "The 48-hour bump" in references/postdemo.md owns that silence.

**Who sends.** On all three channels the hosted rail sends, never the run. On LinkedIn the drip sends all three bumps, on both sessions, off the `li_*` fields, in `/messaging` or in the Sales Navigator thread: an SN-delivered demo is bumped inside its own InMail thread through the worker's `/sendSalesMessage` route, which only the founder session can use. On X the drip sends them off `x_state` `demo_await_1` / `demo_await_2` / `demo_await_3` and `x_demo_fu1` / `x_demo_fu2` / `x_demo_fu3`, charged against `caps.dm`. On Instagram the worker sends each rung as a one-DM `demo_bump` sequence off `ig_demo_state` and `ig_demo_fu1` / `ig_demo_fu2` / `ig_demo_fu3`. The daily run sends NOTHING itself; this replaces the direct-send wording of 2026-09-14. The run does two things: it enrols a delivered demo with `enrol_demo_ladder`, and it authors each rung's copy the morning that rung sends with `refresh_demo_ladder_copy`. Both are global-only Tribed MCP tools. Both refuse a placeholder, a reply after the delivery, a slot offer in flight and an unproven delivery. Only the refresh refuses a human-owned lead (`automated: false`): enrolment is the handoff, a hand-parked lead enrols after the run has read the thread, and the write flips `automated` to true. The thread-read-first gate stays: read the thread before you enrol or author, and go ahead only when the tail is ours. Email has no hosted rail and no ladder yet. Mechanics per channel are in references/pipeline.md, in "Backfill: put every delivered demo on the clock" and in "Daily pickup: the Instagram, email and X demo ladder".

**The timing is enforced in code, so do not restate it from memory.** `DEMO_FU1_DELAY_DAYS` (3), `DEMO_FU2_DELAY_DAYS` (4), `DEMO_FU3_DELAY_DAYS` (4) and `DEMO_LADDER_REST_DAYS` (14) live in `NobleAdmin/mcp/src/repos/demoLadder.ts`, the one module all three rails share. If a constant changes, change this table in the same commit, the same rule `FOLLOWUP_2_DELAY_DAYS` carries above.

**Bump 1, 3 days after delivery, the one thing inside their app.** Name a single feature that is really in their build and give them a reason to poke at it, in their own words. Curiosity, not pressure. No price, no booking link, no count of how long they have been quiet. One question at the end, answerable in a word.

**Bump 2, 4 days later, the audit offer.** This bump OFFERS the audit. It does not deliver one. Write one line naming what the audit would look at in THEIR funnel, something specific you saw, then one question answerable in a word: "want me to put it together?". No link, no page, no price, no booking link. Build nothing before the send. We only build the audit once they agree to it, because a page nobody asked for is work thrown at silence. A yes is a reply, so it stops the ladder like any other reply. The run that handles that reply builds the page with the sales toolset (`enable_tools` for `sales`, read `get_proposal_guide`, then `upsert_proposal`) and sends the share link in the answer, stamping `data.proposalUrl`. When the account's configured stages include `audit_proposal`, the rail moves the lead to that stage on the bump 2 send.

**Bump 3, 4 days after that, the video.** Offer the 60-second video of their own app, or offer to walk them through it. This is the asset Mode 5 owns, and this is the only bump on the ladder that offers it. Never write a [video link] placeholder here: offer the video, let them say yes, then send it. No price, no booking link, one soft question.

### Examples

**Demo bump 1, LinkedIn (English):**
Tim, one thing worth two minutes: ask the AI coach what to do with a client who stalls at week six. Its answer comes out of your own method. Did you get to it?

**Demo bump 1, Instagram (English):**
Kat, ok slightly nosy question. the habits tab in your app splits your 12 weeks into daily reps, and i want to know if that matches how you run it. did you open it?

**Demo bump 1, X (English):**
Marco, your app has the AI coach trained on your own rest-week guidance. most people test that one first. did you open it yet?

The bump 2 examples carry no link, because there is no page yet. Each one names one real thing in that prospect's funnel and ends on a one-word question.

**Demo bump 2, LinkedIn (English):**
Tim, I keep looking at the step between your free cohort and the paid program. I can write up what I'd change there, on one page, yours to keep. Want me to put it together?

**Demo bump 2, Instagram (English):**
Kat, the part i'd dig into is the week after someone dms you about coaching. i can map that whole path and send it over. want me to?

**Demo bump 2, neutral Spanish:**
Lucia, lo que me llama la atención es que después de la primera charla no hay un paso siguiente claro. Te armo una auditoría de esa parte y te la paso. ¿La quieres?

**Demo bump 3, LinkedIn (English):**
Tim, the screen a member lands on day one is hard to picture from a link. I can record 60 seconds of it, or walk you through it live. Which is easier?

**Demo bump 3, Instagram (English):**
Kat, i can film 60 seconds walking through your app, the part a member lands on first, so you don't have to dig around. should i record it?
