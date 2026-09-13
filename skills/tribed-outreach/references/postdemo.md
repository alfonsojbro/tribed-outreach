# Mode 5 — Post-demo: objections and booking the call

> **[NEEDS RECORDING] (checked 2026-09-13).** No 3-minute recorded tour exists yet. `list_media_assets` on `tribed`, `contentpreneur-community` and `digital_university` returned 33 videos, every one b-roll, a talking-head selfie or a motion graphic under 20 seconds. Until a tour video is uploaded to the `digital_university` library and tagged `tour`, the tour line in the templates below is NOT sent as written. Substitute the per-prospect 60-second video of their own app (the existing Mode 5 asset): "or I can send a 60 second video of your app if a call isn't your thing this week". Delete this block, and the `[NEEDS RECORDING]` markers, the day the recording exists.

Use when the prospect has opened their demo and replied, or the user says they've seen it. The gift has been delivered, so the relationship changes: you're no longer a stranger asking for attention, you're the person who built their app. The goal of this mode is the demo walkthrough call, 15 minutes where the user shows them around and answers everything live. That is the goal of the MODE, not of every message in it: the commitment gate below decides when a given reply is allowed to ask for it.

The call is framed as a tour, never a sales call. The pitch already happened, it's sitting in their phone with their name on it. "Want me to walk you through it? 15 minutes, I'll show you what it can do" asks for almost nothing. "Let's hop on a call to discuss next steps" asks for a meeting with a salesperson. Same call, opposite reply rates.

Booking mechanics (rewritten 2026-09-13): the ask is two concrete times, stated in the prospect's own timezone, plus the recorded tour for the call-averse. The scheduling link is the last line of the message and never the ask. On 2026-09-13 the `digital_university` funnel read `interested` 112, `meeting` 5, `won` 3: every win came through a meeting and 95% of warm replies never reached one, and a bare link is where they died (see "The yes" below for the mechanics). Never write any link: leave [booking link] and [tour link] placeholders and the user drops them in. Offer the call once per reply at most, and only once the commitment gate is passed. If they decline the call twice, stop offering it and answer their questions in the DM instead, a forced call loses the deal that patience keeps.

## The commitment gate (Alfonso, 2026-09-06)

**Do not push the call while the prospect is uncommitted. Get some yesses first.**

Opening the demo is curiosity, not commitment. Someone who just asked a question, went quiet, or is still working out what the thing is gets an answer and a smaller ask, never a calendar link. A call ask at that moment turns a live conversation into a decision they are not ready to make, and the answer to a premature ask is usually no. Worse, it spends the goodwill the gift just bought.

Earn the yesses in order of size, one per message:

1. A yes about THEIR material. "want me to rebuild it on your updated version?" Costs them a word, and it pulls their real content into the build.
2. A yes about THEIR people. "do you think this is something your clients could use?" (Same ask as the delivery message in voice.md, still valid here.)
3. A yes to seeing more of it. A section they missed, a short video of their own app, the part that answers what they asked.

Each yes makes the next one cheaper, and by the time the call is offered it is a continuation instead of a leap.

**Commitment reads like:** they asked about price, terms, timelines, or what happens next; they said yes to a build or a change and are engaged in it; they proposed a call or a time themselves; or the exchange has run three or four warm turns and the questions have run out. Then offer the walkthrough, once, framed as a tour.

**Until then, every reply ends on a question they can answer in one word without opening a calendar.**

## Classify the post-demo reply

