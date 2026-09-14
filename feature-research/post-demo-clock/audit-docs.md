# Audit: post-demo clock, DOC half (plan items 7 to 13)

Date: 2026-09-14. Repo: tribed-outreach, branch main. Working tree only, nothing committed.

## Files changed

- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/references/followups.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/references/postdemo.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/references/pipeline.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/commands/daily-outreach.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/SKILL.md`

No other file was touched. NobleAdmin was not touched.

## What changed, per file

### followups.md (item 7)

Appended one new section, `## The post-demo ladder (Mode 5 silence)`, after the pre-demo ladder and
its examples. It holds:

- the four-row table (demo delivered day 0 `li_demo_message`; bump 1 at +3 `li_demo_fu1`; bump 2 at
  +4 `li_demo_fu2`; stop at +14 quiet days, `nextAction: "Park (demo ladder done)"`, stage unchanged);
- the bump-1 note that an absent `data.demo_opened_at` becomes an "did the link open" angle only once
  tracking is live;
- the stop rule: any reply parks `li_state: "reply_due"` with `advanceTo: "replied"`, and a lead with
  `data.slots_offered_at` belongs to postdemo.md's 48-hour bump, not this ladder;
- the constants note, mirroring the existing `FOLLOWUP_2_DELAY_DAYS` note: `DEMO_FU1_DELAY_DAYS` 3,
  `DEMO_FU2_DELAY_DAYS` 4, `DEMO_LADDER_REST_DAYS` 14 in
  `NobleAdmin/mcp/src/repos/linkedinDrip.ts`, table and constants change in the same commit;
- one prose paragraph per touch, and four fresh examples: bump 1 LinkedIn English, bump 1 Instagram
  English, bump 2 LinkedIn English, bump 2 neutral Spanish.

Copy check on all four: first name first, under 35 words, no em dashes, no banned phrase, one
question at the end, no URL, nothing lifted from an existing example. `[video link]` appears only in
the Spanish bump 2, the one message that carries the video.

### postdemo.md (item 8)

Replaced the `**Silence after opening the demo.**` paragraph with a pointer to followups.md's
post-demo ladder (bump 1 at +3, bump 2 at +4, then park) and the statement that the only silence rule
outranking it is "The 48-hour bump" below, for a lead whose last message from us was the two-slot
offer. Nothing else in the file changed.

### pipeline.md (items 9, 10, 11)

1. "How the stack is wired", new bullet right after the sequence-map table, documenting the drip's
   demo-delivery rail (2026-09-09) for the first time: `data.li_demo_message`,
   `data.li_demo_copy_at`, `data.li_demo_sent_at`, `data.demo_delivered`; sends only on a thread tail
   of "theirs"; bypasses `isHumanOwned` via `isDemoDeliveryDue`; on success hands the lead to
   `li_state: "demo_await_1"` with `nextActionAt` +3 when `li_demo_fu1` exists, else parks
   `reply_due` +1 as before.
2. Fill 1, new bullet after `await_fu2`: `demo_await_1` to `data.li_demo_fu1` and `demo_await_2` to
   `data.li_demo_fu2`, both stamping `data.li_demo_copy_at` in the same write, never `li_copy_at`.
3. New subsection `### Backfill: put every delivered demo on the clock`, placed after Fill 1 and
   before Fill 1b. Carries the measured reason (2026-09-14: 62 leads at `demo_sent`, 45 overdue,
   oldest since 2026-08-06, none automated), the selection rule, the three shapes that are not ladder
   work (thread tail "theirs" is a reply the run owes; an explicit hold or date in `nextAction` stays,
   naming Neal Foard, Marilia Coutinho, Jason Dorland and Dr Rosa; a demo with no thread is a delivery
   bug, reported by name, naming Nicole Gordon and Iona Holloway), and the Instagram and email case
   (same clock, review queue, `slot: "demo_fu1"` / `"demo_fu2"`).
4. Job D step 4, one sentence appended to the `demo_tracked_url` paragraph: both keys are written by
   the unmerged branch `feat/demo-open-tracking` as of 2026-09-14, so an absent `demo_opened_at` means
   UNKNOWN until it lands and bump-1 copy must not claim they have not opened the link.

### commands/daily-outreach.md (item 12)

