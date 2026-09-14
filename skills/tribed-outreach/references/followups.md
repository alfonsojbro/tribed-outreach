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

The ladder above is for a prospect who never replied to the opener. This one is for a prospect whose demo was delivered and who then went quiet. Same shape: two touches, each one gives something, then stop. The clock starts at the DELIVERY of the demo, not at the reply that asked for it. The same clock runs on LinkedIn, Instagram, email and X.

| Touch | When | Angle | Copy field |
|---|---|---|---|
| Demo delivered | day 0 | the link itself, by the drip's demo rail or by hand | `li_demo_message` on LinkedIn |
| Demo bump 1 | 3 days after delivery, no reply | curiosity, anchored on ONE thing inside THEIR app ("did you find the AI coach, ask it about deloading"). When open tracking is live and `data.demo_opened_at` is absent, ask whether the link opened instead | `li_demo_fu1` on LinkedIn, `demo_fu1` on every other channel |
| Demo bump 2 | 4 days after bump 1, no reply | the 60-second video of their own app, or the walkthrough. One soft question, no price, no booking link | `li_demo_fu2` on LinkedIn, `demo_fu2` on every other channel |
| Stop | 14 quiet days after bump 2 | `nextAction: "Park (demo ladder done)"`, stage unchanged, door open. The daily report lists them; nobody auto-marks them lost | none |

Any reply at any point stops the ladder: the lead parks for a reply (`data.li_state: "reply_due"` on LinkedIn, `data.demo_ladder_state: "reply_due"` on the other channels) with `advanceTo: "replied"`, and the run drafts the answer instead of the next bump. A lead carrying `data.slots_offered_at` is NOT on this ladder: its last message was the two-slot offer, so "The 48-hour bump" in references/postdemo.md owns that silence.

**Who sends.** On LinkedIn the drip sends both bumps, on both sessions, off the `li_*` fields. That half is live, deployed 2026-09-14. The run authors the copy each morning: `li_demo_fu1` for a `demo_await_1` lead and `li_demo_fu2` for a `demo_await_2` lead. On Instagram, email and X the daily run sends them itself, direct-send under Alfonso's authorization of 2026-09-14. There is no review queue and no dashboard approval on this ladder. The one hard gate is a thread read before every send: read the thread first, every time, and send only when the last message is ours and nothing in the thread changes the copy. The `log_outreach_touch` entry carrying the copy verbatim is the record of what went out. Mechanics per channel are in references/pipeline.md, in "Backfill: put every delivered demo on the clock" and in "Daily pickup: the Instagram, email and X demo ladder".

**The timing is enforced in code, so do not restate it from memory.** `DEMO_FU1_DELAY_DAYS` (3), `DEMO_FU2_DELAY_DAYS` (4) and `DEMO_LADDER_REST_DAYS` (14) live in `NobleAdmin/mcp/src/repos/linkedinDrip.ts`. If a constant changes, change this table in the same commit, the same rule `FOLLOWUP_2_DELAY_DAYS` carries above.

**Bump 1, 3 days after delivery, the one thing inside their app.** Name a single feature that is really in their build and give them a reason to poke at it, in their own words. Curiosity, not pressure. No price, no booking link, no count of how long they have been quiet. One question at the end, answerable in a word.

**Bump 2, 4 days later, the video.** Offer the 60-second video of their own app, or offer to walk them through it. This is the asset Mode 5 owns, and this is the only bump on the ladder that offers it. Use a [video link] placeholder when the message carries the video, never a real URL. No price, no booking link, one soft question.

### Examples

**Demo bump 1, LinkedIn (English):**
Tim, one thing worth two minutes: ask the AI coach what to do with a client who stalls at week six. Its answer comes out of your own method. Did you get to it?

**Demo bump 1, Instagram (English):**
Kat, ok slightly nosy question. the habits tab in your app splits your 12 weeks into daily reps, and i want to know if that matches how you run it. did you open it?

**Demo bump 1, X (English):**
Marco, your app has the AI coach trained on your own rest-week guidance. most people test that one first. did you open it yet?

**Demo bump 2, Instagram (English):**
Kat, i can film 60 seconds walking through your app, the part a member lands on first, so you don't have to dig around. should i record it?

**Demo bump 2, LinkedIn (English):**
Tim, the screen a member lands on day one is hard to picture from a link. I can record 60 seconds of it, or walk you through it live. Which is easier?

**Demo bump 2, neutral Spanish:**
Lucia, te grabé un video de 60 segundos de tu app, lo que ve un miembro el primer día: [video link]. ¿Le ves sentido para tu gente?
