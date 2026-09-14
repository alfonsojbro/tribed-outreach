# Post-demo follow-up clock

Status: DRAFT, awaiting Alfonso's approval. Written 2026-09-14.

## The problem (measured today)

- 62 leads sit at stage `demo_sent` on `digital_university`, 81 more at `interested` (most with a `demo_url`).
- After the demo goes out, nothing automated touches the lead again. The drip's demo-delivery rail
  (linkedinDrip.ts, 2026-09-09) sends the link, then parks the lead at `li_state: "reply_due"` with
  `nextActionAt` +1 day. Hand-delivered demos are parked `li_state: "human"` or `"done"`.
- Every "bump if quiet" is a prose `nextAction` a person must read and act on. 45 of the 62 are
  overdue, the oldest since 2026-08-06. No one is working that queue, so a ghosted demo is a dead demo.
- postdemo.md has a one-line "Silence after opening the demo" rule and a 48-hour slot bump, but no
  clock, no second touch, and no field the drip could act on.
- The docs never mention the drip's demo rail (`li_demo_message`, `li_demo_copy_at`,
  `li_demo_sent_at`), and pipeline.md Job D describes `demo_tracked_url` / `demo_opened_at` as live,
  but that code sits on the unmerged branch `feat/demo-open-tracking` (2 commits, tip 2026-09-14).

## The ladder (the decision this plan asks Alfonso to confirm)

Same shape as the pre-demo ladder: two touches, each gives something, then stop. Clock starts at
the demo delivery, not at the reply that asked for it.

| Touch | When | Angle | Copy field |
|---|---|---|---|
| Demo delivered | day 0 | the link (existing rail, or by hand) | `li_demo_message` |
| Demo bump 1 | +3 days, no reply | curiosity, anchored on ONE thing inside THEIR app ("did you find the AI coach, ask it about deloading"). If `demo_opened_at` is absent and tracking is live: "did the link open ok on your side?" instead | `li_demo_fu1` |
| Demo bump 2 | +4 days after bump 1, no reply | the 60-second video of their own app, or the walkthrough, one soft question, no price, no booking link | `li_demo_fu2` |
| Stop | +14 days after bump 2, still quiet | `nextAction: "Park (demo ladder done)"`, stage unchanged, door open. The daily run's report lists them; nobody auto-marks lost | none |

Any reply at any point stops the ladder: thread tail `"theirs"` -> `li_state: "reply_due"`,
`advanceTo: "replied"`, exactly what the demo rail does today. A lead that hit the Mode 5 slot
offer (`data.slots_offered_at` set) is NOT on this ladder; the 48-hour bump owns it.

Timing constants live in code (`DEMO_FU1_DELAY_DAYS = 3`, `DEMO_FU2_DELAY_DAYS = 4`,
`DEMO_LADDER_REST_DAYS = 14`) and the doc table cites them, same rule as `FOLLOWUP_2_DELAY_DAYS`.

## Who sends (amended 2026-09-14 on Alfonso's instruction)

- LinkedIn: the drip, both sessions, routed by `data.li_account` via `sessionFor`. Templated
  follow-ups are already drip-sent (FU1, FU2, demo delivery); this is the same class of message.
- Instagram, email and X: the daily run sends the bump itself, direct-send under Alfonso's
  authorization of 2026-09-14, no review queue and no dashboard approval. The one hard gate is a
  thread read before every send: only when the last message is ours and nothing in the thread
  changes the copy. Fields are channel-neutral (`demo_ladder_state`, `demo_fu1`, `demo_fu2`,
  `demo_fu_copy_at`, `demo_fu1_sent_at`, `demo_fu2_sent_at`), `nextActionAt` carries the clock,
  and the `log_outreach_touch` entry with the copy verbatim is the record. Send tools: Instagram
  `send_instagram_dm` (Meta 24h window is closed by day 3), email Gmail `reply` on the same thread,
  X `send_x_dm_session`. No new drip rails for these channels in this plan; a hosted rail can
  follow if the once-a-day run proves too slow for a 3-day clock.

## Code changes (NobleAdmin, branch `feat/li-drip-demo-ladder` off `main`)

`mcp/src/repos/linkedinDrip.ts`
1. Add `"demo_await_1"` and `"demo_await_2"` to `DRIP_OWNED_STATES` (:902) so `isHumanOwned` releases
   them to the drip, and add the three delay constants next to `FOLLOWUP_2_DELAY_DAYS` (:78).
2. Demo rail success path (:2273-2292): when `data.li_demo_fu1` exists, write `li_state:
   "demo_await_1"` and `nextActionAt: plusDays(DEMO_FU1_DELAY_DAYS)` instead of `reply_due` +1.
   When it does not exist, keep today's behaviour (reply_due) so a lead is never drip-owned with no copy.
3. New leg, modelled on the `await_fu2` branch (:2953-3073), for both states:
   - same-day gate on the demo rail's own stamp `li_demo_copy_at` (never the shared `li_copy_at`),
     placeholder check, park `plusDays(0)` when stale, same as the demo rail (:2224-2233);
   - reply guard: `checkDemoThreadTail` (:1034). `"theirs"` -> reply_due + `advanceTo: "replied"`;
     `"no_thread"` -> `li_state: "human"`, `automated: false` (a demo with no thread is a data bug);
     `"unreadable"` -> defer +1; `"ours"` -> send;
   - send `li_demo_fu1` / `li_demo_fu2` through `worker.sendMessage` with `accountId: session`,
     `budget.messages--`, stamp `li_demo_fu1_sent_at` / `li_demo_fu2_sent_at`, touch note quoting
     the status, then advance: fu1 -> `demo_await_2` at +4d; fu2 -> `li_state: "done"`,
     `nextAction: "Park (demo ladder done)"`, `nextActionAt: plusDays(DEMO_LADDER_REST_DAYS)`.