Step 0 QUOTA FILL, the "LinkedIn copy, both sessions" bullet: the due drip-owned list now also names
`demo_await_1` demo bump 1s and `demo_await_2` demo bump 2s, with the `data.li_demo_copy_at` stamp
called out, and the bullet ends by naming the pipeline.md backfill step. Step 3 REPORT: the demo
ladder line added to the digest list, worded as specified.

### SKILL.md (item 13)

`version: 1.21.0` to `1.23.0`. Nothing else. SKILL.md line 96 ("follow-up 2 offers a 60-second
explainer video") was left alone as instructed.

## Deviations from the plan

One, small. In pipeline.md Fill 1, the sentence that builds the authoring list enumerated the
drip-owned states (`to_invite` / `awaiting_accept` / `await_fu2`) one paragraph above the bullet
list. Adding the two bullets without adding the two states there would have left the section
instructing the run to build a list that excludes the very leads the new bullets describe. I added
`demo_await_1` / `demo_await_2` to that parenthetical. Same paragraph, same in-scope file, no other
text changed.

No other deviation. No em dash was added anywhere (the em dashes visible in the diff of
daily-outreach.md and Job D are pre-existing text on the modified lines).

## Test results

No code changed, so no test suite applies. Checks run instead:

- `git diff -U0 | grep '^+' | grep '—'` returns only pre-existing prose on modified lines; no added
  sentence carries an em dash.
- Word count and one-question check on each of the four new examples: pass.
- `git diff --stat`:

```
 commands/daily-outreach.md                     |  4 ++--
 skills/tribed-outreach/SKILL.md                |  2 +-
 skills/tribed-outreach/references/followups.md | 33 ++++++++++++++++++++++++++
 skills/tribed-outreach/references/pipeline.md  | 20 ++++++++++++++--
 skills/tribed-outreach/references/postdemo.md  |  2 +-
 5 files changed, 55 insertions(+), 6 deletions(-)
```

## Open risks

- The docs now describe `demo_await_1` / `demo_await_2` as drip-owned states, and the code that makes
  them so (plan items 1 to 6, `linkedinDrip.ts`) is not written yet. Between this commit and that
  one, a lead written to `demo_await_1` is invisible to the drip and to `isHumanOwned`, so it would
  sit still rather than send. Run the backfill only after the code half lands.
- pipeline.md line 43 still describes `isHumanOwned` as a whitelist of exactly `to_invite` /
  `awaiting_accept` / `await_fu2`. That sentence becomes stale when item 1 adds the two demo states.
  It sits outside this task's stated edits, so it was left; the code half should update it.
- Job D now carries two adjacent statements about an absent `demo_opened_at`: the older flat claim
  ("means they have not opened it") and the new dated override ("means UNKNOWN until the branch
  lands"). The override reads after it and is dated, which matches the file's house style, but the
  older clause should be deleted the day `feat/demo-open-tracking` merges.
- The named leads (Neal Foard, Marilia Coutinho, Jason Dorland, Dr Rosa, Nicole Gordon, Iona
  Holloway) and the 62/45 counts are a 2026-09-14 snapshot handed down in the plan; they were not
  re-measured in this run.

## Amendment pass (IG, email, X direct-send)

Date: 2026-09-14. Same branch (`main`), working tree only, nothing committed. Scope change from
Alfonso: the post-demo ladder runs on Instagram, email and X with the same clock, and the daily run
SENDS those bumps itself. No review queue, no dashboard approval. The one hard gate is a thread read
before every send.

### Files changed

- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/references/followups.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/references/pipeline.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/commands/daily-outreach.md`

No other file was touched in this pass. `postdemo.md` and `SKILL.md` keep the first pass's edits and
were not opened for writing. NobleAdmin was not touched.

### What changed, per file

**followups.md, post-demo ladder section.**

- The intro sentence now states the same clock runs on LinkedIn, Instagram, email and X.
- The table's "Copy field" column is channel-neutral: `li_demo_fu1` / `li_demo_fu2` on LinkedIn,
  `demo_fu1` / `demo_fu2` on every other channel. The delivery row names `li_demo_message` as the
  LinkedIn field only.
- The reply-stop sentence now names both park fields: `data.li_state: "reply_due"` on LinkedIn,
  `data.demo_ladder_state: "reply_due"` elsewhere.
- New **Who sends** paragraph: LinkedIn is the drip on both sessions; Instagram, email and X are the
  daily run, direct-send under Alfonso's authorization of 2026-09-14, no review queue and no
  dashboard approval, thread read first every time, send only when the last message is ours and
  nothing in the thread changes the copy, and the `log_outreach_touch` entry with the copy verbatim
  is the record. It points at pipeline.md for the per-channel mechanics.
- Two new examples: demo bump 1 on X, demo bump 2 on Instagram. Both casual register, first name
  first, under 30 words (24 and 24), one question at the end, no em dash, no banned phrase, no URL,
  and no wording reused from an existing example.

**pipeline.md.**

- "Backfill: put every delivered demo on the clock": the closing paragraph that sent Instagram and
  email bumps to the review queue with `slot: "demo_fu1"` / `"demo_fu2"` is GONE. In its place:
  a paragraph naming the direct-send authorization and the channel-neutral fields
  (`data.demo_ladder_state` with `"await_1"` / `"await_2"` / `"done"`, `data.demo_fu1`,
  `data.demo_fu2`, `data.demo_fu_copy_at` under the same same-day rule, `data.demo_fu1_sent_at`,
  `data.demo_fu2_sent_at`, `nextActionAt` carrying the clock); a four-row table of thread read and
  send tool per channel (LinkedIn drip; Instagram `unibox_sync_instagram` + `unibox_get_thread`
  falling back to `read_instagram_session_inbox`, sending with `send_instagram_dm` because the Meta
  24-hour window is closed by day 3; email Gmail `search_threads` / `get_thread` and Gmail `reply` on
  the SAME thread per Mode 6, stamping `data.email_bumped_at`; X `read_x_session_inbox` and
  `send_x_dm_session`); the hard thread-read gate, with the theirs-tail park, the defer-one-day rule
  for an unreadable thread or a failed sync, and "never send blind"; and the per-send rule
  (`log_outreach_touch` with the copy verbatim, `automated: true`, `nextAction` naming the next rung,
  `nextActionAt` the next due date, and a non-confirmed send stamping nothing).
- The `isHumanOwned` bullet (line ~43) now lists `demo_await_1` / `demo_await_2` alongside
  `to_invite` / `awaiting_accept` / `await_fu2`. This closes the first pass's second open risk.
- The LinkedIn rows and the `li_*` fields are unchanged everywhere else.

**commands/daily-outreach.md.**

- Step 3 REPORT: the demo ladder line is now counted per channel, worded
  "demo ladder (li / ig / em / x): N on the clock, N bump-1 sent, N bump-2 sent, N parked done,
  N replied off the ladder".
- Intro paragraph: one sentence added after the IG-and-email human-approval sentence, saying the
  post-demo bumps on Instagram, email and X are the one exception, shipped from the run under
  Alfonso's 2026-09-14 authorization after a thread read, never from the review queue.

### Deviations from the plan

None. Every edit is inside the three files named for this pass. The selection sentence in the
backfill still opens on `channel "li"` leads; the new block says the non-LinkedIn channels are
selected the same way on their own channel, rather than rewriting that sentence, to keep the diff
minimal.

### Test results

No code changed, so no test suite applies. Checks run instead:

- `grep` for a `slot: "demo_fu1"` / `"demo_fu2"` review-queue trace across `skills/` and `commands/`:
  no hit. The review-queue idea is gone from the ladder.
- `git diff -U0 | grep '^+' | grep '—'`: every hit is pre-existing prose on a modified line. No
  sentence added in this pass carries an em dash.
- Word count and one-question check on the two new examples: pass.
- `git diff --stat` (cumulative, both passes):

```
 commands/daily-outreach.md                     |  6 ++--
 skills/tribed-outreach/SKILL.md                |  2 +-
 skills/tribed-outreach/references/followups.md | 41 ++++++++++++++++++++++++++
 skills/tribed-outreach/references/pipeline.md  | 33 +++++++++++++++++++--
 skills/tribed-outreach/references/postdemo.md  |  2 +-
 5 files changed, 76 insertions(+), 8 deletions(-)
```

Nothing was committed.

### Open risks

- The docs now describe a run-driven send path on Instagram, email and X that no code enforces. The
  thread-read gate is prose, so a run that skips it leaves no guard behind it. The LinkedIn half has
  the drip's own tail check; these three do not.
- `data.demo_ladder_state`, `data.demo_fu1`, `data.demo_fu2`, `data.demo_fu_copy_at`,
  `data.demo_fu1_sent_at` and `data.demo_fu2_sent_at` are new keys in the `data` bag. No tool
  validates them, so a typo on any one is silent and the lead simply never sends.
- The first pass's other open risks stand: the LinkedIn ladder code (`linkedinDrip.ts`) is still
  unwritten, so the backfill must wait for it, and Job D still carries the older flat claim about an
  absent `demo_opened_at` next to the new dated override.

## Review fix pass

Date: 2026-09-14. Repo: tribed-outreach, branch main. Working tree only, nothing committed.
NobleAdmin was not touched.

### Files changed

- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/references/followups.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/references/pipeline.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/commands/daily-outreach.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/SKILL.md`

`postdemo.md` keeps the first pass's edit and was not opened for writing in this pass.

### Where each item landed

- **B1, dated pending marker in three places.** pipeline.md demo-rail bullet in "How the stack is
  wired": the hand-off sentence now ends with the marker, the do-NOT-write rule for
  `li_state: "demo_await_1"` (the drip does not serve it, the write destroys the `reply_due` park),
  and the note that the Instagram, email and X half runs now. pipeline.md Backfill subsection: the
  same marker on the paragraph that writes `demo_await_1`. commands/daily-outreach.md, the backfill
  sentence at the end of the "LinkedIn copy, both sessions" bullet: the same marker, worded for the
  run.
- **B2, Guardrails carve-out.** pipeline.md Guardrails, the "Unattended runs never send anything that
  was not queued as a draft first" bullet now ends with the ladder carve-out, the 2026-09-14
  authorization, the thread read in the Backfill subsection, and the touch log as the record.
- **B3, "Nothing sends itself" carve-out.** pipeline.md, one sentence added to that bullet pointing
  at the ladder and naming the thread read as the gate in place of a draft.
- **B4, Backfill predicate.** Rewritten as six bullets. The last one is the exclusion: skip any lead
  whose `data.li_state` is `demo_await_1` / `demo_await_2` / `done`, any lead carrying a
  `data.demo_ladder_state`, and any lead carrying `data.li_demo_fu1_sent_at` or
  `data.demo_fu1_sent_at`. "The backfill is the entry point only. It never rewinds a lead."
- **B5, daily pickup.** New pipeline.md subsection "Daily pickup: the Instagram, email and X demo
  ladder", placed right under the backfill, with four numbered rungs (bump 1, bump 2, tail theirs,
  unreadable thread) and the per-channel sending account (Instagram from the `accountId` in
  `configuredAccounts` whose unibox holds the thread, email from Alfonso's Gmail, X from the
  `digital_university` session). commands/daily-outreach.md QUOTA FILL carries it as its own bullet,
  "Demo ladder, Instagram, email and X (its own step, and it runs now)", not inside the LinkedIn
  copy bullet.
- **N1, state strings aligned.** pipeline.md now reads
  `data.demo_ladder_state` (`"demo_await_1"` | `"demo_await_2"` | `"done"`, plus `"reply_due"`).
  No bare `"await_1"` / `"await_2"` remains anywhere under `skills/` or `commands/`.
- **N2, SKILL.md follow-up 2.** The clause now says follow-up 2 offers the customized proposal, and
  that the 60-second video belongs to the post-demo ladder in Mode 5. Nothing else in that bullet.
- **N3, followups.md video claim narrowed** to "the only bump on the ladder that offers it".
- **N4, pointer loop broken.** followups.md line 3 now sends a quiet-after-demo reader to
  "The post-demo ladder (Mode 5 silence)" at the end of that same file. SKILL.md's Mode 5 row adds
  "went quiet after: the post-demo ladder in references/followups.md".
- **N5, Gmail tool names kept**, with "(Gmail MCP)" added after the first mention of
  `search_threads` / `get_thread` and of `reply` in the channel table.
- **N6, copy.** Demo bump 1 Instagram rewritten on the confession skeleton and
  "that's the bit people poke at first" is gone. Demo bump 2 LinkedIn rewritten detail-first, leading
  on the day-one screen. Both: first name, one question, no link, no em dash, no banned phrase,
  34 and 32 words.
- **N7, Job D stale clause deleted.** The flat "`demo_opened_at` absent means they have not opened
  it, not that tracking is broken" is gone; the dated UNKNOWN sentence stands alone.

### Deviations

None. Every edit is inside the five files the plan names.

### Test results

No code changed, so no test suite applies. Checks run instead:

- `git diff -U0 | grep '^+' | grep '—'`: every hit is pre-existing prose on a modified line. No
  sentence added in this pass carries an em dash.
- `grep -rn '"await_1"\|"await_2"' skills/ commands/`: no hit.
- Word count and one-question check on the two rewritten examples: 34 and 32 words, one question each.
- `git diff --stat` (cumulative, all three passes):

```
 commands/daily-outreach.md                     |  7 ++--
 skills/tribed-outreach/SKILL.md                |  6 +--
 skills/tribed-outreach/references/followups.md | 43 ++++++++++++++++++-
 skills/tribed-outreach/references/pipeline.md  | 57 +++++++++++++++++++++++---
 skills/tribed-outreach/references/postdemo.md  |  2 +-
 5 files changed, 102 insertions(+), 13 deletions(-)
```

Nothing was committed.

### Open risks

- The pending markers are prose. Nothing enforces them, so a run that skips the marker can still
  write `demo_await_1` on a LinkedIn lead and lose that lead's `reply_due` park. Delete all three
  markers the day `feat/li-drip-demo-ladder` is deployed.
- The Instagram, email and X send path is still guarded only by the prose thread-read gate. No code
  enforces it.
- `data.demo_ladder_state` now shares LinkedIn's state vocabulary. That is easier to read, but a
  reader who greps for `demo_await_1` gets both the LinkedIn `li_state` and the channel-neutral
  field, so the code half must keep the two fields apart.

## Review fix pass 2

Date: 2026-09-14. Repo: tribed-outreach, branch main. Working tree only, nothing committed.
NobleAdmin was not touched.

### Files changed

- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/references/pipeline.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/references/followups.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/commands/daily-outreach.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/SKILL.md`

`postdemo.md` keeps its earlier edit and was not opened for writing in this pass.

### Where each item landed

- **B1, backfill exclusion.** pipeline.md, the "It is not already on the ladder" bullet in
  "Backfill: put every delivered demo on the clock". `done` is no longer an exclusion. The bullet
  now skips `li_state` `demo_await_1` / `demo_await_2`, any `data.demo_ladder_state`, and any
  `data.li_demo_fu1_sent_at` / `data.demo_fu1_sent_at`, and states that a lead at `done` is on the
  ladder only when one of those stamps is present. Without them it is a hand-delivered demo and it
  qualifies, which is the class the backfill exists for.
- **B2, the Instagram / email / X entry write.** pipeline.md, a new paragraph right after the
  channel-neutral field list in the same subsection: write `data.demo_ladder_state: "demo_await_1"`,
  set `nextActionAt` to delivery plus 3 days (today when past), author `data.demo_fu1` with
  `data.demo_fu_copy_at` in the same write, and the note that this half is gated on no branch and
  runs today. Placed at the end of that paragraph block rather than mid-paragraph so the field names
  are defined before they are used.
- **B3, the four missing "not deployed" markers.** pipeline.md line ~43, appended inside the
  `isHumanOwned` whitelist parenthetical; the bullet below it now reads `to_invite` /
  `awaiting_accept` / `await_fu2`, "plus `demo_await_1` / `demo_await_2` once
  `feat/li-drip-demo-ladder` is deployed". pipeline.md Fill 1, on the drip-owned predicate sentence
  and on the `demo_await_1` / `demo_await_2` authoring bullet. followups.md "Who sends", on the
  LinkedIn sentence. commands/daily-outreach.md, the first clause of the QUOTA FILL LinkedIn copy
  bullet, which now says the two demo states are authored only once the branch is deployed, so the
  bullet no longer authorises and then forbids the same write.
- **B4, account gates on the Instagram and X sends.** pipeline.md, a new short paragraph under
  "Read the thread first": an account stood down by the Instagram two-step preflight sends no bumps
  that day, an Instagram bump is charged against `caps.dm.remainingToday` and defers one day at 0,
  and an X bump needs a clear `get_x_session_account_health` read in the same run. Mirrored as three
  sentences at the end of the daily-outreach.md demo ladder bullet.
- **N1, daily pickup selector.** pipeline.md "Daily pickup": the selector is now fenced to
  `data.demo_ladder_state` `"demo_await_1"` or `"demo_await_2"`, and a lead parked `done` that comes
  due again at +14 matches no rung, is reported under "parked done", and has its `nextActionAt`
  cleared.
- **N2, `advanceTo: "replied"`.** Added next to `reply_due` in the Daily pickup rung 3 (tail theirs)
  and in the backfill's "Thread tail theirs" bullet, matching followups.md.
- **N3, both subsection names.** followups.md "Who sends" now points at both
  "Backfill: put every delivered demo on the clock" and
  "Daily pickup: the Instagram, email and X demo ladder".
- **N4, send rungs after the reply backlog.** daily-outreach.md demo ladder bullet rewritten:
  QUOTA FILL authors the copy and writes the entry stamps only, and the send rungs run in step 1,
  after the reply backlog (1.0), never inside QUOTA FILL.
- **N5, no double authoring.** pipeline.md Daily pickup rung 1: a `data.demo_fu_copy_at` stamped
  today is fresh, so copy the backfill authored the same morning is kept and nothing is re-authored.
- **N6, SKILL.md Mode 5 row.** "Prospect opened the demo (replied or went quiet after)" is now
  "Prospect has the demo (replied, or went quiet after delivery)". The rest of the row is unchanged.

### Deviations from the fix list

One placement choice, no content change. B2 asked for the entry write "after the selection
sentence". It is written at the end of that same paragraph block, one sentence later, so that
`data.demo_ladder_state` and `data.demo_fu_copy_at` are introduced before they are used.

### Test results

No code changed, so no test suite applies. Checks run instead:

- `git diff -U0 | grep '^+' | grep '—'`: 7 hits, every one pre-existing prose on a modified line.
  No sentence added in this pass carries an em dash.
- No new run-executed sentence exceeds 25 words; the daily-outreach.md bullet was split into short
  sentences for that reason.
- `git status --short`: only the five doc files are modified. NobleAdmin was never opened.
- `git diff --stat` (cumulative, all four passes):

```
 commands/daily-outreach.md                     |  7 +--
 skills/tribed-outreach/SKILL.md                |  6 +--
 skills/tribed-outreach/references/followups.md | 43 +++++++++++++++++-
 skills/tribed-outreach/references/pipeline.md  | 63 +++++++++++++++++++++++---
 skills/tribed-outreach/references/postdemo.md  |  2 +-
 5 files changed, 107 insertions(+), 14 deletions(-)
```

Nothing was committed.

### Open risks

- The "not deployed" marker now sits in six places across three files. All six must be deleted the
  day `feat/li-drip-demo-ladder` is deployed, or the docs will forbid a rail that works.
- `li_state: "done"` still carries three meanings (retired invite, InMail sent, hand-delivered demo,
  plus ladder parked). The backfill now reads the stamps to tell them apart, but nothing in code
  enforces that reading.
- The Instagram cap and X health gates are prose, like the thread-read gate. No code enforces them.

## Review fix pass 3

Date: 2026-09-14. Working tree only, nothing committed. `references/x.md` is newly in scope this
pass and is added to the Files changed list above in spirit: the full set is now
`commands/daily-outreach.md`, `skills/tribed-outreach/SKILL.md`,
`skills/tribed-outreach/references/followups.md`, `skills/tribed-outreach/references/pipeline.md`,
`skills/tribed-outreach/references/postdemo.md`, `skills/tribed-outreach/references/x.md`.
NobleAdmin was not opened.

### Blocking

- **B1, where the Daily pickup runs.** pipeline.md, the "Daily pickup" intro sentence. It no longer
  says "right after the backfill". It now splits the two halves by phase: the backfill sits in step
  0, QUOTA FILL, where it authors the copy and writes the entry stamps and sends nothing; the send
  rungs run in SHIP, after the reply backlog (1.0), never inside QUOTA FILL. That matches
  daily-outreach.md, which already said the same.
- **B2, the email bump cap.** pipeline.md, a new paragraph next to "The ladder is inside the account
  gates": an email bump spends the same mailbox cap as a cold send, per email.md's 5 a day with a
  hard ceiling of 15, the lead defers one day when the day's budget is spent, and a halted
  `email_day_ledger` sends no bumps. Mirrored as three short sentences at the end of
  daily-outreach.md's demo ladder gate list.
- **B3, the `send_x_dm_session` carve-out.** x.md, right under "The direct tools ... are for
  Alfonso's explicit one-target commands only", in the file's own voice: since 2026-09-14, on
  Alfonso's instruction, the daily run may call `send_x_dm_session` for ONE case, a post-demo ladder
  bump, after the `read_x_session_inbox` thread read, behind a clear `get_x_session_account_health`,
  charged to `caps.dm`; every other daily X send still goes through the drip. A second paragraph
  carries the drip guard: an X ladder lead must not hold `x_state: "to_touch"` with a due
  `nextActionAt`, park it `done` instead. The same guard is stated in pipeline.md next to the
  account gates, and the pipeline.md X table row now points at the carve-out by file.
- **B4, `demo_ladder_state` on a LinkedIn lead.** pipeline.md, appended to the "Instagram, email and
  X demos run the same clock" sentence: the key is never written on a `channel "li"` lead, the
  LinkedIn half uses `li_state` only, and a LinkedIn lead carrying it is excluded from the backfill
  forever. daily-outreach.md's QUOTA FILL LinkedIn bullet is rewritten: the LinkedIn half is REPORT
  ONLY until `feat/li-drip-demo-ladder` deploys, so the run counts the qualifying LinkedIn leads,
  names the count in the digest, and writes nothing. The REPORT line now says the `li` column reads
  0 with that count beside it ("li 0 (N waiting on the branch)").

### Nits

- **N1, the backfill's "Thread tail theirs" bullet.** pipeline.md: the park field is now named per
  channel (`li_state: "reply_due"` on LinkedIn, `data.demo_ladder_state: "reply_due"` elsewhere),
  both with `advanceTo: "replied"`, and the lead is handed to step 1.0 rather than drafted at step
  0. The bullet says explicitly that the reply backlog owns the draft.
- **N2, the SHIP bullet.** daily-outreach.md step 1, a new bullet directly after the reply-backlog
  bullet: "Demo ladder send rungs (pipeline.md 'Daily pickup ...'), Instagram, email and X, inside
  the account gates", plus what it does in three short sentences and "Nothing on LinkedIn."
- **N3, the measured counts.** pipeline.md: the 62/45 line is now "(measured 2026-09-14)" with a
  sentence saying to recount before quoting. The two named-lead bullets read "(measured
  2026-09-14)" with the same instruction, matching the file's existing "(measured YYYY-MM-DD)" style.

### Deviations from the fix list

None. Every item landed where the list asked.

### Checks run

No code changed, so no test suite applies.

- `git diff -U0 | grep '^+' | grep '—'`: 7 hits, all pre-existing prose on modified lines. No
  sentence added this pass carries an em dash.
- Every run-executed sentence added this pass is under 25 words. Three were split for that reason,
  all in daily-outreach.md.
- `git status --short`: only the six doc files are modified.
- `git diff --stat` (cumulative, all passes):

```
 commands/daily-outreach.md                     |  8 +--
 skills/tribed-outreach/SKILL.md                |  6 +--
 skills/tribed-outreach/references/followups.md | 43 ++++++++++++++++-
 skills/tribed-outreach/references/pipeline.md  | 67 +++++++++++++++++++++++---
 skills/tribed-outreach/references/postdemo.md  |  2 +-
 skills/tribed-outreach/references/x.md         | 15 ++++++
 6 files changed, 127 insertions(+), 14 deletions(-)
```

Nothing was committed.

### Open risks

- followups.md's "Who sends" still describes the LinkedIn half as drip-sent once the branch lands.
  It does not repeat the new REPORT ONLY framing; daily-outreach.md is the run-executed copy and it
  does. Worth a look if the two are meant to read identically.
- The "not deployed" marker now sits in seven places across four files. All must be removed the day
  `feat/li-drip-demo-ladder` deploys.
- The X drip guard, the email cap and the IG cap are all prose. No code enforces any of them.

## Review fix pass 4

Date: 2026-09-14. Working tree only, nothing committed. NobleAdmin was not touched.

Files changed in this pass:

- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/references/pipeline.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/skills/tribed-outreach/references/x.md`
- `/Users/alfonsobriceno/Documents/Projects/tribed-outreach/commands/daily-outreach.md`

Items:

- **B1** (pipeline.md, Fill 4, the "It is reference, not work" bold clause): appended the post-demo
  ladder carve-out, dated 2026-09-14. It names the three writes (`data.demo_ladder_state`,
  `data.demo_fu*`, `x_state: "done"`) and keeps the no-draft, no-probe half of the rule.
- **B2** (daily-outreach.md, SHIP bullet "Demo ladder send rungs"): "due today" is now "due today or
  earlier", matching the QUOTA FILL bullet and pipeline.md.
- **N1** (pipeline.md, backfill LinkedIn paragraph): the backfill ALWAYS writes `data.li_demo_sent_at`
  (delivery date, ISO) on a hand-delivered demo that has none, because the drip's record-side reply
  guard keys on that stamp. Placed before the branch-gate bold sentence, so the gate still reads last.
- **N2** (pipeline.md Fill 3, and daily-outreach.md QUOTA FILL Instagram bullet): reserve the day's due
  Instagram ladder bumps out of `caps.dm.remainingToday` before the cold sequences spend it. The
  pipeline.md copy names the count (`ig` leads due today or earlier at `demo_await_1` / `demo_await_2`).
- **N3** (pipeline.md, the email cap sentence): the cold email leg (phase 1e) subtracts the ladder bumps
  already sent that day from the 5, so bumps plus cold sends never go over the mailbox cap.
- **N4** (x.md, the "Re-arm before sourcing a stranger" rule): the re-arm to `"to_touch"` now carries
  "unless the lead is on the post-demo ladder (`data.demo_ladder_state` set)".
- **N5** (daily-outreach.md, QUOTA FILL demo-ladder bullet): names the remedy, "Park it
  `x_state: "done"` before you write `nextActionAt`."
- **N6** (daily-outreach.md, REPORT list): the "li column reads 0" sentence moved out from between
  "demos built," and "stale invites, errors". It now sits after "errors." and the list reads whole.
- **N7** (daily-outreach.md, REPORT ONLY LinkedIn count): the count is on the cheap predicate only
  (stage `demo_sent` or `interested`, a `data.demo_url`, a delivery on record) with no thread read, so
  it does not cost 62 thread reads a morning.
- **N8** (pipeline.md, the "IG is never auto-sent" line): appended
  "(the post-demo ladder bumps are the dated exception, see Guardrails)". The existing em dash on that
  line is unchanged.

Deviations: none. No new em dash was written in this pass. Every added sentence is short, active, and
one idea.

Tests: none exist for documentation. Verification was textual: each edit matched exactly one unique
anchor string, and the run failed closed on any other count.

Open risks: N8 points the reader at the `## Guardrails` heading in pipeline.md, which exists but does
not itself repeat the ladder carve-out. If a later pass moves that heading, the pointer goes stale.

`git diff --stat` after this pass:

```
 commands/daily-outreach.md                     | 10 ++--
 skills/tribed-outreach/SKILL.md                |  6 +--
 skills/tribed-outreach/references/followups.md | 43 ++++++++++++++-
 skills/tribed-outreach/references/pipeline.md  | 73 ++++++++++++++++++++++----
 skills/tribed-outreach/references/postdemo.md  |  2 +-
 skills/tribed-outreach/references/x.md         | 18 ++++++-
 6 files changed, 133 insertions(+), 19 deletions(-)
```

## Deploy-day marker removal (2026-09-14)

`feat/li-drip-demo-ladder` merged into NobleAdmin main (615578e) and the MCP deployed to Fly at 03:27Z (build 615578e). The seven "not deployed / REPORT ONLY / pending branch" markers were rewritten so the LinkedIn half reads as live: pipeline.md (demo-rail bullet, isHumanOwned list, drip-acts-only list, Fill 1, Backfill LinkedIn paragraph), followups.md ("Who sends"), commands/daily-outreach.md (backfill and digest). The `feat/demo-open-tracking` note in Job D now says merged as #22, live on the next MCP redeploy. SKILL.md 1.23.0 -> 1.24.0.