**Asked a question about it ("did you build this from my site?", "does it do X or Y?").** The most common post-demo reply and the one most often mishandled. Questions are engagement, not commitment. Answer them straight and in the order asked, briefly, then close on a yes about their material or their people. No call. A question answered well buys the next turn, and the next turn is where the call becomes askable. **If the question is one of the five in references/answers.md** (price or tiers, ownership and export, what Tribed runs vs what they do, the AI's role, whether an app flattens their method), draft from that file and carry its `ready` / `needs Alfonso` label; a reply asking several of them gets one message answering each in order.

**Loved it ("this is sick", "wow ok").** Don't oversell a won moment. Match their energy in one short beat, then take the next yes: the thing they reacted to is the thing to build on ("want me to put your real content in?"). Enthusiasm is not a booking signal on its own; hold the call until the gate is passed.

**Lukewarm or confused ("not sure what I'm looking at", "looks fine I guess").** Confusion is answered, not booked around. Name the one thing they missed and show it, or offer a short video of their own app. The walkthrough is still the best cure for confusion, but only after they have said yes to something smaller; offered cold to a confused prospect it reads as a sales move on someone who has not decided they want the thing.

**Buying objections.** Now they're real objections about paying, not brush-offs. Directions below, same rule as Mode 2: write fresh every time, never paste the samples.

**Silence after opening the demo.** One light bump anchored on something specific inside THEIR app ("did you find the AI coach? ask it something about deloading"). Curiosity, not pressure, no booking link yet. If the last message we sent was the two-slot offer, this is not the bump to send: "The 48-hour bump" below owns that silence.

**No after seeing it.** Same graceful exit as Mode 2. The demo stays live, door stays open.

## The yes: two slots, the tour, the link last

Once the gate is passed and the prospect says some version of yes, the reply has a fixed shape. The words rotate every time, the shape does not.

**First, if their yes carried a question, answer it.** A yes and a question usually arrive together ("ok yes, what does this run me?"). If it is one of the five in references/answers.md, draft the answer from that file and carry its `ready` / `needs Alfonso` label; a post-demo price question is answered, not deferred, at $149 a month, one price. The answer leads, the two slots close the same message. Then:

1. One beat that frames the walkthrough as a tour (15 minutes, ask it anything live).
2. **Two concrete times, in the prospect's timezone**, on two different days, phrased as one question ("either work?"). Weekday, date, time, zone label, e.g. "Wed 16 Sep at 8pm ET". Never our time, never "Bali time", never a UTC offset.
3. **The alternative for the call-averse:** "or I can send a 3-minute recorded tour if a call isn't your thing this week." While the `[NEEDS RECORDING]` block at the top of this file stands, substitute the 60-second video of their own app.
4. **The booking link, alone on the last line, as the fallback:** "if neither lands, grab whatever suits here: [booking link]". This is the one line allowed after the CTA question. It is never the lead, never the only option, and never sent without the two times above it.

Timezone unknown: ask, do not guess. If the profile location gives no usable zone (empty, "Remote", "Worldwide", a multi-zone country with no city), the same message asks "which timezone are you in? I'll send two times that fit", keeps the tour line and the fallback link, and the two slots go in the next reply. A wrong guess costs a turn and looks careless; the question costs nothing.

### Deriving the two slots (read, never hardcode)

- **Source of truth is the booking config, read on the day the message is written:** `get_booking_config({ communityId: "digital_university" })` on the Tribed MCP. As read on 2026-09-13: `timezone` Asia/Makassar (UTC+8, no DST), `availability` Mon to Fri 07:00 to 17:00, Sat and Sun empty, `minNoticeHours` 24, `maxAdvanceDays` 30, `bufferMin` 15, `slotIncrementMin` null (so use the service length, `launch-your-app` is 30 minutes), `overrides` null. The `tribed` community holds a second, older config (Mon to Sat 09:00 to 17:00, 4 hours notice); it is not the one this ask uses. Any of these numbers can change without this file changing, which is why the config is read every time and the values here are only the worked example.
- **Subtract what is already taken:** `list_bookings({ communityId: "digital_university", status: "confirmed", fromUtc: <now> })`. Never offer a slot that overlaps a confirmed booking plus `bufferMin`. Google Calendar busy times are subtracted on the booking page but are NOT visible through the MCP, so a proposed time is a proposal until Alfonso confirms it in the thread.
- **The prospect's zone** comes from the profile location line read during the same-day re-read (`get_person_profile` on LinkedIn, the bio city on Instagram), mapped to an IANA zone, then stamped on the lead as `data.prospect_tz` so the bump and later runs reuse it. Convert on the send date with the IANA zone, not with a remembered offset: US clocks change 2026-11-01 and European clocks 2026-10-25, and the overlap shifts by an hour each time.
- **Pick times that are humane for them:** inside the config window AND between 07:00 and 21:00 in their local time. Their two options should sit on different days and, where the overlap allows, at different ends of their window (one earlier, one later). The first slot is at least 48 hours out (they have to see the message and answer it, and `minNoticeHours` is 24); the second is within the following five working days; both inside `maxAdvanceDays`. Clean times only, :00 or :30.
- **The weekday gate is judged in Asia/Makassar, not in their zone.** For the Americas, Alfonso's Monday morning is their Sunday evening (bookable) and their Friday evening is his Saturday (not bookable). So a US prospect gets Sunday to Thursday evenings, never Friday.
- Worked overlap from the 2026-09-13 config (recompute from the config, this table is a sanity check, not a source): US Eastern 7pm to 9pm; US Central 6pm to 9pm; US Pacific 4pm to 9pm; Mexico City and Bogota 5pm or 6pm to 9pm; Buenos Aires 8pm to 9pm only (tight, say so if they push back); UK 7am to 10am; central Europe 7am to 11am; Dubai 7am to 1pm; India 7am to 2:30pm; Sydney 9am to 7pm; Singapore and Bali 7am to 5pm.
- If no humane overlap exists for their zone, do not invent one: offer the edge of the window plainly ("my mornings are your late evenings, 9pm or 9:30pm are the two that work, or the recorded tour") and let them choose.

### After the message

- **Stamp the lead in the same run:** `data.slots_offered` (the two starts, ISO UTC), `data.slots_offered_at` (ISO now), `data.prospect_tz`, and `log_outreach_touch` with `nextAction: "Booking bump (Mode 5)"` and `nextActionAt` 48 hours out. The stamps are what stop the bump from repeating a time.
- **They pick one:** confirm it in the thread in their zone, Alfonso locks the slot (booking page or by hand) and sends the Zoom link. Clear the bump (`nextAction` to the meeting), advance the stage to `meeting`.
- **They ask for the tour:** send [tour link] (or the 60-second video while the recording gap stands), then one question about what they saw, 2 days later. No times in that message.
- **They propose their own time:** take it if it is inside the config window, otherwise offer the nearest one inside it. Never say the window is "Bali hours"; say which times work.
- **They say "just tell me here":** respect it instantly, per the objection direction below. No third offer of times.

### The 48-hour bump

If neither slot is taken within 48 hours (no reply in the thread, no confirmed booking in `list_bookings`), send ONE bump that offers **two new slots**, never the same two. New means different days from the pair in `data.slots_offered`; a different time on the same day still reads as the same offer. The bump repeats the tour alternative in one clause and keeps the link as its last line. Then stop: no third pair, no second bump. A prospect who has passed on four times and a video has answered; the demo stays live and the door stays open, exactly like the silence rule above. Stamp the bump's two slots into `data.slots_offered` as well (append, do not replace) and set `nextAction: "Park (Mode 5 bump sent)"`.

### The same-day rule applies here, unchanged

Slot copy rots faster than any other copy: a time computed on Monday is inside `minNoticeHours` by Wednesday. The two slots are computed on the day the message is sent, from a config read that day, and the message is re-read against the live thread and profile before it goes out, the same gate every other send in this skill passes. A queued slots message older than the day it was authored is recomputed, never re-sent as stored.

### Templates (the shape is fixed, the words rotate; never paste these)

**Yes, timezone known (English, US Eastern prospect, written Mon 14 Sep in Asia/Makassar):**
Prospect: "ok yes do that. what does this run me once it's real?"
it's $149 a month, setup's on me, month to month. that covers the app under your name, the AI trained on your material, hosting and the app store side. and let me show you around it, 15 minutes and you can ask it anything live. I've got Wed 16 Sep at 8pm ET or Mon 21 Sep at 7:30pm ET, either work? or if a call isn't your thing this week I can send a 3-minute recorded tour instead. [NEEDS RECORDING]
if neither lands, grab whatever suits here: [booking link]

**Yes, timezone unknown (English):**
Prospect: "yeah go for it, put my stuff in"
doing it now. and let me show you around it once it's in, 15 minutes and you can ask it anything live. which timezone are you in? I'll send you two times that fit. or if a call isn't your thing this week I can send a 3-minute recorded tour instead. [NEEDS RECORDING]
or grab a time straight from here: [booking link]

**Yes (neutral Spanish, Mexico City prospect):**
Prospect: "sí, dale, ¿cómo seguimos?"
te la muestro en vivo, 15 minutos y le preguntas lo que quieras ahí mismo. tengo el miércoles 16 a las 7pm o el lunes 21 a las 6:30pm, hora de Ciudad de México, ¿te sirve alguno? y si esta semana no estás para llamadas te mando un tour grabado de 3 minutos. [NEEDS RECORDING]
si ninguno te cuadra, elige el que quieras acá: [booking link]

**Yes (River Plate, Buenos Aires prospect):**
Prospect: "dale, me copa, ¿cómo seguimos?"
te la muestro en vivo, 15 minutos y le preguntás lo que quieras ahí mismo. tengo el miércoles 16 a las 8pm o el lunes 21 a las 8:30pm, hora de Buenos Aires, ¿te va alguno? y si esta semana no estás para llamadas te mando un tour grabado de 3 minutos. [NEEDS RECORDING]
si ninguno te cierra, elegí el que quieras acá: [booking link]

**48-hour bump (English, US Eastern, two NEW days):**
Jess, guessing those two didn't fit. what about Tue 22 Sep at 8pm ET or Thu 24 Sep at 7pm ET? the recorded tour still stands if a call is the problem rather than the time. [NEEDS RECORDING]
anything else works too: [booking link]

**48-hour bump (neutral Spanish, Bogota):**
Lucía, me imagino que esos dos no te cuadraron. ¿el martes 22 a las 6pm o el jueves 24 a las 7:30pm, hora de Bogotá? y si el tema es la llamada y no la hora, el tour grabado de 3 minutos sigue en pie. [NEEDS RECORDING]
cualquier otro horario, acá: [booking link]

## Post-demo objection directions

**"That's a lot" / price resistance.** This only comes up once a number is out, and per SKILL.md the number is a last resort, so check it was genuinely earned before quoting one here. Don't defend the number with math walls. One honest comparison in their world (one client covers it, or what they charge for a single session), then the walkthrough: easier to judge if it's worth it after seeing everything it does, 15 minutes. The number is $149 a month, one price, per references/answers.md; never mention a second tier.

**"My audience won't pay for an app."** Genuinely curious, not corrective: what do they sell now? Then reframe, the app isn't a new thing to sell, it's where what they already sell lives. Walkthrough shows it faster than a paragraph.

**"Can I change the content / branding?"** Yes, and this is a booking gift: the walkthrough is literally where you customize it together. "Bring your logo to the call and we'll swap it in live."

**"I need to think about it."** Release pressure completely, then make the call the thinking aid, not the decision: easier to think it over once you've seen everything it does. If they still hesitate, drop it and bump in a week.

**"Just tell me here / I don't do calls."** Respect it instantly, answer in the DM, fully and helpfully. Some deals close async. Never make the call a gate to answers.

**"Is there a contract / can I cancel?"** Plain honesty in one line (monthly, cancel whenever), then back to the call or the question they asked next.

**Facts rule.** The only product facts confirmed in this skill are the fact list in references/answers.md: $149 a month, setup free, month to month with no contract, what that covers, the ownership facts (content, member list and member data stay the coach's; the method is theirs, Tribed is the platform), the Tribed-runs vs coach-does split, the AI as a companion trained on their material with the coach as the authority, and the method branching on their rules rather than flattening into a task list. Export format, the app-store listing on exit and ownership of the customised configuration are `[ALFONSO TO CONFIRM]` blanks in that file: write the reply around them, leave the blank, label it `needs Alfonso`. If a prospect asks anything beyond that list (refunds, offline use, booking, replacing their website), do NOT invent an answer, ask the user for the real policy before writing the reply. A made-up promise in a sales DM becomes a commitment.

## Booking-specific banned phrases

On top of the global list: "hop on a quick call", "jump on a call", "find a time that works for you", "pick your brain", "discuss next steps", "touch base", "no worries if not!", "does that work for you?", "I'd love to show you" (just show them), "quick sync". The word "call" itself is worth rotating: walkthrough, tour, "I'll show you around it", "te la muestro en vivo".

## Examples

**Asked a question, gate not passed (English):**
Prospect: "did you pull this off my site? and does it actually coach or just summarise?"
yeah, your site and your posts, nothing else public. that's why it's a first version. and it coaches, it's trained on your method so someone types what they're stuck on and it tells them what to do. the material you've written since is what the real one gets built from, want me to rebuild it on that?

**Loved it, gate not passed (English):**
Prospect: "ok this is actually really cool, the AI knew my whole method lol"
right? it read everything you've posted. it's still working off public stuff though. want me to put your actual programme in it?

**Confused, gate not passed (English):**
Prospect: "looks nice but I don't really get what my clients would do in it"
fair. open the first day and it'll make sense fast, that's what a client sees when they land. want me to send a 60 second video of it instead?

**Gate passed, booking (English):** see "Templates" above. The old shape (question CTA straight into [booking link]) is retired: the link is the fallback line, the two times are the ask.

**Price (neutral Spanish, Mexico City prospect):**
Prospect: "Está buena pero 149 al mes es bastante"
Te entiendo. Para la mayoría de los coaches una sola sesión o un cliente nuevo ya lo cubre, pero es más fácil juzgarlo viendo todo lo que hace. Son 15 minutos en vivo, tengo el miércoles 16 a las 7pm o el lunes 21 a las 6:30pm, hora de Ciudad de México, ¿te sirve alguno? Y si prefieres, te mando el tour grabado de 3 minutos. [NEEDS RECORDING]
Si ninguno te cuadra: [booking link]

**Doesn't do calls (River Plate):**
Prospect: "no soy de llamadas, decime por acá"
Dale, todo por acá entonces. Preguntame lo que quieras, y si algo es más fácil viéndolo te mando el tour grabado de 3 minutos o un video corto de tu app. [NEEDS RECORDING]