4. The three mirrors (the file's own warning): the due `where` predicate (:3169-3182),
   `isSendReady` (:827-884, tier A so the leg does not starve), and `readDripStatus` (:3781-3842)
   so `dueActionable` / `needsCopy` count the new states and `get_linkedin_session_health` shows them.
5. Counters: `c.demoFu1++` / `c.demoFu2++` on the tick summary line.

`mcp/src/repos/linkedinDrip.test.ts`
6. Tests: demo send lands in `demo_await_1` with +3; stale copy parks; `"theirs"` tail parks
   reply_due; fu1 send advances to `demo_await_2` at +4; fu2 send parks done at +14; a lead with
   `slots_offered_at` is skipped; `readDripStatus` counts a due `demo_await_1` in `needsCopy`.

No change to the outreach tools, no schema change (all new keys live in the `data` bag).

## Doc changes (tribed-outreach, then synced to NobleAdmin's bundled copy)

`skills/tribed-outreach/references/followups.md`
7. New section "The post-demo ladder (Mode 5 silence)" with the table above, the stop rule, and two
   examples per touch (LinkedIn English, Instagram, neutral Spanish). Keep the file the single source.

`skills/tribed-outreach/references/postdemo.md`
8. Replace the "Silence after opening the demo" paragraph with a pointer to followups.md's
   post-demo ladder; state that the 48-hour slot bump is the only silence rule that outranks it.

`skills/tribed-outreach/references/pipeline.md`
9. Fill 1: `demo_await_1` -> author `li_demo_fu1`, `demo_await_2` -> author `li_demo_fu2`, stamp
   `li_demo_copy_at` in the same write. Document the demo rail's fields for the first time.
10. New "Backfill: put every delivered demo on the clock" step, run once per lead: for every lead
    at `demo_sent` or `interested` with a `demo_url` and a delivery on record (`li_demo_sent_at`,
    `demo_delivered: true`, or a touch that says the link went out), no `slots_offered_at`, and a
    thread tail of `"ours"`: set `li_state: "demo_await_1"`, `nextActionAt` = delivery date + 3
    (today when that is already past), author `li_demo_fu1`. Leads whose thread tail is `"theirs"`
    are replies the run owes, not ladder work: park reply_due and draft the answer. Leads with an
    explicit hold in `nextAction` (Neal Foard, Marilia Coutinho, Jason Dorland, Dr Rosa) keep it.
11. Job D: mark the `demo_tracked_url` / `demo_opened_at` paragraph as pending merge of
    `feat/demo-open-tracking`, so bump-1 copy does not depend on a field that is not written yet.

`commands/daily-outreach.md`
12. Quota-fill bullet: add the two demo states to the "EVERY due drip-owned lead" list, and the
    backfill step, and a report line "demo ladder: N on the clock, N bump-1 sent, N bump-2 sent,
    N parked done, N replied".

`skills/tribed-outreach/SKILL.md`
13. Version bump (1.21.0 -> 1.22.0 is already taken by the tracked-link commit; use the next
    number at commit time).

NobleAdmin `mcp/skills/tribed-outreach/references/{followups,postdemo,pipeline}.md`
14. Sync copies of 7-9 (they are byte-identical to tribed-outreach today).

## Files touched

NobleAdmin (`feat/li-drip-demo-ladder`):
- mcp/src/repos/linkedinDrip.ts
- mcp/src/repos/linkedinDrip.test.ts
- src/components/screens/outreach/components/linkedinSequence.ts (added after review: the dashboard's mirror of DRIP_OWNED_STATES)
- src/components/screens/outreach/components/LinkedInLeadSequences.tsx (added after review: state labels for the two ladder states)
- mcp/skills/tribed-outreach/** (full bundled-copy sync to tribed-outreach 1.23.0; the copy was already behind on SKILL.md, postdemo, pipeline, x, replies, linkedin from earlier commits, so a partial sync would leave a mixed snapshot)

tribed-outreach (`main`, one commit `skill(followups): the post-demo ladder puts every delivered demo on a clock (1.23.0)`):
- skills/tribed-outreach/references/followups.md
- skills/tribed-outreach/references/postdemo.md
- skills/tribed-outreach/references/pipeline.md
- commands/daily-outreach.md
- skills/tribed-outreach/SKILL.md
- skills/tribed-outreach/references/x.md (added after review 3: the direct-send tool rule gets a dated carve-out for demo-ladder bumps, per Alfonso's 2026-09-14 instruction)

## Out of scope

- Merging `feat/demo-open-tracking` (separate decision; the ladder works without it and gets the
  opened/not-opened branch in bump 1 for free once it lands).
- Hosted drip rails for the Instagram, email and X bumps (run-driven for now).
- Auto-marking quiet demos as lost.

## Order

Docs first (7-13, they define the copy the run authors), then code (1-6), then sync (14),
then the backfill run (10) on the next daily run. Reviewer scope = the union of the two lists above.
