# Mode 4 — LinkedIn cold outreach

Same gift-first move, same rule zero, register shifted one notch toward "colleague you like": full sentences more often, no slang flood, still contractions and warmth, never corporate. LinkedIn coaches are drowning in automated "I'd love to connect" sequences, so sounding like a human with a specific reason matters even more here than on Instagram.

What to gather: sub-mode (note or InMail), name and niche, the anchor detail from their activity or profile (a recent post is a stronger anchor than their headline), language. If no anchor detail is visible, ask, same as rule zero.

## 4A — Connection request note (~300 char limit)

Goal: the accept, nothing else. One line anchored on their specific detail, one line hinting something was built for them. No price, no features, no CTA beyond the implicit accept. Keep under 280 characters to be safe, confirm the count before outputting.

Vary these as much as DMs. Three notes that all read "your X content is great. I built something around your method." are one template, not three notes. Rotate which half leads: sometimes the detail first, sometimes the odd fact that the thing exists.

**Example (detail-first, English):**
James, your deload-without-losing-your-mind post was the most useful thing I read this week. I ended up building a demo app around your method, it's waiting on the other side of this request.

**Example (reveal-first, English):**
Maria, slightly odd intro: I already built you an app. Your 5-Minute Reset gave me the idea. Accept and I'll show you.

**Example (neutral Spanish):**
Ana, tu serie sobre movilidad sin dolor me hizo armar algo: una demo de una app con tu método adentro. Si aceptas te la muestro.

## 4B — InMail (no prior connection)

Lands in their inbox cold, so they expect slightly more context than a DM, but it earns deletion just as fast. Three to five sentences. Anchor detail first or reveal first, one clause max of business upside, name the app if it helps, one soft question. No emoji-heavy register, no "Dear", no signature block.

**Example (English, nutrition coach, detail-first):**
Hey Sarah, your post on why maintenance phases fail stuck with me, I've quoted it twice since. I build branded apps for coaches and ended up making a demo of one around your method, AI coaching and tracking included, with your name on it. It exists and it's free to look at. Want me to send it over?

**Example (neutral Spanish, mindset coach, reveal-first):**
Hola Carlos, esto va a sonar raro pero ya te hice una app. Tu publicación sobre disciplina sin motivación me dio la idea y terminé armando una demo con tu método, coaching con IA y seguimiento para tu gente. Ya existe y verla no cuesta nada. ¿Te la mando?

## 4C — When the invite is never accepted

Policy agreed 2026-08-23, final form settled the same day across two sessions. The drip enforces it (`mcp/src/repos/linkedinDrip.ts`); this section governs what you write and what you stamp.

**Retire at 21 days, unconditionally.** `PENDING_LOST_DAYS` is a hard floor. An owed InMail does NOT extend it (Alfonso rejected a 42-day exemption explicitly), so the InMail escalation window is day 3 to day 21. A perfect-tier lead that reaches day 21 without its InMail is an upstream failure (authoring job, credit ledger), not a reason to keep the lead alive. The retire also fires from the error path when the profile read fails, because that unconditional +1 day defer is exactly how the 2026-07-23 cohort reached 31 days against a 21 day floor.

**Never write a second connection note.** No withdraw, no re-invite, no second note in different copy.

- **No withdraw.** LinkedIn's weekly cap counts invites SENT, not outstanding, so withdrawing refunds no quota we can spend. Its one real effect is re-invite eligibility after a ~3-week block, which we never use. There is no withdraw endpoint on the worker either. The pending invite is left to expire on LinkedIn's own schedule.
- **No re-invite.** The first note not landing is not evidence that better copy would have.
- **No follow-up**, and nothing fires if they accept months later. They fall out of the sequence.

The retire is terminal: `li_state` goes to `"done"` and `nextActionAt` is cleared. Both are needed. A lead left in `awaiting_accept` with only its date cleared looks parked but re-arms the moment anything writes a date back. The retire note names an undelivered demo (`data.demo_url`) when one exists.

**The Open Profile exception.** The ONE thing that survives the floor. A 2nd-degree profile whose TOP CARD shows a Message control is an Open Profile: a free message reaches them with no connection and no InMail credit. Verified live 2026-08-23 on leadwithak, denisehansard-executivelifecoach and emily-bouchard-5a9a049, all three 31 days pending, all three reachable, all three carrying a finished demo. The drip stamps `data.li_open_profile_at`, logs ONE touch naming the demo, and hands to the daily session for same-day copy; later ticks defer quietly. Detection is scoped to the top card (cut at "Explore Premium profiles" / "· 3rd", whole-line "Message") because the Premium rail below lists other people with their own Message buttons.

**Do not invent a park value.** The drip owns exactly three `li_state` values and treats every other value as somebody else's lead. Since 2026-08-23 an unknown `li_state` write is REJECTED at the repo layer (`assertKnownLiState` in `mcp/src/repos/linkedinLeadState.ts`). Allowed: `to_invite`, `awaiting_accept`, `await_fu2`, `human`, `done`, `connected`. Need a new one? Add it there first.

### The InMail escalation gate (both, not either)

- `data.li_source === "sales_navigator"` — which LinkedIn surface the lead came from, `"sales_navigator"` or `"search"`. **Stamp it at discovery.** Never inferred; an unstamped lead reads as not-Sales-Navigator and never escalates. It lives in the `data` bag, NOT in `channel`: `channel` stays `"li"` for every LinkedIn lead, because it is the routing key the drip and the reply sync filter on, and a second value there drops the lead out of both.
- `data.icp_tier === "perfect"` — the strict Job B step 5 bar in `pipeline.md`.

A perfect-ICP lead blocked only on the missing source tag is called out in the drip's daily touch rather than going quiet. Stamp the real source and it escalates on the next tick.

One InMail, ever, authored the day it ships, day 3 to day 21. Copy it like 4B. **Read the thread and the note we sent first** (`data.li_note` is on the record): if the note already promised a demo, the InMail delivers the link (`data.demo_url`) instead of teasing a second time. Where no demo exists, use the 4B tease.

**Sending the InMail closes the lead.** `li_state` goes to `"done"`, `nextActionAt` clears, the stage stays. The drip does not poll it afterwards. A reply lands in the SALES NAVIGATOR inbox, not `/messaging/` — the daily run reads it with `get_sales_inbox({ filter: "ACCEPTED" })` and `get_sales_conversation({ thread_id })`, and answers INTO THE SAME SN THREAD with `send_sales_message` (dry run with `confirm_send: false` first, Alfonso approves the copy, then `confirm_send: true`; a reply spends no InMail credit). Never answer an InMail reply with a new InMail, and never conclude "no reply" from `/messaging/` reads. Full contract in pipeline.md, "The InMail escalation". Known gap, accepted by Alfonso: a silent ACCEPTANCE after the InMail (no message) is not noticed and follow-up 1 never fires.

### Do not build a demo for an unaccepted invite

Three of the 2026-07-23 cohort had a full demo built three weeks after their invite had already gone unaccepted. When the retire fires it now names the undelivered demo in the touch note. Check the invite was accepted, or that the lead is an Open Profile, or that an InMail is going out, before queueing demo work.
